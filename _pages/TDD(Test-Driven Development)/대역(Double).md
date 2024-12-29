---
title: "테스트 대역(Test Double)"
tags:
    - java
date: "2024-12-23"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> **Martin Fowler** : *테스트 대역은 테스트를 목적으로 프로덕션 객체를 대체하는 모든 경우를 포괄하는 일반적인 용어이다.*
*(Test Double is a generic term for any case where you replace a production object for testing purposes.)* 

> **Gerard Meszaros** : *테스트를 작성할 때, DOC를 사용할 수 없거나 사용하지 않기로 선택한 경우, 이를 테스트 대역으로 대체할 수 있다. 테스트 대역은 실제 DOC와 동일하게 동작할 필요는 없으며, 단지 실제 DOC와 동일한 API를 제공하여 SUT가 이를 실제 DOC로 인식하게 만들면 된다.*
*(When we are writing a test in which we cannot (or chose not to) use a real depended-on component (DOC), we can replace it with a Test Double. The Test Double doesn't have to behave exactly like the real DOC; it merely has to provide the same API as the real one so that the SUT thinks it is the real one!)*   
>
**SUT (System Under Test)**
    - 테스트 대상 시스템을 의미한다.
    - 단위 테스트에서의 SUT는 테스트 대상이 되는 클래스, 객체, 메서드를 의미한다.
**DOC (Depended-On Component)**
    - SUT가 의존하는 개별 클래스 또는 대규모 컴포넌트를 의미한다.

# 테스트 대역 종류

|종류|설명|부가설명|관련검증|
|---|---|---|
|스텁(Stub)|테스트만을 위한 단순한 구현을 가지고 있는 객체||상태 검증(State Verification)|
|가짜(Fake)|프로덕션에는 적합하지 않지만 실제 동작하는 구현을 가지고 있는 객체||상태 검증|
|스파이(Spy)|호출된 내역을 가지고 있는 객체, Stub이기도 하다.|호출된 내역이란 테스트 중 Spy 객체의 **메서드가 몇 번 호출되었는지, 어떤 인자로 호출되었는지**에 대한 정보이다.|상태 검증, 행위 검증|
|**모의(Mock)**|실제 객체를 흉내 내는 가짜 객체, **Stub과 Spy를 대신할 수 있다.**|**mock 객체의 주요 기능**은 기대한 대로 상호작용하는지 확인(= 행위 검증)하는 것이다.|행위 검증(Behavior Verification)|

# 테스트 대역 예시 - TDD ([참고 : TDD 흐름과 예시](https://leejae-h.github.io/TDD(Test-Driven%20Development)/TDD%20%ED%9D%90%EB%A6%84%EA%B3%BC%20%EC%98%88%EC%8B%9C.html))
<img src="https://github.com/user-attachments/assets/ac4b9f53-ea43-4740-8714-3710a3561605" style="width:590px;" />
- **UserRegister(테스트 대상)** : 회원 가입에 대한 핵심 로직을 수행
- WeakPasswordChecker : 암호가 약한지 검사
- UserRepository : 회원 정보 저장 및 조회 기능
- EmailNotifier : 이메일 발송 기능

## 첫 번째 테스트 : 암호가 약한 경우 회원 가입에 실패 - Stub 사용
1. 테스트 코드 작성
    ```java
    public class UserRegisterTest {
        private UserRegister userRegister;
        private StubWeakPasswordChecker stubPasswordChecker = 
            new StubWeakPasswordChecker();
        
        @BeforeEach
        void setUp() {
            userRegister = new UserRegister(stubPasswordChecker);
        }

        @DisplayName("약한 암호면 가입 실패")
        @Test
        void weakPassword() {
            // given
            stubPasswordChecker.setWeak(true);

            // when & then
            assertThatThrownBy(() -> userRegister.register("id", "pw", "email"))
                .isInstanceOf(WeakPasswordException.class);
        }
    }
    ```

2. 구현
    ```java
    public class UserRegister {
        private WeakPasswordChecker passwordChecker;

        public UserRegister(WeakPasswordChecker passwordChecker) {
            this.passwordChecker = passwordChecker;
        }

        public void register(String id, String pw, String email) {
            if(passwordChecker.checkPasswordWeak(pw)) {
                throw new WeakPasswordException();
            }
        }
    }
    ```
    ```java
    public interface WeakPasswordChecker {
        boolean checkPasswordWeak(String pw);
    }
    ```
    ```java
    public class StubWeakPasswordChecker implements WeakPasswordChecker {
        private boolean weak;

        public void setWeak(boolean weak) {
            this.weak = weak;
        }

        @Override
        public boolean checkPasswordWeak(String pw) {
            return weak;
        }
    }
    ```
    ```java
    public class WeakPasswordException extends RuntimeException {
    }
    ```

## 두 번째 테스트 : 동일 ID를 가진 회원이 존재할 경우 회원 가입에 실패 - Fake 사용
1. 테스트 코드 작성
    ```java
    public class UserRegisterTest {
        private UserRegister userRegister;
        private StubWeakPasswordChecker stubPasswordChecker = 
            new StubWeakPasswordChecker();
        private MemoryUserRepository fakeRepository = 
            new MemoryUserRepository(); // 추가

        @BeforeEach
        void setUp() {
            userRegister = new UserRegister(stubPasswordChecker, 
                fakeRepository); // 추가
        }

        @DisplayName("이미 같은 ID가 존재하면 가입 실패")
        @Test
        void dupIdExists() {
            // given
            fakeRepository.save(new User("id", "pw1", "email1"));

            // when & then
            assertThatThrownBy(() -> userRegister.register("id", "pw2", "email2"))
                .isInstanceOf(DubIdException.class);
        }
    }
    ```

2. 구현
    ```java
    public class UserRegister {
        private WeakPasswordChecker passwordChecker;
        private UserRepository userRepository; // 추가

        public UserRegister(WeakPasswordChecker passwordChecker,
            UserRepository userRepository) {
            this.passwordChecker = passwordChecker;
            this.userRepository = userRepository; // 추가
        }

        public void register(String id, String pw, String email) {
            if(passwordChecker.checkPasswordWeak(pw)) {
                throw new WeakPasswordException();
            }

            // 추가
            User user = userRepository.findById(id);
            if (user != null) {
                throw new DupIdException(); 
            }
            userRepository.save(new User(id, pw, email));
        }
    }
    ```
    ```java
    public interface UserRepository {
        void save(User user);
        User findById(String id);
    }
    ```
    ```java
    public class MemoryUserRepository implements UserRepository {
        private Map<String, User> users = new HashMap<>();

        @Override
        public void save(User user) {
            users.put(user.getId(), user);
        }    
        
        @Override
        public User findById(String id) {
            return users.get(id);
        }
    }
    ```
    ```java
    public class DubIdException extends RuntimeException {
    }
    ```

## 세 번째 테스트 : 회원 가입 성공 시 메일 발송 - Spy 사용
- 이메일 발송 여부를 어떻게 확인할 수 있을까?
    - 이를 확인할 수 있는 방법 중 하나는, UserRegister가 EmailNotifier의 메일 발송 기능을 실행할 때 이메일 주소로 "email@naver.com"을 사용했는지 확인하는 것이다. **이런 용도로 사용할 수 있는 것이 스파이(Spy) 대역이다.**

1. 테스트 코드 작성
    ```java
    public class UserRegisterTest {
        private UserRegister userRegister;
        private StubWeakPasswordChecker stubPasswordChecker = 
            new StubWeakPasswordChecker();
        private MemoryUserRepository fakeRepository = 
            new MemoryUserRepository();
        private SpyEmailNotifier spyEmailNotifier =
            new SpyEmailNotifier(); // 추가

        @BeforeEach
        void setUp() {
            userRegister = new UserRegister(stubPasswordChecker, 
                fakeRepository,
                spyEmailNotifier); // 추가
        }

        @DisplayName("가입하면 메일을 전송함")
        @Test
        void whenRegisterThenSendMail() {
            // when
            userRegister.register("id", "pw", "email@naver.com");

            // then
            // sendRegisterEmail가 호출되었는지 확인
            assertThat(spyEmailNotifier.isCalled()).isTrue();
            // sendRegisterEmail가 호출될 때, 인자로 "email@naver.com"가 사용되었는지 확인
            assertThat(spyEmailNotifier.getEmail()).isEqualTo("email@naver.com");
        }
    }
    ```

2. 구현
    ```java
    public class UserRegister {
        private WeakPasswordChecker passwordChecker;
        private UserRepository userRepository; 
        private EmailNotifier emailNotifier; // 추가

        public UserRegister(WeakPasswordChecker passwordChecker,
            UserRepository userRepository,
            EmailNotifier emailNotifier) {
            this.passwordChecker = passwordChecker;
            this.userRepository = userRepository; 
            this.emailNotifier = emailNotifier; // 추가
        }

        public void register(String id, String pw, String email) {
            if(passwordChecker.checkPasswordWeak(pw)) {
                throw new WeakPasswordException();
            }

            User user = userRepository.findById(id);
            if (user != null) {
                throw new DupIdException(); 
            }
            userRepository.save(new User(id, pw, email));

            emailNotifier.sendRegisterEmail(email); // 추가
        }
    }
    ```
    ```java
    public interface EmailNotifier {
        void sendRegisterEmail(String email);
    }
    ```
    ```java
    public class SpyEmailNotifier implements EmailNotifier {
        private boolean called;
        private String email;

        public boolean isCalled() {
            return called;
        }

        public String getEmail() {
            return email;
        }

        @Override
        public void sendRegisterEmail(String email) {
            this.called = true;
            this.email = email;
        }
    }
    ```

## Stub과 Spy 대체 - Mock 사용
```java
public class UserRegisterTest {
    private UserRegister userRegister;
    private WeakPasswordChecker mockPasswordChecker = 
        Mockito.mock(WeakPasswordChecker.class); // 수정
    private MemoryUserRepository fakeRepository = 
        new MemoryUserRepository();
    private EmailNotifier mockEmailNotifier =
        Mockito.mock(EmailNotifier.class); // 수정

    @BeforeEach
    void setUp() {
        userRegister = new UserRegister(mockPasswordChecker, 
            fakeRepository,
            mockEmailNotifier); // 수정
    }

    // Mock 객체를 사용하여 Stub을 대신함
    @DisplayName("약한 암호면 가입 실패")
    @Test
    void weakPassword() {
        // given
        BDDMochito.given(mockPasswordChecker.checkPasswordWeak("pw"))
            .willReturn(true); 

        // when & then
        assertThatThrownBy(() -> userRegister.register("id", "pw", "email"))
            .isInstanceOf(WeakPasswordException.class);
    }

    // Mock 객체를 사용하여 Spy를 대신함
    @DisplayName("가입하면 메일을 전송함")
    @Test
    void whenRegisterThenSendMail() {
        // when
        userRegister.register("id", "pw", "email@naver.com");

        // then
        ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class); 
        
        // sendRegisterEmail가 호출되었는지 확인 + 인자를 captor에 저장
        BDDMockito.then(mockEmailNotifier)
            .should().sendRegisterEmail(captor.capture());
        String realEmail = captor.getValue(); 

        // sendRegisterEmail가 호출될 때, 인자로 "email@naver.com"가 사용되었는지 확인
        assertThat(realEmail).isEqualTo("email@naver.com");
    }
}
```

## Behavior Verification - Mock 사용
```java
@DisplayName("회원 가입시 암호 검사 수행함")
@Test
void checkPassword() {
    // when
    userRegister.register("id", "pw", "email");

    // then
    BDDMockito.then(mockPasswordChecker) // 해당 Mock 객체의
        .should() // 특정 메서드가 호출됐는지 검증하는데
        .checkPasswordWeak(BDDMockito.anyString()); // 임의의 String 타입 인자를 이용해서 checkPasswordWeak 메서드 호출 여부 확인
}
```

# 대역 사용의 이점과 필요성
- **대기 시간을 줄여준다.**
    - ex) 다른 개발자가 A 기능을 구현하고 있을 때, 구현이 완료될 때까지 기다리지 않고 A 기능을 테스트할 수 있다.
    - ex) DB 서버가 구축되기 전에도 데이터가 올바르게 저장되는지 확인할 수 있다.
    - ex) 회원 가입 성공 시 메일 발송 여부를 확인할 때, 메일함에 메일이 도착할 때까지 기다릴 필요가 없다.
