---
title: "JDBC"
tags:
    - java
date: "2024-06-12"
thumbnail: "/assets/img/thumbnail/sample.png"
---

# 애플리케이션 서버와 데이터베이스
![](https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/06234841-3bf9-4e74-8a07-fcc190872d35){:class="img-lg"}

1. **커넥션 연결** : 주로 TCP/IP를 사용해서 커넥션을 연결한다.  
2. **SQL 전달** : 애플리케이션 서버는 DB가 이해할 수 있는 SQL을 연결된 커넥션을 통해 DB에 전달한다.  
3. **결과 응답** : DB는 전달된 SQL을 수행하고 그 결과를 응답한다. 애플리케이션 서버는 응답 결과를 활용한다.  

- **문제**
    - 각각의 데이터베이스마다 커넥션 연결, SQL 전달, 결과를 응답 받는 방법이 모두 다르다. 즉, <span style="color:blueviolet">데이터베이스 종류를 변경하면 애플리케이션 서버의 데이터베이스 사용 코드도 함께 변경</span>해야 한다. 
    - 이런 문제를 해결하기 위해 <span style="color:red">JDBC라는 자바 표준</span>이 등장한다.

# JDBC 
<center><img src="https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/80c95130-f67c-45e6-b262-5469d4a6da74" width="650" height="230"></center>
- **JDBC**(Java Database Connectivity)는 데이터베이스와의 연결 및 상호작용을 위한 자바 API이다.  
    - <span style="color:red">JDBC API는 자바 패키지 java.sql과 javax.sql에 여러 표준 인터페이스와 클래스를 정의하고 있다.</span>  
        - java.sql 패키지는 JDBC API의 핵심 부분을 구성하며, javax.sql 패키지는 JDBC API의 확장 부분을 구성한다. 
    - 각 DB 벤더는 자신의 DB에 맞도록 <span style="color:red">JDBC API에서 정의한 표준 인터페이스들을 구현해서 라이브러리로 제공하는데, 이것을 JDBC 드라이버라고 한다.</span>
        - 즉, JDBC 표준 인터페이스의 구현체는 JDBC 드라이버이다.   
    - 개발자는 JDBC 표준 인터페이스만 사용해서 개발하면 되고, <span style="color:blueviolet">애플리케이션 로직은 JDBC 표준 인터페이스에만 의존한다.<span> 
        - 데이터베이스를 다른 종류의 데이터베이스로 변경하고 싶다면 JDBC 드라이버만 변경하면 되므로, 애플리케이션 서버의 사용 코드를 그대로 유지할 수 있다.    
