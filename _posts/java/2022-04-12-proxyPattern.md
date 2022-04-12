---
title:  "프록시 패턴" 
excerpt: "프록시 패턴에 대해 알아보자"

categories:
  - Design Patterns
tags:
  - [프록시 패턴]

toc: true
toc_sticky: true

date: 2022-04-12
last_modified_at: 2022-04-12
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}


**GOF 디자인 패턴**
둘다 프록시를 사용하는 방법이지만 GOF 디자인 패턴에서는 이 둘을 `의도(intent)` 에 따라서 프록시 패턴과 데코레이터 패턴으로 구분한다.  
* 프록시 패턴: 접근 제어가 목적
* 데코레이터 패턴: 새로운 기능 추가가 목적

둘다 프록시를 사용하지만, `의도` 가 다르다는 점이 핵심이다.  
용어가 프록시 패턴이라고 해서 해당 패턴만 프록시를 사용하는 것은 아니다.  
데코레이터 패턴도 프록시를 사용한다.

>**참고**: 프록시라는 개념은 클라이언트 서버라는 큰 개념안에서 자연스럽게 발생할 수 있다. 프록시는 객체안에서의 개념도 있고, 웹 서버에서의 프록시도 있다. 객체안에서 객체로 구현되어있는가, 웹 서버로 구현되어 있는가 처럼 규모의 차이가 있을 뿐 근본적인 역할은 같다.

## 🔔 프록시 패턴 - 예제 코드1

