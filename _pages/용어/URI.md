---
title: "URI"
tags:
    - web
date: "2024-02-18"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# URI란?
- **URI(Uniform Resource Identifier)** : *인터넷에 있는 자원을 나타내는 유일한 주소*  
    - URI는 자원의 위치(URL)뿐만 아니라 자원의 고유한 이름(URN)을 모두 포함하는 개념이다. 
    - 즉, URL과 URN은 URI의 유형이다.<span style="color:blueviolet">(URI ⊃ URL, URN)</span>  

> - <span style="color:blueviolet">웹에서 사용되는 URI은 대부분 URL이다.</span>

# URL이란?
- **URL(Uniform Resource Locator)** : <span style="color:green">*웹 페이지를 포함한 다양한 유형의 웹 자원의 위치 또는 해당 자원에 대한 특정 작업을 식별하는 주소*</span>  
<br>
- **URL 구조** : **scheme://host:port/path?query#fragment**
    - **scheme**
        - 자원에 접근하기 위해 사용되는 프로토콜  
            -> 주로 HTTP나 HTTPS 사용  
    <br>
    - **host**
        - 자원이 호스팅 되는 서버(일반적으로 웹 서버)의 도메인 네임 또는 IP 주소  
    <br>
    - (port)
        - 서버(일반적으로 웹 서버)에 접속하기 위한 통로  
            -> 생략되면 기본 포트로 연결됨    
            -> 기본적으로 HTTP의 경우 80 포트, HTTPS의 경우 443 포트를 사용  
    <br>
    - **path**
        - 서버(일반적으로 웹 서버) 내에서 자원의 경로    
            -> 파일 시스템의 경로와 유사  
    <br>
    - (query)  
        - 요청에 대한 추가적인 매개변수  
            -> 서버(일반적으로 웹 서버)에 전달하는 추가 질문으로, 일반적으로 key-value 형식  
    <br>
    - (fragment)
        - 웹 페이지 내의 특정 위치
            - fragment는 웹 서버에 전송되는 요청에는 포함되지 않으며, 클라이언트 측에서만 처리됨   
<br>  

> - 특정 브라우저에서는 URL을 사용하여 로컬 파일 시스템의 폴더와 파일을 탐색할 수 있다.      
- ex) Chrome에서 OS에 맞게 파일 경로를 입력하면, 로컬 파일 시스템의 해당 위치에 있는 폴더와 파일을 나타낸다.  
        Linux : file://127.0.0.1/home/username  
        Windows : file://localhost/C:\Users/username\  
        MacOS : file://127.0.0.1/Users/username
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/d60da152-e0ce-4393-b77d-11d56190705c" width="400" height="300"></center>  

# URN이란?
- **URN(Uniform Resource Name)** : *자원의 고유한 이름*
    - URN은 이름이 변하지 않는 한, 자원의 위치가 변경되더라도 문제 없이 동작한다. 이는 자원의 위치와 상관없이 이름만으로 식별할 수 있다는 개념이다. 
    - 즉, 자원의 위치가 아닌 자원의 이름에 중점을 둔다.   
<br>
- **URN 구조 : urn:namespace:specific-string**
    - **urn**
        - URN이라는 접두어를 나타낸다.  
    <br>
    - **namespace**
        - URN의 유형을 식별하는 데 사용되는 네임스페이스 식별자이다.  
    <br>
    - **specific-string**
        - 네임스페이스 내에서 자원을 고유하게 식별하는데 사용되는 특정 문자열이나 이름이다.  

---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
https://ko.wikipedia.org/wiki/%ED%86%B5%ED%95%A9_%EC%9E%90%EC%9B%90_%EC%8B%9D%EB%B3%84%EC%9E%90  
https://ko.wikipedia.org/wiki/URL  
https://hstory0208.tistory.com/entry/URI와-URL-비슷해보이는데-차이점이-뭘까-완벽-정리  
https://developer.mozilla.org/ko/docs/Learn/Common_questions/Web_mechanics/What_is_a_URL  
https://ittrue.tistory.com/186  
chatgpt  
bard
</div>
</details>
