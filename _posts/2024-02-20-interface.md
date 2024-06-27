---
title: "[Java] 인터페이스"
description: 
author: Enxec
date: 2024-02-20
categories: [Language, Java]
tags: [java, 자바, Interface, 인터페이스]
pin: false
math: true
mermaid: true
image:
  path: /thumbnail/java-logo.png
  lqip: 
  alt: 
---

## 인터페이스란?
---
인터페이스는 일종의 추상클래스다. 인터페이스는 추상클래스처럼 추상메서드를 갖지만 추상클래스보다 추상화 정도가 높아서 추상클래스와 달리 몸통을 갖춘 일반 메서드 또는 멤버변수를 구성원으로 가질 수 없다. 오직 추상메서드와 상수만을 멤버로 가질 수 있으며, 그 외의 다른 어떠한 요소도 허용치 않는다.

앞서 추상클래스를 '미완성 설계도'라고 하였는데, 인터페이스는 구현된 것은 아무 것도 없고 밑그림만 그려져 있는 '기본 설계도'라고 할 수 있다. 인터페이스도 추상클래스처럼 완성되지 않은 불완전한 것이기 때문에 그 자체만으로 사용되기 보다는 다른 클래스를 작성하는데 도움 줄 목적으로 작성된다.

<br>

## 인터페이스의 작성
---
인터페이스를 작성하는 것은 클래스를 작성하는 것과 같다. 다만 키워드로 class 대신 interface를 사용한다는 것만 다르다. 그리고 interface에도 클래스와 같이 접근제어자로 public 또는 default를 사용할 수 있다.

```java
interface 인터페이스명 {
    public static final 타입 상수이름 = 값;
    public abstract 메서드명(매개변수목록);
}
```

