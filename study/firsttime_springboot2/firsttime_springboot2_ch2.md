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