- **다양한 상황에 대해 테스트할 수 있다.**
    - ex) 외부 API를 사용할 때, 해당 API 서버의 응답을 5초 이내에 받지 못하는 상황을 테스트해볼 수 있다.
- **테스트 수행이 외부에 영향을 주면 안되는 경우**, 대역을 통해 해결할 수 있다.
    - ex) 은행 계좌에서 돈을 인출하는 기능을 테스트할 때, 실제 인출 요청이 API 서버에 전달되어서는 안된다.
- **테스트 내에서 같은 요청을 보냈을 때 외부의 응답이 항상 동일할 것이라고 신뢰할 수 없는 경우**, 대역을 통해 해결할 수 있다.
    - ex) 발급받은 오픈 API의 Key가 만료되면 응답이 달라진다.
    - ex) 외부 API 서버에 장애가 발생하면 응답이 달라진다.

---
- 참고 자료
테스트 주도 개발 시작하기(저자: 최범균)   
https://martinfowler.com/bliki/TestDouble.html  
http://xunitpatterns.com/Test%20Double.html  
http://xunitpatterns.com/SUT.html  
http://xunitpatterns.com/DOC.html  
https://gyuwon.github.io/blog/2020/05/10/do-you-really-need-test-doubles.html  
https://beomseok95.tistory.com/295  
https://jonghoonpark.com/2023/12/05/test-double-for-well-grounded-java-developer  
https://greeng00se.github.io/test-double  
https://umbum.dev/1233/  
https://f-lab.kr/insight/test-code-and-mock-object  