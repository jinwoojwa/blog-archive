---
title: HTTP 핵심 특성
---

# HTTP 핵심 특성

## HTTP란

`HTTP(HyperText Transfer Protocol)`는 웹 브라우저(클라이언트)와 웹 서버가 데이터를 주고받기 위해 사용하는 통신 규약(프로토콜)이다. 클라이언트가 요청(Request)을 보내고 서버가 응답(Response)을 보내는 방식으로 동작한다.

## HTTP의 특징

### 비연결성 (Connectionless)

브라우저가 서버에 요청을 보내는 순간 잠깐 연결되었다가, 응답을 받으면 곧바로 연결이 종료된다. 웹의 특성상 다수의 클라이언트가 동시에 서버와 통신하기 때문에, 연결을 계속 유지하면 서버 자원이 낭비된다.

### 무상태 (Stateless)

서버가 이전 요청의 상태를 기억하지 않는다는 의미다. 모든 요청은 독립적으로 처리되며, 이전 요청의 정보가 자동으로 유지되지 않는다. 그래서 로그인 정보처럼 상태를 유지해야 하는 경우에는 `쿠키(Cookie)`나 `세션(Session)`을 별도로 사용해야 한다.

## HTTP 요청 프로토콜

다음과 같은 URL을 예로 들어보자.

```text
http://localhost:8080/MyApp/member/login.html
```

이 URL은 아래 요소들로 구성된다.

| 번호 | 요소 이름                      | 값           | 의미                                         |
| ---- | ------------------------------ | ------------ | -------------------------------------------- |
| ①    | Protocol (프로토콜)            | `http`       | 클라이언트와 서버가 통신하는 규약            |
| ②    | Host (호스트)                  | `localhost`  | 요청을 보낼 서버의 주소 또는 도메인          |
| ③    | Port (포트)                    | `8080`       | 서버 내부의 특정 서비스에 접속하기 위한 번호 |
| ④    | Context Path (컨텍스트 경로)   | `MyApp`      | 웹 애플리케이션을 식별하는 경로              |
| ⑤    | Directory Path (디렉터리 경로) | `member`     | 애플리케이션 내부의 하위 경로                |
| ⑥    | Resource (리소스)              | `login.html` | 요청하는 실제 파일 또는 자원                 |

### 요청 프로토콜 구조

HTTP 요청 메시지는 `Start-Line`, `Message Header`, `Message Body`로 구성된다.

```text
[Web Client] ---- HTTP Request ----> [Web Server]
                     │
                     ├─ Start-Line
                     ├─ Message Header
                     ├─ CRLF
                     └─ Message Body
```

#### Start-Line

Start-Line에는 요청과 관련된 핵심 정보, 즉 요청 방식, 요청 URI, 프로토콜/버전이 담긴다.

<br>

**요청 방식(Method)**

REST API 관점에서 자주 사용하는 Method는 다음과 같다.

- `GET`: 조회
- `POST`: 생성
- `PUT`: 전체 수정
- `PATCH`: 부분 수정
- `DELETE`: 삭제

<br>

**요청 URI**

- 요청 URL: `http://localhost:8080/MyApp/member/login.html`
- 요청 URI: `/MyApp/member/login.html`

URI는 서버 내부의 자원을 식별하는 경로다. HTTP 요청 메시지의 Start-Line에는 URL 전체가 아니라 URI만 포함된다는 점에 유의한다.

<br>

**프로토콜/버전**

`HTTP/1.1`과 같은 형태로 표기한다.

사용자가 브라우저에서 아래 URL을 요청하면

```text
http://localhost:8080/MyApp/member/login.html
```

Start-Line에는 다음과 같은 정보가 자동으로 설정된다.

```text
GET     /MyApp/member/login.html     HTTP/1.1
│              │                        │
│              │                        └─ 프로토콜/버전
│              └──────────────────────── 요청 URI
└─────────────────────────────────────── 요청 방식(Method)
```

#### Message Header

Message Header는 `Key: Value` 형태로 구성되며, 요청에 대한 부가 정보를 전달하는 역할을 한다.

| Key               | 설정 정보                                  |
| ----------------- | ------------------------------------------ |
| `Host`            | 요청하려는 서버 호스트 이름과 포트 번호    |
| `User-Agent`      | 브라우저 이름과 버전 정보                  |
| `Accept`          | 브라우저가 처리할 수 있는 MIME Type 목록   |
| `Accept-Charset`  | 브라우저가 처리할 수 있는 문자 인코딩 목록 |
| `Accept-Language` | 브라우저가 처리할 수 있는 언어 목록        |
| `Cookie`          | `key=value` 형태의 쿠키 정보               |

> MIME Type은 데이터의 형식을 나타내는 표준 규격이다.
>
> - text/html
> - text/plain
> - application/json
> - application/xml
> - image/png
> - image/jpeg
>
> HTTP Header의 Content-Type을 통해 이 데이터 형식을 전달한다.

#### Message Body

Message Body는 주로 `POST`, `PUT`, `PATCH` 요청에서 사용되며, 사용자가 입력한 데이터(JSON, Form 데이터 등)가 담긴다. `GET` 요청은 일반적으로 Message Body를 사용하지 않는다.

예시는 다음과 같다.

```json
{
  "id": "user1",
  "password": "1234"
}
```