일반적인 클래스의 멤버들과 달리 인터페이스의 멤버들은 다음과 같은 제약사항이 있다.
- 모든 멤버변수는 public static final 이어야 하며, 이를 생략할 수 있다.
- 모든 메서드는 public abstract 이어야 하며, 이를 생략할 수 있다.
  (단, static 메서드와 디폴트 메서드는 예외(JDK1.8부터)

인터페이스에 정의된 모든 멤버에 예외없이 적용되는 사항이기 때문에 제어자를 생략할 수 있는 것이며, 편의상 생략하는 경우가 많다. 생략된 제어자는 컴파일 시에 컴파일러가 자동적으로 추가해준다.

```java
interface PlayingCard {
    public static final int SPADE = 4;
    final int DIAMOND = 3;
    static int HEART = 2;
    int CLOVER = 1;
    
    public abstract String getCardNumber();
    String getCardKind();
}
```

원래는 인터페이스의 모든 메서드는 추상메서드이어야하는데, JDK1.8부터 인터페이스에 static메서드와 디폴트 메서드의 추가를 허용하는 방향으로 변경되었다. 실무에서 아직 JDK1.8을 사용하지 않는 곳이 많기 때문에, JDK1.8이전의 규칙과 이후의 규칙을 모두 알고 있어야 한다.

<br>

## 인터페이스의 상속
---
인터페이스는 인터페이스로부터만 상속받을 수 있으며, 클래스와는 달리 다중상속, 즉 여러 개의 인터페이스로부터 상속을 받는 것이 가능하다.

💡 __참고__  
인터페이스는 클래스와 달리 Object클래스와 같은 최고 조상이 없다.

```java
interface Movable {
    /** 지정된 위치(x, y)로 이동하는 기능의 메서드 */
    void move(int x, int y);
}

interface Attackable {
    /** 지정된 대상(u)을 공격하는 기능의 메서드 */
    void attack (Unit u);
}

interface Fightable extends Movable, Attackable{

}
```

클래스의 상속과 마찬가지로 자손 인터페이스(Fightable)는 조상 인터페이스(Moveable, Attackable)에 정의된 멤버를 모두 상속받는다.

그래서 Fightable자체에는 정의된 멤버가 하나도 없지만 조상 인터페이스로부터 상속 받은 두개의 추상메서드, move(int x, int y)와 attack(Unit u)을 멤버로 갖게 된다.

<br>

## 인터페이스의 구현
---
인터페이스도 추상클래스처럼 그 자체로는 인스턴스를 생성할 수 없으며, 추상클래스가 상속을 통해 추상메서드를 완성하는 것처럼, 인터페이스도 자신에 정의된 추상메서드의 몸통을 만들어주는 클래스를 작성해야 하는데, 그 방법은 추상클래스가 자신을 상속받는 클래스를 정의하는 것과 다르지 않다. 다만 클래스는 확장한다는 의미의 키워드 'extends'를 사용하지만 인터페이스는 구현한다는 의미의 키워드 'implements'를 사용할 뿐이다.

```java
class 클래스이름 implements 인터페이스이름 {
    // 인터페이스에 정의된 추상메서드를 구현해야 한다.
}

class Fighter implements Fightable {
    public void move(int x, int y) {
        /* 내용 생략 */
    }
    
    public void attack(Unit u) {
        /* 내용 생략 */
    }
}
```

💡 __참고__  
이 때 'Fighter클래스는 Fightable인터페이스를 구현한다.' 라고 한다.

만일 구현하는 인터페이스의 메서드 중 일부만 구현한다면, abstract를 붙여서 추상클래스로 선언해야 한다.

```java
abstract class Fighter implements Fightable {
    public void move(int x, int y) {
        /* 내용 생략 */
    }
}
```

그리고 다음과 같이 상속과 구현을 동시에 할 수도 있다.

```java
class Fighter extends Unit implements Fightable {
    public void move(int x, int y) {
        /* 내용 생략 */
    }
    
    public void attack(Unit u) {
        /* 내용 생략 */
    }
}
```

💡 __참고__  
인터페이스의 이름에는 주로 Fightable과 같이 '~ 을 할 수 있는'의 의미인 'able'로 끝나는 것들이 많은데, 그 이유는 어떠한 기능 또는 행위를 하는데 필요한 메서드를 제공한다는 의미를 강조하기 위해서이다. 또한 그 인터페이스를 구현한 클래스는 '~를 할 수 있는' 능력을 갖추었다는 의미이기도 하다. 이름이 'able'로 끝나는 것은 인터페이스라고 추측할 수 있지만, 모든 인터페이스의 이름이 반드시 'able'로 끝나야 하는 것은 아니다.

<br>

## 인터페이스를 이용한 다중상속
---
두 조상으로부터 상속받는 멤버 중에서 멤버변수의 이름이 같거나 메서드의 선언부가 일치하고 구현 내용이 다르다면 이 두 조상으로부터 상속받는 자손클래스는 어느 조상의 것을 상속받게 되는 것인지 알 수 없다. 어느 한 쪽으로부터의 상속을 포기하던가, 이름이 충돌하지 않도록 조상클래스를 변경하는 수 밖에 없다.

그래서 다중상속은 장점도 있지만 단점이 더 크다고 판단하였기 때문에 자바에서는 다중 상속을 허용하지 않는다. 그러나 또 다른 객체지향언어인 C++에서는 다중상속을 허용하기 때문에 자바는 다중상속을 허용하지 않는다는 것이 단점으로 부각되는 것에 대한 대응으로 '자바도 인터페이스를 이용하면 다중상속이 가능하다.' 라고 하는 것일 뿐 자바에서 인터페이스로 다중상속을 구현하는 경우는 거의 없다.
이러한 이유로 인터페이스가 다중상속을 위한 것으로 오해를 사곤한다.

인터페이스는 static상수만 정의할 수 있으므로 조상클래스의 멤버변수와 충돌하는 경우는 거의 없고 충돌된다 하더라도 클래스 이름을 붙여서 구분이 가능하다. 그리고 추상메서드는 구현내용이 전혀 없으므로 조상클래스의 메서드와 선언부가 일치하는 경우에는 당연히 조상 클래스 쪽의 메서드를 상속받으면 되므로 문제되지 않는다. 그러나 이렇게 하면 상속받는 멤버의 충돌은 피할 수 있지만, 다중상속의 장점을 잃게 된다. 만일 두 개의 클래스로부터 상속을 받아야 할 상황이라면, 두 조상클래스 중에서 비중이 높은 쪽을 선택하고 다른 한쪽은 클래스 내부에 멤버로 포함시키는 방식으로 처리하거나 어느 한쪽의 필요한 부분을 뽑아서 인터페이스로 만든 다음 구현하도록 한다.

예를 들어, 다음과 같이 Tv클래스와 VCR클래스가 있을 때, TVCR클래스를 작성하기 위해 두 클래스로부터 상속을 받을 수만 있으면 좋겠지만 다중상속을 허용하지 않으므로, 한쪽만 선택하여 상속받고 나머지 한 쪽은 클래스 내에 포함시켜서 내부적으로 인스턴스를 생성해서 사용하도록 한다.

```java
public class Tv {
    protected boolean power;
    protected int channel;
    protected int volume;
    
    public void power() {
        power = !power;
    }
    
    public void channelUp() {
        channel++;
    }
    
    public void channelDown() {
        channel--;
    }
    
    public void volumnUp() {
        volume++;
    }
    
    public void volumnDown() {
        volume--;
    }
}

public class VCR {
    protected int counter; // VCR의 카운터
    
    public void play() {
        // Tape을 재생한다.
    }
    
    public void stop() {
        // 재생을 멈춘다.
    }
    
    public void reset() {
        counter = 0;
    }
    
    public int getCounter() {
        return counter;
    }
    
    public void setCounter(int c) {
        counter = c;
    }
}
```

VCR클래스에 정의된 메서드와 일치하는 추상메서드를 갖는 인터페이스를 작성한다.

```java
public interface IVCR {
    public void play();
    public void stop();
    public void reset();
    public int getCounter();
    public void setCounter(int c);
}
```

이제 IVCR 인터페이스를 구현하고 Tv클래스로부터 상속받는 TVCR클래스를 작성한다. 이때 VCR클래스 타입의 참조변수를 멤버변수로 선언하여 IVCR인터페이스의 추상메서드를 구현하는데 사용한다.

```java
public class TVCR extends Tv implements IVCR {
    VCR vcr = new VCR();
    
    public void play() {
        vcr.play(); // 코드를 작성하는 대신 VCR인스턴스의 메서드를 호출한다.
    }
    
    public void stop() {
        vcr.stop();
    }
    
    public void reset() {
        vcr.reset();
    }
    
    public int getCounter() {
        return vcr.getCounter();
    }
    
    public void setCounter(int c) {
        vcr.setCounter(c);
    }
}
```

IVCR인터페이스를 구현하기 위해서는 새로 메서드를 작성해야하는 부담이 있지만 이처럼 VCR클래스의 인스턴스를 사용하면 손쉽게 다중상속을 구현할 수 있다. 또한 VCR클래스의 내용이 변경되어도 변경된 내용이 TVCR클래스에도 자동적으로 반영되는 효과도 얻을 수 있다.

사실 인터페이스를 새로 작성하지 않고도 VCR클래스를 TVCR클래스에 포함시키는 것만으로도 충분하지만, 인터페이스를 이용하면 다형적 특성을 이용할 수 있다는 장점이 있다.

<br>

## 인터페이스를 이용한 다형성
---
다형성에 대해 학습할 때 자손클래스의 인스턴스를 조상타입의 참조변수로 참조하는 것이 가능하다는 것을 배웠다. 인터페이스 역시 이를 구현한 클래스의 조상이라 할 수 있으므로 해당 인터페이스 타입의 참조변수로 이를 구현한 클래스의 인스턴스를 참조할 수 있으며, 인터페이스 타입으로의 형변환도 가능하다. 인터페이스 Fightable을 클래스 Fighter가 구현했을 때, 다음과 같이 Fighter인스턴스를 Fightable타입의 참조변수로 참조하는 것이 가능하다.

```java
Fightable f = (Fightable)new Fighter();
또는
Fightable f = new Fighter();
```

💡 __참고__  
Fightable타입의 참조변수로는 인터페이스 Fightable에 정의된 멤버들만 호출이 가능하다.
따라서 인터페이스는 다음과 같이 메서드의 매개변수의 타입으로 사용될 수 있다.

```java
void attack(Fightable f) {
    //...
}
```

인터페이스 타입의 매개변수가 갖는 의미는 메서드 호출 시 해당 인터페이스를 구현한 클래스의 인스턴스를 매개변수로 제공해야한다는 것이다. 그래서 attack메서드를 호출할 때 매개변수로 Fightable인터페이스를 구현한 클래스의 인스턴스를 넘겨주어야 한다.

```java
class Fighter extends Unit implements Fightable {
    public void move(int x, int y) {
        /* 내용 생략 */
    }
    
    public void attack(Fightable f) {
        /* 내용 생략 */
    }
}
```

위와 같이 Fightable 인터페이스를 구현한 Fighter클래스가 있을 때, attack메서드의 매개변수로 Fighter인스턴스를 넘겨 줄 수 있다. 즉, attack(new Fighter())와 같이 할 수 있다는 것이다. 그리고 다음과 같이 메서드의 리턴타입으로 인터페이스의 타입을 지정하는 것 역시 가능하다.

```java
Fightable method() {
    ...
    Fighter f = new Fighter();
    return f;
}
```

리턴타입이 인터페이스라는 것은 메서드가 해당 인터페이스를 구현한 클래스의 인스턴스를 반환한다는 것을 의미한다.

<br>

## 인터페이스의 장점
---
인터페이스를 사용하는 이유와 그 장점을 정리해 보면 다음과 같다.

- 개발시간을 단축시킬 수 있다.
- 표준화가 가능하다.
- 서로 관계없는 클래스들에게 관계를 맺어 줄 수 있다.
- 독립적인 프로그래밍이 가능하다.

<br>

__1) 개발 시간을 단축시킬 수 있다.__  

