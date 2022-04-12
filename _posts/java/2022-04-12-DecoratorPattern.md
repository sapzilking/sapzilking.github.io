---
title:  "데코레이터 패턴" 
excerpt: "데코레이터 패턴에 대해 알아보자"

categories:
  - Design Patterns
tags:
  - [데코레이터 패턴]

toc: true
toc_sticky: true

date: 2022-04-12
last_modified_at: 2022-04-12
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

지난 번에 알아본 [프록시패턴](https://sapzilking.github.io/design%20patterns/proxyPattern/)에 이어 오늘은 `데코레이터 패턴`에 대해 알아보자.

데코레이터 패턴을 이해하기 위한 예제 코드를 작성해보자. 먼저 데코레이터 패턴을 도입하기 전 코드를 아주 단순하게 만들어보자.

## 🔔 데코레이터 패턴 - 예제 코드1
 
![img12](https://user-images.githubusercontent.com/93430103/162970258-69999308-1cca-4a09-9bb5-3b0f866667fe.png)

![img13](https://user-images.githubusercontent.com/93430103/162970284-9e68218f-6c81-48b4-bd69-983deedeb816.png)


**Component 인터페이스**
```java
  public interface Component {
      String operation();
  }
```
`Component` 인터페이스는 단순히 `String operation()` 메서드를 가진다.

**RealComponent**
```java
  @Slf4j
  public class RealComponent implements Component {
      @Override
      public String operation() {
          log.info("RealComponent 실행");
          return "data";
      }
  }
```
* `RealComponent` 는 `Component` 인터페이스를 구현한다. 
* `operation()` : 단순히 로그를 남기고 `"data"` 문자를 반환한다.


**DecoratorPatternClient**
```java
  @Slf4j
  public class DecoratorPatternClient {
      private Component component;
      
      public DecoratorPatternClient(Component component) {
          this.component = component;
      }
      
      public void execute() {
          String result = component.operation();
          log.info("result={}", result);
      }
  }
```
* 클라이언트 코드는 단순히 `Component` 인터페이스를 의존한다.
* `execute()` 를 실행하면 `component.operation()` 을 호출하고, 그 결과를 출력한다.

**DecoratorPatternTest**
```java
  @Slf4j
  public class DecoratorPatternTest {
      @Test
      void noDecorator() {
          Component realComponent = new RealComponent();
          DecoratorPatternClient client = new DecoratorPatternClient(realComponent);
          
          client.execute();
      }
  }
```
테스트 코드는 `client -> realComponent` 의 의존관계를 설정하고, `client.execute()` 를 호출한다.

**실행 결과**
```
RealComponent - RealComponent 실행 
DecoratorPatternClient - result=data
```
여기까지는 앞서 [프록시 패턴](https://sapzilking.github.io/design%20patterns/proxyPattern/) 에서 설명한 내용과 유사하고 단순해서 이해하는데 어려움은 없을 것이다.

## 🔔 데코레이터 패턴 - 예제 코드2
**부가 기능 추가**  
앞서 설명한 것 처럼 프록시를 통해서 할 수 있는 기능은 크게 접근 제어와 부가 기능 추가라는 2가지로 구분한다.  
앞서 프록시 패턴에서 캐시를 통한 접근 제어를 알아보았다. 이번에는 프록시를 활용해서 부가 기능을 추가해보자. 이렇게 프록시로 부가 기능을 추가하는 것을 데코레이터 패턴이라 한다.

데코레이터 패턴: 원래 서버가 제공하는 기능에 더해서 부가 기능을 수행한다. 
* 예) 요청 값이나, 응답 값을 중간에 변형한다.
* 예) 실행 시간을 측정해서 추가 로그를 남긴다.

<br>

**응답 값을 꾸며주는 데코레이터**  
응답 값을 꾸며주는 데코레이터 프록시를 만들어보자.

![img14](https://user-images.githubusercontent.com/93430103/162971825-f9db09fe-f799-481a-805f-c5f76adf4c76.png)

![img15](https://user-images.githubusercontent.com/93430103/162971843-7f6be41c-fa60-4ccb-bd06-394fc28525fa.png)


**MessageDecorator**
```java
  @Slf4j
  public class MessageDecorator implements Component {
      private Component component;
      
      public MessageDecorator(Component component) {
          this.component = component;
      }
      
      @Override
      public String operation() {
          log.info("MessageDecorator 실행");
          String result = component.operation();
          String decoResult = "*****" + result + "*****"; 
          log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);
          
          return decoResult;
      }
  }
```
`MessageDecorator` 는 `Component` 인터페이스를 구현한다.  
프록시가 호출해야 하는 대상을 `component` 에 저장한다.  
`operation()` 을 호출하면 프록시와 연결된 대상을 호출(`component.operation()`) 하고, 그 응답 값에
`*****` 을 더해서 꾸며준 다음 반환한다.  
예를 들어서 응답 값이 `data` 라면 다음과 같다. 
* 꾸미기 전: `data`
* 꾸민 후 : `*****data*****`

**DecoratorPatternTest - 추가**
```java
  @Test
  void decorator1() {
      Component realComponent = new RealComponent();
      Component messageDecorator = new MessageDecorator(realComponent);
      DecoratorPatternClient client = new DecoratorPatternClient(messageDecorator);
      
      client.execute();
  }
```
`client -> messageDecorator -> realComponent` 의 객체 의존 관계를 만들고 `client.execute()` 를 호출한다.

**실행 결과**
```
MessageDecorator - MessageDecorator 실행
RealComponent - RealComponent 실행
MessageDecorator - MessageDecorator 꾸미기 적용 전=data, 적용 후=*****data*****
DecoratorPatternClient - result=*****data*****
```
실행 결과를 보면 `MessageDecorator` 가 `RealComponent` 를 호출하고 반환한 응답 메시지를 꾸며서 반환한 것을 확인할 수 있다.

## 🔔 데코레이터 패턴 - 예제 코드3
**실행 시간을 측정하는 데코레이터**  
이번에는 기존 데코레이터에 더해서 실행 시간을 측정하는 기능까지 추가해보자.

![img16](https://user-images.githubusercontent.com/93430103/162973295-45467d12-7a12-4095-bc09-dbd42bc94942.png)

![img17](https://user-images.githubusercontent.com/93430103/162973361-8f851045-e67d-4eb8-920a-b9e0cc203487.png)

**TimeDecorator**
```java
  @Slf4j
  public class TimeDecorator implements Component {
      private Component component;
      
      public TimeDecorator(Component component) {
          this.component = component;
      }
      
      @Override
      public String operation() {
          log.info("TimeDecorator 실행");
          long startTime = System.currentTimeMillis();
          String result = component.operation();
          long endTime = System.currentTimeMillis();
          long resultTime = endTime - startTime; 
          log.info("TimeDecorator 종료 resultTime={}ms", resultTime); 
          
          return result;
      }
  }
```
`TimeDecorator` 는 실행 시간을 측정하는 부가 기능을 제공한다. 대상을 호출하기 전에 시간을 가지고 있다가, 대상의 호출이 끝나면 호출 시간을 로그로 남겨준다.

**DecoratorPatternTest - 추가**
```java
  @Test
  void decorator2() {
      Component realComponent = new RealComponent();
      Component messageDecorator = new MessageDecorator(realComponent);
      Component timeDecorator = new TimeDecorator(messageDecorator);
      DecoratorPatternClient client = new DecoratorPatternClient(timeDecorator);
      
      client.execute();
  }
```
`client -> timeDecorator -> messageDecorator -> realComponent` 의 객체 의존관계를 설정하고, 실행한다.

**실행 결과**
```
TimeDecorator 실행
MessageDecorator 실행
RealComponent 실행
MessageDecorator 꾸미기 적용 전=data, 적용 후=*****data***** 
TimeDecorator 종료 resultTime=7ms
result=*****data*****
```
실행 결과를 보면 `TimeDecorator` 가 `MessageDecorator` 를 실행하고 실행 시간을 측정해서 출력한 것을 확인할 수 있다.

## 🔔 프록시 패턴과 데코레이터 패턴 정리
![img18](https://user-images.githubusercontent.com/93430103/162974214-180105e8-322f-4fad-840b-7b75214fe8f4.png)
여기서 생각해보면 `Decorator` 기능에 일부 중복이 있다. 꾸며주는 역할을 하는 `Decorator` 들은 스스로 존재할 수 없다. 항상 꾸며줄 대상이 있어야 한다. 따라서 내부에 호출 대상인 `component` 를 가지고 있어야 한다. 그리고 `component` 를 항상 호출해야 한다. 이 부분이 중복이다. 이런 중복을 제거하기 위해 `component` 를 속성으로 가지고 있는 `Decorator` 라는 추상 클래스를 만드는 방법도 고민할 수 있다. 이렇게 하면 추가로 클래스 다이어그램에서 어떤 것이 실제 컴포넌트 인지, 데코레이터인지 명확하게 구분할 수 있다. 여기까지 고민한 것이 바로 GOF에서 설명하는 데코레이터 패턴의 기본 예제이다.

### 🔔 프록시 패턴 vs 데코레이터 패턴
여기까지 진행하면 몇가지 의문이 들 것이다.
* Decorator 라는 추상 클래스를 만들어야 데코레이터 패턴일까? 
* 프록시 패턴과 데코레이터 패턴은 그 모양이 거의 비슷한 것 같은데?

**의도(intent)**  
사실 프록시 패턴과 데코레이터 패턴은 그 모양이 거의 같고, 상황에 따라 정말 똑같을 때도 있다. 그러면 둘을 어떻게 구분하는 것일까?  
디자인 패턴에서 중요한 것은 해당 패턴의 겉모양이 아니라 그 패턴을 만든 의도가 더 중요하다. 따라서 의도에 따라 패턴을 구분한다.

* 프록시 패턴의 의도: 다른 개체에 대한 접근을 제어하기 위해 대리자를 제공
* 데코레이터 패턴의 의도: 객체에 추가 책임(기능)을 동적으로 추가하고, 기능 확장을 위한 유연한 대안 제공

**정리**  
프록시를 사용하고 해당 프록시가 접근 제어가 목적이라면 프록시 패턴이고, 새로운 기능을 추가하는 것이 목적이라면 데코레이터 패턴이 된다.

<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
