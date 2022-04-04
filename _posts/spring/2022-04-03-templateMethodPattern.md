---
title:  "템플릿 메서드 패턴" 
excerpt: "템플릿 메서드 패턴에 대해 알아보자"

categories:
  - spring
tags:
  - [디자인 패턴, 템플릿 메서드 패턴]

toc: true
toc_sticky: true

date: 2022-04-04
last_modified_at: 2022-04-04
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

디자인 패턴중 하나인 `템플릿 메서드 패턴`에 대해 알아보자.  

## 개념
기존 코드에 로그 추적기를 추가해야 되는 상황이 생겼다고 가정해 보자.

**로그 추적기 도입 전 코드**
```java
  //OrderControllerV0 코드 
  @GetMapping("/v0/request")
  public String request(String itemId) {
      orderService.orderItem(itemId);
      return "ok";
  }

  //OrderServiceV0 코드
  public void orderItem(String itemId) {
      orderRepository.save(itemId);
  } 
```

**로그 추적기 도입 후 코드**
```java
  //OrderControllerV3 코드 
  @GetMapping("/v3/request")
  public String request(String itemId) {
      TraceStatus status = null;
      try {
          status = trace.begin("OrderController.request()");
          orderService.orderItem(itemId); //핵심 기능
          trace.end(status);
      } catch (Exception e) {
          trace.exception(status, e);
          throw e; 
      }

      return "ok";
  }

  //OrderServiceV3 코드
  public void orderItem(String itemId) {
      TraceStatus status = null;
      try {
          status = trace.begin("OrderService.orderItem()"); 
          orderRepository.save(itemId); //핵심 기능
          trace.end(status);
      } catch (Exception e) {
          trace.exception(status, e);
          throw e;
      }
  }
 
```

여기서 TraceStatus의 역할을 자세하게 알 필요는 없고 그냥 로그를 찍기 위한 클래스 라고 생각하면 된다.  

로그 추적기 도입 전과 후의 코드를 비교해서 보자.
도입 전 코드는 해당 메서드가 실제 처리해야 하는 핵심 기능만 깔끔하게 남아있다. 반면에 로그 추적기 도입 후 에는 핵심 기능보다 로그를 출력해야 하는 부가 기능 코드가 훨씬 더 많고 복잡하다.  
앞으로 코드를 설명할 때 핵심 기능과 부가 기능으로 구분해서 설명하겠다.

<br>

## 핵심 기능 vs 부가 기능
* **핵심 기능**은 해당 객체가 제공하는 고유의 기능이다. 예를 들어서 `orderService`의 핵심 기능은 주문로직이다.  
메서드 단위로 보면 orderService.orderItem()의 핵심 기능은 주문 리포지토리를 호출하는 orderRepository.save(itemId)코드가 핵심 기능이다.
* **부가 기능**은 핵심 기능을 보조하기 위해 제공되는 기능이다. 예를 들어서 로그 추적 로직, 트랜잭션 기능이 있다. 이러한 부가 기능은 단독으로 사용되지는 않고, 핵심 기능과 함께 사용된다. 예를 들어서 로그 추적 기능은 어떤 핵심 기능이 호출되었는지 로그를 남기기 위해 사용한다. 그러니까 핵심 기능을 보조하기 위해 존재한다.

로그 추적기 도입전에는 핵심 기능만 있지만, 로그 추적기를 추가한 코드는 핵심 기능과 부가 기능이 함께 섞여있다. 로그 추적기 도입 후 코드를 보면 핵심 기능 코드보다 부가 기능을 처리하기 위한 코드가 더 많아졌다. 소위 배보다 배꼽이 큰 상황이다. 만약 클래스가 수백 개라면 어떻게 하겠는가?

이 문제를 좀 더 효율적으로 처리할 수 있는 방법이 있을까?
로그 추적기 도입 후 코드를 유심히 잘 살펴보면 다음과 같이 동일한 패턴이 있다.

