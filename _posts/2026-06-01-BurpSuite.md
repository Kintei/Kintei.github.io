---
layout: single
title: "[모의해킹] 04. Burp 및 Proxy"
date: 2026-06-01
categories:
  - security study
tags:
  - Burp_Suite
  - Web_Proxy
---
# Burp Suite와 Web Proxy 정리

## 1. Web Proxy란?

### 1.1 Proxy의 기본 개념

Proxy는 클라이언트와 서버 사이에서 요청과 응답을 대신 전달해 주는 중간 서버 또는 중계 도구를 의미한다.

일반적인 웹 통신은 사용자의 브라우저가 웹 서버에 직접 요청을 보내고, 웹 서버가 다시 브라우저로 응답을 반환하는 방식으로 이루어진다.

```text
Web Browser  →  Web Server
Web Browser  ←  Web Server
```

하지만 Proxy를 사용하면 브라우저와 웹 서버 사이에 Proxy Tool이 위치하게 된다.

![Proxy 설명](../../../../images/Proxy_Tool_Description.png)

위 이미지처럼 Web과 Web Server 사이에 Proxy Tool이 들어가면 요청과 응답은 다음과 같은 흐름으로 이동한다.

```text
Web Browser  →  Proxy Tool  →  Web Server
Web Browser  ←  Proxy Tool  ←  Web Server
```

즉, 브라우저가 서버로 직접 요청을 보내는 것이 아니라 먼저 Proxy Tool로 요청을 보내고, Proxy Tool이 다시 Web Server로 요청을 전달한다. 서버의 응답도 마찬가지로 Proxy Tool을 거쳐 브라우저로 돌아온다.

보안 분석에서는 이 중간 위치를 이용하여 HTTP 요청과 응답을 확인하거나 수정할 수 있다.

예를 들어 사용자가 로그인 버튼을 눌렀을 때 브라우저가 서버로 어떤 ID, 비밀번호, 쿠키, 토큰 값을 보내는지 Proxy Tool을 통해 확인할 수 있다.

---

### 1.2 Forward Proxy와 Reverse Proxy 차이

Proxy는 위치와 목적에 따라 Forward Proxy와 Reverse Proxy로 나눌 수 있다.

#### Forward Proxy

Forward Proxy는 클라이언트 쪽에 가까운 Proxy이다.

사용자의 브라우저나 프로그램이 인터넷 서버에 직접 접근하지 않고, Forward Proxy를 통해 외부 서버에 접근한다.

```text
Client  →  Forward Proxy  →  Web Server
```

주요 목적은 다음과 같다.

* 내부 사용자의 외부 인터넷 접근 제어
* 특정 사이트 접속 차단
* 캐싱을 통한 속도 향상
* 사용자의 실제 IP 숨김
* 보안 장비를 통한 트래픽 검사
* 웹 요청과 응답 분석

Burp Suite의 Proxy는 브라우저와 웹 서버 사이에 위치하여 요청과 응답을 중간에서 확인하므로 Forward Proxy 방식에 가깝다.

#### Reverse Proxy

Reverse Proxy는 서버 쪽에 가까운 Proxy이다.

외부 사용자가 서버에 접근할 때 실제 웹 서버 앞단에서 요청을 받아 내부 서버로 전달한다.

```text
Client  →  Reverse Proxy  →  Internal Web Server
```

주요 목적은 다음과 같다.

* 실제 서버 IP 숨김
* 로드 밸런싱
* SSL/TLS 종료
* 웹 방화벽 적용
* 캐싱 및 성능 최적화
* 여러 내부 서버로 요청 분산

예를 들어 Nginx, Apache, Cloudflare, AWS ALB 같은 구성은 Reverse Proxy 역할을 할 수 있다.

#### 차이 정리

| 구분      | Forward Proxy          | Reverse Proxy            |
| ------- | ---------------------- | ------------------------ |
| 위치      | 클라이언트 앞단               | 서버 앞단                    |
| 보호 대상   | 클라이언트                  | 서버                       |
| 주 사용 목적 | 사용자의 요청 제어, 익명성, 분석    | 서버 보호, 부하 분산, 보안         |
| 예시      | Burp Proxy, 사내 인터넷 프록시 | Nginx, Cloudflare, 로드밸런서 |

---

### 1.3 Web Proxy가 HTTP/HTTPS 요청을 중간에서 보는 방식

HTTP는 암호화되지 않은 평문 통신이다. 따라서 Proxy가 중간에 있으면 요청과 응답 내용을 그대로 확인할 수 있다.

