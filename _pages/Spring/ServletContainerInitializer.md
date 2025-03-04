---
title: "(Tomcat) - web.xml을 대체하는 ServletContainerInitializer (+ServletContextListener)"
tags:
    - tomcat
date: "2024-05-10"
thumbnail: "/assets/img/thumbnail/sample.png"
---

> - **web.xml과 ServletContainerInitializer**
    - **web.xml** : 서블릿 컨테이너가 웹 애플리케이션을 초기화할 때 필요한 정보를 제공한다.
    - **ServletContainerInitializer** : 서블릿 컨테이너가 시작될 때 ServletContainerInitializer를 구현한 클래스를 실행하여 웹 애플리케이션을 초기화한다.
    - <span style="color:red">ServletContext는 서블릿 컨테이너라고 볼 수 있다. 따라서 web.xml이나 ServletContainerInitializer의 핵심은 결국 ServletContext 객체를 초기화하기 위함이다.</span>

---
# ServletContainerInitializer
```java
public interface ServletContainerInitializer 
{
    void onStartup(Set<Class<?>> var1, ServletContext var2) throws ServletException;
}
```
- Servlet 3.0부터 ServletContainerInitializer의 등장으로 기존의 web.xml 기반 대신 <span style="color:red">코드 기반으로 서블릿 컨테이너를 초기화하는 작업(**ServletContext 객체 초기화**)</span>이 가능해졌다. 
    - ServletContainerInitializer 인터페이스를 구현한 클래스가 있으면, ServletContainerInitializer를 구현한 클래스는 <span style="color:blueviolet">서블릿 컨테이너가 시작될 때 로드되고 인스턴스화되며 onStartup 메소드가 호출된다.</span>  
<br>
- **ServletContainerInitializer 구현**
    - ServletContainerInitializer 인터페이스를 구현하는 클래스를 만든 후, /META-INF/services/jakarta.servlet.ServletContainerInitializer 파일을 생성하고 ServletContainerInitializer 인터페이스를 구현한 클래스의 이름을 해당 파일에 적어준다.
    - ServletContainerInitializer 인터페이스를 구현하는 클래스에 @HandlesTypes 어노테이션으로 클래스를 지정해주면, 서블릿 컨테이너가 시작될 때 지정된 클래스를 찾아서 ServletContainerInitializer 인터페이스를 구현하는 클래스의 onStartup() 메소드에 지정된 클래스와 ServletContext 객체를 인자로 넣어준다.  

# ServletContext 객체 초기화
> - 서블릿 컨테이너가 시작될 때 수행되는 웹 애플리케이션을 초기화하는 작업인 <span style="color:red">"ServletContext 객체 초기화"</span>에 대해서 web.xml 방식과 ServletContainerInitializer 방식의 예시를 살펴보자.

>1. web.xml 방식
    - 서블릿 컨테이너가 시작되고 웹 애플리케이션이 초기화될 때, 서블릿 컨테이너는 <span style="color:blueviolet">web.xml을 읽은 후 ServletContext 객체를 초기화</span>한다.
2. ServletContainerInitializer 방식
    - 서블릿 컨테이너가 시작되고 웹 애플리케이션이 초기화될 때, 서블릿 컨테이너는 <span style="color:blueviolet">ServletContainerInitializer 인터페이스를 구현한 클래스를 찾은 후 해당 클래스를 로딩하고 인스턴스화한다. 이후 해당 클래스의 onStartup() 메소드를 호출하여 ServletContext 객체를 초기화</span>한다.    

<br>
## ServletContext 객체 초기화 1 - ServletContext 초기화 파라미터 설정, 서블릿 등록 등
1. **web.xml 방식**
    - 웹 애플리케이션 구조
        ```bash
/src
        /main
          /java
            /com
              /example
                ExampleServlet.java
          /webapp
            /WEB-INF
              web.xml
        ``` 
    - 서블릿 클래스 생성
        ```java
        public class ExampleServlet extends HttpServlet 
        {
            @Override
            protected void service(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException
                {
                    System.out.println("service 메소드 호출");
                }    
        }
        ```
    - web.xml 작성
        ```xml
        <!-- ServletContext 초기화 파라미터 설정 -->
        <context-param>
            <param-name>contextParamName_example</param-name>
            <param-value>contextParamValue_example</param-value>
        </context-param>

        <!-- 서블릿 등록 -->
        <servlet>
            <servlet-name>exampleServlet</servlet-name>
            <servlet-class>com.example.ExampleServlet</servlet-class>
        </servlet>
        
        <servlet-mapping>
            <servlet-name>exampleServlet</servlet-name>
            <url-pattern>/example</url-pattern>
        </servlet-mapping>
        
        <!-- 필터, 리스너 등 웹 애플리케이션 구성 요소 관련 -->
        ...
        ```

