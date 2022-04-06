---
title:  "전략 패턴" 
excerpt: "전략 패턴에 대해 알아보자"

categories:
  - spring
tags:
  - [디자인 패턴, 전략 패턴]

toc: true
toc_sticky: true

date: 2022-04-06
last_modified_at: 2022-04-06
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

지난번에 알아본 `템플릿 메서드 패턴`에 이어 오늘은 `전략 패턴`에 대해 알아보자.  


## 🔔 전략 패턴 - 시작
전략 패턴의 이해를 돕기 위해 템플릿 메서드 패턴에서 만들었던 동일한 예제를 사용하겠다.

<br>

**ContextV1Test**
```java
  @Slf4j
  public class ContextV1Test {
    @Test
    void strategyV0() {
      logic1();
      logic2();
    }

  private void logic1() {
    long startTime = System.currentTimeMillis();
    //비즈니스 로직 실행
    log.info("비즈니스 로직1 실행");
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis();
    long resultTime = endTime - startTime;
    log.info("resultTime={}", resultTime);
  }

  private void logic2() {
    long startTime = System.currentTimeMillis(); 
    //비즈니스 로직 실행
    log.info("비즈니스 로직2 실행");
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis(); 
    long resultTime = endTime - startTime; 
    log.info("resultTime={}", resultTime);
  }
}
```
**실행 결과**
```
비즈니스 로직1 실행 
resultTime=5
비즈니스 로직2 실행 
resultTime=1
```

<br>

## 🔔 전략 패턴 - 예제1 (필드에 전략을 보관하는 방식)
탬플릿 메서드 패턴은 부모 클래스에 변하지 않는 템플릿을 두고, 변하는 부분을 자식 클래스에 두어서 상속을 사용해서 문제를 해결했다.  
전략 패턴은 변하지 않는 부분을 Context 라는 곳에 두고, 변하는 부분을 Strategy 라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 해서 문제를 해결한다. 상속이 아니라 위임으로 문제를 해결하는 것이다.  
전략 패턴에서 Context 는 변하지 않는 템플릿 역할을 하고, Strategy 는 변하는 알고리즘 역할을 한다.

>GOF 디자인 패턴에서 정의한 전략 패턴의 의도는 다음과 같다.  
알고리즘 제품군을 정의하고 각각을 캡슐화하여 상호 교환 가능하게 만들자. 전략을 사용하면 알고리즘을
사용하는 클라이언트와 독립적으로 알고리즘을 변경할 수 있다.

