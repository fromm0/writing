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

- 프로파일에 따른 환경 구성 분리

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

- YAML 파일 매핑하기






