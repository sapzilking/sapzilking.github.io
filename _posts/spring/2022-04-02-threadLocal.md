---
title:  "쓰레드 로컬" 
excerpt: "쓰레드 로컬을 이용하여 동시성 문제를 해결해 보자"

categories:
  - spring
tags:
  - [동시성 문제 해결, 쓰레드 로컬]

toc: true
toc_sticky: true

date: 2022-04-02
last_modified_at: 2022-04-02
---

인프런에 있는 김영한 님의 **스프링 핵심 원리 - 고급편** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [스프링 핵심원리 - 고급편]강의 들으러 가기!](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B3%A0%EA%B8%89%ED%8E%B8/dashboard){:target="_blank"}
{: .notice--warning}

지난번에 `동시성 문제`에 대해 알아보았다.

오늘은 `동시성 문제` 해결 방법 중 쓰레드 로컬을 이용한 방법에 대해 알아보자.

```java
@Slf4j
public class ThreadLocalService {

    private ThreadLocal<String> nameStore = new ThreadLocal<>();

    public String logic(String name) {
        log.info("저장 name{} => nameStore={}", name, nameStore.get());
        nameStore.set(name);
        sleep(1000);
        log.info("조회 nameStore={}", nameStore.get());
        return nameStore.get();
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
지난번 코드와 nameStore변수의 타입이 달라진것 말고는 동일한 코드이다.

## 🔔 테스트
위의 코드로 아래와 같은 테스트를 진행해 보자.  

```java
@Slf4j
public class ThreadLocalServiceTest {

    private ThreadLocalService service = new ThreadLocalService();

    @Test
    void field() {
        log.info("main start");
        Runnable userA = () -> {
            service.logic("userA");
        };
        Runnable userB = () -> {
            service.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start();
//        sleep(2000); // 동시성 문제 발생X
        sleep(100); // 동시성 문제 발생O
        threadB.start();

        sleep(3000); // 메인 쓰레드 종료 대기
        log.info("main exit");

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

위의 테스트의 실행 결과이다.

![img02](https://user-images.githubusercontent.com/93430103/161389488-1eb3ecba-6a91-40cb-8739-a232debd77e9.png)

thread-A는 userA를, thread-B는 userB를 조회해 오는걸 볼 수 있다.  
이 처럼 `쓰레드 로컬`을 사용하면 각 쓰레드마다 별도의 내부 저장소를 가지기 때문에,
`동시성 문제`가 해결되는걸 볼 수 있다.

## 쓰레드 로컬 사용 시 주의사항

쓰레드 로컬의 값을 사용 후 제거하지 않고 그냥 두면 WAS(톰캣)처럼 쓰레드 풀을 사용하는 경우에 심각한 문제가 발생할 수 있다.  

아래의 예시를 보자.  

사용자A 저장 요청 이미지
![img03](https://user-images.githubusercontent.com/93430103/161391470-56a4b17e-cb1f-4c91-b949-59ae887bdc71.png)


1. 사용자A가 저장 HTTP를 요청했다.
2. WAS는 쓰레드 풀에서 쓰레드를 하나 조회한다.
3. 쓰레드 `thread-A`가 할당되었다.
4. `thread-A`는 사용자A의 데이터를 쓰레드 로컬에 저장한다.
5. 쓰레드 로컬의 `thread-A`전용 보관소에 사용자A 데이터를 보관한다.

사용자A 저장 요청 종료 이미지
![img04](https://user-images.githubusercontent.com/93430103/161391471-3b194e7c-a498-48ed-a9a4-3991fd40e1b8.png)

1. 사용자A의 HTTP 응답이 끝난다.
2. WAS는 사용이 끝난 thread-A 를 쓰레드 풀에 반환한다. 쓰레드를 생성하는 비용은 비싸기 때문에 쓰레드를 제거하지 않고, 보통 쓰레드 풀을 통해서 쓰레드를 재사용한다.
3. thread-A 는 쓰레드풀에 아직 살아있다. 따라서 쓰레드 로컬의 thread-A 전용 보관소에 사용자A 데이터도 함께 살아있게 된다.


사용자B 조회 요청
이미지
![img05](https://user-images.githubusercontent.com/93430103/161391473-e855f4fd-0004-4ff5-920c-ea8099f867d5.png)

1. 사용자B가 조회를 위한 새로운 HTTP 요청을 한다.
2. WAS는 쓰레드 풀에서 쓰레드를 하나 조회한다.
3. 쓰레드 thread-A 가 할당되었다. (물론 다른 쓰레드가 할당될 수 도 있다.)
4. 이번에는 조회하는 요청이다. thread-A 는 쓰레드 로컬에서 데이터를 조회한다. 5. 쓰레드 로컬은 thread-A 전용 보관소에 있는 사용자A 값을 반환한다.
6. 결과적으로 사용자A 값이 반환된다.
7. 사용자B는 사용자A의 정보를 조회하게 된다.

결과적으로 사용자B는 사용자A의 데이터를 확인하게 되는 심각한 문제가 발생하게 된다.  
이런 문제를 예방하려면 사용자A의 요청이 끝날 때 쓰레드 로컬의 값을 ThreadLocal.remove() 를 통해서 꼭 제거해야 한다.


> 쓰레드 로컬을 모두 사용하고 나면 꼭 remove 메서드를 호출해서 쓰레드 로컬에 저장된 값을 제거해 주자!


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