예시는 다음과 같다.

```http
GET /login HTTP/1.1
Host: example.com
Cookie: session=abc123
```

하지만 HTTPS는 TLS 암호화를 사용하기 때문에 일반적인 Proxy는 내부 내용을 바로 볼 수 없다.

Burp Suite 같은 웹 보안 분석용 Proxy는 HTTPS 트래픽을 확인하기 위해 Burp의 CA 인증서를 브라우저나 기기에 설치한다.

그러면 브라우저는 Burp를 신뢰할 수 있는 인증기관처럼 인식하고, Burp는 브라우저와 서버 사이에서 각각 암호화 연결을 맺는다.

```text
Browser  ⇄  Burp Proxy  ⇄  Web Server
```

이 구조에서 Burp는 브라우저가 보낸 요청을 복호화해서 보여주고, 사용자가 요청을 수정하면 수정된 요청을 다시 서버로 전달할 수 있다.

즉, Web Proxy는 단순히 요청을 전달하는 역할뿐만 아니라, 보안 테스트 환경에서는 요청과 응답을 분석하고 변조하는 도구로도 사용된다.

---

### 1.4 Web Proxy를 사용하는 이유

Web Proxy를 사용하는 이유는 다음과 같다.

#### 1. 요청과 응답 확인

브라우저 화면에서는 보이지 않는 실제 HTTP 요청과 응답을 확인할 수 있다.

확인 가능한 정보는 다음과 같다.

* HTTP Method
* URL
* Query String
* Request Header
* Cookie
* Authorization Header
* POST Body
* JSON 데이터
* Response Header
* Response Body
* 상태 코드

#### 2. 요청 수정

클라이언트가 서버로 보내는 값을 중간에서 수정할 수 있다.

예를 들어 다음과 같은 값을 변경해 볼 수 있다.

* 사용자 ID
* 게시글 번호
* 상품 가격
* 권한 값
* 쿠키
* 토큰
* 검색어
* JSON Body 값

이를 통해 서버가 클라이언트에서 전달한 값을 그대로 신뢰하는지, 서버 측 검증을 제대로 수행하는지 확인할 수 있다.

#### 3. 보안 테스트

Web Proxy는 웹 취약점 분석에 자주 사용된다.

예를 들어 다음과 같은 취약점 테스트에 활용할 수 있다.

* 인증 우회
* 접근 제어 취약점
* IDOR
* SQL Injection
* XSS
* CSRF
* 파일 업로드 취약점
* 세션 관리 문제
* 입력값 검증 문제

#### 4. 웹 개발 디버깅

개발 중 프론트엔드와 백엔드 사이에서 실제로 어떤 데이터가 오가는지 확인할 수 있다.

예를 들어 API 요청이 정상적으로 보내졌는지, 서버 응답이 예상과 다른 이유가 무엇인지 확인할 수 있다.

#### 5. 모바일 앱 트래픽 분석

모바일 기기의 Wi-Fi Proxy 설정을 Burp가 실행 중인 PC로 지정하면 모바일 앱이 서버와 통신하는 내용을 확인할 수 있다.

단, HTTPS 트래픽을 확인하려면 모바일 기기에도 Burp CA 인증서를 설치해야 한다.

---

### 1.5 Web Proxy 사용 시 주의점

Web Proxy는 요청과 응답을 중간에서 확인하고 수정할 수 있는 강력한 도구이다. 따라서 반드시 허가된 환경에서만 사용해야 한다.

주의할 점은 다음과 같다.

* 본인 소유 또는 명시적으로 허가받은 서비스에서만 테스트한다.
* 다른 사람의 계정, 세션, 개인정보가 포함된 트래픽을 무단으로 수집하지 않는다.
* 회사나 학교 네트워크에서 사용할 경우 내부 정책을 확인한다.
* Burp CA 인증서를 설치한 브라우저나 기기는 관리에 주의한다.
* 테스트가 끝난 뒤 불필요한 Proxy 설정은 원래대로 되돌린다.
* 테스트가 끝난 뒤 불필요한 CA 인증서는 제거하는 것이 좋다.
* 실제 운영 서비스에 과도한 요청을 보내지 않는다.
* Scope를 설정하여 허가받은 대상만 테스트한다.

특히 Intruder 같은 자동화 기능은 짧은 시간에 많은 요청을 보낼 수 있으므로 실습 환경, CTF, 허가받은 진단 대상에서만 사용해야 한다.

---

## 2. Burp Suite란?

### 2.1 Burp Suite 개요

