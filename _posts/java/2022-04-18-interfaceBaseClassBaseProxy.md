---
title:  "인터페이스 기반, 구체 클래스 기반 프록시 " 
excerpt: "다양한 상황에서 프록시를 적용해보자."

categories:
  - Design Patterns
tags:
  - [프록시, 인터페이스 기반 프록시 적용, 구체 클래스 기반 프록시 적용]

toc: true
toc_sticky: true

date: 2022-04-18
last_modified_at: 2022-04-18
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

프록시 적용 방법 중 아래 2가지 방법의 장∙단점에 대해 학습해보자.
1. 인터페이스 기반 프록시 적용
2. 구체 클래스 기반 프록시 적용

## 🔔 인터페이스 기반 프록시 적용
인터페이스 기반 클래스에 프록시를 적용하는 방법을 학습해보자.  
다음에 보이는 각 controller, service, repository는 인터페이스와 구현체가 각각 존재한다.  
기존 코드 수정 없이 새로운 기능(로그추적 기능)을 적용할 수 있을까?  먼저 프록시를 도입하기 전에 기본 코드를 작성해보자.  

### 🔔 인터페이스 기반 프록시 - 예제1
**OrderControllerV1**
```java
@RequestMapping
@ResponseBody
public interface OrderControllerV1 {

    @GetMapping("/v1/request")
    String request(@RequestParam("itemId") String itemId);

    @GetMapping("/v1/no-log")
    String noLog();
}
```
**OrderControllerV1Impl**
```java
public class OrderControllerV1Impl implements OrderControllerV1 {

    private final OrderServiceV1 orderService;

    public OrderControllerV1Impl(OrderServiceV1 orderService) {
        this.orderService = orderService;
    }

    @Override
    public String request(String itemId) {
        orderService.orderItem(itemId);
        return "ok";
    }

    @Override
    public String noLog() {
        return "ok";
    }
}
```

**OrderServiceV1**
```java
public interface OrderServiceV1 {
    void orderItem(String itemId);
}
```
**OrderServiceImpl**
```java
public class OrderServiceImpl implements OrderServiceV1 {

    private final OrderRepositoryV1 orderRepository;

    public OrderServiceImpl(OrderRepositoryV1 orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public void orderItem(String itemId) {
        orderRepository.save(itemId);
    }
}
```

**OrderRepositoryV1**
```java
public interface OrderRepositoryV1 {
    void save(String itemId);
}
```

**OrderRepositoryV1Impl**

```java
public class OrderRepositoryV1Impl implements OrderRepositoryV1 {

    @Override
    public void save(String itemId) {
        //저장 로직
        if (itemId.equals("ex")) {
            throw new IllegalStateException("예외 발생!");
        }
        sleep(1000);
    }

    private void sleep(int mills) {
        try {
            Thread.sleep(mills);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

V1 App의 기본 클래스 의존 관계와 런타임시 객체 인스턴스 의존 관계는 다음과 같다.

**V1 기본 클래스 의존 관계**
![img23](https://user-images.githubusercontent.com/93430103/163751409-b4847fa5-2898-45ae-954b-a07e56128b74.png)

**V1 런타임 객체 의존 관계**
![img24](https://user-images.githubusercontent.com/93430103/163751417-49ed1e3f-3b14-41cf-9b84-d0d6074b91d5.png)

**여기에 로그 추적용 프록시를 추가하면 다음과 같다.**

**V1 프록시 의존 관계 추가**
![img25](https://user-images.githubusercontent.com/93430103/163751426-fc926228-84c5-458e-846f-485bc162683f.png)
`Controller` , `Service` , `Repository` 각각 인터페이스에 맞는 프록시 구현체를 추가한다. (그림에서 리포지토리는 생략했다.)

**V1 프록시 런타임 객체 의존 관계**
![img26](https://user-images.githubusercontent.com/93430103/163751430-b4942fc6-1897-4b4c-affe-dc4fa57319e9.png)

그리고 애플리케이션 실행 시점에 프록시를 사용하도록 의존 관계를 설정해주어야 한다. 이 부분은 빈을 등록하는 설정 파일을 활용하면 된다. (그림에서 리포지토리는 생략했다.)

그럼 실제 프록시를 적용해보자.

### 🔔 인터페이스 기반 프록시 - 예제2

**OrderRepositoryInterfaceProxy**
```java
@RequiredArgsConstructor
public class OrderRepositoryInterfaceProxy implements OrderRepositoryV1 {
    private final OrderRepositoryV1 target;
    private final LogTrace logTrace;
    
