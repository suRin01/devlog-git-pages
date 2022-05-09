---
layout: post
title: "디자인 패턴 - 싱글톤 편"
subtitle: ""
date: 2022-05-03 12:30:15 +0900
background: "/img/posts/05.jpg"
---

개발 공부하다, 그런 생각이 들었다.
그냥 로직을 짜기만 하는게 아니라 전체적인 구조를 어떻게 하면 더 잘 짤 수 있을까,
개발 원칙에 어긋나지 않으면서 내가 원하는 기능을 구현할 수 있을까. 라는 생각들

개발 공부하면서 누군가는 알고리즘 테스트르 준비용으로 백준이나 릿코드 등에서 문제를 풀어보라고도 했고,
컴퓨터 구조에 대해 공부해보라고도 했는데

컴퓨터 구조는 대학 수업 때 배우기도 하고(완벽히 이해하고 암기하고 있다라고는 절대 말 못하지만) 알고리즘 테스트 준비는 조금씩 해 가고 있기 때문에
누군가 디자인 패턴에 대해서 물어봤을때 대답할 만한건 싱글톤 패턴과 팩토리 패턴밖에 말 못하겠더라.

그래서 디자인 패턴에 대해서 공부해보기로 했다!(?)

# 디자인 패턴?

- 소프트웨어를 설계할 때 특정 맥락에서 자주 발생하는 고질적인 문제들이 또 발생했을 때 재사용할 할 수있는 훌륭한 해결책
- “바퀴를 다시 발명하지 마라(Don’t reinvent the wheel)” 이미 만들어져서 잘 되는 것을 처음부터 다시 만들 필요가 없다는 의미이다.

- 각기 다른 소프트웨어 모듈이나 기능을 가진 다양한 응용 소프트웨어 시스템들을 개발할 때도 서로 간에 공통되는 설계 문제가 존재하며 이를 처리하는 해결책 사이에도 공통점이 있다. 이러한 유사점을 패턴이라 한다.
- 패턴은 공통의 언어를 만들어주며 팀원 사이의 의사 소통을 원활하게 해주는 아주 중요한 역할을 한다.

# 디자인 패턴 구조

디자인 패턴은 콘텍스트, 문제, 해결의 구조를 가진다
콘텍스트는 문제가 발생되는 상황을 기술하고, 패턴이 유용하지 못한 상황을 나타내기도 한다.
문제는 패턴이 적용되어 해결될 필요가 있는 디자인 이슈들을 기술하고, 제약 사항과 영향력도 문제 해결을 위해 고려되어야 한다.
해결은 무넺를 해결하도록 설계를 구성하는 요소들과 그 요소들 사이의 관게, 책임, 협력 관게를 기술한다. 해결은 반드시 구체적인 구현 방법이나 언어에 의존적이지 않으며 다양한 상황에 적용할 수 있는 일종의 탬플릿이다.

# 디자인 패턴의 종류

가장 유명한 패턴은 GoF의 디자인 패턴이다.
소프트웨어 개발 영역에서 디자인 패턴을 구체화하고 체계화한 사람 4명이 23가지의 디자인 패턴을 정리하고 각각의 디자인 패턴을 생성, 구조, 행위 3가지로 분류했다.

- 생성 패턴
  추상 팩토리, 빌더, 팩토리 매서드, 프로토타입, 싱틀톤
- 구조 패턴
  어댑터, 브리지, 컴퍼지트, 데코레이터, 파사드, 플라이웨이트, 프록시
- 행위 패턴
  책임 연쇄, 커맨드, 인터프리터, 이터레이터, 미디에이터, 메멘토, ,옵서버, 스테이트, 스트레티지, 템플릿 메서드, 비지터

## 생성 패턴

객체 생성에 관련된 패턴으로, 객체의 생성과 조합을 캡슐화해 특정 객체가 생성되거나 변경되어도 프로그램 구조에 영향을 크ㅡ게 받지 않도록 유연성을 제공한다.

## 구조 패턴

클래스나 객체를 조합해 더 큰 구조를 만드는 패턴
예를 들어 서로다른 인터페이스를 지닌 2개의 객체를 묶어 단일 인터페이스를 제공하거나, 객체들을 서로 묶어 새로운 기능을 제공하는 패터

## 행위 패턴

객체나 클래스 사이의 알고리즘이나 책임 분배에 관한 패턴

오늘은 이 중, 가장 많이 접해봤고 믾이 쓰이게 될 것 같은 싱글톤 패턴에 대해서 공부해 볼 예정이다.

# Singleton Pattern

싱글톤 패턴이란 전역 변수를 사용하지 않고 객체를 하나만 생성하도록 하며, 생성된 객체를 어디서든지 참조할 수 있도록 하는 패턴이다.

하나의 인스턴스만을 생성하는 책임이 있으며 getInstance()메소드를 통해 모든 클라이언트들에게 동일한 인터페이스를 반환하는 작업을 수행한다.

## 문제 상황

싱글톤 패턴을 가장 많이 사용했던 적은 전역으로 사용되어야 하고, 하나의 객체를 유지해야 하는 DB커넥션 풀을 생성할 때 사용한 사례이다.
커넥션 풀을 하나로 유지하고, 그 풀을 통해서 커넥션을 만들어내고 조절함하고, 서비스 레이어에서 여러 클래스에서 사용하기 때문에 가장 자주 사용하게 되었다.

## 싱글톤 패턴의 예시