2. **ServletContainerInitializer 방식**
    - 웹 애플리케이션 구조
        ```bash
/src
        /main
          /java
            /com
              /example
                ExampleServlet.java
                MyServletContainerInitializer.java
                MyAppInitializer.java
                MyAppInitializerV1.java
          /resources
            /META-INF
              /services
                jakarta.servlet.ServletContainerInitializer
        ``` 
    - 서블릿 클래스 생성
        ```java
        public class ExampleServlet extends HttpServlet 
        {
            @Override
            protected void service(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException
                {
                    System.out.println("service 메소드 호출");
                }    
        }
        ```
    - ServletContainerInitializer 구현 클래스 생성
        ```java
        @HandlesTypes(MyAppInitializer.class)
        public class MyServletContainerInitializer implements ServletContainerInitializer 
        {
            @Override
            public void onStartup(Set<Class<?>> classes, ServletContext servletContext) throws ServletException 
            {
                for (Class<?> myAppInitClass : classes) {
                    try {
                        MyAppInitializer initializer = (MyAppInitializer) myAppInitClass.getDeclaredConstructor.newInstance();
                        initializer.onStartup(servletContext);
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                }
            }
        }
        ```
    - MyAppInitializer 인터페이스 작성 및 구현
        ```java
        public interface MyAppInitializer
        {
            void onStartUp(ServletContext servletContext);
        }
        ```
        ```java
        public class MyAppInitializerV1 implements MyAppInitializer
        {
            @Override    
            public void onStartUp(ServletContext servletContext) 
            {
                // setInitParamater와 setAtturibute는 용도와 동작 방식 측면에서 여러 차이가 있음
                    // setInitParameter()
                        // - 설정 후 변경 불가 -> 이미 해당 초기화 파라미터가 설정되있으면 실패함  
                        // - 값은 문자열 형식
                        // - 설정은 주로 초기화 시에 한 번만 수행  
                        // - 웹 애플리케이션의 전역 설정에 주로 사용  
                        // - web.xml의 context-param 태그와 유사함 -> 읽기 전용  
                    // setAttribute()
                        // - 설정, 변경, 제거 가능  
                        // - 값은 객체 형식
                        // - 런타임 동안 동적으로 관리  
                        // - 애플리케이션 컴포넌트 간 데이터 공유 및 상태 저장에 주로 사용   

                // ServletContext 초기화 파라미터 설정
                servletContext.setInitParameter("contextParamName_example", "contextParamValue_example");

                // 서블릿 등록
                ServletRegistration.Dynamic exampleServlet = servletContext.addServlet("exampleServlet", new ExampleServlet());

                exampleServlet.addMapping("/example");
            }
        }
        ```

    - /META-INF/services/jakarta.servlet.ServletContainerInitializer 파일 생성 후 아래 내용 작성
        ```bash
com.example.MyServletContainerInitializer
        ```  

- => <span style="color:red">**ServletContainerInitializer, web.xml의 핵심은 결국 ServletContext 객체를 초기화하는 것**</span>   
    - ServletContainerInitializer 방식의 코드(MyAppInitializerV1.class)에서 서블릿을 등록하는 부분인 servletContext.addServlet()를 보면, ServletContext 객체에 서블릿을 등록하는 형태로 이루어진다. <span style="color:red">즉, 서블릿을 ServletContext 객체에 등록하는 것이 서블릿 등록의 본질이라고 볼 수 있다.</span><span style="color:red">**마찬가지로, ServletContext 초기화 파라미터 설정은 물론, 필터나 리스너 등록 등의 작업도 결국은 ServletContext 객체에 등록하는 작업이다.**</span>  
    - web.xml 방식에서도 서블릿, 필터, 리스너, ServletContext 초기화 파라미터 등의 설정을 작성해두면, 해당 설정들이 ServletContext 객체에 등록된다. 따라서, ServletContainerInitializer와 web.xml은 ServletContext 객체를 초기화하는 작업을 수행하는 방식이 다를 뿐, 결과적으로 두 방식 모두 ServletContext 객체를 초기화한다.

