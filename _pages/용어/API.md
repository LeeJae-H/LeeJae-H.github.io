---
title: "API"
tags:
    - web
date: "2024-02-29"
thumbnail: "/assets/img/thumbnail/sample.png"
---

>- **API**는 <span style="color:green">"소프트웨어 간의 인터페이스 또는 소프트웨어와 하드웨어 간의 인터페이스"</span>라고 말할 수 있다. 

>- <span style="color:green">"소프트웨어 간의 인터페이스"</span> 의미에 초점을 두고 **API**를 좀 더 구체적으로 얘기해보자면 다음과 같다.   
    - A 소프트웨어가 B 소프트웨어의 다양한 기능 중 하나를 이용하려고 할 때, A 소프트웨어가 B 소프트웨어에 어떻게 접근하는 지, 어떤 정보를 전달하고 어떤 정보를 받는 지 등의 <span style="color:red"> 통신 방법 집합</span>을 **API**라고 하는 것이다. 그리고 API에 대해 기술하는 문서나 표준, 즉 설명서를 "**API 문서(사양, 규격)**"라고 한다. 

# API란?
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/01f71d52-4454-4ea8-878b-c47bf2936d3d" width="640" height="300"></center>
- **API(Application Programming Interface)** : <span style="color:green">*어떤 소프트웨어의 기능을 사용할 수 있도록 해주는 인터페이스*</span>   
    - 클라이언트는 API를 통해 <span style="color:red">다른 소프트웨어(라이브러리, 외부 서비스, 운영체제 등)에서 제공하는 기능을 사용</span>할 수 있다.  

> - **API 제공 예시 1 : 라이브러리 API**  
    -> 라이브러리는 기능을 호출하고 사용할 수 있도록 해주는 API를 제공한다.    
    -> 클라이언트에서 <span style="color:red">라이브러리에 실질적으로 접근하는 방법은 함수 호출을 통한 접근</span>이다.  
    ex) Mongoose, Numpy, Pandas 등  
<br>   
> - **API 제공 예시 2 : 외부 서비스 API**  
    -> 외부 서비스는 기능을 호출하고 사용할 수 있도록 해주는 API를 제공한다.    
    -> 클라이언트에서 <span style="color:red">외부 서비스에 실질적으로 접근하는 방법은 대부분 URL(엔드포인트)과 HTTP 요청을 통한 접근</span>이다.  
        ex) Google Maps API, Twitter API 등  
<br>
> - **API 제공 예시 3 : 애플리케이션 서버 API**  
    -> 애플리케이션 서버는 기능을 호출하고 사용할 수 있도록 해주는 API를 제공한다.    
    -> 클라이언트에서 <span style="color:red">애플리케이션 서버에 실질적으로 접근하는 방법은 대부분 URL(엔드포인트)과 HTTP 요청을 통한 접근</span>이다.  
        ex) 데이터 저장, 검색 등의 비즈니스 로직 => 애플리케이션(모바일, 웹)에서 사용할 백엔드 기능

- **Web API** : <span style="color:green">*웹 기술을 기반으로 한 API*</span>
    - ex) HTTP API, WebSocket API 등  
<br>
- **HTTP API** : *<span style="color:green">HTTP 프로토콜을 사용하여 통신하는 API</span>*
    - 다양한 클라이언트에서 HTTP API를 호출하여 <span style="color:red">데이터(주로 JSON 형식)만</span> 주고 받는다. UI 화면이 필요하면 클라이언트가 별도로 처리한다.
        - 앱 클라이언트(모바일 앱, PC 앱)
        - 웹 클라이언트(React, Vue.js 등)
        - 웹 브라우저에서 자바스크립트를 통한 HTTP API 호출
        - 서버에서 HTTP API 호출
    - 외부 서비스에서 제공하는 API, 애플리케이션 서버에서 제공하는 API 등 <span style="color:blueviolet">우리가 사용하는 API는 대부분 HTTP API</span>이다.  

---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
https://ko.wikipedia.org/wiki/API  
https://en.wikipedia.org/wiki/API  
https://aws.amazon.com/ko/what-is/restful-api/  
https://www.ibm.com/kr-ko/topics/api 
https://ko.wikipedia.org/wiki/%EC%9B%B9_API  
https://www.guru99.com/ko/what-is-api.html  
https://www.quora.com/What-is-a-Web-API  
https://www.quora.com/Is-an-API-just-a-library  
https://www.inforad.co.kr/single-post/api-library  
chatgpt  
bard  
</div>
</details>