Burp Suite는 PortSwigger에서 만든 웹 애플리케이션 보안 테스트 도구이다.

웹 브라우저와 웹 서버 사이에 Proxy로 위치하여 HTTP/HTTPS 요청과 응답을 확인하고, 수정하고, 반복 전송하고, 자동화된 테스트를 수행할 수 있다.

Burp Suite는 웹 취약점 진단, 모의해킹, CTF, 웹 보안 학습에서 많이 사용된다.

대표적인 기능은 다음과 같다.

* Proxy
* Intercept
* HTTP History
* Repeater
* Intruder
* Decoder
* Comparer
* Sequencer
* Extensions
* Collaborator

Burp Suite의 핵심은 브라우저와 서버 사이에 위치하여 통신 내용을 직접 확인할 수 있다는 점이다.

```text
Browser  →  Burp Suite  →  Web Server
```

---

### 2.2 Burp Suite를 사용하는 이유

Burp Suite를 사용하는 가장 큰 이유는 웹 요청과 응답을 직접 확인하고 조작할 수 있기 때문이다.

브라우저 화면에서는 버튼, 입력창, 페이지 결과만 보이지만 실제 내부에서는 HTTP 요청과 응답이 오간다.

Burp Suite를 사용하면 다음과 같은 내용을 확인할 수 있다.

* 로그인 요청에 포함된 ID와 비밀번호
* 서버가 발급하는 세션 쿠키
* API 요청에 포함된 JSON 값
* 게시글 작성 시 전달되는 파라미터
* 파일 업로드 요청 구조
* 권한 확인에 사용되는 토큰
* 서버 응답 코드와 에러 메시지

또한 요청 값을 직접 수정해서 서버의 검증 로직을 확인할 수 있다.

예를 들어 게시글 조회 요청이 다음과 같다고 가정한다.

```http
GET /board/view?id=100 HTTP/1.1
Host: test.example.com
Cookie: session=abc123
```

Burp에서 `id=100`을 `id=101`로 바꿔 보낼 수 있다.
이때 서버가 권한 검사를 제대로 하지 않으면 다른 사용자의 게시글이 조회될 수 있다.

이처럼 Burp Suite는 웹 애플리케이션이 클라이언트 입력값을 어떻게 처리하는지 확인하는 데 유용하다.

---

### 2.3 Burp Suite Community VS Professional

Burp Suite는 대표적으로 Community Edition과 Professional Edition으로 나뉜다.

#### Burp Suite Community Edition

Community Edition은 무료로 사용할 수 있는 버전이다.

주요 특징은 다음과 같다.

* 기본 Proxy 기능 사용 가능
* Intercept 사용 가능
* HTTP History 확인 가능
* Repeater 사용 가능
* Decoder 사용 가능
* Comparer 사용 가능
* Sequencer 일부 기능 사용 가능
* 학습, CTF, 기본 실습에 적합
* Intruder 기능에 속도 제한이 있음
* 자동 취약점 스캐너 기능은 제한적이거나 제공되지 않음

처음 Burp Suite를 배우거나 CTF 문제를 풀 때는 Community Edition으로도 충분하다.

#### Burp Suite Professional

Professional Edition은 유료 버전이다.

주요 특징은 다음과 같다.

* Community 기능 포함
* 자동 취약점 스캐너 사용 가능
* Intruder 속도 제한 완화
* 고급 분석 기능 제공
* Collaborator 기능 활용 가능
* 업무용 웹 취약점 진단에 적합
* 반복 작업 자동화에 유리
* 확장 기능 활용에 유리

실제 웹 취약점 진단 업무에서는 Burp Suite Professional이 자주 사용된다. 자동 스캐너, 빠른 Intruder, 프로젝트 관리 등 업무 효율을 높이는 기능이 있기 때문이다.

하지만 Professional Edition이 반드시 정답인 것은 아니다. 학습, CTF, 개인 실습, 커스텀 자동화 관점에서는 Community Edition과 Python 스크립트를 함께 사용하는 방식도 매우 유용하다. 특히 Intruder처럼 반복 요청을 보내는 기능은 Python으로 직접 구현할 수 있으며, 이 과정에서 HTTP 요청 구조, 인증 흐름, 세션 처리, 응답 비교 로직을 더 깊게 이해할 수 있다.

따라서 처음 학습할 때는 Community Edition으로 요청과 응답 구조를 익히고, 반복 작업이나 특수한 테스트는 Python으로 직접 자동화해 보는 것이 실력 향상에 도움이 된다. 실무에서는 시간 효율, 자동 스캐너, 프로젝트 관리 기능이 필요할 때 Professional Edition을 사용하는 방식으로 구분할 수 있다.