일단 인터페이스가 작성되면, 이를 사용해서 프로그램을 작성하는 것이 가능하다. 메서드를 호출하는 쪽에서는 메서드의 내용에 관계없이 선언부만 알면 되기 떄문이다. 그리고 동시에 다른 한 쪽에서는 인터페이스를 구현하는 클래스를 작성하게 하면, 인터페이스를 구현하는 클래스가 작성될 때까지 기다리지 않고도 양쪽에서 동시에 개발을 진행할 수 있다.

__2) 표준화가 가능하다.__  
프로젝트에 사용되는 기본 틀을 인터페이스로 작성한 다음, 개발자들에게 인터페이스를 구현하여 프로그램을 작성하도록 함으로써 보다 일관되고 정형화된 프로그램의 개발이 가능하다.

__3) 서로 관계없는 클래스들에게 관계를 맺어 줄 수 있다.__  
서로 상속관계에 있지도 않고, 같은 조상클래스를 가지고 있지 않은 서로 아무런 관계도 없는 클래스들에게 하나의 인터페이스를 공통적으로 구현하도록 함으로써 관계를 맺어 줄 수 있다.

__4) 독립적인 프로그래밍이 가능하다.__  
인터페이스를 이용하면 클래스의 선언과 구현을 분리시킬 수 있기 때문에 실제구현에 독립적인 프로그램을 작성하는 것이 가능하다. 클래스와 클래스간의 직접적인 관계를 인터페이스를 이용해서 간접적인 관계로 변경하면, 한 클래스의 관련된 다른 클래스에 영향을 미치지 않는 독립적인 프로그래밍이 가능하다.

