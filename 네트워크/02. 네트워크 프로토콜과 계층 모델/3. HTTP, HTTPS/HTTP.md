## HTTP 정의

> HTTP: Hyper Text Transfer Protocol

**HTTP는 web에서 server와 client가 서로 데이터를 주고 받기 위해 사용하는 통신 규약이다.** application level의 protocol이며, TCP/IP 위에서 동작한다. 기본적으로는 web 문서 간 link를 통해 연결할 수 있는 protocol인데, 텍스트 뿐만 아니라 사진, JSON, 영상, 파일 등 거의 모든 형태의 데이터를 전송할 수 있다.


## HTTP 요청 / 응답 모델

**HTTP 통신은 server/client 모델을 따르므로, server와 client가 나뉘어진 구조로 구성되어 있다.** client에서 request(요청)를 보내면 server는 response(응답)를 처리한다.

**이렇게 server와 client를 분리한 이유는 책임을 분리하기 위해서이다.** client는 UI를 그리는 책임만 갖고, server는 비즈니스 로직과 데이터를 처리하는 책임만을 갖는다. 책임을 분리하면 각자의 영역에서 독립적인 발전이 가능하며, 트래픽에 따른 scale 확장과 각 진영의 코드의 유지보수가 용이해진다는 엄청난 장점을 갖는다.


## HTTP 메소드

**HTTP 메소드는 client가 web server에게 resource에 대한 특정 동작을 수행하도록 call하는 방법이다.** HTTP 메소드는 총 9가지의 종류가 존재한다. GET(조회), POST(생성 또는 변경), PUT/PATCH(수정), DELETE(삭제) 등의 작업을 지정할 수 있다.

### GET vs POST

| 특성 | GET | POST |
|------|-----|------|
| 목적 | 서버의 리소스에서 데이터를 요청 | 서버의 리소스를 새로 생성하거나 업데이트 |
| 데이터 전송 방식 | URL의 'query string'으로 전송 (URL 주소 끝에 파라미터 형태로 데이터 전송)  | HTTP 메시지 본문(body)에 데이터를 추가하여 전송 |
| 데이터 노출 | URL에 데이터가 노출됨  | URL에 데이터가 노출되지 않음  |
| 데이터 길이 제한 | URL 길이 제한으로 인해 제한적 (일반적으로 2,048자 이하)  | 길이 제한이 상대적으로 큼 (일반적으로 2GB 이하)  |
| 보안성 | 낮음 (URL에 데이터가 노출)  | 상대적으로 높음 (body에 데이터 포함)  |
| 캐싱 | 브라우저에서 자동으로 캐싱됨   | 브라우저에서 캐싱되지 않음  |
| 멱등성(idempotent) | 멱등 (여러 번 요청해도 결과 동일)  | 멱등하지 않음 (요청마다 결과가 달라질 수 있음)  |
| 브라우저 히스토리 | 요청 내역이 브라우저 히스토리에 남음 | 요청 내역이 브라우저 히스토리에 남지 않음 |
| 북마크 가능 여부 | 가능 | 불가능 |
| 주요 사용 사례 | 데이터 조회, 검색 기능, 페이지 이동  | 로그인, 회원가입, 게시글 작성, 파일 업로드  |

> 멱등성(冪等性, idempotent): 여러 번 연산해도 결괏값이 일정한 성질.

**핵심 포인트 1. 사용 목적**

기본적으로 **데이터 조회 시에는 GET**을 사용한다. POST의 경우에는 데이터 생성 뿐만 아니라 변경 자체가 가능하여 수정 및 삭제 시에도 사용 가능하다. 하지만 원래 목적에 따르면 데이터 수정 시에는 PUT 또는 PATCH, 데이터 삭제 시에는 DELETE를 사용하는 것이 올바르다.

**의문점. 그럼 POST의 정체성은 무엇인가?**

RFC 문서에서 POST를 '요청 본문은 타깃 리소스의 정의대로 처리한다.'로 정의한다. client가 request를 보내면 server에서는 모든 작업을 알아서 처리해야 한다는 것이다. 그래서 **server에서 정의한 다양한 작업을 모두 처리할 수 있는 메소드이다.** RFC 문서에서도 form에 입력된 데이터를 불러올 때, 블로그에 메시지를 업로드할 때, 이미 존재하는 리소스에 데이터를 추가할 때 모두 POST 메소드를 사용할 수 있다고 명시돼 있다.