#### 정리

| 구분       | Community      | Professional  |
| -------- | -------------- | ------------- |
| 비용       | 무료             | 유료            |
| Proxy    | 사용 가능          | 사용 가능         |
| Repeater | 사용 가능          | 사용 가능         |
| Intruder | 제한 있음          | 제한 적음         |
| Scanner  | 제한적            | 사용 가능         |
| 추천 용도    | 학습, CTF, 기본 실습 | 실무 진단, 자동화 분석 |

---

## 3. Burp Proxy 설정 방법

### 3.1 Burp 자체 브라우저 사용 방법

Burp Suite에는 자체 Chromium 기반 브라우저가 포함되어 있다.

가장 간단한 방법은 Burp를 실행한 뒤 자체 브라우저를 사용하는 것이다.

```text
Proxy → Intercept → Open Browser
```

기본 흐름은 다음과 같다.

1. Burp Suite 실행
2. Proxy 탭 이동
3. Intercept 탭 확인
4. Open Browser 클릭
5. 열린 브라우저에서 테스트 대상 사이트 접속
6. Burp의 HTTP History 또는 Intercept에서 요청 확인

Burp 자체 브라우저를 사용하면 별도의 브라우저 Proxy 설정이나 인증서 설치 과정을 줄일 수 있다.

따라서 처음 Burp를 실습할 때는 자체 브라우저를 사용하는 것이 가장 편하다.

---

### 3.2 외부 브라우저 수동 프록시 설정 방법

Chrome, Firefox, Edge 같은 외부 브라우저를 Burp와 연결하려면 브라우저의 Proxy 설정을 수동으로 변경해야 한다.

Burp의 기본 Proxy Listener는 보통 다음 주소를 사용한다.

```text
127.0.0.1:8080
```

브라우저의 Proxy 설정을 다음과 같이 지정한다.

```text
HTTP Proxy: 127.0.0.1
Port: 8080

HTTPS Proxy: 127.0.0.1
Port: 8080
```

설정 후 브라우저에서 웹사이트에 접속하면 요청이 Burp를 거쳐 이동한다.

```text
Browser  →  Burp Proxy  →  Web Server
```

테스트가 끝난 뒤에는 브라우저 Proxy 설정을 원래대로 되돌려야 한다.

Proxy 설정을 그대로 두고 Burp를 종료하면 브라우저가 계속 `127.0.0.1:8080`으로 요청을 보내려고 하기 때문에 인터넷 접속이 되지 않을 수 있다.

---

### 3.3 모바일 앱 트래픽 분석을 위한 수동 프록시 설정

Burp 자체 브라우저는 PC 웹 요청을 분석할 때 편리하지만, 모바일 앱의 요청을 확인하려면 스마트폰에서 직접 Burp Proxy를 바라보도록 수동 프록시 설정을 해야 한다.

기본 흐름은 다음과 같다.

1. PC와 스마트폰을 같은 네트워크에 연결한다.
2. Burp Proxy Listener가 PC의 특정 IP와 포트에서 대기하도록 설정한다.
3. 스마트폰 Wi-Fi 설정에서 프록시 서버를 PC IP와 Burp 포트로 지정한다.
4. 스마트폰에 Burp CA 인증서를 설치한다.
5. 앱 실행 후 발생하는 HTTP/HTTPS 요청을 Burp Proxy History에서 확인한다.

단, HTTPS 앱 트래픽의 경우 인증서 고정, 즉 Certificate Pinning이 적용되어 있으면 Burp CA 인증서를 설치해도 요청이 정상적으로 보이지 않을 수 있다.


---

### 3.4 Burp Proxy Listener 설정

Proxy Listener는 Burp가 클라이언트의 요청을 받아들이는 주소와 포트를 의미한다.

기본값은 보통 다음과 같다.

```text
Interface: 127.0.0.1
Port: 8080
```

`127.0.0.1`은 로컬호스트 주소이므로 같은 PC에서만 접근할 수 있다.

외부 모바일 기기에서 Burp로 연결하려면 Listener를 모든 인터페이스에서 접근 가능하도록 설정해야 할 수 있다.

```text
Bind to address: All interfaces
Port: 8080
```

이렇게 설정하면 같은 네트워크에 있는 모바일 기기가 PC의 IP 주소와 포트를 통해 Burp에 접근할 수 있다.

```text
Mobile Device  →  PC IP:8080  →  Burp Proxy
```

