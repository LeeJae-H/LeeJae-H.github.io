---
title: "(Spring) - Spring MVC 요청 처리 과정 (+Interceptor)"
tags:
    - spring
date: "2024-05-01"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# Spring MVC 요청 처리 과정 (인터셉터 X)
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/61cd5d4e-50e1-4592-9a75-543107f64ed7" width="700" height="330"></center>
1. Client가 Web Server로 HTTP Request 보냄  
    
2. 정적 콘텐츠의 경우 Web Server에서 처리, 동적 콘텐츠의 경우 Web Server가 Servlet Container로 요청 전달<span style="color:red"> - (1)</span>  

3. 요청을 받은 Servlet Container는 HttpServletRequest, HttpServletResponse 객체를 생성

4. Servlet Container는 요청을 처리할 Thread를 Thread Pool에서 가져옴

5. Thread는 DispatcherServlet의 doDispatch() 메소드 호출<span style="color:blueviolet">(doDispatch() 메소드를 호출하기까지의 과정은 생략)</span><span style="color:red"> - (2)</span>  
    ```java
    protected void doDispatch(HttpServletRequest request, 
                              HttpServletResponse response) 
                              throws Exception 
        {
            ...
            // 5-1. 핸들러 조회 - (3)
            // 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러) 조회
            mappedHandler = this.getHandler(processedRequest);
            if (mappedHandler == null) {
                this.noHandlerFound(processedRequest, response);
                return;
            }

            ...
            // 5-2. 핸들러 어댑터 조회 
            // 핸들러를 실행할 수 있는 핸들러 어댑터 조회
            HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
            
            ...
            // 5-3. 핸들러 어댑터 실행 - (4)
            // 핸들러 어댑터가 핸들러 실행
                // 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            
            ...
            this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
        }

    private void processDispatchResult(HttpServletRequest request, 
                                       HttpServletResponse response, 
                                       @Nullable HandlerExecutionChain mappedHandler, 
                                       @Nullable ModelAndView mv, 
                                       @Nullable Exception exception) 
                                       throws Exception 
        {
            ...
            this.render(mv, request, response);
            ...
        }
    
    protected void render(ModelAndView mv, 
                          HttpServletRequest request, 
                          HttpServletResponse response) 
                          throws Exception 
        {
            String viewName = mv.getViewName();
            View view;

            ...
            // 5-4. 뷰 찾기 - (5)
            // 뷰 리졸버를 통해 뷰 찾기
                // 뷰 리졸버는 View 반환
            view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);

            ...
            // 5-5. 뷰 렌더링
            // HTML 등의 응답 생성
                // 주로 템플릿 엔진을 통해 동적으로 생성
            view.render(mv.getModelInternal(), request, response);
        }
    ```
    5-6. 응답 값을 HttpServletResponse 객체에 담음

6. Servlet Container는 응답 값을 HTTP Response로 바꾼 후 Web Server로 전송

7. HttpServletRequest, HttpServletResponse 객체를 소멸시키고 Thread를 종료

# Spring MVC 요청 처리 과정 (인터셉터 O)
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/3d064f77-b9fe-4890-8435-758f00346f1b" width="700" height="330"></center>
1. Client가 Web Server로 HTTP Request 보냄  
    
2. 정적 콘텐츠의 경우 Web Server에서 처리, 동적 콘텐츠의 경우 Web Server가 Servlet Container로 요청 전달<span style="color:red"> - (1)</span>  

3. 요청을 받은 Servlet Container는 HttpServletRequest, HttpServletResponse 객체를 생성

4. Servlet Container는 요청을 처리할 Thread를 Thread Pool에서 가져옴

5. Thread는 DispatcherServlet의 doDispatch() 메소드 호출<span style="color:blueviolet">(doDispatch() 메소드를 호출하기까지의 과정은 생략)</span><span style="color:red"> - (2)</span>  
    ```java
    protected void doDispatch(HttpServletRequest request, 
                              HttpServletResponse response) 
                              throws Exception 
        {
            ...
            // 5-1. 핸들러 조회 - (3)
            // 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러) 조회
            mappedHandler = this.getHandler(processedRequest);
            if (mappedHandler == null) {
                this.noHandlerFound(processedRequest, response);
                return;
            }

            ...
            // 5-2. 핸들러 어댑터 조회 
            // 핸들러를 실행할 수 있는 핸들러 어댑터 조회
            HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
            
            ...
            // 5-3. 인터셉터 호출(핸들러 어댑터 호출 전) - (4)
            // applyPreHandle() 메소드에서 preHandle() 메소드 호출
                // preHandle()의 반환 값이 true : 다음으로 진행함
                // preHandle()의 반환 값이 false : 다음으로 진행하지 않음
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
            }

            ...
            // 5-4. 핸들러 어댑터 실행 - (5)
            // 핸들러 어댑터가 핸들러 실행
                // 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            
            ...
            // 5-5. 인터셉터 호출(핸들러 어댑터 호출 후) - (6)
            // applyPostHandle() 메소드에서 postHandle() 메소드 호출
            mappedHandler.applyPostHandle(processedRequest, response, mv);

            ...
            this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
        }

    private void processDispatchResult(HttpServletRequest request, 
                                       HttpServletResponse response, 
                                       @Nullable HandlerExecutionChain mappedHandler, 
                                       @Nullable ModelAndView mv, 
                                       @Nullable Exception exception) 
                                       throws Exception 
        {
            ...
            this.render(mv, request, response);

            ...
            // 5-8. 인터셉터 호출(뷰 렌더링 후) - (8)
            // triggerAfterCompletion() 메소드에서 afterCompletion() 메소드 호출            
            if (mappedHandler != null) {
                mappedHandler.triggerAfterCompletion(request, response, (Exception)null);
            }   
        }
    
    protected void render(ModelAndView mv, 
                          HttpServletRequest request, 
                          HttpServletResponse response) 
                          throws Exception 
        {
            String viewName = mv.getViewName();
            View view;

            ...
            // 5-6. 뷰 찾기 - (7)
            // 뷰 리졸버를 통해 뷰 찾기
                // 뷰 리졸버는 View 반환
            view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);

            ...
            // 5-7. 뷰 렌더링
            // HTML 등의 응답 생성
                // 주로 템플릿 엔진을 통해 동적으로 생성
            view.render(mv.getModelInternal(), request, response);
        }
    ```
    5-9. 응답 값을 HttpServletResponse 객체에 담음

6. Servlet Container는 응답 값을 HTTP Response로 바꾼 후 Web Server로 전송

7. HttpServletRequest, HttpServletResponse 객체를 소멸시키고 Thread를 종료

---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/context-hierarchy.html#page-title  
https://codevang.tistory.com/248  
https://gowoonsori.com/blog/spring/architecture/    
https://velog.io/@suhongkim98/DispatcherServlet%EA%B3%BC-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88-%EC%83%9D%EC%84%B1-%EA%B3%BC%EC%A0%95  
https://jaeseo.tistory.com/entry/DispatcherServlet-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95%EA%B3%BC-Special-Bean-Types  
https://jeonyoungho.github.io/posts/Spring-MVC-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95/  
https://jaeseo.tistory.com/entry/DispatcherServlet-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95%EA%B3%BC-Special-Bean-Types  
https://yoonbing9.tistory.com/80  
https://velog.io/@seculoper235/3  
https://appleg1226.tistory.com/31  
</div>
</details>
