---
title:  "테스트 코드 태깅과 필터링" 
excerpt: "테스트 코드에서 필터링을 거는 방법에 대해 알아보자"

categories:
  - JUnit
tags:
  - [testcode, tag, filtering]

toc: true
toc_sticky: true

date: 2022-04-17
last_modified_at: 2022-04-17
---

인프런에 있는 백기선 님의 **더 자바, 애플리케이션을 테스트하는 다양한 방법** 강의를 듣고 정리한 내용 입니다. 😀    
[🌜 [더 자바, 애플리케이션을 테스트하는 다양한 방법]강의 들으러 가기!](https://www.inflearn.com/course/the-java-application-test/dashboard){:target="_blank"}
{: .notice--warning}

`@Tag` annotation을 이용해서 테스트 코드를 필터링 하는 방법에 대해 알아보자.

## 🔔 필요한 상황
아래 예제와 같이 테스트가 2개 있다고 가정해보자.  
fast_test는 실행시간이 얼마 걸리지 않는 테스트 이고, slow_test는 실행시간이 오래 걸리는 테스트이다.  
예를 들어 로컬에서는 fast_test만 실행하고 ci서버에서는 전체 테스트를 실행하도록 하고 싶다면 어떻게 해야 하는지 알아보자.

```java
  @Test
  @DisplayName("간단한 테스트")
  @Tag("fast")
  void fast_test() {
    System.out.println("fast_test 실행");
  }
  @Test
  @DisplayName("복잡한 테스트")
  @Tag("slow")
  void slow_test() {
    System.out.println("slow_test 실행");
  }
```

### 🔔 첫 번째 방법 (IntelliJ 설정)

1. Edit Configurations 클릭  
![img01](https://user-images.githubusercontent.com/93430103/163678807-676b422c-6ba1-4cb0-9d99-c15e66ccf6ea.png)

2. Class 부분에서 Tags 선택 및 실행할 태그 이름 입력  
![img02](https://user-images.githubusercontent.com/93430103/163712437-880164bc-aa0f-4968-85f8-ca9a38cc3133.png)

3. 아래와 같이 이제 fast태그가 붙은 테스트만 실행할 수 있다.  
![img03](https://user-images.githubusercontent.com/93430103/163712438-da682948-37bb-464d-a957-790730363f0e.png)

### 🔔 두 번째 방법 (Maven 설정)
이번엔 maven 설정을 이용해서 테스트 코드 필터링 작업을 해보자
```
<profiles>
    <profile>
        <id>default</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <groups>fast</groups>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
    <profile>
        <id>ci</id>
        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-surefire-plugin</artifactId>
<!--                        <configuration>-->
<!--                            <groups>fast | slow</groups>-->
<!--                        </configuration>-->
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```
여기서는 2개의 profile을 추가해 주었다.  
따로 옵션 없이 테스트를 실행하면 fast 태그가 붙은 테스트가 실행되고, profile 옵션으로 id에 ci를 준 뒤, 실행하면 모든 테스트가 실행되도록 설정하였다.  

configuration에서 groups에 필터링 할 태그를 여러개 설정해 줄 수도 있고, 생략하면 태그 구분없이 전체 테스트가 실행된다.  
사용 가능한 태그 표현식은 [여기](https://junit.org/junit5/docs/current/user-guide/#running-tests-tag-expressions)를 참고하자.

그럼 이제 터미널에서 maven test를 진행해 보자.

![img04](https://user-images.githubusercontent.com/93430103/163713718-6a810573-e896-4b7d-9f28-0d20f0414944.png)  
![img05](https://user-images.githubusercontent.com/93430103/163713720-df3783c3-10c4-46d0-baf8-c1bf6861f90d.png)

![img06](https://user-images.githubusercontent.com/93430103/163713721-197cd294-520f-4824-997f-6209370046d0.png)  
![img07](https://user-images.githubusercontent.com/93430103/163713723-f4a83b77-952a-4c16-8060-d6deb22e840b.png)

테스트 결과 의도한 대로 동작하는걸 확인할 수 있다.

<br>

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}
<br>
