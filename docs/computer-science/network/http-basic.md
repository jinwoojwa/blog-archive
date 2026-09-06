---
title: HTTP 핵심 특성
---

## 1. HTTP란

`HTTP(HyperText Transfer Protocol)`

- 웹 브라우저(클라이언트)와 웹 서버가 데이터를 주고받기 위해 사용하는 통신 규약(프로토콜)
- 요청(Request)과 응답(Response) 방식으로 동작

## 2. HTTP 특징

### 2.1 비연결성 (Connectionless)

- 브라우저가 서버에 요청하는 순간 잠깐 연결되었다가, 응답을 받은 후 곧바로 연결이 종료됨
- 웹의 특성상 다수의 클라이언트가 동시에 서버와 통신하므로 연결을 계속 유지하면 서버 자원이 낭비됨

### 2.2 무상태 (Stateless)

- 서버가 이전 요청의 상태를 기억하지 않는 것을 의미
- 모든 요청은 독립적으로 처리되며, 이전 요청 정보가 자동으로 유지되지 않음
- 따라서 로그인 정보와 같은 상태를 유지하기 위해 `쿠키(Cookie)`, `세션(Session)` 등을 사용함

## 3. HTTP 요청 프로토콜

```text
http://localhost:8080/MyApp/member/login.html
```

| 번호 | 요소 이름                      | 값           | 의미                                         |
| ---- | ------------------------------ | ------------ | -------------------------------------------- |
| ①    | Protocol (프로토콜)            | `http`       | 클라이언트와 서버가 통신하는 규약            |
| ②    | Host (호스트)                  | `localhost`  | 요청을 보낼 서버의 주소 또는 도메인          |
| ③    | Port (포트)                    | `8080`       | 서버 내부의 특정 서비스에 접속하기 위한 번호 |
| ④    | Context Path (컨텍스트 경로)   | `MyApp`      | 웹 애플리케이션을 식별하는 경로              |
| ⑤    | Directory Path (디렉터리 경로) | `member`     | 애플리케이션 내부의 하위 경로                |
| ⑥    | Resource (리소스)              | `login.html` | 요청하는 실제 파일 또는 자원                 |

### 3.1 HTTP 요청 프로토콜 구조

HTTP 요청 메시지는 `Start-Line`, `Message Header`, `Message Body` 로 구성됨

```text
[Web Client] ---- HTTP Request ----> [Web Server]
                     │
                     ├─ Start-Line
                     ├─ Message Header
                     ├─ CRLF
                     └─ Message Body
```

#### 3.1.1 Start-Line

- 요청과 관련된 핵심 정보(요청 방식, 요청 URI, 프로토콜/버전)가 포함됨

<br>

**① 요청 방식(Method)**

REST API 관점에서 자주 사용하는 Method

- `GET` : 조회
- `POST` : 생성
- `PUT` : 전체 수정
- `PATCH` : 부분 수정
- `DELETE` : 삭제

<br>

**② 요청 URI**

- 요청 URL : `http://localhost:8080/MyApp/member/login.html`
- 요청 URI : `/MyApp/member/login.html`

※ URI는 서버 내부 자원을 식별하는 경로이며, HTTP 요청 메시지의 Start-Line에는 URL 전체가 아닌 URI가 포함됨

<br>

**③ 프로토콜/버전**

- `HTTP/1.1`

사용자가 브라우저에서 아래 URL을 요청하면

```text
http://localhost:8080/MyApp/member/login.html
```

Start-Line에는 다음과 같은 정보가 자동으로 설정됨

```text
GET     /MyApp/member/login.html     HTTP/1.1
│              │                        │
│              │                        └─ 프로토콜/버전
│              └──────────────────────── 요청 URI
└─────────────────────────────────────── 요청 방식(Method)
```

#### 3.1.2 Message Header

- `Key: Value` 형태로 구성됨
- 요청에 대한 부가 정보를 전달함