    @Override
    public void save(String itemId) {
        TraceStatus status = null;
        try {
            status = logTrace.begin("OrderRepository.request()");

            //target 호출
            target.save(itemId);
            
            logTrace.end(status);
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

프록시를 만들기 위해 인터페이스를 구현하고 구현한 메서드에 `LogTrace` 를 사용하는 로직을 추가한다.

**OrderServiceInterfaceProxy**
```java
@RequiredArgsConstructor
public class OrderServiceInterfaceProxy implements OrderServiceV1 {
    private final OrderServiceV1 target;
    private final LogTrace logTrace;
    
    @Override
    public void orderItem(String itemId) {
        TraceStatus status = null;
        try {
            status = logTrace.begin("OrderService.orderItem()");
            
            //target 호출
            target.orderItem(itemId);
            
            logTrace.end(status);
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

**OrderControllerInterfaceProxy**

```java
@RequiredArgsConstructor
public class OrderControllerInterfaceProxy implements OrderControllerV1 {
    private final OrderControllerV1 target;
    private final LogTrace logTrace;
    
    @Override
    public String request(String itemId) {
        TraceStatus status = null;
        try {
            status = logTrace.begin("OrderController.request()");

            //target 호출
            String result = target.request(itemId);
            
            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
        
    @Override
    public String noLog() {
        return target.noLog();
    }
}
```

`noLog()` 메서드는 로그를 남기지 않아야 한다. 따라서 별도의 로직 없이 단순히 `target` 을 호출하면 된다.

**AppV1Config (프록시 적용 전 Configuration)** 
```java
@Configuration
public class AppV1Config {

    @Bean
    public OrderControllerV1 orderControllerV1() {
        return new OrderControllerV1Impl(orderServiceV1());
    }

    @Bean
    public OrderServiceV1 orderServiceV1() {
        return new OrderServiceImpl(orderRepositoryV1());
    }

    @Bean
    public OrderRepositoryV1 orderRepositoryV1() {
        return new OrderRepositoryV1Impl();
    }
}
```

**InterfaceProxyConfig (프록시 적용 후 Configuration)** 
```java
@Configuration
public class InterfaceProxyConfig {
    @Bean
    public OrderControllerV1 orderController(LogTrace logTrace) {
        OrderControllerV1Impl controllerImpl = new OrderControllerV1Impl(orderService(logTrace));
        return new OrderControllerInterfaceProxy(controllerImpl, logTrace);
    }
    
    @Bean
    public OrderServiceV1 orderService(LogTrace logTrace) {
        OrderServiceV1Impl serviceImpl = new OrderServiceV1Impl(orderRepository(logTrace));
        return new OrderServiceInterfaceProxy(serviceImpl, logTrace);
    }
    
    @Bean
    public OrderRepositoryV1 orderRepository(LogTrace logTrace) {
        OrderRepositoryV1Impl repositoryImpl = new OrderRepositoryV1Impl();
        return new OrderRepositoryInterfaceProxy(repositoryImpl, logTrace);
    }
}
```

`LogTrace` 가 아직 스프링 빈으로 등록되어 있지 않은데, 이 부분은 바로 다음에 등록할 것이다.

**V1 프록시 런타임 객체 의존 관계 설정**

* 이제 프록시의 런타임 객체 의존 관계를 설정하면 된다. 기존에는 스프링 빈이 `orderControlerV1Impl` , `orderServiceV1Impl` 같은 실제 객체를 반환했다. 하지만 이제는 프록시를 사용해야한다. 따라서 프록시를 생성하고 **프록시를 실제 스프링 빈 대신 등록한다. 실제 객체는 스프링 빈으로 등록하지 않는다.**
* 프록시는 내부에 실제 객체를 참조하고 있다. 예를 들어서 `OrderServiceInterfaceProxy` 는 내부에 실제 대상 객체인 `OrderServiceV1Impl` 을 가지고 있다.
* 정리하면 다음과 같은 의존 관계를 가지고 있다.
     * `proxy -> target`
     * `orderServiceInterfaceProxy -> orderServiceV1Impl`
* 스프링 빈으로 실제 객체 대신에 프록시 객체를 등록했기 때문에 앞으로 스프링 빈을 주입 받으면 **실제 객체 대신에 프록시 객체가 주입**된다.
* 실제 객체가 스프링 빈으로 등록되지 않는다고 해서 사라지는 것은 아니다. 프록시 객체가 실제 객체를 참조하기 때문에 프록시를 통해서 실제 객체를 호출할 수 있다. 쉽게 이야기해서 프록시 객체 안에 실제 객체가 있는 것이다.

![img27](https://user-images.githubusercontent.com/93430103/163752771-4a8a7ea5-5b5e-4ca5-81ab-dbcf519137b7.png)

`AppV1Config` 를 통해 프록시를 적용하기 전

* 실제 객체가 스프링 빈으로 등록된다. 빈 객체의 마지막에 `@x0..` 라고 해둔 것은 인스턴스라는 뜻이다.

![img28](https://user-images.githubusercontent.com/93430103/163753110-d91c8048-a8cc-48e0-84eb-562e85bbd19d.png)

`InterfaceProxyConfig` 를 통해 프록시를 적용한 후

* 스프링 컨테이너에 프록시 객체가 등록된다. 스프링 컨테이너는 이제 실제 객체가 아니라 프록시 객체를 스프링 빈으로 관리한다.

* 이제 실제 객체는 스프링 컨테이너와는 상관이 없다. 실제 객체는 프록시 객체를 통해서 참조될 뿐이다. 
* 프록시 객체는 스프링 컨테이너가 관리하고 자바 힙 메모리에도 올라간다. 반면에 실제 객체는 자바 힙 메모리에는 올라가지만 스프링 컨테이너가 관리하지는 않는다.

![img29](https://user-images.githubusercontent.com/93430103/163753212-7127f563-dfd2-46f6-a77c-b848d6d7a543.png)

최종적으로 이런 런타임 객체 의존관계가 발생한다. (리포지토리는 생략했다.)


**ProxyApplication**
```java
@Import(InterfaceProxyConfig.class)
public class ProxyApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProxyApplication.class, args);
    }
    
    @Bean
    public LogTrace logTrace() {
          return new ThreadLocalLogTrace();
    }
}
```

* @Bean : 먼저 LogTrace 스프링 빈 추가를 먼저 해주어야 한다.
* @Import(InterfaceProxyConfig.class) : 프록시를 적용한 설정 파일을 사용하자.

## 🔔 구체 클래스 기반 프록시 적용
이번에는 구체 클래스에 프록시를 적용하는 방법을 학습해보자.  
다음에 보이는 `ConcreteLogic` 은 인터페이스가 없고 구체 클래스만 있다.  
이렇게 인터페이스가 없어도 프록시를 적용할 수 있을까?  먼저 프록시를 도입하기 전에 기본 코드를 작성해보자.

### 🔔 구체 클래스 기반 프록시 - 예제1
**ConcreteLogic**
 ```java
 @Slf4j
public class ConcreteLogic {
    public String operation() {
        log.info("ConcreteLogic 실행");
        return "data";
    }
}
 ```
 `ConcreteLogic` 은 인터페이스가 없고, 구체 클래스만 있다. 여기에 프록시를 도입해야 한다.

![img19](https://user-images.githubusercontent.com/93430103/163748816-37b19cfb-762e-4584-9450-e71f6ff6ffbc.png)

![img20](https://user-images.githubusercontent.com/93430103/163748884-995f710c-824a-475f-a3a6-83c9f275ce82.png)

**ConcreteClient**

```java
public class ConcreteClient {
    private ConcreteLogic concreteLogic;
      
    public ConcreteClient(ConcreteLogic concreteLogic) {
        this.concreteLogic = concreteLogic;
    }
    
    public void execute() {
        concreteLogic.operation();
    }
}
```

**ConcreteProxyTest**

```java
public class ConcreteProxyTest {
    @Test
    void noProxy() {
        ConcreteLogic concreteLogic = new ConcreteLogic();
        ConcreteClient client = new ConcreteClient(concreteLogic);
        client.execute();
    }
}
```
코드가 단순해서 이해하는데 어려움은 없을 것이다.


### 🔔 구체 클래스 기반 프록시 - 예제2
**클래스 기반 프록시 도입**  
지금까지 인터페이스를 기반으로 프록시를 도입했다. 그런데 자바의 다형성은 인터페이스를 구현하든, 아니면 클래스를 상속하든 상위 타입만 맞으면 다형성이 적용된다. 쉽게 이야기해서 인터페이스가 없어도 프록시를 만들수 있다는 뜻이다. 그래서 이번에는 인터페이스가 아니라 클래스를 기반으로 상속을 받아서 프록시를 만들어보겠다.

![img21](https://user-images.githubusercontent.com/93430103/163749419-65016068-45a5-4fa8-92ec-241bf779c7ca.png)

![img22](https://user-images.githubusercontent.com/93430103/163749425-2abb2a65-d4de-4754-a156-745a65aa01c9.png)

**TimeProxy**

```java
@Slf4j
public class TimeProxy extends ConcreteLogic {
    private ConcreteLogic realLogic;
      
    public TimeProxy(ConcreteLogic realLogic) {
        this.realLogic = realLogic;
    }
      
    @Override
    public String operation() {
        log.info("TimeDecorator 실행");
        long startTime = System.currentTimeMillis();
        
        String result = realLogic.operation();
        
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeDecorator 종료 resultTime={}", resultTime);
        
        return result;
    }
}
```
`TimeProxy` 프록시는 시간을 측정하는 부가 기능을 제공한다. 그리고 인터페이스가 아니라 클래스인 `ConcreteLogic` 를 상속 받아서 만든다.

***ConcreteProxyTest - addProxy() 추가***

```java
public class ConcreteProxyTest {
    @Test
    void noProxy() {
        ConcreteLogic concreteLogic = new ConcreteLogic();
        ConcreteClient client = new ConcreteClient(concreteLogic);
        client.execute();
    }

    @Test
    void addProxy() {
        ConcreteLogic concreteLogic = new ConcreteLogic();
        TimeProxy timeProxy = new TimeProxy(concreteLogic);
        ConcreteClient client = new ConcreteClient(timeProxy);
        client.execute();
    }
}
```

여기서 핵심은 `ConcreteClient` 의 생성자에 `concreteLogic` 이 아니라 `timeProxy` 를 주입하는 부분이다.
`ConcreteClient` 는 `ConcreteLogic` 을 의존하는데, 다형성에 의해 `ConcreteLogic` 에 `concreteLogic` 도 들어갈 수 있고, `timeProxy` 도 들어갈 수 있다.

**ConcreteLogic에 할당할 수 있는 객체**
* ConcreteLogic = concreteLogic (본인과 같은 타입을 할당)
* ConcreteLogic = timeProxy (자식 타입을 할당)

**실행 결과**
```
TimeDecorator 실행 
ConcreteLogic 실행 
TimeDecorator 종료 resultTime=1
```
실행 결과를 보면 인터페이스가 없어도 클래스 기반의 프록시가 잘 적용된 것을 확인할 수 있다.



**인터페이스 기반 프록시 vs 클래스 기반 프록시**
* 인터페이스가 없어도 클래스 기반으로 프록시를 생성할 수 있다.
* 클래스 기반 프록시는 해당 클래스에만 적용할 수 있다. 인터페이스 기반 프록시는 인터페이스만 같으면 모든 곳에 적용할 수 있다.
* 클래스 기반 프록시는 상속을 사용하기 때문에 몇가지 제약이 있다.  
  * 부모 클래스의 생성자를 호출해야 한다.  
  * 클래스에 final 키워드가 붙으면 상속이 불가능하다.
  * 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩 할 수 없다.

이렇게 보면 인터페이스 기반의 프록시가 더 좋아보인다. 맞다. 인터페이스 기반의 프록시는 상속이라는 제약에서 자유롭다.  
프로그래밍 관점에서도 인터페이스를 사용하는 것이 역할과 구현을 명확하게 나누기 때문에 더 좋다.  
인터페이스 기반 프록시의 단점은 인터페이스가 필요하다는 그 자체이다. 인터페이스가 없으면 인터페이스 기반 프록시를 만들 수 없다.

이론적으로는 모든 객체에 인터페이스를 도입해서 역할과 구현을 나누는 것이 좋다. 이렇게 하면 역할과 구현을 나누어서 구현체를 매우 편리하게 변경할 수 있다. 하지만 실제로는 구현을 거의 변경할 일이 없는 클래스도 많다.  
인터페이스를 도입하는 것은 구현을 변경할 가능성이 있을 때 효과적인데, 구현을 변경할 가능성이 거의 없는 코드에 무작정 인터페이스를 사용하는 것은 번거롭고 그렇게 실용적이지 않다. 이런곳에는 실용적인 관점에서 인터페이스를 사용하지 않고 구체 클래스를 바로 사용하는 것이 좋다 생각한다. (물론 인터페이스를 도입하는 다양한 이유가 있다. 여기서 핵심은 인터페이스가 항상 필요하지는 않다는 것이다.)

**너무 많은 프록시 클래스**
지금까지 프록시를 사용해서 기존 코드를 변경하지 않고, 로그 추적기라는 부가 기능을 적용할 수 있었다.  
그런데 문제는 프록시 클래스를 너무 많이 만들어야 한다는 점이다. 잘 보면 프록시 클래스가 하는 일은 `LogTrace` 를 사용하는 것인데, 그 로직이 모두 똑같다. 대상 클래스만 다를 뿐이다. 만약 적용해야 하는 대상 클래스가 100개라면 프록시 클래스도 100개를 만들어야한다.  
프록시 클래스를 하나만 만들어서 모든 곳에 적용하는 방법은 없을까?  

다음에는 동적 프록시 기술을 이용해서 이 문제를 해결해 보자.

<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