주의할 점은 모든 인터페이스로 열면 같은 네트워크의 다른 기기도 접근할 수 있다는 것이다.
따라서 신뢰할 수 있는 네트워크에서만 사용해야 한다.

---

### 3.5 Burp CA 인증서 설치 이유

HTTPS는 TLS 암호화를 사용하므로 일반 Proxy는 요청과 응답 내용을 볼 수 없다.

Burp가 HTTPS 트래픽을 복호화해서 보여주려면 브라우저 또는 기기가 Burp의 CA 인증서를 신뢰해야 한다.

인증서를 설치하지 않으면 다음과 같은 문제가 발생할 수 있다.

* HTTPS 사이트 접속 시 인증서 오류 발생
* 브라우저에서 보안 경고 표시
* 요청은 보이지만 HTTPS 내용이 정상적으로 표시되지 않음
* 모바일 앱 HTTPS 트래픽 분석 실패

Burp CA 인증서는 보통 Burp Proxy를 실행한 상태에서 브라우저로 다음 주소에 접속해 받을 수 있다.

```text
http://burp
```

이후 브라우저 또는 운영체제의 신뢰할 수 있는 루트 인증기관에 인증서를 등록한다.

테스트가 끝난 뒤 더 이상 필요하지 않으면 인증서를 제거하는 것이 좋다.

---

### 3.6 HTTPS 트래픽이 보이지 않을 때 확인할 것

HTTPS 트래픽이 Burp에 보이지 않을 때는 다음 항목을 확인한다.

#### 1. 브라우저 Proxy 설정 확인

브라우저가 Burp의 Listener 주소와 포트로 요청을 보내고 있는지 확인한다.

```text
127.0.0.1:8080
```

#### 2. Burp Proxy Listener 상태 확인

Burp의 Proxy Listener가 켜져 있는지 확인한다.

#### 3. Intercept 상태 확인

Intercept가 켜져 있으면 요청이 Burp에서 멈춘다.

브라우저가 계속 로딩 중이라면 Intercept에서 요청이 대기 중인지 확인하고, 필요하면 `Forward`를 누르거나 `Intercept off`로 변경한다.

#### 4. CA 인증서 설치 여부 확인

HTTPS 내용을 확인하려면 Burp CA 인증서가 브라우저 또는 기기에 설치되어 있어야 한다.

#### 5. 브라우저별 인증서 저장소 확인

Firefox는 운영체제 인증서 저장소와 별도로 자체 인증서 저장소를 사용할 수 있다.
따라서 Firefox를 사용할 경우 Burp CA 인증서를 Firefox에 별도로 설치해야 할 수 있다.

#### 6. 모바일 기기와 PC 네트워크 확인

모바일 분석 시에는 PC와 모바일 기기가 같은 Wi-Fi에 연결되어 있어야 한다.

또한 PC 방화벽이 Burp Listener 포트, 예를 들어 8080 포트를 막고 있지 않은지 확인한다.

#### 7. Certificate Pinning 여부 확인

일부 모바일 앱이나 프로그램은 Certificate Pinning을 사용한다.

이 경우 Burp CA 인증서를 설치해도 앱이 Burp 인증서를 신뢰하지 않아 HTTPS 트래픽이 보이지 않을 수 있다.

---

### 3.7 Scope 설정

Scope는 Burp에서 테스트 대상 범위를 지정하는 기능이다.

웹 보안 테스트를 할 때 모든 사이트의 요청을 분석하면 불필요한 트래픽이 너무 많이 쌓인다.

예를 들어 브라우저는 테스트 대상 사이트뿐만 아니라 검색 엔진, 광고, 이미지 서버, CDN, 확장 프로그램 관련 요청도 함께 보낼 수 있다.

따라서 테스트 대상 도메인만 Scope에 넣고 나머지는 제외하는 것이 좋다.

예시는 다음과 같다.

```text
Target Scope:
https://test.example.com
```

Scope를 설정하면 다음과 같은 장점이 있다.

* 테스트 대상 요청만 필터링 가능
* HTTP History 관리가 쉬워짐
* 불필요한 외부 요청 제거 가능
* 실수로 다른 사이트를 테스트하는 것을 방지
* Scanner나 Intruder 사용 시 범위 통제 가능
* 허가된 범위 안에서만 테스트하기 쉬움

보안 테스트에서는 Scope를 명확히 설정하는 것이 중요하다.
허가받은 범위를 벗어난 요청 수정이나 자동화 테스트는 문제가 될 수 있다.