| Key               | 설정 정보                                  |
| ----------------- | ------------------------------------------ |
| `Host`            | 요청하려는 서버 호스트 이름과 포트 번호    |
| `User-Agent`      | 브라우저 이름과 버전 정보                  |
| `Accept`          | 브라우저가 처리할 수 있는 MIME Type 목록   |
| `Accept-Charset`  | 브라우저가 처리할 수 있는 문자 인코딩 목록 |
| `Accept-Language` | 브라우저가 처리할 수 있는 언어 목록        |
| `Cookie`          | `key=value` 형태의 쿠키 정보               |

> MIME Type : 데이터의 형식을 나타내는 표준 규격
> 예시
>
> - text/html
> - text/plain
> - application/json
> - application/xml
> - image/png
> - image/jpeg
>
> HTTP Header의 Content-Type을 통해 데이터 형식을 전달한다.

#### 3.1.3 Message Body

- 주로 `POST`, `PUT`, `PATCH` 요청에서 사용됨
- 사용자가 입력한 데이터(JSON, Form 데이터 등)가 포함됨
- `GET` 요청은 일반적으로 Message Body를 사용하지 않음

예시

```json
{
  "id": "user1",
  "password": "1234"
}
```

#### 3.1.4 HTTP 요청 프로토콜 전체 예시

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

## 4. HTTP 응답 프로토콜

- 서버는 브라우저로부터 전송된 HTTP 요청 프로토콜로부터 정보를 추출하여 요청을 처리
- HTTP 응답 프로토콜을 생성하여 처리 결과를 브라우저로 전송

### 4.1 HTTP 응답 프로토콜 구조

HTTP 응답 메시지는 `Status-Line`, `Message Header`, `Message Body` 로 구성됨

```text
[Web Server] ---- HTTP Response ----> [Web Client]
                      │
                      ├─ Status-Line
                      ├─ Message Header
                      ├─ CRLF
                      └─ Message Body
```

#### 4.2.1 Status-Line

- 응답과 관련된 핵심 정보(프로토콜/버전, 상태 코드, 상태 메시지)가 포함됨

<br>

**① 프로토콜/버전**

- `HTTP/1.1`

<br>

**② 상태 코드(Status Code)**

- 서버의 요청 처리 결과를 숫자로 표현

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

**③ 상태 메시지(Status Message)**

- 상태 코드에 대한 설명

예시

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

응답이 성공적으로 처리되었다면 Status-Line은 다음과 같이 설정됨

```text
HTTP/1.1     200     OK
│             │       │
│             │       └─ 상태 메시지
│             └──────── 상태 코드
└────────────────────── 프로토콜/버전
```

#### 4.2.2 Message Header

- `Key: Value` 형태로 구성됨
- 응답에 대한 부가 정보를 전달함

| Key              | 설정 정보                   |
| ---------------- | --------------------------- |
| `Content-Type`   | 응답 데이터의 MIME Type     |
| `Content-Length` | 응답 데이터 크기            |
| `Set-Cookie`     | 브라우저에 저장할 쿠키 정보 |
| `Cache-Control`  | 캐시 정책                   |
| `Location`       | 리다이렉트 주소             |

예시

```text
Content-Type: text/html;charset=UTF-8
Content-Length: 1024
Set-Cookie: JSESSIONID=ABC123
```

#### 4.2.3 Message Body

- 서버가 클라이언트에게 전달하는 실제 데이터
- HTML, JSON, 이미지 등의 데이터가 포함될 수 있음

JSON 응답 예시

```json
{
  "id": 1,
  "name": "kim"
}
```

HTML 응답 예시

```html
<html>
  <body>
    <h1>Hello World</h1>
  </body>
</html>
```

#### 4.2.4 HTTP 응답 프로토콜 전체 예시

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

## 5. 정리

- `HTTP`는 요청(Request)과 응답(Response) 기반의 통신 프로토콜이다.
- `HTTP`는 비연결성, 무상태 특징을 가진다.
- 요청 메시지 = `Start-Line + Header + Body`
- 응답 메시지 = `Status-Line + Header + Body`
- Header는 부가 정보, Body는 실제 데이터를 담는다.
- 상태 유지를 위해 `Cookie`와 `Session`을 사용한다.
