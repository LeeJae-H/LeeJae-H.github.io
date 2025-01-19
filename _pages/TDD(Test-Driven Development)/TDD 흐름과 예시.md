---
title: "TDD 흐름과 예시"
tags:
    - java
date: "2024-12-08"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> ***TDD(Test-Driven Development)**는 매우 짧은 개발 사이클을 반복하는 소프트웨어 개발 프로세스 중 하나이다. 개발자는 먼저 요구사항을 검증하는 자동화된 테스트 케이스를 작성한다. 그런 후에, 그 테스트 케이스를 통과하기 위한 최소한의 코드를 생성한다. 마지막으로 작성한 코드를 표준에 맞도록 리팩토링한다.* <span style="color:gray">(feat. 위키백과)</span>

# TDD 흐름
<img src="https://github.com/user-attachments/assets/71820640-3cc8-4aea-9d42-c99ff18f6972" style="width:700px;" />
1. **테스트(Red)** : 기능을 검증하는 테스트 코드를 먼저 작성한다.
    - 테스트에 대한 구현을 하지 않았으므로 **실패해야 한다.** 
    - 실패 예시 : 존재하지 않는 객체나 메서드 등을 작성하여 실패(컴파일 에러), 객체나 메서드가 이미 존재하지만 테스트할 상황에 대한 구현이 되어 있지 않아서 실패 등
2. **코딩(Green)** : 테스트를 통과할 만큼만 코드를 작성한다.  
    - **지금까지 작성한 테스트들을 통과할 만큼만 구현을 진행하며, 아직 추가하지 않은 테스트들을 고려하지 않는다.**
3. **리팩토링(Refactor)** : 테스트를 통과한 뒤에는 개선할 코드가 있으면 리팩토링한다. 리팩토링을 수행한 뒤에는 다시 테스트를 실행해서 기존 기능이 망가지지 않았는지 확인한다.
    - 테스트 코드도 코드이기 때문에 유지보수 대상이다. 즉, 테스트 메서드에서 발생하는 중복을 제거하거나 의미가 잘 드러나게 코드를 수정할 필요가 있다.
    - 그러나 **테스트 코드의 중복을 무턱대고 제거하면 안된다.** 중복을 제거한 뒤에도 테스트 코드의 가독성이 떨어지지 않고 수정이 용이한 경우에만 중복을 제거해야 한다.   

=> *이 과정들을 반복하면서 점진적으로 기능을 완성해 나가는 것이 전형적인 TDD의 흐름이며, 테스트 코드를 먼저 작성함으로써 테스트가 개발을 주도하게 된다* 


# TDD 예시(단위 테스트) - 암호 검사기
> - *암호 검사기 기능을 TDD로 구현해보자. 암호 검사기는 문자열을 검사해서 규칙을 준수하는지에 따라 암호를 "강함", "보통", "약함"으로 구분한다.*
    - *검사할 규칙은 다음 두 가지이다.*
        - *길이가 8글자 이상*
        - *0부터 9 사이의 숫자를 포함*
    - *암호의 구분은 다음과 같다.*
        - *2개의 규칙 모두 충족 => "강함"*
        - *1개의 규칙 충족 => "보통*"
        - *0개의 규칙 충족 => "약함"*

- 첫 번째 테스트를 잘 선택하는 것이 중요하다. 첫 번째 테스트를 선택할 때에는 **가장 쉽거나 가장 예외적인 상황을 선택**해야 한다.
    - 모든 규칙을 충족하는 경우 -> 해당 상황을 선택!
    - 모든 규칙을 충족하지 않는 경우
    - 값이 없는 경우(null 또는 빈 문자열)

## 첫 번째 테스트 : 모든 규칙을 충족하는 경우
1. 테스트 코드 작성
    - 테스트할 기능을 제공할 클래스의 이름, 테스트 메서드 이름, 메서드의 리턴 타입 등을 결정했다.
    - PasswordStrengthMeter 타입과 PasswordStrength 타입이 존재하지 않으므로 컴파일 에러가 발생한다.
    ```java
    public class PasswordStrengthMeterTest {
        @Test
        void meetsAllCriteria_Then_Strong() {
            PasswordStrengthMeter meter = new PasswordStrengthMeter();
            PasswordStrength result = meter.meter("ab12!@AB");
            assertThat(result).isEqualTo(PasswordStrength.STRONG);
        }
    }
    ```