```java
  TraceStatus status = null;
  try {
      status = trace.begin("message"); 
      //핵심 기능 호출
      trace.end(status);
  } catch (Exception e) {
      trace.exception(status, e);
      throw e; 
  }
```

위의 코드에서 로그 추적기를 사용하는 구조는 모두 동일하다.  
중간에 핵심 기능을 사용하는 코드만 다를 뿐이다.  
부가 기능과 관련된 코드가 중복이니 중복을 별도의 메서드로 뽑아내면 될 것 같다. 그런데, `try ~ catch` 는 물론이고, 핵심 기능 부분이 중간에 있어서 단순하게 메서드로 추출하는 것은 어렵다.

## 변하는 것과 변하지 않는 것을 분리
좋은 설계는 변하는 것과 변하지 않는 것을 분리하는 것이다.  
여기서 핵심 기능 부분은 변하고, 로그 추적기를 사용하는 부분은 변하지 않는 부분이다. 이 둘을 분리해서 모듈화해야 한다.  
`템플릿 메서드 패턴(Template Method Pattern)`은 이런 문제를 해결하는 디자인 패턴이다.

## 🔔 템플릿 메서드 패턴 - 예제1

**TemplateMethodTest**
```java
  @Slf4j
  public class TemplateMethodTest {
    @Test
    void templateMethodV0() {
      logic1();
      logic2();
    }

    private void logic1() {
      long startTime = System.currentTimeMillis(); //비즈니스 로직 실행
      log.info("비즈니스 로직1 실행");
      //비즈니스 로직 종료
      long endTime = System.currentTimeMillis(); long resultTime = endTime - startTime; log.info("resultTime={}", resultTime);
    }

    private void logic2() {
      long startTime = System.currentTimeMillis(); //비즈니스 로직 실행
      log.info("비즈니스 로직2 실행");
      //비즈니스 로직 종료
      long endTime = System.currentTimeMillis();
      long resultTime = endTime - startTime;
      log.info("resultTime={}", resultTime);
    }
  }
```
<br>

**실행 결과**
```
비즈니스 로직1 실행
resultTime=5
비즈니스 로직2 실행
resultTime=1
```
`logic1()` 과 `logic2()` 는 시간을 측정하는 부분과 비즈니스 로직을 실행하는 부분이 함께 존재한다.

* 변하는 부분: 비즈니스 로직
* 변하지 않는 부분: 시간 측정

이제 템플릿 메서드 패턴을 사용해서 변하는 부분과 변하지 않는 부분을 분리해 보자.

## 🔔 템플릿 메서드 패턴 - 예제2