<br>
## ServletContext 객체 초기화 2 - 서블릿에 대한 추가 설정
1. **web.xml 방식**
    - web.xml 작성
        ```xml
        <servlet>
            <servlet-name>exampleServlet</servlet-name>
            <servlet-class>com.example.ExampleServlet</servlet-class>

            <!-- 서블릿 초기화 파라미터 설정 -->
            <init-param>
                <param-name>servletParamName_example</param-name>
                <param-value>servletParamValue_example</param-value>
            </init-param>

            <!-- 서블릿 초기화 시점 설정 -->
            <load-on-startup>1</load-on-startup>
        </servlet>
        
        <!-- 서블릿 매핑 -->
        <servlet-mapping>
            <servlet-name>exampleServlet</servlet-name>
            <url-pattern>/example</url-pattern>
        </servlet-mapping>
        ```

2. **ServletContainerInitializer 방식**
    - MyAppInitializer 구현
        ```java
        public class MyAppInitializerV1 implements MyAppInitializer
        {
            @Override    
            public void onStartUp(ServletContext servletContext) 
            {
                ServletRegistration.Dynamic exampleServlet = servletContext.addServlet("exampleServlet", new ExampleServlet());

                // 서블릿 초기화 파라미터 설정
                exampleServlet.setInitParameter("servletParamName_example", "servletParamValue_example");

                // 서블릿 초기화 시점 설정
                exampleServlet.setLoadOnStartup(1);

                // 서블릿 매핑
                exampleServlet.addMapping("/example");
            }
        }
        ```        

- => <span style="color:red">**서블릿 초기화 파라미터 설정, 서블릿 매핑 등의 서블릿에 대한 추가 설정 작업도 ServletContext 객체를 초기화하는 작업의 일부로 간주**</span>
    - ServletContainerInitializer 방식의 코드(MyAppInitializerV1.class)를 보면, ServletContext 객체에 서블릿을 등록할 때 <span style="color:red">addServlet() 메소드가 사용되고 이 메소드는 ServletRegistration.Dynamic 객체를 반환한다. **즉, ServletRegistration.Dynamic 객체는 ServletContext를 통해 반환되며, 반환된 ServletRegistration.Dynamic 객체를 통해 서블릿에 대한 추가 설정을 수행할 수 있다.**</span>
    - ServletContext의 getServletRegistration() 메소드를 사용하여 등록된 서블릿의 정보를 가져올 수 있다.   
<br>
<br>
**3. @WebServlet 어노테이션 방식**     
    - 서블릿 클래스 생성(@WebServlet)
        ```java
        @WebServlet(name = "exampleServlet", 
                    urlPatterns = "/example",
                    initParams = {@WebInitParam(name = "servletParamName_example", value = "servletParamValue_example")},
                    loadOnStartup = 1)
        public class ExampleServlet extends HttpServlet 
        {
            @Override
            protected void service(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException
                {
                    System.out.println("service 메소드 호출");
                }    
        }
        ```
    - @WebServlet을 사용하면 web.xml이나 ServletContainerInitializer 없이 서블릿 등록 및 매핑, 서블릿 초기화 파라미터 설정 등 서블릿에 대한 설정을 할 수 있다. 
        - @WebServlet은 <span style="color:blueviolet">개별 서블릿에 대한 간편한 설정을 제공</span>하지만, 리스너 설정이나 ServletContext 객체의 초기화 파라미터 설정 등 <span style="color:blueviolet">복잡한 설정이나 웹 애플리케이션 전체 설정이 필요한 경우 web.xml이나 ServletContainerInitializer가 필요</span>하다.

# ServletContextListener
```java
public interface ServletContextListener extends EventListener 
{
    default void contextInitialized(ServletContextEvent sce) 
    {
        // 웹 애플리케이션이 초기화(시작)될 때 호출 
            // 필터나 서블릿이 초기화되기 전에 호출됨
    }

    default void contextDestroyed(ServletContextEvent sce) 
    {
        // 웹 애플리케이션이 종료될 때 호출
            // 모든 필터와 서블릿이 파괴된 후에 호출됨
    }
}
```
- ServletContextListener 인터페이스를 구현한 클래스는 <span style="color:red">웹 애플리케이션의 ServletContext가 변경될 때 알림을 받는다. </span>
    - ServletContextListener는 <span style="color:blueviolet">ServletContext의 시작 및 종료 이벤트를 처리할 수 있으며, 웹 애플리케이션의 초기화(시작) 및 종료 시에 특정 작업을 수행하기 위해 사용</span>된다.  
    - ServletContextListener는 주로 웹 애플리케이션 초기화 및 종료 시 필요한 설정 작업을 수행한다. 데이터베이스 연결을 설정하거나, 로깅을 구성하는 등의 다양한 작업을 포함한다.    
