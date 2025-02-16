---
title: "(Tomcat) - Tomcat의 요청 처리 과정"
tags:
    - tomcat
date: "2024-05-03"
thumbnail: "/assets/img/thumbnail/sample.png"
---
# Tomcat의 요청 처리 과정 (필터 X)
<img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/72a914b4-8403-4964-b363-758006a1ed4a" style="width:600px;" />
1. Client가 Web Server로 HTTP Request 보냄  

2. 정적 콘텐츠의 경우 Web Server에서 처리, 동적 콘텐츠의 경우 Web Server가 Servlet Container로 요청 전달<span style="color:red"> - (1)</span>  

3. 요청을 받은 Servlet Container는 HttpServletRequest, HttpServletResponse 객체를 생성

4. Servlet Container는 주로 web.xml 또는 어노테이션 또는 ServletContainerInitializer 인터페이스의 구현체를 통해서 요청한 URL에 맞는 서블릿을 찾음<span style="color:red"> - (2)</span>
    - Servlet Container에 해당 서블릿 인스턴스가 없는 경우 
        - Servlet Container가 서블릿 인스턴스를 생성하고 서블릿의 init() 메소드 호출 
    - Servlet Container에 해당 서블릿 인스턴스가 있는 경우 
        - 서블릿 인스턴스를 다시 생성하지 않고 재사용 **(싱글톤 패턴)**
<br>  
5. Servlet Container는 요청을 처리할 Thread를 Thread Pool에서 가져옴 **(멀티 스레드 환경)**

6. Thread는 서블릿의 service() 메소드 호출<span style="color:red"> - (3)</span>  
    ```java
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException { 
        ... 
    }
    ```
    6-1. service() 메소드에서는 요청에 따라 doGET() 또는 doPost() 메소드 호출
    6-2. 응답 값을 HttpServletResponse 객체에 담음

7. Servlet Container는 응답 값을 HTTP Response로 바꾼 후 Web Server로 전송

8. HttpServletRequest, HttpServletResponse 객체를 소멸시키고 Thread를 종료

# Tomcat의 요청 처리 과정 (필터 O)
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/2ab63e42-2fa8-47b6-9cd9-0e1d409029f5" width="700" height="350"></center>
1. Client가 Web Server로 HTTP Request 보냄  
    
2. 정적 콘텐츠의 경우 Web Server에서 처리, 동적 콘텐츠의 경우 Web Server가 Servlet Container로 요청 전달<span style="color:red"> - (1)</span>  

3. 요청을 받은 Servlet Container는 HttpServletRequest, HttpServletResponse 객체를 생성

4. Servlet Container는 주로 web.xml 또는 어노테이션 또는 ServletContainerInitializer 인터페이스의 구현체를 통해서 요청한 URL에 맞는 필터와 서블릿을 찾음<span style="color:red"> - (2)</span>  
    - 필터
        - **Tomcat이 시작될 때 Servlet Container는 모든 필터 인스턴스를 생성(싱글톤 패턴)하고 init() 메소드 호출**
    - Servlet Container에 해당 서블릿 인스턴스가 없는 경우 
        - Servlet Container가 서블릿 인스턴스를 생성하고 서블릿의 init() 메소드 호출 
    - Servlet Container에 해당 서블릿 인스턴스가 있는 경우 
        - 서블릿 인스턴스를 다시 생성하지 않고 재사용 **(싱글톤 패턴)**
<br>
5. Servlet Container는 FilterChain을 생성<span style="color:red"> - (3)</span>  
    - FilterChain은 필터들과 서블릿의 연결고리라고 볼 수 있음
<br>
6. Servlet Container는 요청을 처리할 Thread를 Thread Pool에서 가져옴 **(멀티 스레드 환경)**

7. Thread는 첫 번째 필터의 doFilter() 메소드 호출<span style="color:red"> - (4)</span>  
    ```java
    @Override
    public void doFilter(ServletRequest request,ServletResponse response,
                                            FilterChain chain) throws ServletException, IOException {
        ...
        chain.doFilter(request, response);
    }
    ```
    7-1. chain.doFilter(request, response)에서는 다음 필터가 있으면 필터를 호출하고, 다음 필터가 없으면 서블릿 service() 메소드 호출
    7-2. service() 메소드에서는 요청에 따라 doGET() 또는 doPost() 메소드 호출
    7-3. 서블릿의 service() 메소드가 끝나면 이전 필터로 돌아가고, 반복해서 첫 번째 필터까지 돌아감   
    7-4. 응답 값을 HttpServletResponse 객체에 담음  

8. Servlet Container는 응답 값을 HTTP Response로 바꾼 후 Web Server로 전송

9. HttpServletRequest, HttpServletResponse 객체를 소멸시키고 Thread를 종료

---
- 참고 자료
https://ko.wikipedia.org/wiki/웹_컨테이너   
https://ko.wikipedia.org/wiki/아파치_톰캣  
https://itwiki.kr/w/아파치_톰켓  
https://docs.spring.io/spring-security/reference/servlet/architecture.html  
https://www.inflearn.com/questions/1132952/servlet%EC%97%90-%EB%8C%80%ED%95%B4-%EC%A0%9C-%EC%83%9D%EA%B0%81%EC%9D%84-%ED%95%9C%EB%B2%88-%EC%A0%95%EB%A6%AC%ED%95%B4%EB%B4%A4%EC%8A%B5%EB%8B%88%EB%8B%A4   
https://stackoverflow.com/questions/3386254/servlets-filters-and-threads-in-tomcat  
https://d-memory.tistory.com/37  
https://taes-k.github.io/2020/02/16/servlet-container-spring-container/  
https://forsaken.tistory.com/entry/Servlet-JSP-%EC%84%9C%EB%B8%94%EB%A6%BF-filter  
https://velog.io/@bey1548/Servlet-Filter  
https://curiousjinan.tistory.com/entry/spring-filterchain-dofilter  
https://steady-coding.tistory.com/599  
https://tecoble.techcourse.co.kr/post/2021-05-23-servlet-servletcontainer/  
https://github.com/mangdo/TIL/blob/main/Spring/Controller/HttpServletRequest.md  
https://velog.io/@zini9188/Spring-Security-Filter%EC%99%80-FilterChain  
https://cbw1030.tistory.com/215  
https://velog.io/@yoho98/%EC%84%9C%EB%B8%94%EB%A6%BFServlet%EA%B3%BC-%EC%84%9C%EB%B8%94%EB%A6%BF-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88Servlet-Container-y88kny7g  
https://forsaken.tistory.com/entry/Servlet-JSP-%EC%84%9C%EB%B8%94%EB%A6%BF-filter  