<br> 
- <span style="color:red">**주요 JDBC 표준 인터페이스와 클래스**</span> 
    - **Connection, Statement, ResultSet** (인터페이스, java.sql) 
        - 데이터베이스와의 연결, 쿼리 실행, 쿼리 결과의 표준을 정의한다.
    - **Driver** (인터페이스, java.sql)
        - 데이터베이스와 연결을 설정하는 방법의 표준을 정의한다.
    - **DriverManager** (클래스, java.sql)
        - JDBC 드라이버들을 관리하고, 등록된 드라이버 중에서 적합한 드라이버의 Connection 객체를 획득하는 기능을 제공한다. 
    - **DataSource** (인터페이스, javax.sql)    
        - <span style="color:red">Connection 객체를 획득하는 방법의 표준</span>을 정의한다.
        - [커넥션 풀과 DataSource](https://leejae-h.github.io/posts/61)
    - **SQLException** (클래스, java.sql)
        - JDBC에서 데이터베이스와의 상호작용 중에 발생할 수 있는 모든 예외를 처리하는 기본 예외 클래스이다.
        
<br>
## **Connection, Statement, ResultSet**
![](https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/18bd5fd1-f282-4de4-88e7-e5f6de60e274){:class="img-lg"}
![](https://github.com/LeeJae-H/LeeJae-H.github.io/assets/122717063/54082f27-55f7-4c48-ab1f-f46d0343a002){:class="img-lg"}
- **Connection 주요 메소드**
    ```java
    public interface Connection extends Wrapper, AutoCloseable {
        Statement createStatement() throws SQLException; // SQL 쿼리를 실행할 수 있는 Statement 객체를 생성
        PreparedStatement prepareStatement(String var1) throws SQLException; //  미리 컴파일된 SQL 쿼리를 실행할 수 있는 PreparedStatement 객체를 생성

        void setAutoCommit(boolean var1) throws SQLException; // 자동 커밋 모드로 설정
        void commit() throws SQLException; // 현재 트랜잭션을 커밋
        void rollback() throws SQLException; // 현재 트랜잭션을 롤백

        void close() throws SQLException; // Connection 종료
    }
    ```
- **Statement 주요 메소드**
    ```java
    public interface Statement extends Wrapper, AutoCloseable {
        ResultSet executeQuery(String var1) throws SQLException; // SELECT 문을 실행하고, 결과를 ResultSet 객체로 반환
        int executeUpdate(String var1) throws SQLException; // INSERT, UPDATE, DELETE 문을 실행하고, 영향을 받은 행의 수를 반환
        boolean execute(String var1) throws SQLException; // SQL 문을 실행하고, 쿼리의 실행 결과를 나타내는 boolean 값을 반환

        void close() throws SQLException; // Statement 종료
    }
    ```

- **ResultSet 주요 메소드**
    ```java
    // ResultSet은 테이블 형식이다.
    public interface ResultSet extends Wrapper, AutoCloseable {
        boolean next() throws SQLException; // 커서를 다음 행으로 이동시키며, 결과 집합에 더 이상 행이 없으면 false를 반환

        String getString(String var1) throws SQLException; // 지정된 열의 문자열 값을 반환
        int getInt(String var1) throws SQLException; // 지정된 열의 정수 값을 반환
        double getDouble(String var1) throws SQLException; // 지정된 열의 실수 값을 반환

        void close() throws SQLException; // ResultSet 종료
    }
    ```

<br>
## Driver, DriverManager

```java
public interface Driver {
    // 주어진 URL과 정보를 바탕으로 데이터베이스와 연결하고, 그 연결 객체(Connection 객체)를 반환
    Connection connect(String var1, Properties var2) throws SQLException;
    ...
}
```

```java
public class DriverManager {
    ...
    // Driver 객체를 DriverManager에 등록
    public static void registerDriver(Driver driver) throws SQLException {
        registerDriver(driver, (DriverAction)null);
    }
    public static void registerDriver(Driver driver, DriverAction da) throws SQLException {
        ...
        registeredDrivers.addIfAbsent(new DriverInfo(driver, da));
    }

    // Driver 객체의 connect 메소드를 호출하여 Connection 객체를 획득
    public static Connection getConnection(String url, String user, String password) throws SQLException { 
        ...
        return getConnection(url, info, Reflection.getCallerClass());
    }
    private static Connection getConnection(String url, Properties info, Class<?> caller) throws SQLException {
        ...
        Iterator var5 = registeredDrivers.iterator();
        while(true) {
            while(var5.hasNext()) {
                DriverInfo aDriver = (DriverInfo)var5.next();  
                ...
                    Connection con = aDriver.driver.connect(url, info);
                    if (con != null) {
                        return con;
                    }
                ...
    }
    ...
}
```
- <span style="color:blueviolet">Driver 클래스 로딩 시 Driver 객체가 DriverManager에 등록</span>되는데, Driver 클래스를 로딩하는 방식은 아래와 같이 2가지 방식이 있다.
    - 명시적 로딩 : Class.forName("...") 메소드를 사용하여 Driver 클래스를 명시적으로 로드하는 방식
    - <span style="color:blueviolet">자동 로딩</span> : Java 6 이후, JDBC 4.0부터는 JDBC 드라이버 JAR 파일의 "META-INF/services/java.sql.Driver" 파일이 포함되어 있으면 자동으로 Driver 클래스가 로드되는 방식  
<br>
- DriverManager는 JDBC 드라이버들의 Driver 객체를 관리하고, 등록된 Driver 객체들의 connect 메소드를 호출하여 Connection 객체가 얻어지면 해당 Connection 객체를 반환하는 기능을 제공한다. 
    - <span style="color:blueviolet">Connection 객체를 얻기 위한 이런 복잡한 작업들을 DriverManager가 해주기 때문에, 개발자는 간편하게 DriverManager만 사용해서 Connection 객체를 얻으면 된다.</span>  
<br>
- **DriverManager와 JDBC 드라이버 -  MySQL 예시**
> - MySQL JDBC 드라이버 JAR 파일의 "com.mysql.cj.jdbc 패키지" 내에는 Driver, Connection, Statement, ResultSet 등 JDBC 표준 인터페이스를 구현하는 클래스들이 존재한다.

1. MySQL JDBC 드라이버의 Driver 클래스가 로드될 때, Driver 클래스의 정적 초기화 블록이 실행되어 <span style="color:blueviolet">Driver 객체가 DriverManager에 등록</span>된다.
    ```java
    public class Driver extends NonRegisteringDriver implements java.sql.Driver 
    {
        public Driver() throws SQLException { }

        static {
            try {
                DriverManager.registerDriver(new Driver());
            } catch (SQLException var1) {
                throw new RuntimeException("Can't register driver!");
            }
        }
    }
    ```

2. 애플리케이션에서 DriverManager.getConnection(url, user, password) 메소드를 호출하면, 해당 메소드에서는 <span style="color:blueviolet">등록된 Driver 객체들의 connect 메소드를 호출하여 Connection 객체가 얻어지면 해당 Connection 객체를 반환한다.</span>
    - MySQL JDBC 드라이버의 Driver 객체의 connect 메소드는 MySQL 서버에 연결한 후 <span style="color:blueviolet">ConnectionImpl 객체(Connection 인터페이스의 구현체)를 반환</span>한다.    

---
### JDBC를 사용한 간단한 CRUD 예시
- DBConnectUtil 클래스 작성 (Connection 획득)
    ```java
    public class DBConnectionUtil {
        public static Connection getConnection() {
            try {
                Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
                return connection;
            } catch (SQLException e) {
                throw new IllegalStateException(e);
            }
        }
    }
    ```
- Member 클래스 작성
    ```java
    public class Member {
        private String memberId;
        private int money;

        public Member() {
        }

        public Member(String memberId, int money) {
            this.memberId = memberId;
            this.money = money;
        }
        
        ...
        // Getter, Setter 메소드
        // toString, equals, hashCode 메소드 - @Override
    }
    ```
- JdbcMemberRepository 클래스 작성
    ```java
    @Repository
    public class MemberRepositoryJdbc implements MemberRepository {
        private static final Logger log = LoggerFactory.getLogger(MemberRepositoryJdbc.class);

        // Coonection 획득
        private Connection getConnection() { 
            return DBConnectionUtil.getConnection(); 
        }

        // 리소스 정리
        private void close(Connection con, Statement stmt, ResultSet rs) {
            JdbcUtils.closeResultSet(rs);
            JdbcUtils.closeStatement(stmt);
            JdbcUtils.closeConnection(con);
        }
        
        @Override
        public Member save(Member member) throws SQLException {
            String sql = "insert into member(member_id, money) values(?, ?)";
            Connection con = null;
            PreparedStatement pstmt = null;
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, member.getMemberId());
                pstmt.setInt(2, member.getMoney());
                pstmt.executeUpdate();
                return member;
            } catch (SQLException e) {
                log.error("db error", e);
                throw e;
            } finally {
                close(con, pstmt, null);
            }
        }

        @Override
        public Member findById(String memberId) throws SQLException {
            String sql = "select * from member where member_id = ?";
            Connection con = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, memberId);
                rs = pstmt.executeQuery();
                // 조회 결과가 항상 1건(member_id는 PK)이므로 
                    // while 대신에 if 사용
                if (rs.next()) {
                    Member member = new Member();
                    member.setMemberId(rs.getString("member_id"));
                    member.setMoney(rs.getInt("money"));
                    return member;
                } else {
                    throw new NoSuchElementException("member not found memberId=" + memberId);
                }
            } catch (SQLException e) {
                log.error("db error", e);
                throw e;
            } finally {
                close(con, pstmt, rs);
            }
        }

        @Override
        public void update(String memberId, int money) throws SQLException {
            String sql = "update member set money=? where member_id=?";
            Connection con = null;
            PreparedStatement pstmt = null;
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setInt(1, money);
                pstmt.setString(2, memberId);
                pstmt.executeUpdate();
            } catch (SQLException e) {
                log.error("db error", e);
                throw e;
            } finally {
                close(con, pstmt, null);
            }
        }

        @Override
        public void delete(String memberId) throws SQLException {
            String sql = "delete from member where member_id=?";
            Connection con = null;
            PreparedStatement pstmt = null;
            try {
                con = getConnection();
                pstmt = con.prepareStatement(sql);
                pstmt.setString(1, memberId);
                pstmt.executeUpdate();
            } catch (SQLException e) {
                log.error("db error", e);
                throw e;
            } finally {
                close(con, pstmt, null);
            }
        }
    }
    ```

---
<details>
<summary><span style="color:gray">참고 자료</span></summary>
<div markdown="1">
"김영한 스프링 DB 1편"  
https://ko.wikipedia.org/wiki/JDBC  
https://www.ibm.com/docs/ko/i/7.3?topic=connections-java-drivermanager-class  
https://www.ibm.com/docs/ko/i/7.3?topic=exceptions-java-sqlexception-class  
</div>
</details>

