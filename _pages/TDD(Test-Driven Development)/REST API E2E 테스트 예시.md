---
title: "REST API E2E 테스트 (feat. REST-assured)"
tags:
    - java
date: "2025-01-14"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> 이전 글들을 모두 읽고 오자 !!

> **Dependencies**
    - *Spring Boot 3.4.1*
    - *Junit 5.10.5*
    - *REST-assured 5.3.1*

# REST API E2E 테스트 예시 - 스프링 부트
> - `REST API E2E 테스트`를 작성하는 것은 **E2E 테스트를 자동화**하는 것이다. 우리가 일반적으로 Postman을 사용하여 API를 테스트하는 것과 거의 같은 흐름이며, 예시에서는 아래 두 가지 도구를 사용한다.
    1. @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.**RANDOM_PORT**)
    2. **REST-assured**

## * @SpringBootTest의 MOCK vs RANDOM_PORT
- **MOCK(default 값)** 
    - 서블릿 컨테이너(Tomcat)를 실행하지 않는다. 즉, **서버 실행 없이 Spring Context만 로드**된다.
    - 서버가 실행되지 않기 때문에 **가상으로 HTTP 요청과 응답을 테스트**한다. 이를 위해, MockMvc를 사용한다.
- **RANDOM_PORT**
    - 서블릿 컨테이너(Tomcat)를 실행한다. 즉, **임의의 포트에서 실제로 서버가 실행**된다.
    - 서버가 실행되기 때문에 **실제 HTTP 요청을 보내고 응답을 받으며 테스트**한다. 이를 위해, REST-assured나 TestRestTemplate 등을 사용한다.

## * MockMvc vs Rest-assured
- **MockMvc**
    - **DispatcherServlet을 통해 컨트롤러를 호출하고 응답을 확인**한다.
    - 일반적으로, @WebMvcTest를 사용하는 `Controller 단위 테스트`나 @SpringBootTest(MOCK)을 사용하는 `Controller 통합 테스트`에서 사용된다.
- **Rest-assured**
    - **실제 HTTP 요청을 서버에 보내고 응답을 확인**한다.
    - 일반적으로, @SpringBootTest(RANDOM_PORT)를 사용하는 `REST API E2E 테스트`에서 사용된다.

## 0. 테스트에 사용될 객체들
```java
@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "email")
    private String email;

    @Column(name = "password")
    private String password;

    @Builder
    public User(String email, String password) {
        this.email = email;
        this.password = password;
    }
}
```
```java
public interface UserRepository extends Repository<User, Long> {
    User save(User user);
    Optional<User> findByEmail(String email);
    Optional<User> findById(Long id);
}
```
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    @Transactional
    public Long signUp(UserSignUpRequest userSignUpRequest) {
        if(userRepository.findByEmail(userSignUpRequest.getEmail()).isPresent()) {
            throw new ExpectedException(ErrorCode.ALREADY_EXISTED_USER);
        }

        User user = User.builder()
                            .email(userSignUpRequest.getEmail())
                            .password(userSignUpRequest.getPassword())
                            .build();
        return userRepository.save(user).getId();
    }

    public UserInfoResponse getUserInfo(Long userId) {
        User foundUser = userRepository.findById(userId)
                .orElseThrow(() -> new ExpectedException(ErrorCode.USER_NOT_FOUND));

        return new UserInfoResponse(foundUser.getEmail());
    }
}
```
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    // 회원 가입
    @PostMapping("/signup")
    public ResponseEntity<Long> signUp(@RequestBody UserSignUpRequest userSignUpRequest) {
        Long userId = userService.signUp(userSignUpRequest);
        return new ResponseEntity<>(userId, HttpStatus.CREATED);
    }

    // 내 정보 보기
    @GetMapping("/{id}")
    public ResponseEntity<UserInfoResponse> getUserInfo(@PathVariable("id") Long id) {
        UserInfoResponse userInfo = userService.getUserInfo(id);
        return ResponseEntity.ok(userInfo);
    }
}
```
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserSignUpRequest {
    private String email;
    private String password;

    public UserSignUpRequest(String email, String password) {
        this.email = email;
        this.password = password;
    }
}
```
```java
@Getter
public class UserInfoResponse {
    private String email;

    public UserInfoResponse(String email) {
        this.email = email;
    }
}
```
```java
@Getter
@RequiredArgsConstructor
public class ExpectedException extends RuntimeException {
    private final ErrorCode errorCode;
}
```
```java
@Getter
@RequiredArgsConstructor
public enum ErrorCode {
    ALREADY_EXISTED_USER("ALREADY_EXISTED_USER", "이미 존재하는 유저입니다."),
    USER_NOT_FOUND("USER_NOT_FOUND", "존재하지 않는 유저입니다.");

    private final String errorCode;
    private final String errorMessage;
}
```

## 1. 스프링 부트의 내장 서버를 이용한 REST API E2E 테스트 (feat. REST-assured)
```java
@ActiveProfiles("test")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserApiE2ETest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private UserRepository userRepository;

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        jdbcTemplate.update("truncate table users");
        RestAssured.port = this.port;
    }

    @Test
    @DisplayName("회원 가입 성공")
    void signUp_Success() {
        UserSignUpRequest signUpRequest 
                    = new UserSignUpRequest("joyuri@gmail.com", "yuri123");

        RestAssured.given()
                        .contentType(ContentType.JSON)
                        .body(signUpRequest)
                    .when()
                        .post("/api/users/signup")
                    .then()
                        .statusCode(201)
                        .body("", Matchers.greaterThanOrEqualTo(1));
    }

    @Test
    @DisplayName("내 정보 찾기 성공")
    void getUserInfo_Success() {
        // Given
        User existingUser = User.builder().email("joyuri@gmail.com").password("yuri123").build();
        Long userId = userRepository.save(existingUser).getId();

        // When & Then
        RestAssured.given()
                        .pathParam("id", userId)
                    .when()
                        .get("/api/users/{id}")
                    .then()
                        .statusCode(200)
                        .body("email", Matchers.equalTo("joyuri@gmail.com"));
    }
}
```


---
- 참고 자료
테스트 주도 개발 시작하기(저자: 최범균)   
https://martinfowler.com/articles/practical-test-pyramid.html  