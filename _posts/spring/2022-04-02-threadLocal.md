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

[이미지](/images/image02.png)

thread-A는 userA를, thread-B는 userB를 조회해 오는걸 볼 수 있다.  
이 처럼 `쓰레드 로컬`을 사용하면 각 쓰레드마다 별도의 내부 저장소를 가지기 때문에,
`동시성 문제`가 해결되는걸 볼 수 있다.

> 쓰레드 로컬 사용시 주의점  
쓰레드 로컬을 모두 사용하고 나면 꼭 remove 메서드를 호출해서 쓰레드 로컬에 저장된 값을 제거해 주자!


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
