---
layout: post
title:  "HTTP특징"
subtitle:   "HTTP특징"
date:   2021-07-05 00:35:27 +0900
categories: spring
tags: spring http client-server stateful stateless connectionless
comments: true
---

# HTTP특징
<br>

- 목차
	- [HTTP란?](#http란)
	- [HTTP 역사](#http-역사)
	- [HTTP 특징](#http-특징)
		- [1. 클라이언트 서버 구조](#1-클라이언트-서버-구조)
		- [2. 무상태 프로토콜(Stateless), 비연결성](#2-무상태-프로토콜stateless-비연결성)
			- [Stateful, Stateless 차이](#stateful-stateless-차이)
			- [Stateful, Stateless 정리](#stateful-stateless-정리)
			- [Stateless 한계](#stateless-한계)
			- [비 연결성](#비-연결성)
			- [비 연결성의 한계와 극복](#비-연결성의-한계와-극복)
		- [3. HTTP 메시지](#3-http-메시지)
			- [요청 메세지의 구조](#요청-메세지의-구조)
			- [응답 메세지의 구조](#응답-메세지의-구조)
<br>

## HTTP란?

HTTP란 HyperText Transfer Protocol의 약자로, 인터넷에서, 웹 서버와 사용자의 인터넷 브라우저 사이에 문서를 전송하기 위해 사용되는 통신 규약을 말한다.

- HTML, TEXT
- IMAGE, 음성, 영상, 파일
- JSON, XML (API)
- 거의 모든 형태의 데이터 전송 가능
- 서버간의 데이터를 주고 받을 때도 대부분 HTTP 사용

위와같이 HTTP 메세지에 모든 것을 전송할수 있다.
<br><br>

## HTTP 역사

- HTTP/0.9 1991년: GET 메서드만 지원, HTTP 헤더X
- HTTP/1.0 1996년: 메서드, 헤더 추가
- HTTP/1.1 1997년: 가장 많이 사용, 우리에게 가장 중요한 버전
- RFC2068 (1997) -> RFC2616 (1999) -> RFC7230~7235 (2014)
- HTTP/2 2015년: 성능 개선
- HTTP/3 진행중: TCP 대신에 UDP 사용, 성능 개선

> 참고 : 기반 프로토콜
<br>TCP : HTTP/1.1, HTTP/2
<br>UDP : HTTP/3
<br>현재 HTTP/1.1 주로 사용
<br>HTTP/2, HTTP/3 도 점점 증가

<br><br>

# HTTP 특징

<br>

## 1. 클라이언트 서버 구조

- Request, Response 구조
- 클라이언트는 서버에 요청을 보내고, 응답을 대기
- 서버가 클라이언트 요청에 대한 결과를 만들어서 응답

위와같은 특징으로 서버와 클라이언트는 독립적으로 발전이 가능하다.

예를 들어, 클라이언트는 UI/UX를 그리기에만 집중하고, 서버는 데이터와 로직을 처리하게 한다면, 트래픽이 늘어난다면 클라이언트 수정이 아닌 서버에서 트래픽은 어떻게 처리할지 고민하면 된다.

<br>

## 2. 무상태 프로토콜(Stateless), 비연결성

- 서버가 클라이언트의 상태를 보존 하지 않는다.
- 장점 : 서버 확장성 높음(스케일 아웃)
- 단점 : 클라이언트가 추가 데이터 전송

<br><br>

### Stateful, Stateless 차이

<br>

- 상태유지 - Stateful

```
- 고객: 이 노트북 얼마인가요?
- 점원A: 100만원 입니다.

- 고객: 2개 구매하겠습니다.
- 점원A: 200만원 입니다. 신용카드, 현금중에 어떤 걸로 구매 하시겠어요?

- 고객: 신용카드로 구매하겠습니다.
- 점원A: 200만원 결제 완료되었습니다
```
점원A는 점원이 바뀌지 않는이상 고객이 요청하는 정보를 모두 알고있다. 
만일 다른 점원에게 구매요청을 한다면 그 점원은 고객이 요청하는 것을 모를것이다.(처음부터 다시 요청해야함.)
<br><br>

- 무상태 - Stateless

```
- 고객: 이 노트북 얼마인가요?
- 점원A: 100만원 입니다.

- 고객: 노트북 2개 구매하겠습니다.
- 점원B: 노트북 2개는 200만원 입니다. 신용카드, 현금중에 어떤 걸로 구매 하시겠어요?

- 고객: 노트북 2개를 신용카드로 구매하겠습니다.
- 점원C: 200만원 결제 완료되었습니다.
```
고객은 내가 원하는 모든 정보를 모두 점원A, 점원B, 점원C 에게 전달한다. 이렇게 한다면 점원과 얘기를 할수록 많은 정보를 말해야 하지만(노트북 개수, 신용카드로 구매...) 점원에 구애받지 않고 모든점원에게 구매를 할수 있다.
<br><br>

### Stateful, Stateless 정리

<br>

- Stateful
1. 중간에 다른 점원으로 바뀌면 안된다.
(중간에 다른 점원으로 바뀔 때 상태 정보를 다른 점원에게 미리 알려줘야 한다.)

<br>

- Stateless
1. 중간에 다른 점원으로 바뀌면 안된다.
(중간에 다른 점원으로 바뀔 때 상태 정보를 다른 점원에게 미리 알려줘야 한다.)
2. 중간에 다른 점원으로 바뀌어도 된다.
3. 갑자기 고객이 증가해도 점원을 대거 투입할 수 있다.
4. 갑자기 클라이언트 요청이 증가해도 서버를 대거 투입할 수 있다.
5. 무상태는 응답 서버를 쉽게 바꿀 수 있다. -> 무한한 서버 증설 가능

![그림1](https://sehwan-choi.github.io/assets/img/http/http_feature/stateful.jpg)
<br>

![그림2](https://sehwan-choi.github.io/assets/img/http/http_feature/stateful2.jpg)
<br>

![그림3](https://sehwan-choi.github.io/assets/img/http/http_feature/stateless.jpg)
<br>

![그림4](https://sehwan-choi.github.io/assets/img/http/http_feature/stateless2.jpg)
<br>

![그림5](https://sehwan-choi.github.io/assets/img/http/http_feature/stateless3.jpg)
<br><br>

### Stateless 한계
- 모든 것을 무상태로 설계 할 수 있는 경우도 있고 없는 경우도 있다.
- 무상태의 경우 (로그인이 필요 없는 단순한 서비스 소개 화면)
- 상태유지의 경우 (로그인)
- 로그인한 사용자의 경우 로그인 했다는 상태를 서버에 유지
- 일반적으로 브라우저 쿠키와 서버 세션등을 사용해서 상태 유지
- 상태 유지는 최소한만 사용

<br><br>

### 비 연결성

- HTTP는 기본이 연결을 유지하지 않는 모델
- 일반적으로 초 단위의 이하의 빠른 속도로 응답
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작음
<br> 예). 웹 브라우저에서 계속 연속해서 검색 버튼을 누르지는 않는다.
- 서버 자원을 매우 효율적으로 사용할 수 있음
<br><br>

### 비 연결성의 한계와 극복

- TCP/IP 연결을 새로 맺어야 함 - 3 way handshake 시간 추가
- 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등 수 많은 자원이 함께 다운로드 (자원을 하나 받을때마다 연결하고 해제하고 해야함)
- 지금은 HTTP 지속 연결(Persistent Connections)로 위의 문제 해결
- HTTP/2, HTTP/3에서 더 많은 최적화

<br><br><br>

## 3. HTTP 메시지

<br>

![그림6](https://sehwan-choi.github.io/assets/img/http/http_feature/example.jpg)

위의 사진과 같은 구조로 되어있다.

<br><br>

---
# 요청 메세지의 구조
> __굵게 처리된 부분을 봐주길 바란다.__


---
## start-line


<br>

__GET__ /search?q=hello&hl=ko HTTP/1.1
<br>
요청 메시지 - HTTP 메서드
- 종류 : GET, POST, PUT, DELETE
- 서버가 수행해야 할 동작 지정
	- GET : 리소스 조회
	- POST : 요청 내역 처리
<br><br>

GET __/search?q=hello&hl=ko__ HTTP/1.1
<br>
요청 메시지 - 요청 대상
- absolute-path[?query] (절대경로[?쿼리])
- 절대경로 = "/"로 시작하는 경로
<br><br>

GET /search?q=hello&hl=ko __HTTP/1.1__
<br>
요청 메시지 - HTTP 버전
<br><br>

## HTTP 헤더

Host: www.google.com

<br>

---

# 응답 메세지의 구조

## start-line

<br>

__HTTP/1.1__ 200 OK
<br>
HTTP 버전

<br>

HTTP/1.1 __200__ OK
<br>
HTTP 상태 코드 : 요청 성공, 실패를 나타냄
- 2XX : 성공
- 4XX : 클라이언트 요청 오류
- 5XX : 서버 내부 오류

<br>

HTTP/1.1 200 __OK__
<br>
이유 문구 : 사람이 이해할 수 있는 짧은 상태 코드 설명

<br>

## HTTP 헤더

<br>

__Content-Type: text/html;charset=UTF-8__

__Content-Length: 3423__
<br><br>

- HTTP 전송에 필요한 모든 부가정보 포함

	예) 메시지 바디의 내용, 메시지 바디의 크기, 압축, 인증, 요청 클라이언트(브라우저) 정보, 서버 애플리케이션 정보, 캐시 관리 정보...

<br><br><br>

## HTTP 메시지 바디

```
<html>
	<body>...</body>
<html>
```
- 실제 전송할 데이터
- HTML 문서, 이미지, 영상, JSON 등등 byte로 표현할 수 있는 모든 데이터 전송 가능

<br><br><br>
## References 및 사진 출처

> __김영한의 모든 개발자를 위한 HTTP 웹 기본 지식__