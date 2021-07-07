---
layout: post
title:  "HTTP 캐시"
subtitle:   "HTTP 캐시"
date:   2021-07-08 00:30:27 +0900
categories: spring
tags: spring httpheader cache
comments: true
---

# HTTP 캐시

<br>

- 목차
	- [캐시란?](#캐시란)
	- [캐시 기본 동작](#캐시-기본-동작)
		- [캐시가 없을때](#캐시가-없을때)
		- [캐시를 적용한다면?](#캐시를-적용한다면)
		- [캐시 시간 초과된다면?](#캐시-시간-초과된다면)
	- [검증 헤더와 조건부 요청](#검증-헤더와-조건부-요청)
		- [검증 헤더란?](#검증-헤더란)
		- [검증 헤더 종류](#검증-헤더-종류)
		- [조건부 요청 헤더](#조건부-요청-헤더)
		- [캐시 시간 초과](#캐시-시간-초과)
		- [검증 헤더추가(Last-Modified, If-Modified-Since)](#검증-헤더추가last-modified-if-modified-since)
		- [If-Modified-Since: 이후에 데이터가 수정되었으면?](#if-modified-since-이후에-데이터가-수정되었으면)
		- [Last-Modified, If-Modified-Since 단점](#last-modified-if-modified-since-단점)
		- [검증 헤더추가2(ETag, If-None-Match)](#검증-헤더추가2etag-if-none-match)
		- [ETag, If-None-Match 정리](#etag-if-none-match-정리)
	- [캐시와 조건부 요청 헤더](#캐시와-조건부-요청-헤더)
		- [캐시 제어 헤더](#캐시-제어-헤더)
		- [Cache-Control(max-age, no-cache, no-store)](#cache-controlmax-age-no-cache-no-store)
		- [Pragma](#pragma)
		- [Expires](#expires)
		- [검증 헤더와 조건부 요청 헤더](#검증-헤더와-조건부-요청-헤더)
	- [프록시 캐시](#프록시-캐시)
		- [프록시 캐시 Cache-Control](#프록시-캐시-cache-control)
	- [캐시 무효화](#캐시-무효화)
		- [캐시 무효화 Cache-Control](#캐시-무효화-cache-control)
		- [캐시 무효화 Cache-Control 설명](#캐시-무효화-cache-control-설명)
		- [no-cache vs must-revalidate](#no-cache-vs-must-revalidate)
<br>

# 캐시란?
브라우저에 응답으로 온 HTML이나 JSON같은 데이터가 저장되어 나중에 서버에 요청을 보내지 않고도 브라우저에 저장된 응답을 사용할 수 있습니다.

보통 캐싱은 GET 요청에만 합니다. GET이 REST적의미로 가져오다이기 때문에, 가져온 데이터를 저장해두고 두고두고 쓰는 것이죠. 일반적으로 200(가져오기 성공), 301(다른 주소로 이동 후 가져옴), 404(가져올 게 없음) 상태 코드로 온 응답을 캐싱할 수 있습니다.

즉, 사용자(client)가 웹 사이트(server)에 접속할 때, 정적 컨텐츠(이미지, JS, CSS 등)를 특정 위치(client, network 등)에 저장하여, 웹 사이트 서버에 해당 컨텐츠를 매번 요청하여 받는것이 아니라, 특정 위치에서 불러옴으로써 사이트 응답시간을 줄이고, 서버 트래픽 감소 효과를 볼 수 있습니다.
<br><Br>

# 캐시 기본 동작

## 캐시가 없을때

- 데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운로드 받아야 한다.
- 인터넷 네트워크는 매우 느리고 비싸다.
- 브라우저 로딩 속도가 느리다.
- 느린 사용자 경험

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache2.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache3.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache4.jpg)

캐시가 없다면 위와같이 두번째 요청시에도 첫번째와 같은 이미지를 계속 받아오기 때문에 네트워크 비용이 많이 들것이다.
<br><br>

## 캐시를 적용한다면?

- 캐시 덕분에 캐시 가능 시간동안 네트워크를 사용하지 않아도 된다.
- 비싼 네트워크 사용량을 줄일 수 있다.
- 브라우저 로딩 속도가 매우 빠르다.
- 빠른 사용자 경험

<br>

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache5.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache6.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache7.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache8.jpg)

<br>

## 캐시 시간 초과된다면?
- 캐시 유효 시간이 초과하면, 서버를 통해 데이터를 다시 조회하고, 캐시를 갱신한다.
- 이때 다시 네트워크 다운로드가 발생한다.

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache9.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache10.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache11.jpg)
<br><br>

# 검증 헤더와 조건부 요청

## 검증 헤더란?
캐시 데이터와 서버 데이터가 같은지 검증하는 데이터

<br><br>

## 검증 헤더 종류
- Last-Modified
- ETag
<br><br>

## 조건부 요청 헤더
- 검증 헤더로 조건에 따른 분기
- If-Modified-Since: Last-Modified 사용
- If-None-Match: ETag 사용
- 조건이 만족하면 200 OK
- 조건이 만족하지 않으면 304 Not Modified
<br><br>

## 캐시 시간 초과
-  캐시 유효 시간이 초과해서 서버에 다시 요청하면 다음 두 가지 상황이 나타난다.
	1. 서버에서 기존 데이터를 변경함
	2. 서버에서 기존 데이터를 변경하지 않음
- 캐시 만료후에도 서버에서 데이터를 변경하지 않음
- 생각해보면 데이터를 전송하는 대신에 저장해 두었던 캐시를 재사용 할 수 있다.
- 단 클라이언트의 데이터와 서버의 데이터가 같다는 사실을 확인할 수 있는 방법 필요
<br><br>

## 검증 헤더추가(Last-Modified, If-Modified-Since)

- 캐시 유효 시간이 초과해도, 서버의 데이터가 갱신되지 않으면
- 304 Not Modified + 헤더 메타 정보만 응답(바디X)
- 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신
- 클라이언트는 캐시에 저장되어 있는 데이터 재활용
- 결과적으로 네트워크 다운로드가 발생하지만 용량이 적은 헤더 정보만 다운로드
- 매우 실용적인 해결책

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last2.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last3.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last4.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last5.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last6.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last7.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last8.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/last9.jpg)

<br><br>

## If-Modified-Since: 이후에 데이터가 수정되었으면?
- 데이터 미변경 예시
	- 캐시: 2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 10:00:00
	- 304 Not Modified, 헤더 데이터만 전송(BODY 미포함)
	- 전송 용량 0.1M (헤더 0.1M, 바디 1.0M)
- 데이터 변경 예시
	- 캐시: 2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 11:00:00
	- 200 OK, 모든 데이터 전송(BODY 포함)
	- 전송 용량 1.1M (헤더 0.1M, 바디 1.0M)
<br><br>

## Last-Modified, If-Modified-Since 단점

- 1초 미만(0.x초) 단위로 캐시 조정이 불가능
- 날짜 기반의 로직 사용
- 데이터를 수정해서 날짜가 다르지만, 같은 데이터를 수정해서 데이터 결과가 똑같은 경우
- 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
	- 예) 스페이스나 주석처럼 크게 영향이 없는 변경에서 캐시를 유지하고 싶은 경우
<br><br>

## 검증 헤더추가2(ETag, If-None-Match)
- Last-Modified, If-Modified-Since 단점을 보완
- ETag(Entity Tag)
- 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
	- 예) ETag: "v1.0", ETag: "a2jiodwjekjl3"
- 데이터가 변경되면 이 이름을 바꾸어서 변경함(Hash를 다시 생성)
	- 예) ETag: "aaaaa" -> ETag: "bbbbb"
- 진짜 단순하게 ETag만 보내서 같으면 유지, 다르면 다시 받기!

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag2.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag3.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag4.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag5.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag6.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag7.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag8.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/etag9.jpg)
<br><br>

## ETag, If-None-Match 정리

- 진짜 단순하게 ETag만 서버에 보내서 같으면 유지, 다르면 다시 받기!
- 캐시 제어 로직을 서버에서 완전히 관리
- 클라이언트는 단순히 이 값을 서버에 제공(클라이언트는 캐시 메커니즘을 모름)
- 예)
	- 서버는 배타 오픈 기간인 3일 동안 파일이 변경되어도 ETag를 동일하게 유지
	- 애플리케이션 배포 주기에 맞추어 ETag 모두 갱신
<br><br><br>

# 캐시와 조건부 요청 헤더

<br>

## 캐시 제어 헤더
---

## Cache-Control(max-age, no-cache, no-store)
캐시 지시어(directives)
- Cache-Control: max-age
	- 캐시 유효 시간, 초 단위
- Cache-Control: no-cache
	- 데이터는 캐시해도 되지만, 항상 원(origin) 서버에 검증하고 사용
- Cache-Control: no-store
	- 데이터에 민감한 정보가 있으므로 저장하면 안됨
(메모리에서 사용하고 최대한 빨리 삭제

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/cache-control.jpg)

<br><br>

## Pragma
캐시 제어(하위 호환)
- Pragma: no-cache
- HTTP 1.0 하위 호환

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/pragma.jpg)


<br><br>

## Expires
캐시 만료일 지정(하위 호환)
- expires: Mon, 01 Jan 1990 00:00:00 GMT
- 캐시 만료일을 정확한 날짜로 지정
- HTTP 1.0 부터 사용
- 지금은 더 유연한 Cache-Control: max-age 권장
- Cache-Control: max-age와 함께 사용하면 Expires는 무시

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/expires.jpg)

<br><br>

## 검증 헤더와 조건부 요청 헤더
- 검증 헤더 (Validator) 
	- ETag: "v1.0", ETag: "asid93jkrh2l"
	- Last-Modified: Thu, 04 Jun 2020 07:19:24 GMT
- 조건부 요청 헤더
	- If-Match, If-None-Match: ETag 값 사용
	- If-Modified-Since, If-Unmodified-Since: Last-Modified 값 사용
<br><br>

# 프록시 캐시

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/proxy.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/proxy2.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/proxy3.jpg)

<br><br>

## 프록시 캐시 Cache-Control
캐시 지시어(directives) - 기타
- Cache-Control: public 
	- 응답이 public 캐시에 저장되어도 됨
- Cache-Control: private 
	- 응답이 해당 사용자만을 위한 것임, private 캐시에 저장해야 함(기본값)
- Cache-Control: s-maxage 
	- 프록시 캐시에만 적용되는 max-age
- Age: 60 (HTTP 헤더)
	- 오리진 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)
<br><br>

# 캐시 무효화

## 캐시 무효화 Cache-Control
확실한 캐시 무효화 응답
- Cache-Control: no-cache, no-store, must-revalidate 
- Pragma: no-cache 
- HTTP 1.0 하위 호환

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/must-revalidate.jpg)

<br><br>

## 캐시 무효화 Cache-Control 설명
캐시 지시어(directives) - 확실한 캐시 무효화
- Cache-Control: no-cache 
	- 데이터는 캐시해도 되지만, 항상 원 서버에 검증하고 사용(이름에 주의!)
- Cache-Control: no-store 
	- 데이터에 민감한 정보가 있으므로 저장하면 안됨
(메모리에서 사용하고 최대한 빨리 삭제)
- Cache-Control: must-revalidate 
	- 캐시 만료후 최초 조회시 원 서버에 검증해야함
	- 원 서버 접근 실패시 반드시 오류가 발생해야함 - 504(Gateway Timeout)
	- must-revalidate는 캐시 유효 시간이라면 캐시를 사용함
- Pragma: no-cache 
	- HTTP 1.0 하위 호환
<br><br>

## no-cache vs must-revalidate
- no-cache

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/no-cache.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/no-cache2.jpg)

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/no-cache3.jpg)
<br><br>
- must-revalidate

![그림1](https://sehwan-choi.github.io/assets/img/http/http_cache/must-revalidate2.jpg)
<br><br>

no-cache 는 프록시캐시와 원서버와 간에 순간적인 네트워크 단절이 되어도 프록시캐시에서 설정에 따라 캐시 데이터를 반환 할수 있지만,
must-revalidate 는 원서버에 접근할수 없는 경우 항상 오류가 발생해야 한다.(예. 은행계좌 조회 등..)

<br><br><br>
## References 및 사진 출처

> __김영한의 모든 개발자를 위한 HTTP 웹 기본 지식__