**핵심 포인트 2. 멱등성**

GET 방식은 오직 조회할 때만 사용하기 때문에 몇 번을 호출해도 GET에 의해 결과는 변하지 않으므로 멱등성을 보장한다. 하지만 **POST 방식은 일반적으로 데이터 생성 작업에 사용되기에, 호출할 때마다 결과가 변하므로 멱등성을 보장하지 않는다.** 예를 들어, `/payments` URI에 같은 정보로 3 번의 결제 요청을 하면 `/payments/1`, `/payments/2`, `/payments/3` 서로 다른 URI에 동일한 결제 데이터를 가진 리소스가 하나씩 생성된다.

**핵심 포인트 3. 보안**

GET 방식은 전송할 데이터를 **'query string'** 형태로 보내기 때문에 URL 상에 전송할 데이터가 사용자에게 전부 노출되어 보안적으로 치명적이다. 반면 POST 방식은 전송할 데이터를 **'HTTP 메시지 본문'** 에 담아 보내기 때문에 바로 보이지는 않는다. 하지만 요즘에는 크롬 개발자 도구에서 HTTP 메시지를 다 까볼 수 있어서, 이 특성이 보안적으로 뛰어나다고까지는 못하겠다. 


### PUT vs PATCH

| 특성 | PUT | PATCH |
|------|-----|-------|
| 목적 | 리소스의 완전한 교체 | 리소스의 부분적 수정 |
| 데이터 전송 | 리소스의 전체 표현을 전송 | 변경할 부분만 전송 |
| 리소스 존재 여부 | 없으면 새로 생성 | 일반적으로 기존 리소스가 있어야 함 |
| 멱등성 | 멱등 (여러 번 요청해도 결과 동일) | 일반적으로 멱등하지 않음 |
| 부분 업데이트 | 전체 데이터를 보내야 함 | 변경할 필드만 보내면 됨 |
| 미전송 필드 처리 | 일반적으로 null이나 기본값으로 설정 | 변경되지 않은 상태로 유지 |
| 리소스 상태 | 요청 후 항상 동일한 상태 | 요청에 따라 다른 상태가 될 수 있음 |
| 에러 처리 | 일부 필드만 전송 시 오류 발생 가능 | 일부 필드만 전송해도 정상 처리 |
| 네트워크 효율성 | 전체 데이터 전송으로 비효율적일 수 있음 | 부분 데이터 전송으로 효율적 |
| 사용 사례 | 전체 리소스 업데이트 | 리소스의 일부 필드 업데이트 |

**핵심 포인트 1. 리소스 업데이트 범위**

PUT 요청은 리소스 전체를 업데이트해야만 한다. 리소스의 일부만 수정하고 싶어도 말이다. 반면 PATCH 요청은 부분 업데이트가 가능해 효율적인 처리가 가능하다.

**핵심 포인트 2. 멱등성**

PUT 요청은 항상 리소스 전체를 overriding하기 때문에, 같은 내용으로 몇 번을 요청해도 그 결과는 항상 일정하기에 멱등성을 보장한다. 반면 **PATCH 요청은 동시성 문제를 갖고 있어 멱등성을 보장하지 못한다.** 한 client가 동일한 내용으로 부분 업데이트를 반복적으로 요청하던 도중, 다른 client가 다른 부분에 새 부분 업데이트를 수행하면, 앞서 반복적으로 부분 업데이트를 요청하던 client 입장에서는 난데없이 내가 요청하지 않은 부분에서 데이터가 일부 변경된 것을 목격하게 될 것이다.

## HTTP 상태 코드

자주 사용되는 HTTP 상태 코드에 대해 표로 정리했다.