<br>
- **ServletContextListener 예시**
    > - ServletContextListener를 구현한 클래스는 <span style="color:blueviolet">web.xml 설정, @WebListener 어노테이션, ServletContext의 addListener() 메소드를 통해 등록</span>한다.
    1. **web.xml 방식**
        - ServletContextListener 인터페이스 구현 클래스 생성
            ```java
            public class MyServletContextListener implements ServletContextListener 
            {
                @Override
                public void contextInitialized(ServletContextEvent sce) 
                {
                    // JDBC 드라이버 로드
                    Class.forName("com.mysql.cj.jdbc.Driver");

                    // 데이터베이스 연결 설정
                    String dbUrl = sce.getServletContext().getInitParameter("DB_URL");
                    Connection connection = DriverManager.getConnection(dbUrl, "username", "password");

                    // ServletContext에 연결 객체 저장
                    sce.getServletContext().setAttribute("DBConnection", connection);
                }

                @Override
                public void contextDestroyed(ServletContextEvent sce) 
                {
                    connection.close();
                }
            }
            ```
        - web.xml 작성
            ```xml
            <listener>
                <listener-class>com.example.MyServletContextListener</listener-class>
            </listener>
            ```
    2. **어노테이션 방식**
        - ServletContextListener 인터페이스 구현 클래스 생성(@WebListener)  
            ```java
            @WebListener
            public class MyServletContextListener implements ServletContextListener 
            {
                // 위와 동일
            }
            ```
    3. **addListener() 방식 - ServletContainerInitializer**
        - ServletContextListener 인터페이스 구현 클래스 생성  
            ```java
            public class MyServletContextListener implements ServletContextListener 
            {
                // 위와 동일
            }
            ```
        - MyAppInitializer 구현
            ```java
            public class MyAppInitializerV1 implements MyAppInitializer
            {
                @Override    
                public void onStartUp(ServletContext servletContext) 
                {
                    // ServletContext를 통해 리스너 등록
                    servletContext.addListener(new MyServletContextListener());
                }
            }
            ```  

# ServletContainerInitializer와 ServletContextListener
- **ServletContainerInitializer VS ServletContextListener**
    - 목적
        - ServletContainerInitializer : 서블릿 및 필터 등록 및 매핑, 리스너 등록 등  
        - ServletContextListener : 데이터베이스 연결 설정 및 해제, 로그 설정 등  
    -  호출 시점
        - ServletContainerInitializer : 서블릿 컨테이너 시작 시 호출
        - ServletContextListener : 웹 애플리케이션 초기화(시작) 및 종료 시 호출  
<br>
- <span style="color:red">**ServletContainerInitializer의 onStartUp 메소드와 ServletContextListener의 contextInitialized 메소드 호출 순서**</span>
    1. 서블릿 컨테이너가 시작되면 서블릿 컨테이너는 ServletContainerInitializer 구현 클래스의 onStartUp(ServletContext servletContext) 메소드를 호출
        - <span style="color:blueviolet">onStartUp 메소드에 전달되는 ServletContext 객체는 기본적인 초기화가 완료된 상태이며, onStartUp 메소드를 통해 추가적인 초기화 작업(서블릿, 필터, 리스너 등록 등)을 수행</span>한다.
    2. onStartUp 메소드의 실행이 끝난 후, 서블릿 컨테이너는 등록된 모든 ServletContextListener 구현 클래스의 contextInitialized 메소드를 호출
        - 즉, ServletContextListener의 contextInitialized 메소드가 호출되는 시점은 <span style="color:red">ServletContainerInitializer나 web.xml을 통해 ServletContext 객체가 초기화된 직후</span>이다.


---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
https://tomcat.apache.org/tomcat-7.0-doc/servletapi/javax/servlet/ServletContainerInitializer.html  
https://tomcat.apache.org/tomcat-8.0-doc/servletapi/javax/servlet/ServletContext.html  
https://tomcat.apache.org/tomcat-7.0-doc/servletapi/javax/servlet/ServletRegistration.Dynamic.html  
https://tomcat.apache.org/tomcat-8.5-doc/servletapi/javax/servlet/ServletContextListener.html  
https://docs.oracle.com/javaee%2F6%2Fapi%2F%2F/javax/servlet/ServletContextListener.html  
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/SpringServletContainerInitializer.html  
https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/WebApplicationInitializer.html  
https://kimcoder.tistory.com/511  
https://recordsoflife.tistory.com/490  
https://escapefromcoding.tistory.com/174  
https://offbyone.tistory.com/215  
https://nhs0912.tistory.com/81  
https://blog.naver.com/kitepc/221314687808  
https://velog.io/@rolroralra/Chapter0.-Servlet-컨테이너-Spring-컨테이너   
https://amy-it.tistory.com/86  
https://erjuer.tistory.com/20  
https://scshim.tistory.com/399  
</div>
</details>