![img10](https://user-images.githubusercontent.com/93430103/161970869-ea420176-8d44-4e6e-ae33-ddfc464191d2.png)

<br>

**Strategy 인터페이스**
```java
  public interface Strategy {
    void call();
  }
```
이 인터페이스는 변하는 알고리즘 역할을 한다.

<br>

**StrategyLogic1**
```java
  @Slf4j
  public class StrategyLogic1 implements Strategy {
    @Override
    public void call() {
      log.info("비즈니스 로직1 실행");
    }
  }
```
변하는 알고리즘은 `Strategy` 인터페이스를 구현하면 된다. 여기서는 비즈니스 로직1을 구현했다.

<br>

**StrategyLogic2**
```java
  @Slf4j
  public class StrategyLogic2 implements Strategy {
    @Override
    public void call() {
      log.info("비즈니스 로직2 실행");
    }
  }
```
비즈니스 로직2를 구현했다.

<br>

**ContextV1**
```java
  /**
   * 필드에 전략을 보관하는 방식
   */
  @Slf4j
  public class ContextV1 {
    private Strategy strategy;
      
    public ContextV1(Strategy strategy) {
      this.strategy = strategy;
    }

  public void execute() {
    long startTime = System.currentTimeMillis(); 
    //비즈니스 로직 실행
    strategy.call(); //위임
    //비즈니스 로직 종료
    long endTime = System.currentTimeMillis(); 
    long resultTime = endTime - startTime; 
    log.info("resultTime={}", resultTime);
  }
}
```
`ContextV1` 은 변하지 않는 로직을 가지고 있는 템플릿 역할을 하는 코드이다. 전략 패턴에서는 이것을 컨텍스트(문맥)이라 한다.  
쉽게 이야기해서 컨텍스트(문맥)는 크게 변하지 않지만, 그 문맥 속에서 strategy 를 통해 일부 전략이 변경된다 생각하면 된다.  

Context 는 내부에 Strategy strategy 필드를 가지고 있다. 이 필드에 변하는 부분인 Strategy 의 구현체를 주입하면 된다.  
전략 패턴의 핵심은 Context 는 Strategy 인터페이스에만 의존한다는 점이다. 덕분에 Strategy 의 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향을 주지 않는다.

<br>

**ContextV1Test - 추가**
```java
  /**
   *전략 패턴 적용
   */
  @Test
  void strategyV1() {
    Strategy strategyLogic1 = new StrategyLogic1();
    ContextV1 context1 = new ContextV1(strategyLogic1);
    context1.execute();
    
    Strategy strategyLogic2 = new StrategyLogic2();
    ContextV1 context2 = new ContextV1(strategyLogic2);
    context2.execute();
}
```
전략 패턴을 사용해보자.  
코드를 보면 의존관계 주입을 통해 ContextV1 에 Strategy 의 구현체인 strategyLogic1 를 주입하는 것을 확인할 수 있다. 이렇게해서 Context 안에 원하는 전략을 주입한다. 이렇게 원하는 모양으로 조립을 완료하고 난 다음에 context1.execute() 를 호출해서 context 를 실행한다.

<br>

**전략 패턴 실행 그림**
![img11](https://user-images.githubusercontent.com/93430103/161973007-8169d9e7-2cf6-4cc2-b535-c871cfdbfc86.png)

1. `Context`에 원하는 `Strategy` 구현체를 주입한다.
2. 클라이언트는 `context` 를 실행한다.
3. `context` 는 `context` 로직을 시작한다.
4. `context` 로직 중간에 `strategy.call()` 을 호출해서 주입 받은 `strategy` 로직을 실행한다.
5. `context` 는 나머지 로직을 실행한다.

<br>

**실행 결과**
```
StrategyLogic1 - 비즈니스 로직1 실행 
ContextV1 - resultTime=3
StrategyLogic2 - 비즈니스 로직2 실행 
ContextV1 - resultTime=0
```
## 🔔 전략 패턴 - 예제2
전략 패턴도 익명 내부 클래스를 사용할 수 있다.

<br>

**ContextV1Test - 추가**
```java
  /**
   * 전략 패턴 익명 내부 클래스1
   */
  @Test
  void strategyV2() {
    Strategy strategyLogic1 = new Strategy() {
      @Override
      public void call() {
        log.info("비즈니스 로직1 실행");
      }
    };
    log.info("strategyLogic1={}", strategyLogic1.getClass());
    ContextV1 context1 = new ContextV1(strategyLogic1);
    context1.execute();
    
    Strategy strategyLogic2 = new Strategy() {
      @Override
      public void call() {
        log.info("비즈니스 로직2 실행");
      }
    };
    log.info("strategyLogic2={}", strategyLogic2.getClass());
    ContextV1 context2 = new ContextV1(strategyLogic2);
    context2.execute();
  }
```

**실행 결과**
```
ContextV1Test - strategyLogic1=class 
hello.advanced.trace.strategy.ContextV1Test$1 
ContextV1Test - 비즈니스 로직1 실행
ContextV1 - resultTime=0
ContextV1Test - strategyLogic2=class 
hello.advanced.trace.strategy.ContextV1Test$2 
ContextV1Test - 비즈니스 로직2 실행
ContextV1 - resultTime=0
```
실행 결과를 보면 `ContextV1Test$1` , `ContextV1Test$2`와 같이 익명 내부 클래스가 생성된 것을 확인할 수 있다.

**ContextV1Test -추가**
```java
  /**
   * 전략 패턴 익명 내부 클래스2
   */
  @Test
  void strategyV3() {
    ContextV1 context1 = new ContextV1(new Strategy() {
      @Override
      public void call() { 
        log.info("비즈니스 로직1 실행");
      }
    });
    context1.execute();
    
    ContextV1 context2 = new ContextV1(new Strategy() {
      @Override
      public void call() {
        log.info("비즈니스 로직2 실행"); 
      }
    });
    context2.execute();
  }
```
익명 내부 클래스를 변수에 담아두지 말고, 생성하면서 바로 `ContextV1`에 전달해도 된다.

**ContextV1Test - 추가**
```java
  /**
   * 전략 패턴, 람다
   */
  @Test
  void strategyV4() {
    ContextV1 context1 = new ContextV1(() -> log.info("비즈니스 로직1 실행")); 
    context1.execute();

    ContextV1 context2 = new ContextV1(() -> log.info("비즈니스 로직2 실행"));
    context2.execute();
  }
```
익명 내부 클래스를 자바8부터 제공하는 람다로 변경할 수 있다. 람다로 변경하려면 인터페이스에 메서드가 1개만 있으면 되는데, 여기에서 제공하는 `Strategy` 인터페이스는 메서드가 1개만 있으므로 람다로 사용할 수 있다.

<br>

**정리**  
지금까지 일반적으로 이야기하는 전략 패턴에 대해서 알아보았다. 변하지 않는 부분을 `Context`에 두고 변하는 부분을 `Strategy`를 구현해서 만든다. 그리고 `Context`의 내부 필드에 `Strategy`를 주입해서 사용했다.

<br>

**선 조립, 후 실행**  
여기서 이야기하고 싶은 부분은 `Context`의 내부 필드에 `Strategy`를 두고 사용하는 부분이다.  
이 방식은 `Context`와 `Strategy`를 실행 전에 원하는 모양으로 조립해두고, 그 다음에 `Context`를 실행하는 선 조립, 후 실행 방식에서 매우 유용하다.  
`Context`와 `Strategy`를 한번 조립하고 나면 이후로는 `Context`를 실행하기만 하면 된다.  
우리가 스프링으로 애플리케이션을 개발할 때 애플리케이션 로딩 시점에 의존관계 주입을 통해 필요한 의존관계를 모두 맺어두고 난 다음에 실제 요청을 처리하는 것 과 같은 원리이다.  
이 방식의 단점은 `Context`와 `Strategy`를 조립한 이후에는 전략을 변경하기가 번거롭다는 점이다. 물론 `Context`에 `setter`를 제공해서 `Strategy`를 넘겨 받아 변경하면 되지만, `Context`를 싱글톤으로 사용할 때는 동시성 이슈 등 고려할 점이 많다. 그래서 전략을 실시간으로 변경해야 하면 차라리 이전에 개발한 테스트 코드 처럼 `Context`를 하나더 생성하고 그곳에 다른 `Strategy`를 주입하는 것이 더 나은 선택일 수 있다.

이렇게 먼저 조립하고 사용하는 방식보다 더 유연하게 전략 패턴을 사용하는 방법을 없을까?

## 🔔 전략 패턴 - 예제3 (전략을 파라미터로 전달 받는 방식)
이번에는 전략 패턴을 조금 다르게 사용해보자. 이전에는 `Context`의 필드에 `Strategy`를 주입해서
사용했다. 이번에는 전략을 실행할 때 직접 파라미터로 전달해서 사용해보자.

<br>

**ContextV2**
```java
  /**
   * 전략을 파라미터로 전달 받는 방식
   */
  @Slf4j
  public class ContextV2 {
    public void execute(Strategy strategy) {
      long startTime = System.currentTimeMillis(); 
      //비즈니스 로직 실행
      strategy.call(); //위임
      //비즈니스 로직 종료
      long endTime = System.currentTimeMillis(); 
      long resultTime = endTime - startTime; 
      log.info("resultTime={}", resultTime);
    }
  }
```
`ContextV2`는 전략을 필드로 가지지 않는다. 대신에 전략을 `execute(..)`가 호출될 때 마다 항상 파라미터로 전달 받는다.

<br>

**ContextV2Test**  
```java
  @Slf4j
  public class ContextV2Test {

    /**
     * 전략 패턴 적용
     */
    @Test
    void strategyV1() {
      ContextV2 context = new ContextV2();
      context.execute(new StrategyLogic1());
      context.execute(new StrategyLogic2());
    }
  }
```
`Context`와 `Strategy`를 '선 조립 후 실행'하는 방식이 아니라 `Context`를 실행할 때 마다 전략을 인수로 전달한다.  
클라이언트는 `Context`를 실행하는 시점에 원하는 `Strategy`를 전달할 수 있다. 따라서 이전 방식과 비교해서 원하는 전략을 더욱 유연하게 변경할 수 있다.  
테스트 코드를 보면 하나의 `Context`만 생성한다. 그리고 하나의 `Context`에 실행 시점에 여러 전략을 인수로 전달해서 유연하게 실행하는 것을 확인할 수 있다.

<br>

**전략 패턴 파라미터 실행 그림**
![img12](https://user-images.githubusercontent.com/93430103/161977940-b703b596-77d0-4f3a-b034-556ecde3f389.png)

1. 클라이언트는 Context 를 실행하면서 인수로 Strategy 를 전달한다. 
2. Context 는 execute() 로직을 실행한다.
3. Context 는 파라미터로 넘어온 strategy.call() 로직을 실행한다. 
4. Context 의 execute() 로직이 종료된다.

<br>

**ContextV2Test - 추가**
```java
  /**
   * 전략 패턴 익명 내부 클래스
   */
  @Test
  void strategyV2() {
      ContextV2 context = new ContextV2();
      context.execute(new Strategy() {
        @Override
        public void call() {
          log.info("비즈니스 로직1 실행"); 
        }
      });
      
      context.execute(new Strategy() {
        @Override
        public void call() {
          log.info("비즈니스 로직2 실행"); 
        }
      }); 
  }
```
여기도 물론 익명 내부 클래스를 사용할 수 있다. 코드 조각을 파라미터로 넘긴다고 생각하면 더 자연스럽다.

<br>

**ContextV2Test - 추가**
```java
  /**
   * 전략 패턴 익명 내부 클래스2,람다
   */
  @Test
  void strategyV3() {
    ContextV2 context = new ContextV2(); 
    context.execute(() -> log.info("비즈니스 로직1 실행")); 
    context.execute(() -> log.info("비즈니스 로직2 실행"));
  }
```
람다를 사용해서 코드를 더 단순하게 만들 수 있다.

<br>

**정리**  
* `ContextV1`은 필드에 `Strategy`를 저장하는 방식으로 전략 패턴을 구사했다.  
  * 선 조립, 후 실행 방법에 적합하다.
  * Context 를 실행하는 시점에는 이미 조립이 끝났기 때문에 전략을 신경쓰지 않고 단순히 실행만 하면 된다.
* ContextV2 는 파라미터에 Strategy 를 전달받는 방식으로 전략 패턴을 구사했다.
  * 실행할 때 마다 전략을 유연하게 변경할 수 있다.
  * 단점 역시 실행할 때 마다 전략을 계속 지정해주어야 한다는 점이다.

<br>

**템플릿**  
지금 우리가 해결하고 싶은 문제는 변하는 부분과 변하지 않는 부분을 분리하는 것이다.  
변하지 않는 부분을 템플릿이라고 하고, 그 템플릿 안에서 변하는 부분에 약간 다른 코드 조각을 넘겨서 실행하는 것이 목적이다.  
`ContextV1`, `ContextV2` 두 가지 방식 다 문제를 해결할 수 있지만, 어떤 방식이 조금 더 나아 보이는가?  
지금 우리가 원하는 것은 애플리케이션 의존 관계를 설정하는 것 처럼 선 조립, 후 실행이 아니다. 단순히 코드를 실행할 때 변하지 않는 템플릿이 있고, 그 템플릿 안에서 원하는 부분만 살짝 다른 코드를 실행하고 싶을 뿐이다.  
따라서 우리가 고민하는 문제는 실행 시점에 유연하게 실행 코드 조각을 전달하는 `ContextV2`가 더 적합하다.



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
