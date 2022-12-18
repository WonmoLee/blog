---
title: "[SQL] 데이터베이스 아키텍처"
categories:
  - SQL
tags:
  - SQL
  - Database Architecture
  - 데이터베이스 아키텍처
toc: true
toc_sticky: true
toc_label: "목차"
---

\* 해당 포스팅은 오라클 DB 관련 정보만 다룬다.

# DB 구조
---
DBMS마다 DB에 대한 정의가 조금씩 다르다. 오라클에서는 디스크에 저장된 데이터 집합(Datafile, Redo Log File, Control File 등)을 DB라고 부른다. 그리고 SGA 공유 메모리 영역과 이를 엑세스하는 프로세스 집합을 합쳐서 인스턴스라고 부른다.

![Oracle Architecture](/blog/assets/img/posts/20221113/oracle-architecture.png "Oracle Architecture"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">Oracle Architecture</div>

기본적으로 하나의 인스턴스가 하나의 DB만 액세스하지만, RAC(Real Application Cluster) 환경에서는 여러 인스턴스가 하나의 DB를 액세스할 수 있다. 하나의 인스턴스가 여러 DB를 액세스할 수는 없다.

<br>

# 프로세스
---
오라클은 기본적으로 프로세스를 사용하며, 윈도우 버전에서는 쓰레드를 사용한다. 여기서는 프로세스와 쓰레드를 구분짓지 않고 프로세스로 통칭하여 설명하겠다.

프로세스는 서버 프로세스와 백그라운드 프로세스 집합으로 나뉜다. 서버 프로세스는 전면에서 사용자로부터 전달받은 각종 명령을 처리하고, 백그라운드 프로세스는 뒤에서 묵묵히 할당받은 역할을 수행한다.

## 서버 프로세스
서버 프로세스는 사용자 프로세스와 통신하면서 사용자의 각종 명령을 처리한다. 좀 더 구체적으로 언급하면, SQL을 파싱하고 필요하면 최적화를 수행한다. 커서를 열어 SQL을 실행하면서 블록을 읽어 이 데이터를 정렬해 클라이언트가 요청한 결과 집합을 만들어 네트워크를 통해 전송하는 일련의 작업을 모두 서버 프로세스가 처리해 준다. 스스로 처리하도록 구현되지 않은 기능, 이를 테면 데이터 파일로부터 DB 버퍼 캐시로 블록을 적재하거나 Dirty 블록을 캐시에서 밀어냄으로써 Free 블록을 확보하는일, Redo 로그 버퍼를 비우는 일 등은 OS와 I/O 서브시스템, 백그라운드 프로세스가 대시 처리하도록 시스템 Call을 통해 요청한다.

클라이언트가 서버 프로세스와 연결하는 방식은 DBMS마다 다르지만, 오라클은 전용 서버 방식과 공유 서버 방식 2가지가 존재한다.

### 전용 서버 방식
![전용 서버 방식](/blog/assets/img/posts/20221113/dedicated-server-method.png "전용 서버 방식"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">전용 서버 방식</div>

위 그림은 전용 서버 방식으로 접속할 때 내부적으로 어떤 과정을 거쳐 세션을 수립하고 사용자 명령을 처리하는지 보여준다.

처음 연결 요청을 받은 리스너가 서버 프로세스(Window 환경에서는 쓰레드)를 생성해 주고, 이 서버 프로세스가 단 하나의 사용자 프로세스를 위해 전용 서비스를 제공한다는 점이 특징이다.

만약 SQL을 수행할 때마다 연결 요청을 반복하면 서버 프로세스의 생성과 해제도 반복하게 되므로 DBMS에 매우 큰 부담을 주고 성능을 크게 떨어뜨린다. 따라서 전용 서버 방식을 사용하는 OLTP성 애플리케이션에선 Connection Pooling 기법을 필수적으로 사용해야 한다. 예를 들어 50개의 서버 프로세스와 연결된 50개의 사용자 프로세스를 공유해 반복 재사용하는 방식이다.

### 공유 서버 방식
공유 서버는 말 그대로 하나의 서버 프로세스를 여러 사용자 세션이 공유하는 방식이다. 앞서 설명한 Connection Pooling 기법을 DBMS 내부에 구현해 놓은 것으로 생각하면 쉽다. 즉, 미리 여러 개의 서버 프로세스를 띄어 놓고 이를 공유해 반복 재사용한다.

![공유 서버 방식](/blog/assets/img/posts/20221113/shared-server-method.png "공유 서버 방식"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">공유 서버 방식</div>

위 그림에서 보듯, 공유 서버 방식으로 오라클에 접속하면 사용자 프로세스는 서버 프로세스와 직접 통신하지 않고 Dispatcher 프로세스를 거친다. 사용자 명령이 Dispatcher에게 전달되면 Dispatcher는 이를 SGA에 있는 요청 큐에 등록한다. 이후 가장 먼저 가용한 서버 프로세스가 요청 큐에 있는 사용자 명령을 꺼내서 처리하고, 그 결과를 응답 큐에 등록한다.

응답 큐를 모니터링하던 Dispatcher가 응답 결과를 발견하면 사용자 프로세스에게 전송해 준다.

## 백그라운드 프로세스
- System Monitor(SMON)
  >장애가 발생한 시스템을 재기동할 때 인스턴스 복구를 수행하고, 임시 세그먼트와 익스텐트를 모니터링한다.
- Process Monitor(PMON)
  >이상이 생긴 프로세스가 사용하던 리소스를 복구한다.
- Database Writers(DBWn)
  >버퍼 캐시에 있는 Dirty 버퍼를 데이터 파일에 기록한다.
- Log Writer(LGWR)
  >로그 버퍼 엔트리를 Redo 로그 파일에 기록한다.
- Archiver(ARCn)
  >꽉 찬 Redo 로그가 덮어 쓰여지기 전에 Archive 로그 디렉터리로 백업한다.
- Checkpoint
  >Checkpoint 프로세스는 이전에 Checkpoint가 일어났던 마지막 시점 이후의 데이터베이스 변경 사항을 데이터 파일에 기록하도록 트리거링하고, 기록이 완료되면 현재 어디까지 기록했는지를 컨트롤 파일과 데이터 파일 헤더에 저장한다. 좀 더 자세히 설명하면, Write Ahead Logging 방식(데이터 변경 전에 로그부터 남기는 메커니즘)을 사용하는 DBMS는 Redo 로그에 기록해 둔 버퍼 블록에 대한 변경사항 중 현재 어디까지를 데이터 파일에 기록했는지 체크포인트 정보를 관리해야 한다. 이는 버퍼 캐시와 데이터 파일이 동기화된 시점을 가리키며, 장애가 발생하면 마지막 체크포인트 이후 로그 데이터만 디스크에 기록함으로써 인스턴스를 복구할 수 있도록 하는 용도로 사용된다. 이 정보를 갱신하는 주기가 길수록 장애 발생 시 인스턴스 복구 시간도 길어진다.
- Recoverer(RECO)
  >분산 트랜잭션 과정에 발생한 문제를 해결한다.

<br>

# 데이터 저장 구조
---
오라클은 크게 데이터 파일, 임시 데이터 파일, 로그파일이 존재한다.

## 데이터 파일
![오라클 데이터 파일 구조](/blog/assets/img/posts/20221113/data-file-structure.png "오라클 데이터 파일 구조"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">오라클 데이터 파일 구조</div>

오라클은 물리적으로는 데이터 파일에 데이터를 저장하고 관리한다. 공간을 할당하고 관리하기 위한 논리적인 구조도 크게 다르지 않지만 약간의 차이는 있다.

### 블록
대부분 DBMS에서 I/O는 블록 단위로 이루어진다. 데이터를 읽고 쓸 때의 논리적인 단위가 블록이며, 오라클은 2KB, 4KB, 8KB, 16KB, 32KB의 다양한 블록 크기를 사용할 수 있다. 블록 단위로 I/O 한다는 것은, 하나의 레코드에서 하나의 컬럼만을 읽으려 할 때도 레코드가 속한 블록 전체를 읽게 됨을 뜻한다. SQL 성능을 좌우하는 가장 중요한 성능지표는 액세스하는 블록 개수이며, 옵티마이저의 판단에 가장 큰 영향을 미치는 것도 액세스해야 할 블록 개수다.

### 익스텐트
데이터를 읽고 쓰는 단위는 블록이지만, 테이블스페이스로부터 공간을 할당하는 단위는 익스텐트다. 테이블이나 인덱스에 데이터를 입력하다가 공간이 부족해지면 해당 오브젝트가 속한 테이블스페이스(물리적으로는 데이터 파일)로부터 추가적인 공간을 할당받는다. 이때 정해진 익스텐트 크기의 연속된 블록을 할당받는다. 예를 들어 블록 크기가 8KB인 상태에서 64KB 단위로 익스텐트를 할당하도록 정의했다면, 공간이 부족할 때마다 테이블스페이스로부터 8개의 연속된 블록을 찾아(찾지 못하면 새로운 익스텐트 추가) 세그먼트에 할당해 준다.

익스텐트 내 블록은 논리적으로 인접하지만, 익스텐트끼리 서로 인접하지는 않는다. 예를 들어 어떤 세그먼트에 익스텐트 2개가 할당됐는데, 데이터 파일 내에서 이 둘이 서로 멀리 떨어져 있을 수 있다.
참고로 오라클은 다양한 크기의 익스텐트를 사용하며, 또한 한 익스텐트에 속한 모든 블록을 단일 오브젝트가 사용한다.

### 세그먼트
세그먼트는 테이블, 인덱스, Undo처럼 저장공간을 필요로 하는 DB 오브젝트다. 저장공간을 필요로 한다는 것은 한 개 이상의 익스텐트를 사용함을 뜻한다.

테이블을 생성할 때, 내부적으로는 테이블 세그먼트가 생성된다. 인덱스를 생성할 때, 내부적으로 인덱스 세그먼트가 생성된다. 다른 오브젝트는 세그먼트와 1:1 대응 관계를 갖지만 파티션은 1:M 관계를 갖는다. 즉 파티션 테이블(또는 인덱스)을 만들면, 내부적으로 여러 개의 세그먼트가 만들어진다.

한 세그먼트는 자신이 속한 테이블스페이스 내 여러 데이터 파일에 걸쳐 저장될 수 있다. 즉 세그먼트에 할당된 익스텐트가 여러 데이터 파일에 흩어져 저장되는 것이며, 그래야 디스크 경합을 줄이고 I/O 분산 효과를 얻을 수 있다.

### 테이블스페이스
테이블스페이스는 세그먼트를 담는 컨테이너로서, 여러 데이터 파일로 구성된다. 데이터는 물리적으로 데이터 파일에 저장되지만, 사용자가 데이터 파일을 직접 선택하진 않는다. 사용자는 세그먼트를 위한 테이블스페이스를 지정할 뿐, 실제 값을 저장할 데이터 파일을 선택하고 익스텐트를 할당하는 것은 DBMS의 몫이다.

각 세그먼트는 정확히 한 테이블스페이스에만 속하지만, 한 테이블스페이스에는 여러 세그먼트가 존재할 수 있다. 특정 세그먼트에 할당된 모든 익스텐트는 해당 세그먼트와 관련된 테이블스페이스 내에서만 찾아진다. 한 세그먼트가 여러 테이블스페이스에 걸쳐 저장될 수는 없다. 하지만 앞서 언급했듯이 한 세그먼트가 여러 데이터 파일에 걸쳐 저장될 수는 있다. 한 테이블스페이스가 여러 데이터 파일로 구성되기 때문이다. 지금까지 설명한 내용을 그림으로 요약하면 다음과 같다.

![오라클 저장 구조](/blog/assets/img/posts/20221113/oracle-save-structure.png "오라클 저장 구조"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">오라클 저장 구조</div>

## 임시 데이터 파일
임시 데이터 파일은 특별한 용도로 사용된다. 대량의 정렬이나 해시 작업을 수행하다가 메모리 공간이 부족해지면 중간 결과 집합을 저장하는 용도다. 임시 데이터 파일에 저장되는 오브젝트는 말 그대로 임시로 저장했다가 자동으로 삭제된다. Redo 정보를 생성하지 않기 때문에 나중에 파일에 문제가 생겼을 때 복구되지 않는다. 따라서 백업할 필요도 없다.

오라클에선 임시 테이블 스페이스를 여러 개 생성해 두고, 사용자마다 별도의 임시 테이블스페이스를 지정해줄 수도 있다.

- 예제 쿼리

```sql
CREATE TEMPORARY TABLESPACE BIG_TEMP
TEMPFILE '/usr/local/oracle/oradata/ora10g/big_temp.dbf' SIZE 2000m;

ALTER USER SCOTT TEMPORARY TABLESPACE BIG_TEMP;
```

## 로그 파일
DB 버퍼 캐시에 가해지는 모든 변경사항을 기록하는 파일을 Oracle은 'Redo 로그' 라고 부른다. 변경된 메모리 버퍼 블록을 디스크 상의 데이터 블록에 기록하는 작업은 Random I/O 방식으로 이루어지기 때문에 느리다. 반면 로그 기록은 Append 방식으로 이루어지기 때문에 상대적으로 매우 빠르다. 따라서 대부분 DBMS는 버퍼 블록에 대한 변경사항을 건건이 데이터 파일에 기록하기보다 우선 로그 파일에 Append 방식으로 빠르게 기록하는 방식을 사용한다. 그러고 나서 버퍼 블록과 데이터 파일 간 동기화는 적절한 수단(DBWR, Checkpoint)을 이용해 나중에 배치 방식으로 일괄 처리한다.

사용자의 갱신내용이 메모리상의 버퍼 블록에만 기록된 채 아직 디스크에 기록되지 않았더라도 Redo 로그를 믿고 빠르게 커밋을 완료한다는 의미에서, 이를 'Fast Commit' 메커니즘이라고 부른다. 인스턴스 장애가 발생하더라도 로그 파일을 이용해 언제든 복구 가능하므로 안심하고 커밋을 완료할 수 있는 것이다. Fast Commit은 빠르게 트랜잭션을 처리해야 하는 모든 DBMS의 공통적인 메커니즘이다.

- __Online Redo 로그__  
캐시에 저장된 변경사항이 아직 데이터 파일에 기록되지 않은 상태에서 정전 등으로 인스턴스가 비정상 종료되면, 그때까지의 작업내용을 모두 잃게 된다. 이러한 트랜잭션 데이터의 유실에 대비하기 위해 Oracle은 Online Redo 로그를 사용한다. 마지막 체크포인트 이후부터 사고 발생 직전까지 수행됬던 트랜잭션들을 Redo 로그를 이용해 재현하는 것이며, 이를 '캐시 복구' 라고 한다.  
Online Redo 로그는 최소 두 개 이상의 파일로 구성된다. 현재 사용중인 파일이 꽉 차면 다음 파일로 로그 스위칭이 발생하며, 계속 로그를 써 나가다가 모든 파일이 꽉 차면 다시 첫 번째 파일부터 재사용하는 라운드 로빈 방식을 사용한다.

- __Archived(=Offline) Redo 로그__  
Archived Redo 로그는 오라클에서 Online Redo 로그가 재사용되기 전에 다른 위치로 백업해 둔 파일을 말한다. 디스크가 깨지는 등 물리적인 저장 매체에 문제가 생겼을 때 데이터베이스(또는 미디어) 복구를 위해 사용된다.

<br>

# 메모리 구조
---
메모리 구조는 시스템 공유 메모리 영역과 프로세스 전용 메모리 영역으로 구분된다.

## 시스템 공유 메모리 영역
시스템 공유 메모리는 말 그대로 여러 프로세스가 동시에 엑세스할 수 있는 메모리 영역으로서, Oracle에선 'System Global Area(SGA)' 라고 한다. 공유 메모리를 구성하는 캐시 영역은 매우 다양하지만, 모든 DBMS가 공통적으로 사용하는 캐시 영역으로는 DB 버퍼 캐시, 공유 풀, 로그 버퍼가 있다. 공유 메모리 영역은 그 외에 Large 풀(Large Pool), 자바 풀(Java Pool) 등을 포함하고, 시스템 구조와 제어 구조를 캐싱하는 영역도 포함한다.

시스템 공유 메모리 영역은 여러 프로세스에 공유되기 때문에 내부적으로 래치, 버퍼(Lock), 라이브러리 캐시 Lock/Pin 같은 액세스 직렬화 메커니즘이 사용된다.

### DB 버퍼 캐시
DB 버퍼캐시는 데이터 파일로부터 읽어 들인 데이터 블록을 담는 캐시 영역이다. 인스턴스에 접속한 모든 사용자 프로세스는 서버 프로세스를 통해 DB 버퍼 캐시의 버퍼 블록을 동시에(내부적으로는 버퍼 Lock을 통해 직렬화) 액세스 할 수 있다.

일부 Direct Path Read 메커니즘이 작동하는 경우를 제외하면, 모든 블록 읽기는 버퍼 캐시를 통해 이루어진다. 즉, 읽고자 하는 블록을 먼저 버퍼 캐시에서 찾아보고 없을 때 디스크에서 읽는다. 디스크에서 읽을 때도 먼저 버퍼 캐시에 적재한 후에 읽는다.

데이터 변경도 버퍼 캐시에 적재된 블록을 통해 이루어지며, 변경된 블록(Dirty 버퍼 블록)을 주기적으로 데이터 파일에 기록하는 작업은 DBWR 프로세스의 몫이다.

디스크 I/O는 물리적으로 액세스 암이 움직이면서 헤드를 통해 이루어지는 반면, 메모리 I/O는 전기적 신호에 불과하기 때문에 디스크 I/O에 비교할 수 없을 정도로 빠르다. 디스크에서 읽은 데이터 블록을 메모리 상에 보관해 두는 기능이 모든 DB 시스템에 필수적인 이유다.

__1) 버퍼 블록의 상태__  
모든 버퍼 블록은 아래 세 가지 중 하나의 상태에 놓인다.

- Free 버퍼
  >인스턴스 기동 후 아직 데이터가 읽히지 않아 비어 있는 상태(Clean 버퍼)이거나, 데이터가 담겼지만 데이터 파일과 서로 동기화돼 있는 상태여서 언제든지 엎어 써도 무방한 버퍼 블록을 말한다. 데이터 파일로부터 새로운 데이터 블록을 로딩하려면 먼저 Free 버퍼를 확보해야 한다. Free 상태인 버퍼에 변경이 발생하면 그 순간 Dirty 버퍼로 상태가 바뀐다.

- Dirty 버퍼
  >버퍼에 캐시된 이후 변경이 발생했지만, 아직 디스크에 기록되지 않아 데이터 파일 블록과 동기화가 필요한 버퍼 블록을 말한다. 이 버퍼 블록들이 다른 데이터 블록을 위해 재사용되려면 디스크에 먼저 기록돼야 하며, 디스크에 기록되는 순간 Free 버퍼로 상태가 바뀐다.

- Pinned 버퍼
  >읽기 또는 쓰기 작업이 현재 진행 중인 버퍼 블록을 말한다.

__2) LRU 알고리즘__  
버퍼 캐시는 유한한 자원이므로 모든 데이터를 캐싱해 둘 수 없다. 따라서 모든 DBMS는 사용빈도가 높은 데이터 블록 위주로 버퍼 캐시가 구성되도록 LRU(least recently used) 알고리즘을 사용한다. 모든 버퍼 블록 헤더를 LRU 체인에 연결해 사용빈도 순으로 위치를 옮겨가다가, Free 버퍼가 필요해질 때면 액세스 빈도가 낮은 쪽(LRU end) 데이터 블록부터 밀어내는 방식이다. 아래와 같은 컨베이어 벨트를 연상하면 LRU 알고리즘을 쉽게 이해할 수 있다.

![LRU list](/blog/assets/img/posts/20221113/lru-list.png "LRU list"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">LRU list</div>

### 공유 풀
공유 풀은 딕셔너리 캐시와 라이브러리 캐시로 구성되며, 버퍼 캐시처럼 LRU 알고리즘을 사용한다.

__딕셔너리 캐시__  
DB 딕셔너리는 테이블, 인덱스 같은 오브젝트는 물론 테이블스페이스, 데이터 파일, 세그먼트, 익스텐트, 사용자, 제약에 관한 메타 정보를 저장하는 곳이다. 그리고 딕셔너리 캐시는 말 그대로 딕셔너리먼트, 익스텐트, 사용자, 제약에 관한 메타 정보를 저장하는 곳이다. 그리고 딕셔너리 캐시는 말 그대로 딕셔너리 정보를 캐싱하는 메모리 영역이다. '주문' 테이블을 예로 들면, 입력한 주문 데이터는 데이터 파일에 저장됬다가 버퍼 캐시를 경유해 읽히지만, 테이블 메타 정보는 딕셔너리에 저장됬다가 딕셔너리 캐시를 경유해 읽힌다.

__라이브러리 캐시__  
라이브러리 캐시는 사용자가 수행한 SQL 문과 실행계획, 저장 프로시저를 저장해 두는 캐시영역이다. 사용자가 SQL 명령어를 통해 결과 집합을 요청하면 이를 최적으로 (가장 적은 리소스를 사용하면서 가장 빠르게) 수행하기 위한 처리 루틴을 생성해야 하는데, 그것을 실행계획이라고 한다. 빠른 쿼리 수행을 위해 내부적으로 생성한 일종의 프로시저와 같은 것이라고 이해하면 쉽다.  
쿼리 구문을 분석해서 문법 오류 및 실행 권한 등의 체크, 최적화 과정을 거쳐 실행계획 생성, SQL 실행엔진이 이해할 수 있는 형태로 포맷팅 등의 전 과정을 하드 파싱이라고 한다. 특히 최적화 과정은 하드 파싱을 무겁게 만드는 가장 결정적 요인인데, 같은 SQL을 처리하려고 이런 무거운 작업을 반복 수행하는 것은 매우 비효율적이다. 그래서 같은 SQL에 대한 반복적인 하드파싱을 최소화하기 위한 캐시 공간을 따로 두게 됬고, 그것이 바로 라이브러리 캐시 영역이다. 캐싱된 SQL과 그 실행계획의 재사용성을 높이는 것은 SQL 수행 성능을 높이고 DBMS 부하를 최소화하는 핵심 원리 중 한 가지다.

### 로그 버퍼
DB 버퍼 캐시에 가해지는 모든 변경사항을 로그 파일에 기록한다고 앞서 설명했는데, 로그 엔트리도 파일에 곧바로 기록하는 것이 아니라 먼저 로그 버퍼에 기록한다. 건건이 디스크에 기록하기보다 일정량을 모았다가 기록하면 훨씬 빠르기 때문이다. 좀 더 자세히 설명하면, 서버 프로세스가 데이터 블록 버퍼에 변경을 가하기 전에 Redo 로그 버퍼에 먼저 기록해 두면 주기적으로 LGWR 프로세스가 Redo 로그 파일에 기록한다. 

변경이 가해진 Dirty 버퍼를 데이터 파일에 기록하기 전에 항상 로그 버퍼를 먼저 로그 파일에 기록해야만 하는데, 그 이유는 인스턴스 장애가 발생할 때면 로그 파일에 기록된 내용을 재현해 캐시 블록을 복구하고, 최종적으로 커밋되지 않은 트랜잭션은 롤백해야 한다. 이때, 로그 파일에는 없는 변경내역이 이미 데이터 파일에 기록돼 있으면 사용자가 최종 커밋하지 않은 트랜잭션이 커밋되는 결과를 초래하기 때문이다. 

정리해 보면, 버퍼 캐시 블록을 갱신하기 전에 변경사항을 먼저 로그 버퍼에 기록해야 하며, Dirty 버퍼를 디스크에 기록하기 전에 해당 로그 엔트리를 먼저 로그 파일에 기록해야 하는데, 이를 ‘Write Ahead Logging’이라고 한다. 그리고 로그 버퍼를 주기적으로 로그 파일에 기록한다고 했는데, 늦어도 커밋 시점에는 로그 파일에 기록해야 한다(Log Force at commit). 메모리상의 로그 버퍼는 언제든 유실될 가능성이 있기 때문이다. 로그를 이용한 Fast Commit이 가능한 이유는 로그를 이용해 언제든 복구 가능하기 때문이라고 설명한 것을 상기하기 바란다. 다시 말하지만, 로그 파일에 기록했음이 보장돼야 안심하고 커밋을 완료할 수 있다.

## 프로세스 전용 메모리 영역
오라클은 프로세스 기반 아키텍처이므로 서버 프로세스가 자신만의 전용 메모리 영역을 가질 수 있다. 이를 'Process Global Area(PGA)' 라고 부르며, 데이터를 정렬하고 세션과 커서에 관한 상태 정보를 저장하는 용도로 사용한다.

### PGA
각 Oracle 서버 프로세스는 자신만의 PGA(Process/Program/Private Global Area) 메모리 영역을 할당받고, 이를 프로세스에 종속적인 고유 데이터를 저장하는 용도로 사용한다. PGA는 다른 프로세스와 공유되지 않는 독립적인 메모리 공간으로서, 래치 메커니즘이 필요 없어 똑같은 개수의 블록을 읽더라도 SGA 버퍼 캐시에서 읽는 것보다 훨씬 빠르다.

- __User Global Area(UGA)__  
전용 서버(Dedicated Server) 방식으로 연결할 때는 프로세스와 세션이 1:1 관계를 갖지만, 공유 서버(Shared Server) 방식으로 연결할 때는 1:M 관계를 갖는다. 즉, 세션이 프로세스 개수보다 많아질 수 있는 구조로서, 하나의 프로세스가 여러 개 세션을 위해 일한다. 따라서 각 세션을 위한 독립적인 메모리 공간이 필요해지는데, 이를 ‘UGA(User Global Area)’라고 한다.  
전용 서버 방식이라고 해서 UGA를 사용하지 않는 것은 아니다. UGA는 전용 서버 방식으로 연결할 때는 PGA에 할당되고, 공유 서버 방식으로 연결할 때는 SGA에 할당된다. 구체적으로 후자는, Large Pool이 설정됐을 때는 Large Pool에, 그렇지 않을 때는 Shared Pool에 할당하는 방식이다.

- __Call Global Area(CGA)__  
PGA에 할당되는 메모리 공간으로는 CGA도 있다. Oracle은 하나의 데이터베이스 Call을 넘어서 다음 Call까지 계속 참조되어야 하는 정보는 UGA에 담고, Call이 진행되는 동안에만 필요한 데이터는 CGA에 담는다. CGA는 Parse Call, Execute Call, Fetch Call마다 매번 할당받는다. Call이 진행되는 동안 Recursive Call이 발생하면 그 안에서도 Parse, Execute, Fetch 단계별로 CGA가 추가로 할당된다. CGA에 할당된 공간은 하나의 Call이 끝나자마자 해제돼 PGA로 반환된다.

- __Sort Area__  
데이터 정렬을 위해 사용되는 Sort Area는 소트 오퍼레이션이 진행되는 동안 공간이 부족해질 때마다 청크(Chunk) 단위로 조금씩 할당된다. 세션마다 사용할 수 있는 최대 크기를 예전에는 sort_area_size 파라미터로 설정하였으나, 9i부터는 새로 생긴 workarea_size_policy 파라미터를 auto(기본 값)로 설정하면 Oracle이 내부적으로 결정한다.  
PGA 내에서 Sort Area가 할당되는 위치는 SQL문 종류와 소트 수행 단계에 따라 다르다. DML 문장은 하나의 Execute Call 내에서 모든 데이터 처리를 완료하므로 Sort Area가 CGA에 할당된다. SELECT 문장의 경우, 수행 중간 단계에 필요한 Sort Area는 CGA에 할당되고, 최종 결과집합을 출력하기 직전 단계에 필요한 Sort Area는 UGA에 할당된다.  

---

읽어주셔서 감사합니다. 😊 

__Reference__  
SQL 전문가 가이드 - Kdata 한국데이터산업진흥원  