```java
public class DataBasePool {
  private static DataBasePool pool = null;
  private Printer() { }
  public static DataBasePool getDataBasePool(){
    if (pool == null) {
      pool = new DataBasePool();
    }
    return pool;
  }
  public PoolConnection getConnection() {
    return pool.createConnection();
  }
}
```

예시로, 위와 같은 코드가 있다고 생각하자.
DataBasePool 클래스의 생성자는 priavte으로 선언되어 있고, new DataBasePool()을 통해서 외부에서 생성 할 수 없다.
public으로 선언된 getDataBaseBool 메소드를 실행할 때, 클래스 내의 static pool에 객체가 들어가게 되고, 이제 DataBasePool.getDataBasePool().getConnection()를 통해서 원하는 작업을 수행할 수 있게 된다.

## 싱글톤 패턴의 문제점

싱글톤 패턴은 다중 스레드에서 싱글톤 패턴을 적용한 클래스를 사용할 때, 인스턴스가 1개 이상 생성되는 경우가 발생할 수 있다.

- 경합 조건(Race Condition)이 발생하는 경우
  경합 조건이란, 메모리가 동일한 자원을 2개 이상의 스레드가 이용하려고 경합하는 현장이다.
  DataBasePool 인스턴스가 아직 생성되지 않았을 때 스레드 1이 getDataBaseBool 메소드를 실행시켰을 때, 멤버변수 pool은 아직 null 상태이다.
  그런데 인스턴스가 생성되고 넣어지기 전, 스레드 2가 getDataBasePool을 실행했을 때, 아직 pool은 null 상태이기 때문에 다시 connection pool을 생성하려고 시도한다.
  이제 스레드 1에서 connection pool이 생성되고, pool에 주입하는데 스레드 2 또한 같은 작업을 반복한다.

이와 같은 경우 인스턴스가 2개 생성되게 된다.

다중 스레드 어플리케이션이 아닌 경우에는 아무런 문제가 되지 않겠지만,
다중 스레드 어플리케이션일 경우 문제는 언제든지 발생할 수 있다.

2가지의 방법으로 해결할 수 있다.

1. 정적 변수에 인스턴스를 만들어 바로 초기화 하는 방법(Eager Initializtion)
2. 인스턴스를 만드는 메소드에 동기화하는 방법(Thread-safe Initialization)

### 정적 변수에 인스턴스를 만들어 바로 초기화 하는 방법

```java
public class DataBasePool {
  private static DataBasePool pool = new DataBasePool();
  private Printer() { }
  public static DataBasePool getDataBasePool(){
    return pool;
  }
  public PoolConnection getConnection() {
    return pool.createConnection();
  }
}
```

객체가 생성되기 전 클래스가 메모리에 로딩될 때 만들어져 초기화가 한번만 실행된다.
이 방법은 프로그램 시작-종료까지 없어지지 않고 메모리에 계속 상주하며 클래스에서 생성된 모든 객체에서 참조할 수 있다.

### 인스턴스를 만드는 메소드에 동기화하는 방법

```java
public class DataBasePool {
  private static DataBasePool pool = null;
  private Printer() { }
  public synchronized static DataBasePool getDataBasePool(){
    return pool;
  }
  public PoolConnection getConnection() {
    return pool.createConnection();
  }
}
```

인스턴스를 만드는 메소드를 임계 구역으로 변경한다. 다중 스레드 환경에서 동시에 여러 스레드가 getDataBaseBool메소드를 소유하는 객체에 접근하는 것을 방지한다.
하지만 이 방법은 getInstance()에 Lock을 거는 방식이라, 접근을 위해서 락이 걸려있는지 확인하고, 걸려있다면 락을 풀고 하는 시간 때문에 속도가 상대적으로 느리다.

추가적으로, 싱클톤 패턴을 사용하지 않고 싱글톤 클래스와 같은 효과를 볼 수 있는 방법들 또한 존재한다.

### 정적 클래스

```java
public class DataBasePool {
  private static int counter = 0;
  public synchronized PoolConnection getConnection() {
    system.out.println(counter);
  }
}
```

정적 클래스는 객체를 생성하지 않고 메소드를 바로 사용한다. 정적 메소드를 사용하므로 일반적으로 실행할 때 컴파일 시간에 바인딩되는 인스턴스 메소드를 사용하는 것보다 성능 명에서 우수하다.

하지만 이와 같은 방법은 인터페이스를 구현해야 하는 경우 사용할 수 없다.

### Enum 클래스

```java
public enum EnumSingleton {

    INSTANCE("Initial class info");

    private String info;

    private EnumSingleton(String info) {
        this.info = info;
    }

    public EnumSingleton getInstance() {
        return INSTANCE;
    }
}
```

Thread-safety와 Serialization이 보장되고, Reflection을 방지할 수 있다.
싱글톤에서 발생하는 단일 인스턴스 원칙 위반을 해결하게 된다.

## 싱글톤 사용시 주의할 점

싱글톤이 단순한 패턴이라 프로그래머가 실수할 일은 없지만, 정말 필요하지 않은데 사용하거나, 구현할 때 주의사항이 있게 된다.
싱글톤은 개념적으로는 전역 변수이기 때문에, 변경가능한 경우에는 사용을 피해야 하고, 일반적으로 전역 변수는 남용하면 안된다.
이는 인터페이스의 구현으로 드러낼 수 있는 의존성들이 코드 안에 가려지게 되고, 코드 결합도가 증가할 수 있으며, 어플리케이션의 생명주기와 객체의 생명주기가 같아질 수 이ㅣㅆ어 단위 테스트가 어려워 질 수 있다.
