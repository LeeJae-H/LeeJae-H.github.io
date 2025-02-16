---
title: "Domain Name, DNS"
tags:
    - web
date: "2024-02-22"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# 도메인 네임, DNS란?
- **도메인 네임(Domain Name)** : *IP 주소에 매핑되는 문자열 주소 체계*  
    - 컴퓨터는 IP 주소로 통신하는데, <span style="color:blueviolet">도메인 네임은 서버(대부분 웹 서버)를 쉽게 찾을 수 있도록 해주는 주소</span>라고 할 수 있다. 
    - 도메인 네임은 <span style="color:blueviolet">DNS를 통해 해당하는 IP 주소로 변환</span>되어 웹 브라우저나 다른 응용 프로그램에서 서버를 찾을 수 있게 해준다.
    - 도메인은 영구적으로 소유하는 개념이 아니라, 일정기간 임대하는 개념이다.  
<br>
- **도메인 네임의 계층적 구조 : Subdomain.Domain.TLD**
    - **Subdomain(= Hostname)**
        - Domain의 앞에 오는 <span style="color:blueviolet">추가적인 식별 요소</span>로, 선택적으로 사용된다.  
        -> "www"가 대표적인 Subdomain이며, "blog", "mail", "shop" 등도 사용된다.   
    <br>
    - **Domain(= Second-Level Domain)**
        - TLD 앞에 위치하며, 일반적으로 <span style="color:blueviolet">특정 기업, 조직, 서비스를 식별하는데 사용</span>된다.  
        -> "google"이나 "naver"가 Domain이다.  
    <br>
    - **TLD(Top-Level Domain)**
        - 도메인 이름의 가장 오른쪽에 위치하며, 인터넷에서 <span style="color:blueviolet">특정 종류의 기관, 지역 또는 용도</span>를 나타낸다.  
        -> "com", "org", "net" 등이 대표적인 TLD이며, "kr"(한국), "jp"(일본)과 같은 국가 코드도 TLD에 해당한다.  
 
>- 일반적으로 도메인 네임이라 하면 “Domain.TLD”를 의미한다. 
    - Domain Name : naver.com
 - FQDN(Fully Qualified Domain Name) : *도메인 네임의 전체 이름을 표기하는 방식*
    - FQDN : www.naver.com  

- **DNS(Domain Name System)** : *인터넷에서 도메인 네임을 IP 주소로 변환하는 전세계적인 거대한 <span style="color:green">분산 데이터베이스 시스템</span>*
    - DNS는 <span style="color:blueviolet">여러 DNS Server들이 서로 상호 작용</span>하며 동작한다.  
<br>
- **Domain Name Space** : *<span style="color:green">DNS가 저장 및 관리하는 계층적 구조</span>*      
    <center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/f1edbc3d-2c16-4550-a118-c88f00b7dd6e" width="630" height="400"></center>
    - <span style="color:blueviolet">각 레벨은 그 하위 도메인에 관한 정보를 관리하는 구조를 가진다.</span>   

# DNS 구성 요소
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/0890e992-667c-468e-8979-2b5492dfdefc" width="640" height="340"></center>
#### Stub Resolver ( = DNS Client)
- **Stub Resolver** : <span style="color:green">*클라이언트 기기에 내장된 작은 DNS 클라이언트 소프트웨어*</span>   
    - Stub Resolver는 사용자가 웹 브라우저나 다른 응용 프로그램을 통해 도메인 네임을 입력할 때 동작한다.   
    - Stub Resolver는 수많은 Name Server(Root DNS Server, TLD DNS Server, SLD DNS Server)의 구조를 파악할 필요가 없다.  
<br>
- **Stub Resolver 동작 과정 - 캐시 x**
    1. 사용자가 웹 브라우저에 도메인 네임을 입력하면, 웹 브라우저는 <span style="color:red">사용자의 운영체제에서 설정된 Stub Resolver를 사용</span>한다.  
    -> 운영체제에서 설정된 Stub Resolver는 <span style="color:red">**현재 연결된 ISP의 Local DNS Server(= Resolver)와 통신**</span>한다.  
    <br>
    2. Stub Resolver는 Resolver로 DNS Query를 전송한다.  
    <br>
    3. Stub Resolver는 Resolver로부터 도메인 네임에 대한 IP 주소를 받아 웹 브라우저에 전달한다.  
<br>

## Resolver ( = Local DNS Server  ) => <span style="color:red">ISP가 관리</span>
- **Resolver** : <span style="color:green">*DNS Client로부터 받은 DNS Query를 처리하여 DNS Client에게 IP 주소를 반환하는 서버*</span>  
    - Resolver는 도메인 네임에 대한 IP 주소를 찾기 위해 계층적으로 Name Server들에게 DNS Query를 전송한다.
    - Resolver는 이전에 처리된 DNS Query의 결과를 <span style="color:red">캐시</span>에 저장하여, 동일한 DNS Query에 대한 응답 속도를 향상시킬 수 있다.  