![img08](https://user-images.githubusercontent.com/93430103/162958262-93e11f4d-39dd-44ae-a15c-6faca591c1e8.png)

![img09](https://user-images.githubusercontent.com/93430103/162958604-1381408c-5d1b-4058-a23e-d907f7c8cb2d.png)

<br>

**Subject 인터페이스**
```java
public interface Subject {
      String operation();
}
```

**RealSubject**
```java
  @Slf4j
  public class RealSubject implements Subject {
      @Override
      public String operation() {
          log.info("실제 객체 호출"); 
          sleep(1000);
          return "data";
      }
      
      private void sleep(int millis) {
          try {
              Thread.sleep(millis);
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
```
`RealSubject` 는 `Subject` 인터페이스를 구현했다. `operation()` 은 데이터 조회를 시뮬레이션 하기 위해 1초 쉬도록 했다.  
예를 들어서 데이터를 DB나 외부에서 조회하는데 1초가 걸린다고 생각하면 된다.  
호출할 때 마다 시스템에 큰 부하를 주는 데이터 조회라고 가정하자.

**ProxyPatternClient**
```java
  public class ProxyPatternClient {
      private Subject subject;
      
      public ProxyPatternClient(Subject subject) {
          this.subject = subject;
      }
      
      public void execute() {
          subject.operation();
      }
  }
```
`Subject` 인터페이스에 의존하고, `Subject` 를 호출하는 클라이언트 코드이다.  
 `execute()` 를 실행하면 `subject.operation()` 를 호출한다.

**ProxyPatternTest**
```java
  public class ProxyPatternTest {
      @Test
      void noProxyTest() {
          RealSubject realSubject = new RealSubject();
          ProxyPatternClient client = new ProxyPatternClient(realSubject);
          client.execute();
          client.execute();
          client.execute();
      }
  }
```
테스트 코드에서는 `client.execute()` 를 3번 호출한다. 데이터를 조회하는데 1초가 소모되므로 총 3 초의 시간이 걸린다.

**실행 결과**
```
RealSubject - 실제 객체 호출 
RealSubject - 실제 객체 호출 
RealSubject - 실제 객체 호출
```
**client.execute()을 3번 호출하면 다음과 같이 처리된다.**
* 1. client -> realSubject 를 호출해서 값을 조회한다. (1초)
* 2. client -> realSubject 를 호출해서 값을 조회한다. (1초) 
* 3. client -> realSubject 를 호출해서 값을 조회한다. (1초)

그런데 이 데이터가 한번 조회하면 변하지 않는 데이터라면 어딘가에 보관해두고 이미 조회한 데이터를 사용하는 것이 성능상 좋다. 이런 것을 캐시라고 한다.  
프록시 패턴의 주요 기능은 접근 제어이다. 캐시도 접근 자체를 제어하는 기능 중 하나이다.

이미 개발된 로직을 전혀 수정하지 않고, 프록시 객체를 통해서 캐시를 적용해보자.


## 🔔 프록시 패턴 - 예제 코드2
프록시 패턴을 적용하자.

![img10](https://user-images.githubusercontent.com/93430103/162960442-b6a61fde-cbe2-45ad-b98c-4eb9ed84f03d.png)

![img11](https://user-images.githubusercontent.com/93430103/162960460-895d7bb7-2014-4fa5-858c-f43dcd914559.png)


**ChcheProxy**
```java
  @Slf4j
  public class CacheProxy implements Subject {
      private Subject target;
      private String cacheValue;
      
      public CacheProxy(Subject target) {
          this.target = target;
      }
        
      @Override
      public String operation() {
          log.info("프록시 호출");
          if (cacheValue == null) {
              cacheValue = target.operation();
          }
          return cacheValue;
      }
  }
```

앞서 설명한 것 처럼 프록시도 실제 객체와 그 모양이 같아야 하기 때문에 `Subject` 인터페이스를 구현해야
한다.

* `private Subject target` : 클라이언트가 프록시를 호출하면 프록시가 최종적으로 실제 객체를 호출해야 한다. 따라서 내부에 실제 객체의 참조를 가지고 있어야 한다. 이렇게 프록시가 호출하는 대상을 `target` 이라 한다.
* `operation()` : 구현한 코드를 보면 `cacheValue` 에 값이 없으면 실제 객체(`target`) 를 호출해서 값을 구한다. 그리고 구한 값을 `cacheValue` 에 저장하고 반환한다. 만약 `cacheValue` 에 값이 있으면 실제 객체를 전혀 호출하지 않고, 캐시 값을 그대로 반환한다. 따라서 처음 조회 이후에는 캐시(`cacheValue`) 에서 매우 빠르게 데이터를 조회할 수 있다.

**ProxyPatternTest - cacheProxyTest() 추가**
```java
  public class ProxyPatternTest {
      @Test
      void noProxyTest() {
          RealSubject realSubject = new RealSubject();
          ProxyPatternClient client = new ProxyPatternClient(realSubject);
          
          client.execute();
          client.execute();
          client.execute();
      }

      @Test
      void cacheProxyTest() {
          Subject realSubject = new RealSubject();
          Subject cacheProxy = new CacheProxy(realSubject);
          ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
          
          client.execute();
          client.execute();
          client.execute();
      }
  }
```

**cacheProxyTest()**  
`realSubject` 와 `cacheProxy` 를 생성하고 둘을 연결한다. 결과적으로 `cacheProxy` 가 `realSubject` 를 참조하는 런타임 객체 의존관계가 완성된다. 그리고 마지막으로 `client` 에 `realSubject` 가 아닌 `cacheProxy` 를 주입한다. 이 과정을 통해서 `client -> cacheProxy -> realSubject` 런타임 객체 의존 관계가 완성된다.

`cacheProxyTest()` 는 `client.execute()` 을 총 3번 호출한다. 이번에는 클라이언트가 실제 `realSubject` 를 호출하는 것이 아니라 `cacheProxy` 를 호출하게 된다.

**실행 결과**
```
CacheProxy - 프록시 호출 
RealSubject - 실제 객체 호출 
CacheProxy - 프록시 호출 
CacheProxy - 프록시 호출
```

***client.execute()을 3번 호출하면 다음과 같이 처리된다.***
* 1. client의 cacheProxy 호출 -> cacheProxy에 캐시 값이 없다. -> realSubject를 호출, 결과를 캐시에 저장 (1초)
* 2. client의 cacheProxy 호출 -> cacheProxy에 캐시 값이 있다. -> cacheProxy에서 즉시 반환 (0초)
* 3. client의 cacheProxy 호출 -> cacheProxy에 캐시 값이 있다. -> cacheProxy에서 즉시 반환 (0초)

결과적으로 캐시 프록시를 도입하기 전에는 3초가 걸렸지만, 캐시 프록시 도입 이후에는 최초에 한번만 1 초가 걸리고, 이후에는 거의 즉시 반환한다.


## 🔔 프록시 패턴 - 정리
프록시 패턴의 핵심은 `RealSubject` 코드와 클라이언트 코드를 전혀 변경하지 않고, 프록시를 도입해서 접근 제어를 했다는 점이다.  
그리고 클라이언트 코드의 변경 없이 자유롭게 프록시를 넣고 뺄 수 있다. 실제 클라이언트 입장에서는 프록시 객체가 주입되었는지, 실제 객체가 주입되었는지 알지 못한다.
 

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