<br>

## 인터페이스의 이해
---
인터페이스의 규칙 또는 활용도 중요하지만 본질적인 측면에 대한 이해가 필요하다. 먼저 인터페이스를 이해하기 위해 다음의 두가지 사항을 염두해야한다.

- 클래스를 사용하는 쪽(User)과 클래스를 제공하는 쪽(Provider)이 있다.
- 메서드를 사용(호출)하는 쪽(User)에서는 사용하려는 메서드(Provider)의 선언부만 알면 된다.

```java
class A {
    public void methodA(B b) {
        b.methodB();
    }
}

class B {
    public void methodB() {
        System.out.println("method()");
    }
}

class InterfaceTest {
    public static void main(String args[]) {
        A a = new A();
        a.methodA(new B());
    }
}
```

위 코드를 보자. 클래스 A(User)는 클래스 B(Provider)의 인스턴스를 생성하고 메서드를 호출한다. 이 두 클래스는 서로 직접적인 관계에 있다. 이것을 간단히 'A -> B'라고 표현하자. 이 경우 클래스 A를 작성하려면 클래스 B가 이미 작성되어 있어야 한다. 그리고 클래스 B의 methodB()의 선언부가 변경되면, 이를 사용하는 클래스 A도 변경되어야 한다. 이와 같이 직접적인 관계의 두 클래스는 한 쪽(Provider)이 변경되면 다른 한 쪽(User)도 변경되어야 한다는 단점이 있다. 그러나 클래스 A가 클래스 B를 직접 호출하지 않고 인터페이스를 매개체로 해서 클래스A가 인터페이스를 통해서 클래스 B의 메서드에 접근하도록 하면, 클래스 B에 변경사항이 생기거나 클래스 B와 같은 기능의 다른 클래스로 대체 되어도 클래스 A는 전혀 영향을 받지 않도록 하는 것이 가능하다. 두 클래스간의 관계를 간접적으로 변경하기 위해서는 먼저 인터페이스를 이용해서 클래스 B(Provider)의 선언과 구현을 분리해야한다.

