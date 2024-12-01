---
title: "(Java) String"
tags:
    - java
date: "2024-12-01"
thumbnail: "/assets/img/thumbnail/sample.png"
---
> ***String**은 문자열을 나타내는 클래스이다. Java에서 모든 문자열 리터럴(값)은 String 클래스의 인스턴스로 구현된다.* <span style="color:gray">(feat. Oracle)</span>

## String 객체의 생성과 메모리 저장 방식
- String([참고 : 참조형](https://leejae-h.github.io/Java/%EC%B0%B8%EC%A1%B0%ED%98%95.html))은 **Stack 영역**에 실제 값이 저장된 공간의 **주소를 저장**하고, Heap 영역에 **실제 값을 저장**한다.  
- Java에서 String 객체를 생성하는 방법은 2가지이다.
    1. **문자열 리터럴(값)** : **컴파일 타임**에 String Constant Pool에 객체 생성, 이미 존재한다면 해당 객체 재사용 
    2. **new 키워드** : 일반적인 객체 생성처럼 **런타임**에 Heap 영역에 객체 생성
- ex)
    <img src="https://github.com/user-attachments/assets/d5682108-88e9-4c9c-9580-a7aa9fc744f9" style="width:700px;" />

## String 멤버함수
```java
// equals() : 문자열 내용 비교 -> boolean     
String s1 = "java";
String s2 = "java";
String s3 = new String("java");
System.out.println(s1 == s2); // 출력 : true -> 주소 비교
System.out.println(s1.equals(s2)); // 출력 : true -> 문자열 내용 비교
System.out.println(s1 == s3); // 출력 : false 
System.out.println(s1.equals(s3)); // 출력 : true 

// length() : 문자열 길이 -> int
String s1 = "";
String s2 = " ";
String s3 = "1234";
String s4 = " 12 34 ";
System.out.println(s1.length()); // 출력 : 0
System.out.println(s2.length()); // 출력 : 1
System.out.println(s3.length()); // 출력 : 4
System.out.println(s4.length()); // 출력 : 7

// charAt() : 특정 인덱스의 문자 -> char
String s1 = "1234";
System.out.println(s1.charAt(2)); // 출력 : 3

// substring() : 특정 인덱스 범위의 문자열 -> String
String s1 = "123456";
System.out.println(s1.substring(1)); // 출력 : 23456 -> begin 부터 마지막 까지
System.out.println(s1.substring(0, 3)); // 출력 : 123 -> begin  부터 end - 1 까지
System.out.println(s1.substring(0, 1)); // 출력 : 1

// contains() : 특정 문자열의 존재 여부 -> boolean
String s1 = "ab cdef";
System.out.println(s1.contains("cd")); // 출력 : true

// indexOf() / lastIndexOf() : 특정 문자열의 시작/마지막 인덱스 -> int
String s1 = "aa bb aa bb";
System.out.println(s1.indexOf("b")); // 출력 : 3
System.out.println(s1.lastIndexOf("b")); // 출력: 10

// startsWith() / endsWith() : 문자열이 특정 접두사 또는 접미사로 시작/끝나는지 확인 > boolean
String s1 = "image_file.png";
System.out.println(s1.startsWith("image")); // 출력 : true
System.out.println(s1.endsWith(".png")); // 출력 : true

// split() : 특정 문자열을 기준으로 문자열을 분리하여 배열로 반환 -> String[]
String s1 = "java python";
String[] languages1 = s1.split(" ");
String[] languages2 = s1.split("");
String[] languages3 = s1.split("a");
System.out.println(Arrays.toString(languages1)); // 출력 : [java,python]
System.out.println(Arrays.toString(languages2)); // 출력 : [j,a,v,a, ,p,y,t,h,o,n]
System.out.println(Arrays.toString(languages3)); // 출력 :[j,v, python] -> python 앞에 공백

// replace() : 문자열 내의 특정 문자열을 다른 문자열로 대체 -> String
String s1 = "aaa bbb aaa";
System.out.println(s1.replace("aa", "bb")); // 출력 : bba bbb bba
System.out.println(s1.replace(" ", "")); // 출력 : aaabbbaaa
System.out.println(s1); // 출력 : aaa bbb aaa
```

## String 참고 사항
1. **String은 불변 객체이므로, 값이 변경되면 새로운 객체가 생성된다. 따라서, 잦은 연산은 성능 저하(메모리 낭비, 속도 저하 등)가 발생할 수 있다. 대안으로는 가변 객체인 StringBuilder와 StringBuffer가 있다.**
    - **예시 1 : 객체의 주소**
        ```java
		String a = "안녕";
		System.out.println(a); // 출력 : 안녕
		System.out.println(a.hashCode()); // 출력 : 1611021

		a += "하세요";
		System.out.println(a); // 출력 : 안녕하세요
		System.out.println(a.hashCode()); // 출력 : 803356551
        ```

    - **예시 2 : 연산 속도 비교 : String vs StringBuilder**
        ```java
		long startTime_pre = System.nanoTime();

		// 1. String 연산
		String s = "";
		for (int i = 0; i < 100_000; i++) {
            // for문의 코드 실행할 때마다 새로운 객체 생성

			s += i; // s = s + String.valueOf(i);
		}

		long endTime_pre = System.nanoTime();
		System.out.println("String 실행 시간: " + (endTime_pre - startTime_pre) + " ns");
		// ---------------> String 실행 시간: 3524436890 ns ( = 약 3.524초)

		long startTime_ref = System.nanoTime();

		// 2. StringBuilder 연산
		StringBuilder sb = new StringBuilder();
		for (int i = 0; i < 100_000; i++) {
            // 같은 객체를 수정

			sb.append(i); // sb.append(String.valueOf(i));
		}
		String s_sb = sb.toString();

		long endTime_ref = System.nanoTime();
		System.out.println("StringBuilder 실행 시간: " + (endTime_ref - startTime_ref) + " ns");
		// ---------------> StringBuilder 실행 시간: 3767563 ns ( = 약 0.0038초)

        // 실행시간 차이가 크다.
            // 즉, 잦은 연산이 필요할 때 CPU와 메모리를 낭비하고 싶지 않다면 
            // StringBuilder나 StringBuffer를 사용해야 할 것이다.
        ```

2. **Java 컴파일러는 자동으로 (String s = a + b + c)와 같은 표현식을 StringBuilder를 사용하여 최적화한다.**
    ```java
    String s = "Hello" + " " + "World!"; 
    
    // 위 코드는 컴파일러가 아래의 코드로 자동 변환하여 최적화한다.    
    
    String s = new StringBuilder()
        .append("Hello")
        .append(" ")
        .append("World!")
        .toString();
    ```

---
- 참고 자료
https://docs.oracle.com/javase/8/docs/api/  
https://stackoverflow.com/questions/16783971/string-constant-pool-vs-string-pool  
https://docs.oracle.com/javase/specs/jls/se7/html/jls-3.html#jls-3.10.5  
https://www.youtube.com/watch?v=gp6NY01XFoE  
https://stackoverflow.com/questions/47605/string-concatenation-concat-vs-operator  
https://stackoverflow.com/questions/5234147/why-stringbuilder-when-there-is-string  