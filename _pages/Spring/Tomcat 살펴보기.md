---
title: "(Tomcat) - Tomcat 살펴보기"
tags:
    - tomcat
date: "2024-04-30"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# Tomcat 설치 (Ubuntu)
- **다운로드**  
```shell 
~$ wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.89/bin/apache-tomcat-9.0.89.tar.gz
```  

- **tar.gz 압축 해제**
```shell
~$ tar -zxvf apache-tomcat-9.0.89.tar.gz
```

- **다운로드 완료**
    <img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/665848f1-c6ce-47f9-8c5a-06033c3d328d" style="width:600px;" />

- **Tomcat 시작**
```shell
~/apache-tomcat-9.0.89/bin$ ./startup.sh
```

- **http://localhost:8080 에서 확인**
    <img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/c4053f01-7a02-43ae-89fb-e50fcd59b13c" style="width:600px;" />

- **Tomcat 중지**
```shell
~/apache-tomcat-9.0.89/bin$ ./shutdown.sh
```

# Tomcat 디렉토리 구조
<img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/eba5713c-4a43-4d79-af2d-115b6d2dc388" style="width:450px;" />
- **bin** : <span style="color:green">*Tomcat의 시작, 중지 및 기타 관리 스크립트가 포함된 디렉토리*</span>
    - 주요 파일
        - startup.sh : Tomcat 시작 스크립트
        - shutdown.sh : Tomcat 중지 스크립트
        - catalina.sh : 주요 제어 스크립트  
<br>
- **conf** : <span style="color:green">*Tomcat의 설정 파일들이 포함된 디렉토리*</span>
    - 주요 파일
        - server.xml : Tomcat의 전반적인 설정 파일
        - web.xml : 모든 웹 애플리케이션에 대한 전역 설정 파일
        - context.xml : 개별 웹 애플리케이션에 대한 설정 파일  
<br>
- **lib** : <span style="color:green">*Tomcat을 작동하는 데 필요한 라이브러리들(.jar)이 포함된 디렉토리*</span>  
<br>
- **logs** : <span style="color:green">*Tomcat의 로그 파일들이 저장되는 디렉토리*</span>
    - 주요 파일
        - catalina.out : Tomcat의 표준 출력 및 표준 오류 로그 파일  
<br>
- **temp** : <span style="color:green">*Tomcat이 실행 중 사용하는 임시 파일들(ex) 업로드, 캐시 파일)이 저장되는 디렉토리*</span>  
<br>
- <span style="color:red">**webapps**</span> : <span style="color:green">*배포된 웹 애플리케이션들이 위치하는 디렉토리*</span>  

- **work** : <span style="color:green">*JSP 파일을 서블릿 형태로 변환한 .java 파일과 이를 컴파일한 결과인 .class 파일이 저장되는 디렉토리*</span>
    - JSP는 처음 요청될 때 서블릿(.java)으로 변환되고, 이후 재사용을 위해 컴파일된 서블릿(.class)이 저장된다.  

## webapps 디렉토리
- 각 웹 애플리케이션은 webapps 디렉토리 내에 하나의 디렉토리로 존재하며, 각 웹 애플리케이션은 특정한 디렉토리 구조를 따른다.
    - war 파일을 webapps 디렉토리에 배치(배포)하면, Tomcat은 자동으로 war 파일의 압축을 푼다. 
<br>
- **webapps 디렉토리 내 웹 애플리케이션 디렉토리(war 파일 압축 푼 결과)의 구조**
    - **WEB-INF** : <span style="color:green">*웹 애플리케이션의 핵심 설정 파일과 자원을 포함하는 디렉토리*<span>
        - 클라이언트가 직접 접근할 수 없다.
        - 주요 하위 디렉토리와 파일
            - **web.xml** : 웹 애플리케이션의 서블릿, 필터, 리스너, 초기화 매개변수 등의 설정 파일
            - classes 디렉토리 : 웹 애플리케이션의 컴파일된 파일(.class)들이 위치하는 디렉토리 
            - lib 디렉토리 : 웹 애플리케이션이 사용하는 외부 라이브러리 파일(.jar)들이 위치하는 디렉토리  
    - **META-INF** : <span style="color:green">*JAR 파일 및 웹 애플리케이션 메타데이터를 포함하는 디렉토리*</span>   
    - **기타 정적 자원 디렉토리** : <span style="color:green">*정적 파일(css, js 등)들을 포함하는 디렉토리*</span>
        - 클라이언트가 직접 접근할 수 있다.  

## conf/web.xml과 webapps/WEB-INF/web.xml
- **공통점** : 웹 애플리케이션의 설정을 정의
    - 서블릿, 필터, 리스너, 초기화 매개변수 등을 정의한다.  
<br>
- **차이점** : 적용 범위와 목적
    - **conf/web.xml** : Tomcat에서 실행되는 **모든** 웹 애플리케이션에 적용되는 기본 설정
        - 각 개별 웹 애플리케이션의 web.xml 파일에 정의되지 않은 설정은 여기에서 정의된 기본 설정을 따른다. 
        - 기본 서블릿이나 필터 설정을 제공하여, 모든 애플리케이션이 공통적으로 사용해야 하는 서블릿 또는 필터를 정의할 수 있다.
        - 공통적인 보안 설정이나 JNDI 리소스 등의 설정을 정의할 수 있다.
    - **webapps/WEB-INF/web.xml** : **개별** 웹 애플리케이션에만 적용되는 설정
        - conf/web.xml에서 정의된 전역 설정을 덮어쓰거나 추가한다. 
            - 예를 들어, 특정 서블릿이나 필터가 동일한 이름으로 두 파일에 모두 정의되어 있으면, webapps/WEB-INF/web.xml에 정의된 설정이 우선이다.

---
- 참고 자료
https://tomcat.apache.org/tomcat-5.5-doc/appdev/deployment.html  
https://tomcat.apache.org/download-90.cgi    
https://stackoverflow.com/questions/70216/whats-the-purpose-of-meta-inf  
https://www.youtube.com/watch?v=WdBAto3IQOg&list=PLqaSEyuwXkSoeqnsxz0gYWZMihw519Kfr&index=39  
https://www.youtube.com/watch?v=K84mSiC_q6I&list=PLqaSEyuwXkSoeqnsxz0gYWZMihw519Kfr&index=40  
https://www.youtube.com/watch?v=aP4Lw3SfffQ&list=PLqaSEyuwXkSoeqnsxz0gYWZMihw519Kfr&index=6  
https://lifesteps.tistory.com/84   
https://jake-seo-dev.tistory.com/436  
https://velog.io/@xangj0ng/Linux-Ubuntu-Tomcat-%EC%84%A4%EC%B9%98   
https://jokerkwu.tistory.com/117  
https://xzio.tistory.com/1345  
https://infoinhere.tistory.com/85  
https://yangbox.tistory.com/16  