먼저 다음과 같이 클래스 B에 정의된 메서드를 추상메서드로 정의하는 인터페이스 I를 정의한다.

```java
interface I {
    public abstract void methodB();
}
```

그 다음에는 클래스 B가 인터페이스 I를 구현하도록 한다.

```java
class B implements I {
    public void methodB() {
        System.out.println("methodB in B class");
    }
}
```

이제 클래스 A는 클래스 B 대신 인터페이스 I를 사용해서 작성할 수 있다.

```java
class A {
    public void mehtodA(B b) {
        b.methodB();
    }
}

class A {
    public void methodA(I i) {
        i.methodB();
    }
}
```

💡 __참고__  
methodA가 호출될 때 인터페이스 I를 구현한 클래스의 인스턴스(클래스 B의 인스턴스)를 제공받아야 한다.

클래스 A를 작성하는데 있어서 클래스 B가 사용되지 않았다는 점에 주목하자. 이제 클래스 A와 클래스 B는 'A -> B'의 직접적인 관계에서 'A -> I -> B'의 간접적인 관계로 바뀐 것이다.

<br>

## 디폴트 메서드와 static 메서드
---
- static 메서드  

원래는 인터페이스에 추상 메서드만 선언할 수 있는데, JDK1.8부터 디폴트 메서드와 static메서드도 추가할 수 있게 되었다. static메서드는 인스턴스와 관계가 없는 독립적인 메서드이기 때문에 예전부터 인터페이스에 추가하지 못할 이유가 없었다. 그러나 자바를 보다 쉽게 배울 수 있도록 규칙을 단순히 할 필요가 있어서 인터페이스의 모든 메서드는 추상 메서드이어야 한다는 규칙에 예외를 두지 않았다. 덕분에 인터페이스와 관련된 static메서드는 별도의 클래스에 따로 두어야 했다.

- 디폴트 메서드  

조상 클래스에 새로운 메서드를 추가하는 것은 별 일이 아니지만, 인터페이스의 경우에는 보통 큰 일이 아니다. 인터페이스에 메서드를 추가한다는 것은, 추상 메서드를 추가한다는 것이고, 이 인터페이스를 구현한 기존의 모든 클래스들이 새로 추가된 메서드를 구현해야하기 때문이다. 
인터페이스가 변경되지 않으면 제일 좋겠지만, 아무리 설계를 잘해도 언젠가 변경은 발생하기 마련이다. JDK의 설계자들은 고심 끝에 디폴트 메서드(default method)라는 것을 고안해 내었다. 디폴트 메서드는 추상 메서드의 기본적인 구현을 제공하는 메서드로, 추상 메서드가 아니기 때문에 디폴트 메서드가 새로 추가되어도 해당 인터페이스를 구현한 클래스를 변경하지 않아도 된다.
디폴트 메서드는 앞에 키워드 default를 붙이며, 추상 메서드와 달리 일반 메서드처럼 몸통{} 이 있어야 한다. 디폴트 메서드 역시 접근 제어자가 public이며, 생략가능하다. 대신, 새로 추가된 디폴트 메서드가 기존의 메서드와 이름이 중복되어 충돌하는 경우가 발생한다. 이런 충돌을 해결하는 규칙은 다음과 같다.

1) 여러 인터페이스의 디폴트 메서드 간의 충돌
- 인터페이스를 구현한 클래스에서 디폴트 메서드를 오버라이딩해야 한다.

2) 디폴트 메서드와 조상 클래스의 메서드 간의 충돌
- 조상 클래스의 메서드가 상속되고, 디폴트 메서드는 무시된다.

---

읽어주셔서 감사합니다. 😊 

__Reference__  
자바의 정석 - 남궁성  