<br>
- **Resolver 동작 과정 - 캐시 x**
    1. DNS Client에서 Resolver에게 DNS Query를 전송한다.  
    <br>
    2. Resolver는 먼저 <span style="color:orange">Root DNS Server</span>에게 DNS Query를 보낸다. 
        - <span style="color:orange">Root DNS Server</span>는 <span style="color:brown">TLD DNS Server</span>의 IP 주소를 반환한다.  
    <br>
    3. Resolver는 <span style="color:orange">Root DNS Server</span>로부터 받은 응답을 기반으로, <span style="color:brown">TLD DNS Server</span>에게 DNS Query를 보낸다.
        - <span style="color:brown">TLD DNS Server</span>는 <span style="color:red">SLD DNS Server</span>의 IP 주소를 반환한다.  
    <br>
    4. Resolver는 <span style="color:brown">TLD DNS Server</span>로부터 받은 응답을 기반으로, <span style="color:red">SLD DNS Server</span>에게 DNS Query를 보낸다.
        - <span style="color:red">SLD DNS Server</span>는 도메인 네임에 대한 IP 주소를 반환한다.  
    <br>
    5. Resolver는 최종적으로 <span style="color:red">SLD DNS Server</span>로부터 받은 도메인 네임에 대한 IP 주소를 DNS Client에게 전달한다.   
<br>

## Name Server
- **Name Server** : <span style="color:green">*Domain Name Space에 대한 정보를 가지고 있는 서버*</span>
    - Name Server는 <span style="color:red">계층적 구조</span>로 조직되어 있는데, 이것은 Name Server 간의 상호 작용이 계층적이라는 의미이다.
    - Name Server는 데이터 저장 및 관리, 요청 처리 및 응답 구현 등의 역할을 수행한다.
    - Name Server는 <span style="color:orange">Root DNS Server</span>, <span style="color:brown">TLD DNS Server</span>, <span style="color:red">SLD DNS Server</span>로 분류할 수 있다.     
<br>
- **Root DNS Server**
    - <span style="color:green">DNS의 최상위 계층에 위치하는 서버로, DNS Query 프로세스의 시작점이다.</span> 
    - TLD DNS Server의 IP 주소를 제공한다.
    - ICANN이 직접 관리하는 서버이며, 전세계에 13개의 Root DNS Server가 존재한다.  
<br>  
- **TLD(Top-Level Domain) DNS Server**
    - <span style="color:green">TLD를 관리하는 서버이다.</span>    
        ex) "com", "net"
    - SLD DNS Server의 IP 주소를 제공한다.  
    - 일반적으로 특정 조직이나 단체에 의해 관리된다.  
<br>  
- **SLD(Second-Level Domain) DNS Server(= Authoritative DNS Server)**
    - <span style="color:green">SLD를 관리하는 서버이다.</span>  
        ex) "google(.com)"
    - 도메인 네임에 대한 IP 주소를 제공한다.
    - 일반적으로 도메인/호스팅 업체에 의해 관리된다.
        - 일부 조직이나 기업, 또는 개인이 자체적으로 SLD DNS Server를 운영하거나 관리할 수도 있다.  

# DNS 동작 과정 예시
1. 사용자가 웹 브라우저에 "www.google.com"을 입력한다.  
<br>
2. 먼저, 웹 브라우저와 운영체제는 <span style="color:red">캐시를 확인</span>한다.   
    a. 이전의 결과가 이미 캐시에 저장되어 있는 경우 바로 해당 정보를 반환하고, 아래 과정을 생략한다.    
    b. 웹 브라우저 및 운영체제 캐시에 해당 정보가 없거나 만료된 경우, 웹 브라우저는 DNS Client를 사용하여 Resolver에게 "www.google.com"의 IP 주소를 요청한다.    
<br>
3. Resolver는 <span style="color:red">캐시를 확인</span>한다.  
    a. 이전의 결과가 이미 캐시에 저장되어 있는 경우 바로 해당 정보를 DNS Client에게 해당 정보를 반환하고, 아래 과정을 생략한다.  
    b. Resolver 캐시에 해당 정보가 없거나 만료된 경우, 아래 과정을 진행한다.  
<br>
4. Resolver는 Root DNS Server에게 “com” 도메인에 대한 정보를 요청한다.   
    - Root DNS Server는 “com” 도메인을 관리하는 TLD DNS Server의 IP 주소를 응답한다.
<br>
5. Resolver는 “com” 도메인을 관리하는 TLD DNS Server에게 "google" 도메인에 대한 정보를 요청한다.  
    - TLD DNS Server는 "google" 도메인을 관리하는 SLD DNS Server의 IP 주소를 응답한다.
<br>
6. Resolver는 "google" 도메인을 관리하는 SLD DNS Server에게 “www” 도메인에 대한 정보를 요청한다.
    - SLD DNS Server는 "www.google.com"의 IP 주소인 “209.85.227.104”를 응답한다.
<br>
8. Resolver는 DNS Client에게 “209.85.227.104”를 전달하고, 최종적으로 DNS Client는 웹 브라우저에게 “209.85.227.104”를 전달한다.

---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
https://ko.wikipedia.org/wiki/도메인_네임   
https://ko.wikipedia.org/wiki/도메인_네임_시스템  
https://www.cloudflare.com/ko-kr/learning/dns/glossary/what-is-a-domain-name/  
https://aws.amazon.com/ko/route53/what-is-dns/  
https://www.ibm.com/kr-ko/topics/dns  
https://hanamon.kr/dns란-도메인-네임-시스템-개념부터-작동-방식까지/  
https://velog.io/@qltkd5959/DNSDomain-Name-System란   
https://velog.io/@dnwlsrla40/DNS-Domain-Name-System-zrombqvk  
https://copycode.tistory.com/124  
https://www.quora.com/What-are-domain-names  
chatgpt  
bard
</div>
</details>