---

## 4. Burp 주요 기능

### 4.1 Proxy - Intercept

Intercept는 브라우저와 서버 사이의 HTTP 요청 또는 응답을 중간에서 멈추는 기능이다.

요청이 멈춘 상태에서 사용자는 다음 작업을 할 수 있다.

* 요청 내용 확인
* 파라미터 수정
* Cookie 수정
* Header 수정
* Body 수정
* 요청을 서버로 전달
* 요청을 버림

기본 흐름은 다음과 같다.

```text
Browser → Burp Intercept → Web Server
```

Intercept에서 자주 사용하는 버튼은 다음과 같다.

#### Forward

현재 멈춰 있는 요청을 서버로 전달한다.

#### Drop

현재 요청을 버린다.

#### Intercept on/off

Intercept 기능을 켜거나 끈다.

실습 중 브라우저가 멈춘 것처럼 보이면 Intercept가 켜져 있는지 확인해야 한다.
요청을 수정할 때만 Intercept를 켜고, 일반 탐색 중에는 꺼두어야 사용이 편하다.

---

### 4.2 Proxy - HTTP History

HTTP History는 Burp를 통해 지나간 HTTP 요청과 응답 기록을 보여주는 기능이다.

브라우저에서 웹사이트를 탐색하면 요청이 자동으로 기록된다.

HTTP History에서 확인할 수 있는 정보는 다음과 같다.

* 요청 URL
* HTTP Method
* 상태 코드
* 요청 길이
* 응답 길이
* MIME Type
* 요청 Header
* 응답 Header
* Cookie
* Request Body
* Response Body

HTTP History는 웹 애플리케이션의 구조를 파악할 때 유용하다.

예를 들어 로그인, 게시글 작성, 파일 업로드, 검색, 댓글 작성 같은 기능을 사용한 뒤 History를 보면 어떤 API가 호출되었는지 확인할 수 있다.

또한 특정 요청을 우클릭하여 Repeater, Intruder 등 다른 도구로 보낼 수 있다.

```text
HTTP History 요청 선택 → 우클릭 → Send to Repeater
HTTP History 요청 선택 → 우클릭 → Send to Intruder
```

---

### 4.3 Repeater

Repeater는 하나의 HTTP 요청을 여러 번 수정하고 반복 전송하면서 서버 응답을 비교하는 도구이다.

주로 다음과 같은 테스트에 사용한다.

* 파라미터 값 변경
* 인증 값 변경
* Cookie 변경
* 권한 검증 확인
* 입력값 필터링 확인
* 에러 메시지 확인
* SQL Injection 테스트
* XSS 테스트
* API 요청 분석

기본 사용 흐름은 다음과 같다.

1. HTTP History에서 테스트할 요청 선택
2. 우클릭 후 Send to Repeater 선택
3. Repeater 탭으로 이동
4. 요청 값을 수정
5. Send 클릭
6. 응답 확인
7. 값을 바꿔 다시 Send

예시는 다음과 같다.

```http
GET /user?id=100 HTTP/1.1
Host: test.example.com
Cookie: session=abc123
```

Repeater에서 `id=100`을 `id=101`로 바꿔 요청을 보낼 수 있다.

```http
GET /user?id=101 HTTP/1.1
Host: test.example.com
Cookie: session=abc123
```

이때 서버가 권한 검사를 제대로 하지 않으면 다른 사용자의 정보가 조회될 수 있다.

Repeater는 하나의 요청을 세밀하게 분석할 때 가장 자주 사용하는 기능 중 하나이다.

---

### 4.4 Intruder

Intruder는 HTTP 요청의 특정 위치에 여러 Payload를 자동으로 삽입해 반복 요청을 보내는 도구이다.

즉, 하나의 요청에서 바꿔볼 부분을 지정하고, 여러 값을 넣어 서버 응답 차이를 확인할 수 있다.

주요 사용 예시는 다음과 같다.

* 파라미터 값 대입 테스트
* 디렉터리 또는 파일명 추측
* ID 값 반복 확인
* 입력값 필터링 확인
* 간단한 Brute Force 실습
* CTF 문제 풀이
* 응답 길이, 상태 코드, 응답 시간 비교

기본 흐름은 다음과 같다.

1. HTTP History에서 요청 선택
2. Send to Intruder
3. Positions 탭에서 Payload 위치 지정
4. Payloads 탭에서 사용할 값 목록 입력
5. Attack 실행
6. 응답 코드, 길이, 시간 차이 확인

예시는 다음과 같다.

