---
title: "(Spring) - Spring 살펴보기 (+WebApplicationContext)"
tags:
    - spring
date: "2024-05-02"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# Spring MVC와 DispatcherServlet
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/4f7ebd71-77d4-4392-bfe9-542cd224dba7" width="700" height="300"></center>

- Spring MVC는 프론트 컨트롤러 패턴을 중심으로 설계되었으며, DispatcherServlet은 Spring MVC에서 프론트 컨트롤러 기능을 담당하는 서블릿이다.
    - DispatcherServlet은 모든 HTTP 요청을 받아서 공통 기능을 처리한다. 즉, <span style="color:blueviolet">핸들러(컨트롤러)에서 공통으로 처리해야 하는 부분을 DispatcherServlet에서 핸들러(컨트롤러) 호출 전에 먼저 처리</span>한다.(수문장 역할)
    - 클라이언트로부터 요청이 오면 DispatcherServlet이 직접 모든 작업을 수행하는 것이 아니라, <span style="color:red">요청을 적절한 구성 요소(delegate componenets = **special beans**)로 전달하여 작업을 **위임**</span>한다.

> - <span style="color:red">위임(delegation)</span>
    - 위임은 한 객체가 자신이 직접 작업을 수행하지 않고, 다른 객체가 그 작업을 수행하도록 맡기는 설계 패턴을 의미한다.
    - 간단히 말해서, A 객체가 B 객체를 가지고 있고, A 객체에서 B 객체의 메서드를 호출하여 B 객체가 작업을 수행하면, 이는 A 객체가 B 객체에게 작업을 위임하는 것이다.

> - <span style="color:red">특별한 빈(special bean)</span>
    - 특별한 빈(special bean)이란 스프링이 관리하는 객체인 빈(bean) 중에서, 스프링 프레임워크에서 정해진 역할이나 기능을 수행하기 위해 구현된 빈(bean)을 말한다.
    - ex) HandlerMapping, HandlerAdapter, viewResolver, HandlerExceptionResolver 등

# ApplicationContext, WebApplicationContext, AnnotationConfigWebApplicationContext
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/2c33a644-bea2-4f79-b38f-17dbbba1c55a" width="570" height="330"></center>
- **ApplicationContext**
    - <span style="color:red">ApplicationContext 인터페이스를 구현한 객체(ex) AnnotationConfigWebApplicationContext)</span>를 스프링이 관리하는 빈들이 담겨 있는 컨테이너<span style="color:red">(스프링 컨테이너 = IoC 컨테이너 = DI 컨테이너)</span>라고 한다. 
    - <span style="color:blueviolet">스프링 컨테이너는 빈의 생명주기 관리, 빈의 의존성 주입(DI) 등을 담당한다.</span>
    - ApplicationContext 인터페이스를 구현한 클래스는 여러 종류가 있다.  
<br>
- **WebApplicationContext**
    - <span style="color:red">DispatcherServlet은 자체 설정을 위해 WebApplicationContext를 필요로 한다.</span>
    - WebApplicationContext는 ApplicationContext에 getServletContext() 메소드 등이 추가된 인터페이스로, <span style="color:blueviolet">자신이 속한 ServletContext와 서블릿에 대한 링크</span>를 가지고 있다.  
<br>
- **AnnotationConfigWebApplicationContext**
    - AnnotationConfigWebApplicationContext는 WebApplicationContext 인터페이스를 구현한 클래스이다.
    - AnnotationConfigWebApplicationContext 클래스를 사용할 때 <span style="color:blueviolet">어노테이션 기반의 자바 코드 설정</span>을 넘기면 된다. 자바 코드로된 설정에서는 @Configuration, @Bean 등의 어노테이션을 사용하여 빈을 정의하고 구성한다.  
    - XmlWebApplicationContext는 어노테이션 기반의 자바 코드 설정이 아닌, xml 설정 파일을 사용한다.  
<br>
- **AnnotationConfigWebApplicationContext 예시**
    ```java
    @Configuration
    public class AppConfig 
    {
        @Bean
        public MemberRepository memberRepository() 
        { return new MemoryMemberRepository(); }
        
        @Bean
        public MemberService memberService() 
        { return new MemberServiceImpl(memberRepository()); }
    }
    ```
    ```java
    public class Example 
    {
        public static void main(String[] args) 
        {
            // ApplicationContext ac 
            //    = new AnnotationConfigWebApplicationContext();
            
            // AnnotationConfigWebApplicationContext : 어노테이션 기반 자바 코드 사용 
            AnnotationConfigWebApplicationContext ac 
                = new AnnotationConfigWebApplicationContext();
            ac.register(AppConfig.class);
            ac.refresh(); // 컨텍스트를 초기화하여 등록된 클래스들이 처리되도록 함
            // WebApplicationInitializer를 사용한다면 refresh() 메소드를 직접 호출할 필요가 없음

            MemberService memberService 
                = ac.getBean("memberService", MemberService.class);

            Member member = new Member("A", 20);
            memberService.join(member);
        }
    }
    ```
    > - 위 코드에서, <span style="color:blueviolet">AnnotationConfigWebApplicationContext 객체를 스프링 컨테이너</span>라고 한다.

# Context Hierarchy(계층 구조)
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/4cbc19e4-fd4d-48cb-8c67-ed639ed1c6f6" width="370" height="310"></center>
- 대부분의 웹 애플리케이션은 하나의 웹 애플리케이션에서 단일 WebApplicationContext를 사용하는 것으로 충분하다. 그러나, 더 복잡한 경우에는 Context 계층 구조를 사용할 수 있다.
    - Context 계층 구조란, <span style="color:red">하나의 Root WebApplicationContext가 여러 DispatcherServlet(또는 다른 서블릿)과 공유되고, 각 서블릿마다 자체적인 자식 WebApplicationContext을 가지는 구조</span>이다.   
<br>
- **Context 계층 구조**
    1. **Root WebApplicationContext**
        - Root WebApplicationContext는 데이터 저장소, 비즈니스 서비스와 같이 <span style="color:blueviolet">여러 서블릿에서 공유해야 하는 인프라 빈을 포함</span>
        - Root WebApplicationContext의 빈은 자식 WebApplicationContext에서 상속됨
    2. **Servlet WebApplicationContext (자식 WebApplicationContext)**  
        - <span style="color:blueviolet">각 서블릿에 특화된 빈을 포함</span>  
        - <span style="color:red">Root WebApplicationContext의 빈을 상속함</span>
        - 필요에 따라 Root WebApplicationContext의 빈을 재정의 가능

---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet.html  
https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/special-bean-types.html  
https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/context-hierarchy.html   
https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/container-config.html  
https://www.inflearn.com/questions/226425/servletcontext%EC%99%80-webapplicationcontext%EC%9D%98-%EA%B4%80%EA%B3%84-%EC%A7%88%EB%AC%B8  
https://blog.naver.com/kitepc/221314687808  
https://escapefromcoding.tistory.com/174  
https://mangkyu.tistory.com/18  
https://kimcoder.tistory.com/511  
https://recordsoflife.tistory.com/490  
https://blog.naver.com/PostView.nhn?blogId=inho1213&logNo=220516464519&parentCategoryNo=&categoryNo=19&viewDate=&isShowPopularPosts=false&from=postView  
https://live-everyday.tistory.com/164  
https://velog.io/@ruinak_4127/Context  
https://dev-wnstjd.tistory.com/440  
https://devlogofchris.tistory.com/71  
https://unordinarydays.tistory.com/131  
</div>
</details>