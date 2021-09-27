---
layout: post
title:  "spring Querydsl QClass cannot find symbol 에러발생시"
subtitle:   "spring Querydsl QClass cannot find symbol"
date:   2021-09-28 04:46:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL cannot find symbol
comments: true
---


<br>

- 목차
	- [cannot find symbol](#cannot find symbol)
    
<br>

# cannot find symbol


![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa3.jpg)

위 사진과 같이 Cannot find symbol 에러가 발생할경우 build.gradle에 빌드할 때마다 compileQuerydsl 작업 전에 generated/querydsl package를 삭제하고, 다시 컴파일해서 생성하는 코드를 추가한다.

```gradle
plugins {
	id 'org.springframework.boot' version '2.5.5'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
	id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.8'
	//querydsl 추가
	implementation 'com.querydsl:querydsl-jpa'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

def querydslDir = "$buildDir/generated/querydsl"
querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
configurations {
	querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}

// 아래코드 추가
compileQuerydsl.doFirst {
	if(file(querydslDir).exists() )
		delete(file(querydslDir))
}

```

위 build.gradle에서 아래코드 추가 부분의 코드를 추가한후 다시실행하면 정상적으로 동작한다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__