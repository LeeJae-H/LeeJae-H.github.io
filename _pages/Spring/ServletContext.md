---
title: "(Tomcat) - Servlet Container (feat. ServletContext, ServletConfig)"
tags:
    - tomcat
date: "2024-05-06"
thumbnail: "/assets/img/thumbnail/sample.png"
---
# 서블릿 컨테이너와 웹 애플리케이션
<img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/2a3117e0-853e-46d4-b1c2-edec22a1b10f" style="width:500px;" />
- 서블릿 컨테이너는 웹 애플리케이션을 실행하고 관리하는 데 사용된다.
- 서블릿 컨테이너는 여러 개의 웹 애플리케이션을 호스팅할 수 있으며, 웹 애플리케이션은 서블릿 컨테이너 내에서 독립적으로 실행된다. 

# ServletContext
```java
public interface ServletContext {
    ...
    String getInitParameter(String var1);
    Enumeration<String> getInitParameterNames();
    ...
    URL getResource(String var1) throws MalformedURLException;
    InputStream getResourceAsStream(String var1);
    ...
    Object getAttribute(String var1);
    void setAttribute(String var1, Object var2);
    void removeAttribute(String var1);
    ...
    RequestDispatcher getRequestDispatcher(String var1);
    ...
    void log(String var1);
    void log(String var1, Throwable var2);
    ...
}
```
- **ServletContext 인터페이스**는 서블릿이 서블릿 컨테이너와 통신하는 데 사용되는 메소드들을 정의한다.
    - ServletContext 인터페이스를 구현한 클래스는 ApplicationContext(Facade) 클래스이다. (org.apache.catalina.core.ApplicationContext)  
    - <span style="color:red">ServletContext를 **서블릿 컨테이너**라고 볼 수 있다.</span>  
<br>
- ServletContext 객체는 <span style="color:red">서블릿 컨테이너가 시작될 때</span> 서블릿 컨테이너에 의해 <span style="color:blueviolet">각 웹 애플리케이션마다 한 개</span> 생성된다. 서블릿 컨테이너가 종료될 때 소멸된다. 
    - <span style="color:red">ServletContext 객체는 웹 애플리케이션 전체의 자원 및 설정에 대한 정보를 제공한다.</span>  
<br>
- 서블릿은 ServletContext 객체를 통해 서블릿 컨테이너와 통신하여 다양한 작업(파일 접근, 자원 바인딩, 로그 파일 쓰기 등)을 수행할 수 있다.
    - 서블릿이 초기화될 때 서블릿 컨테이너는 해당 서블릿에 대한 초기화 정보를 담은 ServletConfig 객체를 생성하여 서블릿에게 제공하는데, <span style="color:red">서블릿은 ServletConfig의 getServletContext() 메소드를 통해 ServletContext 객체를 얻을 수 있다.</span>  

# ServletConfig
```java
public interface ServletConfig {
    String getServletName();
    String getInitParameter(String var1);
    Enumeration<String> getInitParameterNames();
    ServletContext getServletContext();
}
```
- <span style="color:blueviolet">ServletConfig 인터페이스는 서블릿 컨테이너가 서블릿을 초기화할 때 서블릿에게 전달하는 정보에 대한 메소드들을 정의한다.</span>
    - ServletContext 인터페이스를 구현한 클래스는 StandardWrapper(Facade) 클래스이다. (org.apache.catalina.core.StandardWrapper)  
<br>    
- ServletConfig 객체는 <span style="color:red">서블릿 컨테이너가 서블릿을 초기화할 때</span> 서블릿 컨테이너에 의해 <span style="color:blueviolet">각 서블릿마다 한 개</span> 생성된다. 서블릿이 소멸될 때 함께 소멸된다. 
    - <span style="color:red">ServletConfig 객체는 ServletContext에 대한 참조와 서블릿에 대한 초기화 매개변수 등을 제공한다.</span>

> - <span style="color:red">서블릿 초기화 시점</span>
    1. 서블릿 컨테이너가 실행될 때 초기화 ('load-on-startup' 속성이 설정된 경우)
    2. 최초 HTTP 요청이 들어올 때 초기화 ('load-on-startup' 속성이 설정되지 않은 경우)

> - 서블릿 초기화 과정
    1. 서블릿 컨테이너가 서블릿 클래스를 로드
    2. 서블릿 컨테이너가 서블릿 클래스의 인스턴스를 생성
    3. <span style="color:red">서블릿 컨테이너가 ServletConfig 객체를 생성</span>
    4. 서블릿 컨테이너가 서블릿 인스턴스의 init(ServletConfig config) 메소드를 호출 