| 코드 | 의미 | 설명 |
|------|------|------|
| **1xx: 정보** |
| 100 | Continue | 요청의 일부가 받아들여졌으며, 클라이언트는 나머지 요청을 계속해야 함 |
| 101 | Switching Protocols | 서버가 프로토콜을 전환하고 있음 |
| **2xx: 성공** |
| 200 | OK | 요청이 성공적으로 처리됨 |
| 201 | Created | 요청이 성공적으로 처리되어 새로운 리소스가 생성됨 |
| 204 | No Content | 요청이 성공적으로 처리되었지만 제공할 콘텐츠가 없음 |
| **3xx: 리다이렉션** |
| 301 | Moved Permanently | 요청한 리소스의 URI가 영구적으로 변경됨 |
| 302 | Found | 요청한 리소스의 URI가 일시적으로 변경됨 |
| 304 | Not Modified | 클라이언트의 캐시된 리소스가 최신 상태임 |
| **4xx: 클라이언트 오류** |
| 400 | Bad Request | 잘못된 문법으로 인하여 서버가 요청을 이해할 수 없음 |
| 401 | Unauthorized | 인증이 필요한 리소스에 대한 인증 실패 |
| 403 | Forbidden | 서버가 요청을 거부함 |
| 404 | Not Found | 요청한 리소스를 서버에서 찾을 수 없음 |
| **5xx: 서버 오류** |
| 500 | Internal Server Error | 서버에 오류가 발생하여 요청을 수행할 수 없음 |
| 502 | Bad Gateway | 게이트웨이나 프록시 역할을 하는 서버가 upstream 서버로부터 잘못된 응답을 받음 |
| 503 | Service Unavailable | 서버가 일시적으로 서비스를 제공할 수 없음 |

## HTTP start line

HTTP start line은 HTTP 메시지의 첫 번째 줄로, 요청(request)과 응답(response)에 따라 다른 형식을 갖는다.

### 요청(Request)의 start line:

- 형식: `<메서드> <요청 URL> <HTTP 버전>`
- 예시: `GET /index.html HTTP/1.1`

**구성 요소:**
- 메서드: GET, POST, PUT 등 HTTP 메서드
- 요청 URL: 요청하는 리소스의 경로
- HTTP 버전: 사용하는 HTTP 프로토콜 버전

### 응답(Response)의 start line:

- 형식: `<HTTP 버전> <상태 코드> <상태 메시지>`
- 예시: `HTTP/1.1 200 OK`

**구성 요소:**
- HTTP 버전: 사용된 HTTP 프로토콜 버전
- 상태 코드: 요청의 처리 결과를 나타내는 3자리 숫자
- 상태 메시지: 상태 코드에 대한 간단한 설명

## HTTP 헤더

HTTP 헤더의 주요 요소들에 대해 표로 정리했다.

| 헤더 분류 | 헤더 이름 | 설명 |
|-----------|-----------|------|
| **일반 헤더** |
| | Date | 메시지가 생성된 날짜와 시간 |
| | Connection | 클라이언트와 서버 간 연결 방식 지정 |
| **요청 헤더** |
| | Host | 요청하는 호스트와 포트 번호 |
| | User-Agent | 클라이언트 프로그램 정보 |
| | Accept | 클라이언트가 받아들일 수 있는 미디어 타입 |
| | Accept-Language | 클라이언트가 선호하는 언어 |
| | Authorization | 인증 토큰 (JWT 등) |
| **응답 헤더** |
| | Server | 서버 소프트웨어 정보 |
| | Content-Type | 응답 본문의 미디어 타입 |
| | Content-Length | 응답 본문의 길이 |
| | Set-Cookie | 클라이언트에 쿠키 설정 |
| | Location | 리다이랙트 경로 지정 |
| **엔티티 헤더** |
| | Content-Encoding | 본문 콘텐츠의 인코딩 방식 |
| | Content-Language | 본문의 자연 언어 |
| | Expires | 리소스의 만료 일자 |
| **보안 헤더** |
| | Content-Security-Policy | 콘텐츠 보안 정책 |
| | Strict-Transport-Security | HTTPS 사용 강제 |
| | X-XSS-Protection | XSS 공격 방어 |
| **캐시 헤더** |
| | Cache-Control | 캐싱 동작 지정 |
| | ETag | 리소스의 특정 버전에 대한 식별자 |
| **CORS 헤더** |
| | Access-Control-Allow-Origin | 교차 출처 리소스 공유 허용 출처 |
| | Access-Control-Allow-Methods | 허용되는 HTTP 메서드 지정 |

