---
title: Java Servlet
---

## 서블릿 (Servlet)이란

- Java 기반 웹 애플리케이션에서 `HTTP 요청을 처리하기 위한` 서버 측 컴포넌트
- 브라우저의 요청을 받아 비즈니스 로직을 수행한 후 응답을 생성하는 역할을 담당
- 초기 Java 웹 개발에서는 동적 웹 페이지를 생성하기 위해 Servlet이 사용되었으며, 이후 View 영역을 분리하기 위해 `JSP`가 등장

**특징**

- Java 코드로 웹 요청 처리
- `WAS(Web Application Server)` 내부에서 실행
- HTTP 프로토콜 기반 요청/응답 처리
- 서블릿 컨테이너가 생명주기를 관리

## 서블릿 컨테이너와 web.xml

### 서블릿 컨테이너

- Servlet의 생성, 초기화, 실행, 소멸을 관리하는 실행 환경
- 대표적인 구현체는 `Apache Tomcat`

**주요 역할**

- Servlet 객체 생성 및 관리
- URL과 Servlet 매핑
- 요청(Request) 및 응답(Response) 객체 생성
- 생명주기 관리
- 멀티스레드 기반 요청 처리

### web.xml

- 초기 Servlet 환경에서는 web.xml을 이용하여 URL과 Servlet을 매핑

```xml
<servlet>
    <servlet-name>HelloServlet</servlet-name>
    <servlet-class>com.example.HelloServlet</servlet-class>
</servlet>

 <servlet-mapping>
    <servlet-name>HelloServlet</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```

- Servlet 3.0 이후에는 애노테이션 기반 설정이 일반적으로 사용됨

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
}
```

## 서블릿 클래스 작성 규칙

- Servlet은 일반적으로 `HttpServlet`을 상속하여 작성

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet( HttpServletRequest request, HttpServletResponse response ) {
         // 요청 처리
    }
}
```

**주요 규칙**

- `HttpServlet` 상속
- URL 매핑 설정
- HTTP 메서드에 맞는 메서드 오버라이딩
  - `doGet()`
  - `doPost()`
  - `doPut()`
  - `doDelete()`
- 상태를 가지는 인스턴스 변수 사용 주의

## 서블릿 계층 구조

```
Servlet <<interface>>
  └─ GenericServlet <<abstract class>>
       └─ HttpServlet <<abstract class>>
            └─ MyServlet
```

### Servlet

- 서블릿의 최상위 인터페이스
- 생명주기 관련 메서드를 정의

```java
init()
service()
destroy()
```

### GenericServlet

- 프로토콜에 독립적인 추상 클래스
- Servlet 인터페이스의 기본 구현을 제공

### HttpServlet

- HTTP 프로토콜에 특화된 Servlet 구현체
- GET, POST 등의 요청을 메서드별로 분리하여 처리

```java
doGet()
doPost()
doPut()
doDelete()
```

## 서블릿 객체 생성과 라이프사이클

- Servlet 객체는 요청마다 생성되지 않음
- 서블릿 컨테이너는 일반적으로 하나의 Servlet 인스턴스를 생성하고 여러 요청을 처리

```
┌──────────────┐
│ Servlet 생성  │
└──────┬───────┘
       │
       ▼
┌──────────┐     ┌─────────────┐     ┌─────────────┐
│ init()   │ ──► │ service()   │ ──► │ destroy()   │
│ 최초 1회   │     │ 요청마다      │     │ 종료 시 1회   │
└──────────┘     └──────┬──────┘     └─────────────┘
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
         doGet()    doPost()    doDelete()
```

1. 애플리케이션 시작 또는 첫 요청 시 Servlet 객체 생성
2. `init()`: 초기화 작업 수행
3. `service()` : 클라이언트 요청마다 호출
   - `HttpServlet`에서는 요청 메서드에 따라 `doGet()`, `doPost()` 등을 호출
4. `destroy()`: 서버 종료 또는 Servlet 제거 시 호출

### Servlet 요청 처리 구조 (스레드 모델)

- Servlet은 요청마다 객체를 생성하지 않고 하나의 인스턴스만 생성
- 대신 각 요청은 별도의 스레드로 처리됨

```text
Servlet 인스턴스 1개
        ▲
        │
 ┌──────┼────────┬────────┐
 │      │        │        │
Thread1 Thread2 Thread3 ThreadN
  │       │       │        │
  ▼       ▼       ▼        ▼
service() service() service() service()
```

- Servlet 객체는 공유됨
- 요청은 멀티스레드로 동시에 처리됨

### Thread Safety 문제

- Servlet 인스턴스 변수는 여러 스레드가 공유함
- 따라서 동시성 문제가 발생할 수 있음

```java
public class HelloServlet extends HttpServlet {
    private int count = 0;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        count++; // 동시 접근 시 문제 발생 가능
    }
}
```

**안전한 방식**

- 지역 변수 사용
- 또는 `synchronized / Atomic` 사용

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    int count = 0;
}
```

## 정리

- Servlet은 Java 기반 웹 요청 처리 기술
- Servlet 생명주기는 WAS 내부의 Servlet Container가 관리
- Servlet은 하나의 인스턴스로 여러 요청을 처리
- 요청은 멀티스레드로 동시에 실행됨
- URL 매핑은 web.xml 또는 @WebServlet을 사용
- 일반적으로 HttpServlet을 상속하여 구현
- Servlet 객체는 하나만 생성되어 여러 요청을 처리
- 생명주기는 `init() → service() → destroy()` 순서로 동작
- 상태를 가지는 인스턴스 변수는 thread-safe하지 않음
- View 작성의 불편함을 해결하기 위해 JSP가 등장