# 서블릿의 초기화 메소드 (GenericServlet)
```java
// HttpServlet 클래스의 부모 클래스인 GenericServlet 클래스
public abstract class GenericServlet implements Servlet, ServletConfig, Serializable 
{
    ...
    private transient ServletConfig config;

    // 서블릿 컨테이너가 먼저 호출하는 메소드
    public void init(ServletConfig config) throws ServletException 
    {
        // 전달받은 ServletConfig 객체를 현재 서블릿 인스턴스의 인스턴스 변수 config에 저장
        this.config = config;
        // init() 메소드 호출
        this.init();
    }

    // 기본 구현으로 아무 작업도 하지않으며, 서블릿에서 필요에 따라 Override
    public void init() throws ServletException { }
    ...

    public ServletConfig getServletConfig() 
    { return this.config; }

    public ServletContext getServletContext() 
    { 
        // ServletConfig의 getServletContext() 메소드
        return this.getServletConfig().getServletContext(); 
    }
    ...
}
```
- 위와 같이 GenericServlet 클래스를 보면, <span style="color:blueviolet">서블릿의 초기화 메소드는 init(ServletConfig config), init() 두 가지</span>가 있다. 
    - 서블릿 컨테이너는 서블릿을 초기화할 때 <span style="color:red">**init(ServletConfig config) 메소드를 먼저 호출**</span>한다.  
<br>
- **서블릿에서 초기화 메소드 Override**
    1. 두 가지 초기화 메소드를 모두 Override 하지 않음
        - 서블릿 컨테이너가 서블릿을 초기화할 때, GenericServlet의 init(ServletConfig config)을 먼저 호출하고 해당 메소드에서 GenericServlet의 init()을 호출함
    2. init()을 Override
        - 서블릿 컨테이너가 서블릿을 초기화할 때, GenericServlet의 init(ServletConfig config)을 먼저 호출하고 해당 메소드에서 서블릿이 Override한 init()을 호출함
    3. init(ServletConfig config)를 Override
        - 서블릿 컨테이너가 서블릿을 초기화할 때, 서블릿에서 Override한 init(ServletConfig config)을 호출함
        - <span style="color:red">이 경우, init(ServletConfig config)에서 super.init(config)를 작성하지 않으면 해당 서블릿의 인스턴스에서는 ServletConfig 객체가 GenericServlet의 private 변수 config에 저장되지 않으며, ServletConfig 객체가 변수에 저장되지 않으면 서블릿에서는 ServletContext 객체에 접근할 수가 없다.</span>

# ServletContext, ServletConfig 예시
- **서블릿에서 초기화 메소드를 Override하지 않았을 때, ServletConfig, ServletContext 객체를 가져오고 사용하기**
    ```java
    public class ExampleServlet extends HttpServlet 
    {
        @Override
        protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException 
        {
            // ServletConfig 객체 가져오기 
            // GenericServlet의 getServletConfig() 메소드
            SerlvetConfig config = getServletConfig();

            // ServletContext 객체 가져오기
            // GenericServlet의 getServletContext() 메소드
            ServletContext context = getServletContext();
            
            // 서블릿의 초기화 매개변수 가져오기
            String configParamValue = 
                    config.getInitParameter("configParamName");

            // 웹 애플리케이션의 초기화 매개변수 가져오기
            String contextParamValue = 
                    context.getInitParameter("contextParamName");
        }
    }

    ```

---
- 참고 자료
https://tomcat.apache.org/tomcat-8.0-doc/servletapi/javax/servlet/ServletContext.html  
https://tomcat.apache.org/tomcat-8.0-doc/servletapi/javax/servlet/ServletConfig.html  
https://javaee.github.io/javaee-spec/javadocs/javax/servlet/Servlet.html  
https://javaee.github.io/javaee-spec/javadocs/javax/servlet/ServletConfig.html  
https://javaee.github.io/javaee-spec/javadocs/javax/servlet/ServletContext.html  
https://vibeee.tistory.com/140  
https://ckddn9496.tistory.com/47  
https://velog.io/@cocodori/ServletContext-ServletConfig  
https://kgvovc.tistory.com/38  
https://blog.naver.com/crint/90068104505  
https://codevang.tistory.com/193  
