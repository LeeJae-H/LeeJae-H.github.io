---
title: "REST API E2E 테스트 예시 (feat. 스프링 부트)"
tags:
    - java
date: "2025-01-14"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> 이전 글들을 모두 읽고 오자 !!

> **Dependencies**
    - *Spring Boot 3.3.6*
    - *Junit 5.10.5*
    - *REST Assured*

# REST API E2E 테스트 예시 - 스프링 부트
## 0. 테스트에 사용될 객체들
```java
@RestController
public class UserController {
    private final UserRepository userRepository;

    @GetMapping("/users")
    public String hello(@PathVariable final String lastName) {
        Optional<Person> foundPerson = personRepository.findByLastName(lastName);

        return foundPerson
             .map(person -> String.format("Hello %s %s!",
                     person.getFirstName(),
                     person.getLastName()))
             .orElse(String.format("Who is this '%s' you're talking about?",
                     lastName));
    }
}
```

## 1. 스프링 부트의 내장 서버를 이용한 REST API E2E 테스트 
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserApiE2ETest {

    @Autowired
    private UserRepository userRepository;

    @LocalServerPort
    private int port;

    @AfterEach
    public void tearDown() throws Exception {
        personRepository.deleteAll();
    }

    @Test
    public void shouldReturnGreeting() throws Exception {
        Person peter = new Person("Peter", "Pan");
        personRepository.save(peter);

        when()
                .get(String.format("http://localhost:%s/hello/Pan", port))
        .then()
                .statusCode(is(200))
                .body(containsString("Hello Peter Pan!"));
    }
}
```


---
- 참고 자료
테스트 주도 개발 시작하기(저자: 최범균)   
https://martinfowler.com/articles/practical-test-pyramid.html  