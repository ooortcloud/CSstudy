# HTTP 통신의 특성
HTTP(하이퍼텍스트 전송 프로토콜)의 주요 특성은 클라이언트와 서버 간의 통신 방식을 정의하는 데 있어 중요한 역할을 합니다. HTTP는 웹 기반 애플리케이션에서 자주 사용되며, 여러 가지 특성을 가지고 있습니다.

### **비연결성 (Statelessness)**
HTTP는 **비연결성**을 특징으로 합니다. 즉, 각 요청과 응답은 독립적으로 처리되며, 이전 요청과 다음 요청 간의 연결 상태를 유지하지 않습니다. 서버는 클라이언트의 상태 정보를 저장하지 않고, 클라이언트가 새로운 요청을 보낼 때마다 그 요청이 처음 요청인 것처럼 처리합니다. 따라서 클라이언트는 필요한 모든 정보를 각 요청에 포함시켜야 합니다.

이로 인해 서버는 동시에 많은 클라이언트의 요청을 처리할 수 있으며, 불필요한 연결 유지로 인한 자원 낭비를 줄일 수 있습니다. 그러나 클라이언트의 상태를 유지해야 하는 경우에는 **쿠키**나 **세션** 같은 기술을 통해 이를 보완할 수 있습니다.

### **무상태성 (Connectionless)**
HTTP는 **무상태성** 프로토콜입니다. 즉, 클라이언트가 서버에 요청을 보내고, 서버는 해당 요청에 대한 응답을 한 후 연결을 끊습니다. 각 요청은 독립적이며, 서버는 이전 요청의 상태를 기억하지 않습니다. 이것은 서버의 자원을 효율적으로 사용할 수 있지만, 상태 유지가 필요한 경우 추가적인 작업이 필요합니다.

- **예:** 온라인 쇼핑몰에서 사용자가 장바구니에 물건을 추가할 때마다 서버는 사용자의 상태를 알지 못합니다. 따라서 쿠키, 세션 또는 토큰 등을 사용하여 상태를 유지해야 합니다.

### **확장성 (Extensibility)**
HTTP는 매우 **확장 가능한** 프로토콜입니다. 다양한 종류의 데이터를 주고받을 수 있으며, 클라이언트와 서버 간에 서로 맞춤형 기능을 추가하기 쉬운 구조를 가지고 있습니다. 특히 헤더 필드가 확장 가능하며, 커스텀 헤더를 정의할 수 있습니다. 이를 통해 보안, 성능 향상, 캐시 제어 등 다양한 기능을 추가할 수 있습니다.

### **유연한 데이터 표현 (Flexible Data Representation)**
HTTP는 **유연한 데이터 표현**을 지원합니다. 요청과 응답의 본문은 텍스트, HTML, JSON, XML, 이미지, 동영상 등 다양한 형태의 데이터를 포함할 수 있습니다. 이를 통해 다양한 형식의 리소스에 접근하고, 클라이언트와 서버 간에 자유롭게 데이터를 주고받을 수 있습니다.

### **HTTP 메서드의 다양성**
HTTP는 여러 가지 **메서드(GET, POST, PUT, DELETE 등)**를 통해 다양한 방식으로 서버와 통신할 수 있습니다. 각 메서드는 특정한 작업을 의미하며, 이를 통해 리소스를 조회, 생성, 수정, 삭제 등의 작업을 할 수 있습니다.

- **GET:** 서버로부터 데이터를 조회
- **POST:** 데이터를 서버에 전송하여 처리 요청
- **PUT:** 데이터를 서버에 저장하거나 업데이트
- **DELETE:** 리소스를 삭제

### **클라이언트-서버 모델 (Client-Server Model)**
HTTP는 **클라이언트-서버 모델**을 따릅니다. 클라이언트는 요청을 보내고, 서버는 요청을 처리하여 응답을 보냅니다. 이 모델을 통해 클라이언트와 서버는 서로 독립적으로 동작하며, 클라이언트는 UI와 같은 사용자 인터페이스 부분에 집중하고, 서버는 데이터 관리나 로직 처리를 담당할 수 있습니다.

### **TCP/IP 기반**
HTTP는 **TCP/IP 프로토콜** 위에서 동작합니다. 이는 안정적인 연결을 보장하며, 데이터의 전송 중 손실을 최소화합니다. HTTP 요청과 응답은 TCP/IP 연결을 통해 주고받으며, 주로 80번(HTTP) 또는 443번(HTTPS) 포트를 사용합니다.



