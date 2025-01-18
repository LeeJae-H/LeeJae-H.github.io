---
title: "REST API E2E 테스트)"
tags:
    - java
date: "2025-01-14"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> 이전 글들을 모두 읽고 오자 !!

> **Dependencies**
    - *Spring Boot 3.3.6*
    - *Junit 5.10.5*
    - *REST-Assured 5.3.1*

# REST API E2E 테스트 예시 - 스프링 부트
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

    public User(String email, String password) {
        this.email = email;
        this.password = password;
    }
}
```
```java
public interface UserRepository extends Repository<User, Long> {
    Optional<User> findByEmail(String email);
    User save(User user);
    Optional<User> findById(Long id);
}
```
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public User signUp(UserSignUpRequest userSignUpRequest) {
        if(userRepository.findByEmail(userSignUpRequest.getEmail()).isPresent()) {
            throw new IllegalArgumentException("이미 가입된 사용자입니다.");
        }

        User newUser = new User(userSignUpRequest.getEmail(), userSignUpRequest.getPassword());
        return userRepository.save(newUser);
    }

    public UserInfoResponse getUserInfo(Long userId) {
        User foundUser = userRepository.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 사용자입니다."));

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
    public User signUp(@RequestBody UserSignUpRequest userSignUpRequest) {
        return userService.signUp(userSignUpRequest);
    }

    // 내 정보 보기
    @GetMapping("/{id}")
    public UserInfoResponse getUserInfo(@PathVariable("id") Long id) {
        return userService.getUserInfo(id);
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

## 1. 스프링 부트의 내장 서버를 이용한 REST API E2E 테스트 
```java
@ActiveProfiles("test")
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@EnableAutoConfiguration(exclude = {SecurityAutoConfiguration.class})  // Spring Security 비활성화
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
    @DisplayName("회원 가입")
    void signUp() {
        UserSignUpRequest signUpRequest = new UserSignUpRequest("test@gmail.com", "test123");

        RestAssured.given()
                        .contentType(ContentType.JSON)
                        .body(signUpRequest)
                    .when()
                        .post("/api/users/signup")
                    .then()
                        .statusCode(200)
                        .body("email", Matchers.equalTo("test@gmail.com"))
                        .body("password", Matchers.equalTo("test123"));
    }

    @Test
    @DisplayName("내 정보 찾기")
    void getUserInfo() {
        // Given
        User existingUser = new User("test@gmail.com", "test123");
        Long userId = userRepository.save(existingUser).getId();

        // When & Then
        RestAssured.given()
                        .pathParam("id", userId)
                    .when()
                        .get("/api/users/{id}")
                    .then()
                        .statusCode(200)
                        .body("email", Matchers.equalTo("test@gmail.com"));
    }
}
```


---
- 참고 자료
테스트 주도 개발 시작하기(저자: 최범균)   
https://martinfowler.com/articles/practical-test-pyramid.html  