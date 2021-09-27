---
layout: post
title:  "spring Querydsl 설정"
subtitle:   "spring Querydsl 설정"
date:   2021-09-28 01:42:27 +0900
categories: spring
tags: spring JPA ORM Mapping QueryDSL Setting
comments: true
---


<br>

- 목차
	- [Querydsl 설정](#querydsl-설정)
	- [테스트](#테스트)
	- [생성된 QType의 클래스를 확인](#생성된-qtype의-클래스를-확인)
    
<br>

# Querydsl 설정

build.gradle

```gradle
plugins {
	id 'org.springframework.boot' version '2.5.5'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10" //  (1)
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
	implementation 'com.querydsl:querydsl-jpa'  //  (2)
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}

//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"    //  (3)

querydsl {      //  (4)
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {    //  (5)
	main.java.srcDir querydslDir
}
configurations {    //  (6)
	querydsl.extendsFrom compileClasspath
}
compileQuerydsl {   //  (7)
	options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝

```

1. querydsl 플러그인을 추가합니다.
2. 라이브러리 dependency를 추가합니다.
3. querydsl에서 사용할 경로를 선언합니다.
4. querydsl 설정을 추가합니다. JPA 사용 여부와 사용할 경로를 지정하였습니다.
5. build시 사용할 sourceSet을 추가합니다.
6. querydsl이 compileClassPath를 상속하도록 설정합니다.
7. querydsl 컴파일시 사용할 옵션을 설정합니다.

위와같이 build.gradle에 (1) ~ (7)까지 추가한다.

<br><br>

# 테스트

- IDE를 사용하는경우

![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa1.jpg)

위 사진과 같이 우측 Gradle -> qureydsl -> Tasks -> other -> compileQuerydsl을 더블클릭 하게 되면 

![그림1](https://sehwan-choi.github.io/assets/img/spring/QueryDSL/jpa2.jpg)

위 사진과 같이 프로젝트 build -> generated -> querydsl 하위에 QMember가 생긴다.


터미널에서 직접 실행시키실 분은 아래 명령어를 프로젝트 루트 폴더에서 수행하시면 된다.

```
./gradlew compileQuerydsl
```

<br><br>

# 생성된 QType의 클래스를 확인

```java
package study.querydsl.entity;

import static com.querydsl.core.types.PathMetadataFactory.*;

import com.querydsl.core.types.dsl.*;

import com.querydsl.core.types.PathMetadata;
import javax.annotation.Generated;
import com.querydsl.core.types.Path;
import com.querydsl.core.types.dsl.PathInits;


/**
 * QMember is a Querydsl query type for Member
 */
@Generated("com.querydsl.codegen.EntitySerializer")
public class QMember extends EntityPathBase<Member> {

    private static final long serialVersionUID = -769675599L;

    private static final PathInits INITS = PathInits.DIRECT2;

    public static final QMember member = new QMember("member1");

    public final NumberPath<Integer> age = createNumber("age", Integer.class);

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public final QTeam team;

    public final StringPath username = createString("username");

    public QMember(String variable) {
        this(Member.class, forVariable(variable), INITS);
    }

    public QMember(Path<? extends Member> path) {
        this(path.getType(), path.getMetadata(), PathInits.getFor(path.getMetadata(), INITS));
    }

    public QMember(PathMetadata metadata) {
        this(metadata, PathInits.getFor(metadata, INITS));
    }

    public QMember(PathMetadata metadata, PathInits inits) {
        this(Member.class, metadata, inits);
    }

    public QMember(Class<? extends Member> type, PathMetadata metadata, PathInits inits) {
        super(type, metadata, inits);
        this.team = inits.isInitialized("team") ? new QTeam(forProperty("team")) : null;
    }

}
```

Item Entity를 상속받아 뭔가를 하는 클래스임을 확인할 수 있다. <br>

> 주의 ! <br>
Entity가 변경되었다면 이 과정을 다시 수행해서 QEntity가 제대로 생성될 수 있게 해야합니다. <br>

그리고 build시에 포함되므로 굳이 git에 포함시킬 필요가 없다.

git Repository 생성시 자동으로 .gitignore 파일에 build 경로가 포함이 되어있어서 신경쓸 필요가 없긴한데, 그렇지 않은 분들은 git으로 소스 코드를 관리할 땐 반드시 해당 경로를 무시하도록 처리해야 한다.

특히 지금 처럼 build 경로로 설정하지 않고 src 경로나 다른 경로로 설정해 .gitignore에서 추가로 설정해야 하는 일이 없게 기본적인 설정을 따라주시는 게 편리하다.

<br><br><br>
## References 및 사진 출처

> __김영한의 실전! Querydsl__