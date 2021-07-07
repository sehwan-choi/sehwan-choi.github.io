---
layout: post
title:  "HTTP 헤더"
subtitle:   "HTTP 헤더"
date:   2021-07-07 23:18:27 +0900
categories: spring
tags: spring httpheader cookie cache
comments: true
---

# HTTP 헤더

<br>

- 목차
	- [HTTP 헤더](#http-헤더)
	- [용도](#용도)
	- [종류](#종류)
	- [표현헤더](#표현헤더)
		- [Content-Type](#content-type)
		- [Content-Encoding](#content-encoding)
		- [Content-Language](#content-language)
		- [Content-Length](#content-length)
	- [전송헤더](#전송헤더)
		- [Content-Length](#content-length)
		- [Content-Encoding](#content-encoding)
		- [Transfer-Encoding](#transfer-encoding)
		- [Range, Content-Range](#range-content-range)
	- [일반헤더](#일반헤더)
		- [From](#from)
		- [Referer](#referer)
		- [User-Agent](#user-agent)
		- [Server](#server)
		- [Date](#date)
	- [특별한 헤더](#특별한-헤더)
		- [Host](#host)
		- [Location](#location)
		- [Allow](#allow)
		- [Retry-After](#retry-after)
		- [Authorization](#authorization)
		- [WWW-Authenticate](#www-authenticate)
<br>

# HTTP 헤더

HTTP/1.1 200 OK<br>
__Content-Type: text/html;charset=UTF-8<br>
Content-Length: 3423__<br>
\<html><br>
 \<body>...\</body><br>
\</html>
<br>

위 코드에서 굵은색 부분이 HTTP 헤더이다.
<br><br>

## 용도
HTTP 전송에 필요한 모든 부가정보
-  예) 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보...
- 표준 헤더가 너무 많음
	- https://en.wikipedia.org/wiki/List_of_HTTP_header_fields
- 필요시 임의의 헤더 추가 가능
	- helloworld: hihi
<br><br>

## 종류

<br>

## __표현헤더__
--- 
## Content-Type
표현 데이터의 형식 설명
- 미디어 타입, 문자 인코딩
	- 예)
	- text/html; charset=utf-8
	- application/json
	- image/png

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/Content-Type.jpg)
<br><br>

## Content-Encoding
표현 데이터 인코딩
- 표현 데이터를 압축하기 위해 사용
- 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
- 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
	- 예)
	- gzip
	- deflate
	- identity

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/Content-Encoding.jpg)
<br><br>

## Content-Language
표현 데이터의 자연 언어
- 표현 데이터의 자연 언어를 표현
- 예)
	- ko
	- en
	- en-US

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/Content-Language.jpg)
<br><br>

## Content-Length
표현 데이터의 길이
- 바이트 단위
- Transfer-Encoding(전송 코딩)을 사용하면 Content-Length를 사용하면 안됨

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/Content-Length.jpg)
<br><br>

## __전송헤더__

---

## Content-Length
단순 전송

<br>

HTTP/1.1 200 OK<br>
Content-Type: text/html;charset=UTF-8<br>
__Content-Length: 3423__ <br>
\<html>
 \<body>...\</body>
\</html>

위 코드의 굵은색 표시
<br><br>

## Content-Encoding
압축 전송

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/encoding.jpg)
<br><br>

## Transfer-Encoding
분할 전송

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/encoding2.jpg)

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/encoding3.jpg)

<br>

위 사진과 같이 Hello, World, \r\n(개행) 이 따로 분할되어 전송된다.

<br><br>


## Range, Content-Range
범위 전송

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/encoding4.jpg)

<br><br>


## 일반헤더
---

## From
유저 에이전트의 이메일 정보

- 일반적으로 잘 사용되지 않음
- 검색 엔진 같은 곳에서, 주로 사용
- 요청에서 사용

<br><br>

## Referer
이전 웹 페이지 주소
- 현재 요청된 페이지의 이전 웹 페이지 주소
- A -> B로 이동하는 경우 B를 요청할 때 Referer: A 를 포함해서 요청
- Referer를 사용해서 유입 경로 분석 가능
- 요청에서 사용
- 참고: referer는 단어 referrer의 오타

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_header/referer.jpg)

<br><br>

## User-Agent
유저 에이전트 애플리케이션 정보

- user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/
537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
- 클리이언트의 애플리케이션 정보(웹 브라우저 정보, 등등)
- 통계 정보
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
- 요청에서 사용

<br><br>

## Server
요청을 처리하는 ORIGIN 서버의 소프트웨어 정보

- Server: Apache/2.2.22 (Debian)
- server: nginx
- 응답에서 사용

<br><br>

## Date
메시지가 발생한 날짜와 시간
- Date: Tue, 15 Nov 1994 08:12:31 GMT
- 응답에서 사용

<br><br>

## __특별한 헤더__

---

## Host
요청한 호스트 정보(도메인)
- 요청에서 사용
- 필수
- 하나의 서버가 여러 도메인을 처리해야 할 때
- 하나의 IP 주소에 여러 도메인이 적용되어 있을 때

GET /search?q=hello&hl=ko HTTP/1.1<br>
__Host: www.google.com__

위 코드의 굵은색이 Host 이다.

<br><br>

## Location
페이지 리다이렉션
- 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동
(리다이렉트)
- 응답코드 3xx에서 설명
- 201 (Created): Location 값은 요청에 의해 생성된 리소스 URI
- 3xx (Redirection): Location 값은 요청을 자동으로 리디렉션하기 위한 대상 리소스를
가리킴

<br><br>

## Allow
허용 가능한 HTTP 메서드
- 405 (Method Not Allowed) 에서 응답에 포함해야함
- Allow: GET, HEAD, PUT

<br><br>

## Retry-After
유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
- 503 (Service Unavailable): 서비스가 언제까지 불능인지 알려줄 수 있음
- Retry-After: Fri, 31 Dec 1999 23:59:59 GMT (날짜 표기)
- Retry-After: 120 (초단위 표기)

<br><br>

## __인증 헤더__

---

## Authorization
클라이언트 인증 정보를 서버에 전달

- Authorization: Basic xxxxxxxxxxxxxxxx

<br><br>

## WWW-Authenticate
리소스 접근시 필요한 인증 방법 정의

- 리소스 접근시 필요한 인증 방법 정의
- 401 Unauthorized 응답과 함께 사용

<br><br>

> 참고 : __위와 같은 Header의 내용은 브라우저에서 F12(개발자옵션) 에서 확인할수 있다.__

<br><br><br>
## References 및 사진 출처

> __김영한의 모든 개발자를 위한 HTTP 웹 기본 지식__