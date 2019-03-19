# 프로젝트 생성
- http://start.spring.io
- Gradle Project, Java, 2.1.3(또는 정식판중 최신)
- Group과 Artiface는 아무값이나 입력 가능
- 의존성 검색 후 추가(Web 는 기본 추가)
- Generate Project 버튼을 클릭
- 프로젝트 압축파일 다운로드및 개발툴에 설정

# HelloWorld

```
package com.naver.pay.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class CommunityApplication {
    public static void main(String[] args) {
        SpringApplication.run(CommunityApplication.class, args);
    }

    @GetMapping
    public String helloWorld() {
        return "Hello World";
    }
}
```

- run application 명령으로 실행
- http://localhost:8080/ 접속 후 결과 확인

# 인텔리제이 상용버전에서 Spring Initializer를 사용해서 프로젝트 생성

# 그레이들 설치및 빌드하기

- 그레이들 멀티 프로젝트 구성하기

# 환경 프로퍼티 파일 설정하기

- src/main/resources 아래 application.properties (또는 application.yml) 파일을 수정한다.

```
server.port: 80
```

## 프로파일에 따른 환경 구성 분리

```application.yml
server:
    port: 80

spring:
    profiles: dev
server:
    port: 8081

spring:
    profiles: real
server:
    port: 8082
```

```
java -jar ... -D spring.profiles.active=dev
```

## YAML 파일 매핑하기

기능 | @Value | @configurationProperties
유연한 바인딩 | X | O
메타데이터 지원 | X | O
SpEL평가 | O | X

유연한 바인딩 : 필드는 낙타표기법으로 선언, 프로퍼티의 키는 다양한 형식(낙타표기법, 케밥표기법, 언더바표기법 등) 으로 선언해서 바인딩
메타데이터 지원 : 프로퍼티의 키에 대한 정보를 메타데이터 파일(JSON)로 제공
SpEL평가


### @Value 살펴보기

```
property:
    test:
        name: property depth test
propertyTest: test
propertyTestList: a,b,c
```

# 자동 환경 설정 이해하기

- Web, H2, JDBC를 비롯한 100여개의 자동설정을 제공함
- @EnableAutoConfiguration (또는 @SpringBootApplication)
    - @EnableAutoConfiguration 는 @Configuration과 함께 사용해야함

## 자동 환경 설정 애노테이션

- @SpringBootApplication = @SpringBootConfiguration + @EnableAutoconfiguration + @ComponentScan

## @EnableAutoConfiguration 살펴보기

- META-INF/spring.factories : 자동 설정 대상 클래스 목록
- META-INF/spring-configuration-metadata.json : 자동 설정에 사용할 프로퍼티 정의 파일
- org/springframework/boot/autoconfigure : 미리 구현해놓은 자동설정 목록

https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html

## 자동 설정 애노테이션 살펴보기

조건 애노테이션 | 적용 조건
@ConditionalOnBean | 해당하는 빈 클래스나 이름이 미리 빈 팩토리에 포함되어 있을 경우
@ConditionalOnClass | 해당하는 클래스가 클래스 경로에 있을 경우