# HTTP 통신 방식

HTTP(하이퍼텍스트 전송 프로토콜)은 웹상에서 클라이언트(주로 웹 브라우저)와 서버 간에 데이터를 주고받기 위해 사용되는 프로토콜입니다. HTTP는 요청(Request)과 응답(Response)의 형식으로 통신을 수행하며, 주로 TCP/IP 프로토콜 위에서 동작합니다. HTTP 통신 방식은 매우 직관적이며, 여러 가지 메서드와 상태 코드를 통해 데이터 전송의 상태를 알 수 있습니다.

### HTTP 요청(Request)
클라이언트가 서버에 데이터를 요청할 때 사용하는 형식입니다. HTTP 요청은 다음과 같은 구성 요소로 이루어져 있습니다.

- **요청 라인(Request Line):** HTTP 메서드, 요청 URL, 그리고 HTTP 버전을 포함합니다.
  - 예: `GET /index.html HTTP/1.1`
- **헤더(Header):** 클라이언트가 서버에게 전달하는 추가적인 정보입니다. 예를 들어, `User-Agent`, `Content-Type` 등이 있습니다.
- **본문(Body):** POST, PUT 같은 메서드에서 주로 사용되며, 서버로 전송할 데이터를 포함합니다.

### HTTP 응답(Response)
서버가 클라이언트의 요청에 대한 응답을 보내는 방식입니다. 응답은 다음과 같은 구성 요소를 가집니다.

- **상태 라인(Status Line):** HTTP 버전, 상태 코드, 그리고 상태 메시지를 포함합니다.
  - 예: `HTTP/1.1 200 OK`
- **헤더(Header):** 서버에서 클라이언트에게 제공하는 추가 정보입니다.
  - 예: `Content-Type: text/html`
- **본문(Body):** 요청된 리소스가 포함되며, HTML, JSON, XML 등의 데이터가 담깁니다.

### HTTP 메서드
HTTP는 다양한 메서드를 사용하여 클라이언트와 서버 간의 상호작용을 정의합니다.

- **GET:** 서버로부터 리소스를 요청하는 메서드입니다. 주로 데이터를 조회할 때 사용합니다.
- **POST:** 서버에 데이터를 제출할 때 사용됩니다. 예를 들어, 양식을 제출할 때 사용됩니다.
- **PUT:** 서버에 데이터를 갱신하거나 새로 저장할 때 사용됩니다.
- **DELETE:** 서버에서 리소스를 삭제할 때 사용됩니다.
- **HEAD:** GET과 비슷하지만, 응답 본문을 제외하고 헤더만 받습니다.
- **OPTIONS:** 서버가 지원하는 HTTP 메서드를 확인할 때 사용됩니다.

### 상태 코드(Status Codes)
서버는 요청에 대한 응답으로 상태 코드를 반환하며, 이는 요청이 성공했는지 또는 오류가 발생했는지 등을 나타냅니다.

- **2xx (성공):** 요청이 성공했음을 의미합니다.
  - 200 OK: 요청 성공
  - 201 Created: 리소스 생성 성공
- **3xx (리다이렉션):** 요청된 리소스가 다른 위치로 이동했음을 의미합니다.
  - 301 Moved Permanently: 리소스가 영구적으로 이동됨
  - 302 Found: 임시적으로 다른 URL로 리소스를 찾음
- **4xx (클라이언트 오류):** 클라이언트 측에서 잘못된 요청을 보냈음을 의미합니다.
  - 400 Bad Request: 잘못된 요청
  - 404 Not Found: 요청한 리소스를 찾을 수 없음
- **5xx (서버 오류):** 서버 측에서 오류가 발생했음을 의미합니다.
  - 500 Internal Server Error: 서버 내부 오류
  - 503 Service Unavailable: 서버가 일시적으로 처리 불가

### 5. HTTP/1.1과 HTTP/2
- **HTTP/1.1:** 가장 널리 사용되는 HTTP 프로토콜 버전으로, Keep-Alive 기능을 지원해 하나의 TCP 연결을 여러 요청에 재사용할 수 있습니다.
- **HTTP/2:** 성능 향상을 위해 도입된 프로토콜로, 요청과 응답을 멀티플렉싱하여 동시에 여러 요청을 처리할 수 있습니다. 헤더 압축 등의 기능도 제공하여 전송 효율성을 높였습니다.