2. 구현
    - 지금까지 작성한 테스트들을 통과할 만큼만 구현한다.
    ```java
    public enum PasswordStrength {
        STRONG
    }
    ```
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            return PasswordStrength.STRONG;
        }
    }
    ```

3. 리팩토링 
    - 아직 개선할 부분이 없어 보인다!

## 두 번째 테스트 : 길이만 8글자 미만이고 나머지 조건은 충족하는 경우
1. 테스트 코드 작성
    - PasswordStrength 열거 타입에 NORMAL이 없으므로 컴파일 에러가 발생하고, 암호의 글자수 확인에 대한 구현이 되어 있지 않아 실패한다.
    ```java
    public class PasswordStrengthMeterTest {
        @Test
        void meetsOtherCriteria_except_for_length_Then_Normal() {
            PasswordStrengthMeter meter = new PasswordStrengthMeter();
            PasswordStrength result = meter.meter("ab12!@A");
            assertThat(result).isEqualTo(PasswordStrength.NORMAL);
        }
    }
    ```

2. 구현
    - 지금까지 작성한 테스트들을 통과할 만큼만 구현한다.
    ```java
    public enum PasswordStrength {
        STRONG, NORMAL
    }
    ```
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            // 추가
            if (s.length() < 8) return PasswordStrength.NORMAL; 

            return PasswordStrength.STRONG;
        }
    }
    ```

3. 리팩토링 
    - 아직 개선할 부분이 없어 보인다!

## 세 번째 테스트 : 숫자를 포함하지 않고 나머지 조건은 충족하는 경우
1. 테스트 코드 작성
    - 컴파일 에러는 발생하지 않지만, 암호의 숫자 포함 여부에 대한 구현이 되어 있지 않아 실패한다.
    ```java
    public class PasswordStrengthMeterTest {
        @Test
        void meetsOtherCriteria_except_for_number_Then_Normal() {
            PasswordStrengthMeter meter = new PasswordStrengthMeter();
            PasswordStrength result = meter.meter("ab!@ABCDEF");
            assertThat(result).isEqualTo(PasswordStrength.NORMAL);
        }
    }
    ```

2. 구현
    - 지금까지 작성한 테스트들을 통과할 만큼만 구현한다.
    ```java
    public enum PasswordStrength {
        STRONG, NORMAL
    }
    ```
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            if (s.length() < 8) return PasswordStrength.NORMAL; 

            // 추가
            boolean containsNum = false;
            for (char ch : s.toCharArray()) {
                if (ch >= '0' && ch <= '9') {
                    containsNum = true;
                    break;
                }
            }
            if (!containsNum) return PasswordStrength.NORMAL;

            return PasswordStrength.STRONG;
        }
    }
    ```

3. 리팩토링 
    - 세 개의 테스트 메서드에서 PasswordStrengthMeter 객체를 생성하는 코드의 중복, 기능을 실행하고 이를 확인하는 코드의 중복을 제거했다.
    - 숫자 포함 여부를 확인하는 코드가 다소 길어지므로 해당 코드를 메서드로 추출해서 가독성을 개선하고 매서드 길이도 줄였다.
    ```java
    public class PasswordStrengthMeterTest {
        private PasswordStrengthMeter meter = new PasswordStrengthMeter();

        @Test
        void meetsAllCriteria_Then_Strong() {
            assertStrength("ab12!@AB");
        }

        @Test
        void meetsOtherCriteria_except_for_length_Then_Normal() {
            assertStrength("ab12!@A");
        }

        @Test
        void meetsOtherCriteria_except_for_number_Then_Normal() {
            assertStrength("ab!@ABCDEF");
        }

        private void assertStrength(String password, PasswordStrength pwstr) {
            PasswordStrength result = meter.meter(password);
            assertThat(result).isEqualTo(pwstr);
        }
    }
    ```
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            if (s.length() < 8) return PasswordStrength.NORMAL; 

            // 수정
            boolean containsNum = meetsContainingNumberCriteria(s);
            if(!containsNum) return PasswordStrength.NORMAL;

            return PasswordStrength.STRONG;
        }

        // 추가
        private boolean meetsContainingNumberCriteria(String s) {
            for (char ch : s.toCharArray()) {
                if (ch >= '0' && ch <= '9') {
                    return true
                }
            }
            return false;
        }
    }
    ```

