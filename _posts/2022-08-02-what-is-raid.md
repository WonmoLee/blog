---
title: "RAID란?"
categories:
  - Storage
tags:
  - 스토리지
  - storage
  - RAID
  - 레이드
toc: true
toc_sticky: true
toc_label: "목차"
---

![RAID](/blog/assets/img/posts/20220802/raid.jpg "RAID"){: width="100%"}
<div style="color: gray; text-align: center; margin-bottom: 30px;">RAID</div> 

# RAID 개념
---
RAID란 여러개의 하드 디스크가 있을 때 동일한 데이터를 다른 위치에 중복해서 저장하는 방법이다. 데이터를 여러개의 디스크에 저장하여 입출력 작업이 균형을 이루게 되어 전체적인 성능을 향상시킨다. 운영체제에서 하나의 RAID는 논리적으로 하나의 디스크로 인식하여 처리된다. 현재 RAID는 데이터를 기록하는 방식과 에러를 체크하는 패리티나 ECC 사용 등 구성방법에 따라 다양한 형태로 존재한다.

# RAID 구성의 변화
---
초기의 RAID는 저용량 하드 디스크를 하나의 디스크로 확장하여 사용하는 것이 주류였으나 현재는 백업을 가능하게 하고 안정적인 데이터의 보존과 유지기능, 속도 향상 등에 사용한다. 단순히 여러 디스크를 하나로 묶어주는 기술로는 Linear가 있다. Linear는 말 그대로 '순차적' 이라는 뜻으로 여러 개의 디스크를 하나로 묶었다고 하더라도 순서를 정해서 하나의 디스크 공간을 전부 사용해야 다른 디스크에 저장된다. 최근에는 단순히 묶어주는 Linear보다는 스트라이핑이나 미러링 기술이 적영된 RAID구성이 보편적이다. RAID를 구성하는 방법도 소프트웨어적 구현과 하드웨어적 구현 등 다양한 방법으로 가능한데, 소프트웨어 RAID는 비용적인 측면에서 유리하나 성능 측면에서는 하드웨어 RAID보다 불리하다. 하드웨어 수준의 RAID에서 주목할 만한 기능은 전원이 켜져있는 상태에서 하드 드라이브를 교체할 수 있는 핫스왑과 베이가 있다.

# RAID 기술
---
- 스트라이핑
  >스트라이핑 기술은 연속된 데이터를 여러 개의 디스크에 라운드로빈 방식으로 기록하는 기술이다. 이 기술은 프로세서가 하나의 디스크에서 읽어 들이는 것보다 빠르게 데이터를 읽거나 쓸 수 있다면 매우 유용하다. 즉, 서로 겹쳐서 읽거나 쓸 수 있도록 설계된 4개의 드라이브가 있을 경우, 보통 하나의 섹터를 읽을 수 있는 시간에 4개의 섹터를 동시에 읽을 수 있다.
- 미러링
  >디스크에 에러가 발생 시 데이터 손실을 막기 위해 추가적으로 하나 이상의 장치에 중복 저장하는 기술이다. 2개의 디스크로 구현했을 경우, 하나의 디스크에 에러가 발생해도 다른 디스크의 데이터는 그대로 보존된다. 그래서 미러링 기술을 '결함 허용' 이라고도 부른다. 이 기법은 하드웨어 방법뿐 아니라, 소프트웨어적으로도 구현가능하다.

# RAID의 종류
---
- RAID-0
  >![RAID-0](/blog/assets/img/posts/20220802/raid0.png "RAID-0"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-0</div>  
  스트라이핑 기술을 사용하여 빠른 입출력 속도를 제공한다. 데이터를 중복이나 패리티 없이 디스크에 분산하여 기록한다. 처리속도는 빠르나, 구성된 디스크 중에 하나라도 오류가 발생하면 데이터 복구가 불가하다.

- RAID-1
  >![RAID-1](/blog/assets/img/posts/20220802/raid1.png "RAID-1"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-1</div> 
  미러링 기술을 사용하여 두 개의 디스크에 데이터를 동일하게 기록한다. 스트라이핑 기술은 사용하지 않으며, 각 드라이브를 동시에 읽을 수 있어서 읽기 성능은 향상되나 쓰기 성능은 단일 디스크와 같다.
  디스크 오류 시 데이터 복구 능력은 탁월하지만, 중복 저장으로 인한 디스크 낭비가 50%에 이른다.

- RAID-2
  >![RAID-2](/blog/assets/img/posts/20220802/raid2.png "RAID-2"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-2</div> 
  디스크들은 스트라이핑 기술을 사용하여 구성하고, 디스크들의 에러를 감지하고 수정하기 위해 ECC(Error Check & Correction) 정보를 사용한다.

- RAID-3
  >![RAID-3](/blog/assets/img/posts/20220802/raid3.png "RAID-3"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-3</div> 
  스트라이핑 기술을 활용하
  ㄷㅊ

- RAID-4
  >![RAID-4](/blog/assets/img/posts/20220802/raid4.png "RAID-4"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-4</div>

- RAID-5
  >![RAID-5](/blog/assets/img/posts/20220802/raid5.png "RAID-5"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-5</div> 

- RAID-6
  >![RAID-6](/blog/assets/img/posts/20220802/raid6.png "RAID-6"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-6</div> 

- RAID-7
  >![RAID-7](/blog/assets/img/posts/20220802/raid7.png "RAID-7"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-7</div> 

- RAID 0+1
  >![RAID 0+1](/blog/assets/img/posts/20220802/raid0+1.png "RAID 0+1"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID 0+1</div> 

- RAID-10
  >![RAID-10](/blog/assets/img/posts/20220802/raid10.png "RAID-10"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-10</div> 

- RAID-53
  >![RAID-53](/blog/assets/img/posts/20220802/raid53.png "RAID-53"){: width="100%"}
  <div style="color: gray; text-align: center; margin-bottom: 30px;">RAID-53</div>  

---

읽어주셔서 감사합니다. 😊

__Reference__  
CenOS7으로 리눅스 마스터 1급 정복하기 - 정성재  