```http
GET /product?id=§100§ HTTP/1.1
Host: test.example.com
```

`§100§` 부분이 Payload가 들어갈 위치이다.

Payload 목록이 다음과 같다면,

```text
100
101
102
103
```

Intruder는 요청을 여러 번 보내며 각각의 응답을 비교할 수 있게 해준다.

#### Community(무료)와 Professional(유료) 차이

Community Edition에서도 Intruder를 사용할 수 있지만 속도나 기능에 제한이 있다.
Professional Edition에서는 더 빠르고 다양한 방식으로 Intruder를 활용할 수 있다.


---

### 4.5 Decoder

Decoder는 인코딩된 데이터를 디코딩하거나, 데이터를 다시 인코딩할 때 사용하는 도구이다.

웹 요청에서는 다양한 인코딩이 사용된다.

예시는 다음과 같다.

* URL Encoding
* Base64
* HTML Encoding
* Hex
* ASCII
* Gzip
* JSON 문자열 이스케이프

예를 들어 다음 값은 URL Encoding된 문자열이다.

```text
%61%64%6D%69%6E
```

Decoder를 사용하면 다음과 같이 확인할 수 있다.

```text
admin
```

또 다른 예로 Basic 인증 헤더는 Base64로 표현된다.

```http
Authorization: Basic YWRtaW46YWRtaW4=
```

이 값을 디코딩하면 다음과 같은 형식이 나온다.

```text
admin:admin
```

Decoder는 요청 안에 숨겨진 값이나 인코딩된 파라미터를 분석할 때 유용하다.

---

### 4.6 Comparer

Comparer는 두 개의 데이터 또는 응답을 비교하는 도구이다.

주로 다음과 같은 상황에서 사용한다.

* 정상 응답과 비정상 응답 비교
* 로그인 성공 응답과 실패 응답 비교
* 권한 있는 요청과 권한 없는 요청 비교
* 파라미터 변경 전후 응답 비교
* 길이가 비슷한 응답의 세부 차이 확인

예를 들어 같은 URL에 대해 다음 두 요청을 보냈다고 가정한다.

```text
id=100
id=101
```

두 응답이 비슷해 보여도 Comparer를 사용하면 변경된 부분을 쉽게 확인할 수 있다.

Comparer는 단어 단위 또는 바이트 단위 비교를 지원하므로, 작은 차이를 찾을 때 유용하다.

---

### 4.7 Sequencer

Sequencer는 토큰이나 세션 값의 무작위성을 분석하는 도구이다.

웹 애플리케이션에서는 다음과 같은 값들이 예측 불가능해야 한다.

* Session ID
* CSRF Token
* Password Reset Token
* Remember-me Token
* 인증 관련 임시 토큰

Sequencer는 여러 개의 토큰 샘플을 수집한 뒤 무작위성이 충분한지 분석한다.

예를 들어 세션 ID가 다음과 같이 단순히 증가한다면 문제가 될 수 있다.

```text
session=10001
session=10002
session=10003
```

이 경우 공격자가 다음 세션 값을 예측할 가능성이 있다.

반대로 충분히 긴 랜덤 값으로 생성된 토큰은 예측이 어렵다.

Sequencer는 세션 관리와 인증 토큰의 안전성을 점검할 때 사용한다.

---

### 4.8 Extensions

Extensions는 Burp Suite에 추가 기능을 설치해 기능을 확장하는 것이다.

Burp는 기본 기능도 강력하지만, 확장 기능을 사용하면 특정 테스트를 더 편하게 수행할 수 있다.

Extensions는 보통 BApp Store 또는 직접 작성한 확장 코드로 설치할 수 있다.

확장 기능을 통해 가능한 작업은 다음과 같다.

* 요청/응답 자동 분석
* 특정 취약점 탐지 보조
* 반복 작업 자동화
* 커스텀 Payload 생성
* Java, Python 기반 확장 기능 사용
* 테스트 결과 정리
* 외부 도구와 연동

대표적으로 웹 보안 테스트에서는 다음과 같은 유형의 확장을 사용할 수 있다.

* JWT 분석 도구
* Autorize 같은 권한 검증 보조 도구
* Logger 계열 도구
* JSON Web Token 관련 도구
* 요청 변조 자동화 도구
* Collaborator 연동 도구

#### Collaborator 개념

Burp Collaborator는 외부에서 들어오는 DNS, HTTP 등의 상호작용을 확인하는 데 사용하는 기능이다.