## 네 번째 테스트 : 값이 없는 경우(null 또는 빈 문자열)
1. 테스트 코드 작성
    - null 또는 빈 문자열을 입력할 경우에, IllegalArgumentException 발생시키는 방법과 유효하지 않은 암호를 의미하는 PasswordStrength.INVALID를 리턴하는 방법을 떠올렸고 이 중에 후자의 방법을 선택했다.
    - PasswordStrength 열거 타입에 INVALID이 없으므로 컴파일 에러가 발생하고, 값이 없는 상황에 대한 구현이 되어 있지 않아 실패한다. 
    ```java
    public class PasswordStrengthMeterTest {
        @Test
        void nullOrEmptyInput_Then_Invalid() {
            assertStrength(null, PasswordStrength.INVALID);
            assertStrength("", PasswordStrength.INVALID);
        }
    }
    ```

2. 구현
    - 지금까지 작성한 테스트들을 통과할 만큼만 구현한다.
    ```java
    public enum PasswordStrength {
        STRONG, NORMAL, INVALID
    }
    ```
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            //추가
            if (s == null || s.isEmpty()) return PasswordStrength.INVALID;
            
            if (s.length() < 8) return PasswordStrength.NORMAL; 
            boolean containsNum = meetsContainingNumberCriteria(s);
            if(!containsNum) return PasswordStrength.NORMAL;
            return PasswordStrength.STRONG;
        }
    }
    ```

3. 리팩토링 
    - 개선할 부분이 없어 보인다!

## 다섯 번째 테스트 : 모든 규칙을 충족하지 않는 경우
1. 테스트 코드 작성
    - PasswordStrength 열거 타입에 WEAK가 없으므로 컴파일 에러가 발생하고, 모든 규칙을 충족하지 않는 상황에 대한 구현이 되어 있지 않아 실패한다.  
    ```java
    public class PasswordStrengthMeterTest {
        @Test
        void meetsNoCriteria_Then_Weak() {
            assertStrength("abc", PasswordStrength.WEAK);
        }
    }
    ```

2. 구현
    - 지금까지 작성한 테스트들을 통과할 만큼만 구현한다.
    - WEAK를 리턴하기 위해 꽤 많은(?) 코드를 수정했다.
    ```java
    public enum PasswordStrength {
        STRONG, NORMAL, INVALID, WEAK
    }
    ```
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            if (s == null || s.isEmpty()) return PasswordStrength.INVALID;
            
            boolean lengthEnough = s.length() >= 8; // 수정
            boolean containsNum = meetsContainingNumberCriteria(s);
            
            // 추가
            if(!lengthEnough && !containsNum) return PasswordStrength.WEAK;
            
            if(!lengthEnough) return PasswordStrength.NORMAL; // 수정
            if(!containsNum) return PasswordStrength.NORMAL;
            return PasswordStrength.STRONG;
        }
    }
    ```

3. 리팩토링
    - 지금까지의 코드도 전체적으로 나쁘진 않았지만, "~개의 규칙을 충족하면 강도가 ~이다."라는 느낌을 주도록(**코드 가독성 개선**) 코드를 리팩토링했다.
    ```java
    public class PasswordStrengthMeter {
        public PasswordStrength meter(String s) {
            if (s == null || s.isEmpty()) return PasswordStrength.INVALID;
            
            int metCounts = getMetCriteriaCounts(s);
            if(metCounts == 0) return PasswordStrength.WEAK;
            if(metCounts == 1) return PasswordStrength.NORMAL;

            return PasswordStrength.STRONG;            
        }

        private int getMetCriteriaCounts(String s) {
            int metCounts = 0;
            if (s.length() >= 8) metCounts++;
            if (meetsContainingNumberCriteria(s)) metCounts++;
            return metCounts; 
        }
    }
    ``` 
    - **테스트에서 메인으로 코드 이동**
        - 마지막으로, src/test/java 소스 폴더에 위치한 PasswordStrength.java 파일과 PasswordStrengthMeter.java 파일을 배포 대상인 src/main/java로 이동해야 비로서 구현이 끝난다.

# 테스트가 개발을 주도
- 가장 먼저 통과해야 할 테스트를 작성했다. 이 과정에서 구현을 생각하지 않았고, 테스트를 추가한 뒤 지금까지 작성한 테스트들을 통과시킬 만큼 기능을 구현했다. 테스트 코드가 추가되면서 **검증하는 범위가 넓어질수록 구현도 점점 완성**되어간다. 
    -> 이렇게 테스트가 개발을 주도해 나간다.

---
- 참고 자료
테스트 주도 개발 시작하기(저자: 최범균)  
https://tech.inflab.com/20230404-test-code/   
https://jayhooney.github.io/tdd/TDD-part1/   