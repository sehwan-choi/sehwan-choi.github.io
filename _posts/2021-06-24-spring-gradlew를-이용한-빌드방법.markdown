---
layout: post
title:  "gradlew를 이용한 빌드방법"
subtitle:   "gradlew를 이용한 빌드방법"
date:   2021-06-24 23:51:27 +0900
categories: spring
tags: spring java springboot gradlew build
comments: true
---

## gradlew를 이용한 빌드방법

---
1. project 폴더로 들어간다.
![그림1](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_1.jpg)

2. project 폴더에서 쉬프트 키를 누른채로 우클릭 하면 사진과 같이 `여기에 PowerShell 창 열기`가 나오며 클릭한다.
![그림2](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_2.png)

3. ./gradlew build 입력!
![그림3](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_3.jpg)

4. 빌드가 시작되며 `BUILD SUCCESSFUL`이 나왔다면 정상적으로 빌드가 된것이다.
![그림4](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_4.jpg)

5. ls(혹은 dir) 명령어를 통해서 확인해보면 build폴더가 생긴것을 확인 할 수 있다.
   빌드완료후 build 폴더에 빌드된 데이터들이 생성된다.
![그림5](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_5.jpg)

6. 실제 실행할수 있는 jar 파일은 libs에 있다.
![그림6](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_6.jpg)

7. java -jar 로 실행시키면 정상적으로 실행 되는 것을 볼수 있다.
![그림7](https://sehwan-choi.github.io/assets/img/spring/gradle/gradle_7.jpg)