#### HTTP 요청 프로토콜 전체 예시

지금까지 살펴본 요소를 하나로 합치면 다음과 같은 형태가 된다.

```text
HTTP Request
┌─────────────────────────────────────────────┐
│ Start-Line                                  │
│ POST /member/login HTTP/1.1                 │
├─────────────────────────────────────────────┤
│ Message Header                              │
│ Host: localhost:8080                        │
│ Content-Type: application/json              │
│ Content-Length: 32                          │
├─────────────────────────────────────────────┤
│ CRLF (빈 줄)                                 │
├─────────────────────────────────────────────┤
│ Message Body                                │
│ {"id":"user1","password":"1234"}            │
└─────────────────────────────────────────────┘
```

## HTTP 응답 프로토콜

서버는 브라우저로부터 전송된 HTTP 요청 프로토콜에서 정보를 추출해 요청을 처리하고, 그 결과를 담은 HTTP 응답 프로토콜을 생성해 브라우저로 전송한다.

### 응답 프로토콜 구조

HTTP 응답 메시지는 `Status-Line`, `Message Header`, `Message Body`로 구성된다.

```text
[Web Server] ---- HTTP Response ----> [Web Client]
                      │
                      ├─ Status-Line
                      ├─ Message Header
                      ├─ CRLF
                      └─ Message Body
```

#### Status-Line

Status-Line에는 응답과 관련된 핵심 정보, 즉 프로토콜/버전, 상태 코드, 상태 메시지가 담긴다.

<br>

**프로토콜/버전**

`HTTP/1.1`과 같은 형태로 표기한다.

<br>

**상태 코드(Status Code)**

서버의 요청 처리 결과를 숫자로 표현한다.

| 범위 | 의미            |
| ---- | --------------- |
| 1xx  | 정보 응답       |
| 2xx  | 성공            |
| 3xx  | 리다이렉션      |
| 4xx  | 클라이언트 오류 |
| 5xx  | 서버 오류       |

| 상태 코드 | 의미                         |
| --------- | ---------------------------- |
| `200`     | 요청 성공                    |
| `201`     | 리소스 생성 성공             |
| `301`     | 영구 이동                    |
| `302`     | 임시 이동                    |
| `400`     | 잘못된 요청                  |
| `401`     | 인증 실패                    |
| `403`     | 접근 권한 없음               |
| `404`     | 요청한 리소스를 찾을 수 없음 |
| `500`     | 서버 내부 오류               |

<br>

**상태 메시지(Status Message)**

상태 코드에 대한 설명이 뒤따른다.

- 200 OK
- 201 Created
- 301 Moved Permanently
- 302 Found
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 500 Internal Server Error

<br>

응답이 성공적으로 처리되었다면 Status-Line은 다음과 같이 설정된다.

```text
HTTP/1.1     200     OK
│             │       │
│             │       └─ 상태 메시지
│             └──────── 상태 코드
└────────────────────── 프로토콜/버전
```

#### Message Header

Message Header는 `Key: Value` 형태로 구성되며, 응답에 대한 부가 정보를 전달하는 역할을 한다.

| Key              | 설정 정보                   |
| ---------------- | ---------------------------- |
| `Content-Type`   | 응답 데이터의 MIME Type     |
| `Content-Length` | 응답 데이터 크기            |
| `Set-Cookie`     | 브라우저에 저장할 쿠키 정보 |
| `Cache-Control`  | 캐시 정책                   |
| `Location`       | 리다이렉트 주소             |

예시는 다음과 같다.

```text
Content-Type: text/html;charset=UTF-8
Content-Length: 1024
Set-Cookie: JSESSIONID=ABC123
```

#### Message Body

Message Body에는 서버가 클라이언트에게 전달하는 실제 데이터가 담긴다. HTML, JSON, 이미지 등 다양한 형식의 데이터가 올 수 있다.

JSON 응답이라면 다음과 같은 형태다.

```json
{
  "id": 1,
  "name": "kim"
}
```

HTML 응답이라면 다음과 같은 형태다.

```html
<html>
  <body>
    <h1>Hello World</h1>
  </body>
</html>
```

#### HTTP 응답 프로토콜 전체 예시

지금까지 살펴본 요소를 하나로 합치면 다음과 같은 형태가 된다.

```text
HTTP Response
┌─────────────────────────────────────────────┐
│ Status-Line                                 │
│ HTTP/1.1 200 OK                             │
├─────────────────────────────────────────────┤
│ Message Header                              │
│ Content-Type: application/json              │
│ Content-Length: 27                          │
├─────────────────────────────────────────────┤
│ CRLF (빈 줄)                                 │
├─────────────────────────────────────────────┤
│ Message Body                                │
│ {"id":1,"name":"kim"}                       │
└─────────────────────────────────────────────┘
```

## 정리

- `HTTP`는 요청(Request)과 응답(Response) 기반의 통신 프로토콜이다.
- `HTTP`는 비연결성, 무상태 특징을 가진다.
- 요청 메시지는 `Start-Line + Header + Body`, 응답 메시지는 `Status-Line + Header + Body`로 구성된다.
- Header는 부가 정보를, Body는 실제 데이터를 담는다.
- 상태를 유지하려면 `Cookie`와 `Session`을 사용한다.