더 자세한 사항은 [여기서](https://gmlwjd9405.github.io/2019/01/28/http-header-types.html) 보면 좋을 듯하다.

### 예시

#### web browser에서 web page 요청 시

**요청 헤더**

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

**응답 헤더**

```
HTTP/1.1 200 OK
Date: Mon, 23 May 2023 12:28:53 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Cache-Control: max-age=3600
```

#### REST API에 데이터 전송 시

**요청 헤더**
```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
User-Agent: PostmanRuntime/7.28.4
Accept: */*
Content-Length: 78
```

**응답 헤더**
```
HTTP/1.1 201 Created
Date: Mon, 23 May 2023 12:30:15 GMT
Server: nginx/1.18.0
Content-Type: application/json
Content-Length: 156
Location: /api/users/12345
Access-Control-Allow-Origin: *
```

## HTTP 특성

### Stateless(무상태성)

**server 입장**

client와 지속적으로 연결을 유지할 필요가 없다. 또한 client의 이전 상태를 저장하고 있을 필요도 없다. 그저 client의 요청에 따라 작업을 처리한 후 response를 해주면 된다. 그래서 **컴퓨터 리소스 부담도 적고, 시스템 확장성도 갖출 수 있고, HA도 높은(고가용성) 엄청난 장점을 갖는다.** 하지만 로그인 상태 유지 등에 대한 추가적인 메커니즘을 구현해야 한다는 단점이 있다.

**client 입장**

client는 확장된 시스템 내에서 아무 server에게나 요청하면 되는 장점이 있다. 하지만 **server들은 이전 상태를 기억하지 못하기 때문에 client는 매 요청 시 모든 문맥을 꾹 눌러 담아서 server에게 전송해야 하므로, overhead가 크다는 심각한 단점이 있다.**

### HTTP Keep-Alive

*이 섹션에서 다루는 Keep-Alive는 HTTP에서의 Keep-Alive에 한정해서 다룬다. TCP Keep-Alive는 OS 단에서 처리하는 방식이며, 순수하게 TCP 연결이 여전히 유효한지 확인하기 위해서 사용된다.*

**HTTP Keep-Alive는 단일 TCP 연결을 유지하면서 TCP 비용 부담없이 http 요청을 보내기 위해 사용하는 기능이다.** TCP 연결을 지속하는 방식을 '지속 커넥션'이라고 부른다. TCP 연결을 유지하면 최초 연결 이후 매 request마다 TCP 3-way-handshake(TCP 연결 요청 시)와 4-way-handshake(TCP 연결 종료 시) 과정을 거칠 필요가 없기 때문에, 통신 부하가 감소한다는 엄청난 장점이 있다. 

다만 커넥션 관리를 잘못하는 경우, 수많은 커넥션이 누적되면서 server 입장에서 stateless의 장점을 잃어버리게 되므로 주의한다.

#### 동작 방식

**요청 header**

HTTP 요청 header의 Connection 속성에 Keep-Alive를 넣어주고 server에게 request를 보낸다. keep-alive 통신 중에는 계속 이 속성을 포함하여 요청해야 끊기지 않고 통신할 수 있다.

**응답 header**

- Connection:
  - keep-alive: HTTP keep-alive 통신이 가능
  - Close: 지금부터 HTTP Keep-alive 통신 종료. 또는 keep-alive 통신 지원 안함 선언.
- Keep-Alive:
  - timeout: n초 동안 지속
  - max: 최대 m회 요청까지 허용.

#### 예시

**요청 header**
```
GET /index.html HTTP/1.1
Host: www.example.com
Connection: keep-alive
...
```

**응답 header**
```
HTTP/1.1 200 OK
Date: Mon, 23 May 2023 12:28:53 GMT
Server: Apache/2.4.41 (Ubuntu)
Connection: keep-alive
Keep-Alive: timeout=5, max=1000
...
```
이 구조에서는 이제부터 5초 동안 최대 1000회 요청만큼 동일한 TCP 통신을 유지하면서 HTTP 통신이 가능해진다.

### HTTP pipelining

**HTTP pipelining은 HTTP/1.1에서 도입된 기능으로, 하나의 TCP 연결을 통해 여러 HTTP request를 연속으로 보내고 그에 대한 response를 순차적으로 받는 방식이다.** 이론적으로 server의 response를 동기적으로 기다리지 않고 여러 request들을 연달아 전송할 수 있기 때문에, 네트워크 지연 시간을 줄이고 전체적인 page 로딩 속도를 향상시키는 엄청난 장점이 있다. 하지만 실질적으로 Head-Of-Line(HOL) blocking 문제로 인해 실제로는 거의 사용되지 않았으며, browser에서도 기본적으로 비활성화된 기능이다.

#### Head-Of-Line(HOL) blocking

**server 입장에서 request를 순차적으로 그리고 동기적으로 처리해서 발생하는 문제.** server에서 먼저 보낸 request의 처리가 지연되면 뒤따르는 모든 request들은 기다리는 수밖에 없다. 결국 pipelining을 쓰나 안쓰나 성능 기대값은 비슷하다는 것이다.

#### 개선 방안

HTTP/2에서 '멀티플랙싱 알고리즘'을 적용하여 HOL bloking 문제를 해결했다. **멀티플랙싱 알고리즘을 적용하면 각 request와 response는 별도의 stream으로 처리할 수 있는데, stream들은 서로 독립적으로 동작하므로 한 stream이 다른 stream에 어떠한 영향도 주지 않는다.** 게다가 stream 간 우선순위 설정도 가능하여 중요 resource를 먼저 처리할 수 있다. 이렇게 되면 request 순서와 관계없이 준비된 응답부터 client에게 response할 수 있고, 이렇게 비동기적으로 응답을 함으로써 비로소 전체적인 page 속도 향상을 기대할 수 있게 됐다.

## HTTP 버전별 특징

### HTTP/0.9

단순히 HTML 파일만 전송 가능하며, GET 메소드만 지원했다. HTTP 헤더와 상태 코드도 존재하지 않은 원시적인 기술이다. 각 request는 독립적으로 처리됐으며, server는 이전 요청에 대한 정보를 저장하지 않는 stateless 특성을 갖게 됐다.

### HTTP/1.0

처음으로 HTTP 헤더와 상태 코드가 도입되었으며, 이전 버전과 구분하기 위해 버전 정보를 포함하기 시작했다. 그리고 GET, POST, HEAD 메소드를 지원하며, 다양한 파일 형식을 전송할 수 있게 개선되었다. 그러나 이 버전은 공식 표준이 아니었으며, stateless 특성으로 인한 문제(비효율적인 통신 방식, 상태 유지 불가)가 대두되기 시작하여 여러 문제들을 해결할 필요가 있었다.

### HTTP/1.1

1997년 초에 처음으로 HTTP의 공식 버전이 출시된 버전이다. 이 버전에서 HTTP의 뜨거운 감자였던 **stateless 문제를 해결하기 위한 'Persistent Connection(지속 연결)' 기술을 도입하여, 하나의 TCP 연결을 유지하여 여러 request와 response를 처리할 수 있게 만들어 효율적으로 통신할 수 있도록 개선했다.** 이외에도 추가 메소드 도입(PUT, DELETE 등), Host header 필수화, chunk 전송 encoking, 개선된 cache 제어 메커니즘 등 다양한 보조 기능들이 추가되어 web 성능과 기능성이 크게 향상됐다. 한편 pipelining 기능을 도입하여 지속 연결 상태에서 비동기적인 HTTP 통신을 구현하려고 했으나, HOL blocking 문제로 인해 실질적으로 사용되지는 못했다.

### HTTP/2

2015년 HTTP/1.1의 성능 개선에 중점을 둔 신규 버전인 HTTP/2가 등장했다. **기존 pipelining의 HOL blocking 문제를 해결하는 '멀티플렉싱 알고리즘'을 도입하여, 지속 연결 상태에서 비동기적인 request-response 처리가 가능하도록 개선했다.** 뿐만 아니라 text 기반 프로토콜에서 binary protocol로 변경하여 메시지 크기를 줄임으로써 속도를 개선했다. 

### HTTP/3

2020년 'QUIC(Quick UDP Internet Connection)' 기반의 HTTP 통신 방식인 HTTP/3 버전이 출시됐다. TCP는 데이터가 정상적으로 들어갔는지 순서는 맞는지 일일이 확인해야 해서 통신 속도가 느리다는 것이 단점인데, UDP는 그런 절차가 최소화되어 있기 때문에 TCP에 비해 매우 빠르다는 엄청난 장점이 있다. 다만 **UDP는 절차가 단순한만큼 빠르지만 통신의 신뢰성이 떨어지는데, QUIC에서는 중간에 데이터 손실이 발생해도 이를 개별적으로 재전송할 수 있게 개선하여 신뢰성까지 챙겼다.** 