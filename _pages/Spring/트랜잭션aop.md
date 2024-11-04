---
title: "(Spring) - 스프링이 제공하는 트랜잭션 AOP"
tags:
    - spring
date: "2024-06-21"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# 서비스 계층
- **애플리케이션 구조**
    - 여러가지 애플리케이션 구조가 있지만, 가장 단순하면서 많이 사용하는 방법은 역할에 따라 3가지 계층으로 나누는 것이다.
        <center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/f0c26013-a725-4e89-a008-41c8a797414c" width="620" height="150"></center>
        - 프레젠테이션 계층
            - UI와 관련된 처리, 웹 요청과 응답, 사용자 요청 검증을 담당
            - 주 사용 기술: 서블릿과 HTTP 같은 웹 기술, 스프링 MVC
        - 서비스 계층
            - 비즈니스 로직을 담당
            - 주 사용 기술: 가급적 특정 기술에 의존하지 않고, 순수 자바 코드로 작성
        - 데이터 접근 계층
            - 실제 데이터베이스에 접근을 담당
            - 주 사용 기술: JDBC, JPA, File, Redis, Mongo  
<br>
- **서비스 계층** ([이전 글](https://leejae-h.github.io/posts/63) 참고)
    1. 서비스 계층은 가급적 <span style="color:blueviolet">특정 구현 기술에 직접 의존해서는 안된다.</span>  
        - 서비스 계층에서 트랜잭션을 사용하기 위해 JDBC 기술에 의존하던 문제  
            -> 트랜잭션 매니저를 통해서 해결했다.
    2. 서비스 계층은 가급적 <span style="color:blueviolet">핵심 비즈니스 로직만 구현해야 한다.</span>
        - 아직 서비스 계층에서 비즈니스 로직 뿐만 아니라 트랜잭션을 처리하는 기술 로직이 함께 포함되어 있는 문제가 남아있다.  
            -> <span style="color:red">스프링 AOP를 통해 프록시를 도입하면 해결할 수 있다.</span>

# 트랜잭션 AOP 
- **트랜잭션 프록시**
    <center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/0aa5524f-8d88-4721-b76c-7fa0f334f93b" width="620" height="220"></center>
    - 프록시를 사용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.
    - 트랜잭션 프록시 코드 예시
        ```java
        public class TransactionProxy {
            private MemberService target;
            public void logic() {
                TransactionStatus status = transactionManager.getTransaction(..);
                try {
                    target.logic(); //실제 대상 호출
                    transactionManager.commit(status);
                } catch (Exception e) {
                    transactionManager.rollback(status);
                    throw new IllegalStateException(e);
                }
            }
        }
        ```
    - 트랜잭션 프록시 적용 후 서비스 코드 예시
        ```java
        public class Service {
            public void logic() { 
                bizLogic(fromId, toId, money); // 비즈니스 로직
            }
        }
        ```
    - <span style="color:blueviolet">스프링이 제공하는 AOP 기능을 사용하면 프록시를 편리하게 적용할 수 있다.</span>
        - 스프링 AOP를 사용하려면 어드바이저, 포인트컷, 어드바이스가 필요하다.  
<br>
- **트랜잭션 AOP**
    <center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/4c10ce3e-75a5-4488-8004-aeef7a2350ff" width="600" height="320"></center>
    - 스프링 AOP를 직접 사용해서 트랜잭션을 처리해도 되지만, <span style="color:red">스프링은 트랜잭션 AOP를 처리하기 위한 모든 기능을 제공</span>한다.
        - 스프링은 트랜잭션 AOP 처리를 위해 다음 클래스를 제공한다.
            - 어드바이저: BeanFactoryTransactionAttributeSourceAdvisor
            - 포인트컷: TransactionAttributeSourcePointcut
            - 어드바이스: TransactionInterceptor
        - 트랜잭션 처리가 필요한 곳에 <span style="color:red">@Transactional 어노테이션</span>만 붙여주면, 스프링의 트랜잭션 AOP는 이 애노테이션을 인식해서 트랜잭션 프록시를 적용해준다.

# 트랜잭션 AOP 예시
> [이전 글](https://leejae-h.github.io/posts/62) 참고

- MemberRepositoryV4 클래스 
    ```java
    @Repository
    public class MemberRepositoryV4 implements MemberRepository {
        // MemberRepositoryV2와 동일
    }
    ```
- MemberServiceV4 클래스 
    ```java
    @Service
    public class MemberServiceV4 {
        private final MemberRepository memberRepository;
        public MemberServiceV4(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }

        @Transactional
        public void accountTransfer(String fromId, String toId, int money) throws SQLException {
            bizLogic(fromId, toId, money); // 비즈니스 로직
        }
        
        private void bizLogic(String fromId, String toId, int money) throws SQLException {
            Member fromMember = memberRepository.findById(fromId);
            Member toMember = memberRepository.findById(toId);
            memberRepository.update(fromId, fromMember.getMoney() - money);
            memberRepository.update(toId, toMember.getMoney() + money);
        }
    }
    ```
- <span style="color:red">서비스 계층에서 아직 JDBC 기술인 SQLException에 의존하지만, 이 부분은 일단 넘어가자.</span>
    - 그래도 간단하게 말하자면, SQLException은 체크 예외인데, 리포지토리 계층에서 체크 예외인 SQLException을 언체크(런타임) 예외로 바꿔서 던지면 된다.

--- 
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
"김영한 스프링 DB 1편"    
https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html  
</div>
</details>