일반적인 취약점 테스트에서는 서버의 응답이 바로 화면에 나타나지 않는 경우가 있다.
이런 경우 외부 도메인으로 요청이 발생하는지 확인하여 취약점 여부를 판단할 수 있다.

예를 들어 서버가 사용자가 입력한 URL로 외부 요청을 보내는지 확인할 때 사용할 수 있다.

활용 가능한 취약점 예시는 다음과 같다.

* SSRF
* Blind XSS
* Out-of-band 취약점
* 외부 DNS 요청 발생 여부 확인
* 서버 측 요청 발생 여부 확인

단, Collaborator는 주로 Professional 환경에서 더 적극적으로 활용되며, 실제 테스트에서는 허가된 범위 안에서만 사용해야 한다.

---

## 5. 정리

### 5.1 Web Proxy와 Burp Proxy의 관계

Web Proxy는 클라이언트와 서버 사이에서 요청과 응답을 중계하는 개념이다.

Burp Proxy는 이 Web Proxy 개념을 웹 보안 테스트에 특화해서 구현한 도구이다.

일반적인 Web Proxy는 트래픽 전달, 캐싱, 접근 제어가 목적일 수 있다.
반면 Burp Proxy는 보안 분석을 위해 HTTP/HTTPS 요청과 응답을 확인하고 수정하는 데 초점이 있다.

정리하면 다음과 같다.

```text
Web Proxy = 중간에서 웹 요청과 응답을 중계하는 개념
Burp Proxy = 웹 보안 테스트를 위해 요청과 응답을 확인/수정할 수 있는 Proxy 도구
```

이미지로 표현하면 다음 구조와 같다.

![Proxy 설명](../../../../images/Proxy_Tool_Description.png)

```text
Web Browser  ⇄  Proxy Tool  ⇄  Web Server
```

Burp Suite는 이 구조에서 Proxy Tool 역할을 수행한다.

따라서 Burp를 사용하면 브라우저와 서버 사이의 통신을 눈으로 확인할 수 있고, 필요한 경우 요청을 수정하거나 Repeater, Intruder 같은 기능으로 반복 분석할 수 있다.

---

### 5.2 Burp를 사용할 때 기억할 점

Burp Suite를 사용할 때 기억해야 할 핵심은 다음과 같다.

1. **Proxy 구조를 이해해야 한다.**

   브라우저의 요청이 Burp를 거쳐 서버로 가는 구조를 이해해야 한다.

   ```text
   Browser → Burp → Web Server
   ```

2. **기본 Proxy 주소는 보통 127.0.0.1:8080이다.**

   외부 브라우저를 사용할 때는 브라우저 Proxy 설정을 이 주소로 맞춘다.

3. **HTTPS 분석에는 CA 인증서가 필요하다.**

   HTTPS 트래픽을 정상적으로 확인하려면 Burp CA 인증서를 브라우저나 기기에 설치해야 한다.

4. **Intercept가 켜져 있으면 요청이 멈춘다.**

   브라우저가 멈춘 것처럼 보이면 Intercept가 켜져 있는지 확인한다.

5. **HTTP History를 먼저 확인한다.**

   어떤 요청이 발생했는지 파악한 뒤 Repeater나 Intruder로 보내 분석한다.

6. **Repeater는 수동 반복 테스트에 적합하다.**

   하나의 요청을 조금씩 수정하면서 서버 반응을 확인할 때 사용한다.

7. **Intruder는 자동화된 반복 테스트에 적합하다.**

   여러 Payload를 넣어 응답 차이를 비교할 때 사용한다.

8. **Decoder는 인코딩된 값을 확인할 때 사용한다.**

   URL Encoding, Base64, Hex 같은 값을 디코딩하거나 다시 인코딩할 수 있다.

9. **Comparer는 응답 차이를 비교할 때 사용한다.**

   정상 응답과 비정상 응답, 권한 있는 응답과 권한 없는 응답의 차이를 확인할 수 있다.

10. **Sequencer는 토큰의 무작위성을 확인할 때 사용한다.**

    세션 ID, CSRF Token, Password Reset Token 같은 값이 예측 가능한지 분석할 수 있다.

11. **Scope를 설정해야 한다.**

    테스트 대상 범위를 명확히 지정해야 불필요한 트래픽을 줄이고, 허가된 범위 안에서만 테스트할 수 있다.

12. **모바일 앱 분석 시 네트워크와 인증서를 확인한다.**

    PC와 모바일 기기가 같은 네트워크에 있어야 하며, HTTPS 분석에는 모바일 기기에도 CA 인증서 설치가 필요하다.

---