템플릿 메서드 패턴 구조 그림
![img06](https://user-images.githubusercontent.com/93430103/161556921-074467bd-07d7-4ae9-9d1b-816ac6d9aa7a.png)

**AbstractTemplate**
```java
  @Slf4j
  public abstract class AbstractTemplate {
    public void execute() {
      long startTime = System.currentTimeMillis(); //비즈니스 로직 실행
      call(); //상속
      //비즈니스 로직 종료
      long endTime = System.currentTimeMillis(); 
      long resultTime = endTime - startTime; 
      log.info("resultTime={}", resultTime);
    }
    protected abstract void call();
  }
```
템플릿 메서드 패턴은 이름 그대로 템플릿을 사용하는 방식이다. 템플릿은 기준이 되는 거대한 틀이다. 템플릿이라는 틀에 변하지 않는 부분을 몰아둔다. 그리고 일부 변하는 부분을 별도로 호출해서 해결한다.
<br>

`AbstractTemplate` 코드를 보자. 변하지 않는 부분인 시간 측정 로직을 몰아둔 것을 확인할 수 있다. 이제 이것이 하나의 템플릿이 된다. 그리고 템플릿 안에서 변하는 부분은 call() 메서드를 호출해서 처리한다. 템플릿 메서드 패턴은 부모 클래스에 변하지 않는 템플릿 코드를 둔다. 그리고 변하는 부분은 자식 클래스에 두고 상속과 오버라이딩을 사용해서 처리한다.

<br>

**SubClassLogic1**
```java
  @Slf4j
  public class SubClassLogic1 extends AbstractTemplate {
      @Override
      protected void call() {
        log.info("비즈니스 로직1 실행"); 
      }
  }
```
변하는 부분인 비즈니스 로직1을 처리하는 자식 클래스이다. 템플릿이 호출하는 대상인 `call()`메서드를 오버라이딩 한다.

**SubClassLogic2**
```java
  @Slf4j
  public class SubClassLogic2 extends AbstractTemplate {
    @Override
    protected void call() { 
      log.info("비즈니스 로직2 실행");
    } 
  }
```
변하는 부분인 비즈니스 로직2를 처리하는 자식 클래스이다. 템플릿이 호출하는 대상인 `call()` 메서드를 오버라이딩 한다.


**TemplateMethodTest - templateMethodV1() 추가**
```java
  /**
   * 템플릿 메서드 패턴 적용
   */
  @Test
  void templateMethodV1() {
    AbstractTemplate template1 = new SubClassLogic1();
    template1.execute();
    AbstractTemplate template2 = new SubClassLogic2();
    template2.execute();
  }
```

**실행 결과**
```
비즈니스 로직1 실행 resultTime=0
비즈니스 로직2 실행 resultTime=1
```

**템플릿 메서드 패턴 인스턴스 호출 그림**
![img07](https://user-images.githubusercontent.com/93430103/161556925-83c55a5e-e93c-43c4-87f1-d9054443929d.png)

`template1.execute()` 를 호출하면 템플릿 로직인 `AbstractTemplate.execute()` 를 실행한다. 여기서 중간에 `call()` 메서드를 호출하는데, 이 부분이 오버라이딩 되어있다. 따라서 현재 인스턴스인 `SubClassLogic1` 인스턴스의 `SubClassLogic1.call()` 메서드가 호출된다.

템플릿 메서드 패턴은 이렇게 다형성을 사용해서 변하는 부분과 변하지 않는 부분을 분리하는 방법이다.

## 🔔 템플릿 메서드 패턴 - 예제3

**익명 내부 클래스 사용하기**
템플릿 메서드 패턴은 `SubClassLogic1`, `SubClassLogic2` 처럼 클래스를 계속 만들어야 하는 단점이 있다. 익명 내부 클래스를 사용하면 이런 단점을 보완할 수 있다.  
익명 내부 클래스를 사용하면 객체 인스턴스를 생성하면서 동시에 생성할 클래스를 상속 받은 자식 클래스를 정의할 수 있다. 이 클래스는 `SubClassLogic1` 처럼 직접 지정하는 이름이 없고 클래스 내부에 선언되는 클래스여서 익명 내부 클래스라 한다.  

**TemplateMethodTest - templateMethodV2() 추가**


```java
  /**
   * 템플릿 메서드 패턴,익명 내부 클래스 사용
   */
  @Test
  void templateMethodV2() {
    AbstractTemplate template1 = new AbstractTemplate() {
      @Override
      protected void call() { 
        log.info("비즈니스 로직1 실행");
      }
    };
    log.info("클래스 이름1={}", template1.getClass()); 
    template1.execute();
    
    AbstractTemplate template2 = new AbstractTemplate() {
      @Override
      protected void call() {
        log.info("비즈니스 로직1 실행");
      }
    };
    log.info("클래스 이름2={}", template2.getClass());
    template2.execute();
  }
```

**실행 결과**
```
클래스 이름1 class hello.advanced.trace.template.TemplateMethodTest$1 
비즈니스 로직1 실행
resultTime=3
클래스 이름2 class hello.advanced.trace.template.TemplateMethodTest$2 
비즈니스 로직2 실행
resultTime=0
```
실행 결과를 보면 자바가 임의로 만들어주는 익명 내부 클래스 이름은 TemplateMethodTest$1 , TemplateMethodTest$2 인 것을 확인할 수 있다.

![img08](https://user-images.githubusercontent.com/93430103/161558373-d0549ea4-4640-4e4b-a242-b60a52f1327b.png)





[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
