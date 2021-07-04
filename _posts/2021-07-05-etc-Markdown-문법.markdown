---
layout: post
title:  "Github Gist에서 마크다운 형식(Markdown) 사용 시 유용한 팁"
subtitle:   "Github Gist에서 마크다운 형식(Markdown) 사용 시 유용한 팁"
date:   2021-07-05 12:35:27 +0900
categories: etc
tags: markdown
comments: true
---

# Github Gist에서 마크다운 형식(Markdown) 사용 시 유용한 팁

## 1. `스페이스바 두 번 + Enter == \n` 일반 표현식은 자동으로 줄바꿈이 안 되는데 줄의 끝에 스페이스 바 두 번 입력하고 줄 바꿈을 하면 한 줄 띄울 수 있다.
> 스페이스바 2번 습관화... 빨리 패치되길 (2019.12.12기준)   

<br>

## 2. 문서내 링크가 가능하다. (목차로 활용하면 유용하다)
> 대부분 모르는 한글이나 특수문자가 입력될 시 유용한 팁 알려드림.  

사용법은 `[링크명(=파란색글씨)](#찾아갈목차명)`   

ex)  
[1. 목차1번 (Apple) ](#1-목차1번-apple)   
[아래 조건이 틀린경우](#1.-목차1번-Apple) 링크가 먹지않는다. `[아래 조건이 틀린경우](#1.-목차1번-Apple)` (틀린 경우)   

<br>

<br>

<br>

<br>

ex) `[1. 목차1번 (Apple) ](#1-목차1번-apple)` (옳은 경우)    

### 1. 목차1번 (Apple)  

<br>

- 조건: 목차명에 영어는 무조건 소문자, 스페이스가 있을 경우 대신 `-` 입력
- 조건: 목차명에 점, 반점, 특수문자, 괄호가 들어갔을 경우 무시해주면 된다.
- 조건: 목차명은 문자열이 유일해야 해야한다. 유일하지 않으면 가장 가까운 곳으로 가는데 이 때 링크명과 목차명이 같아야 할 때 팁은 위 예제와 같이 `[링크명 ]` 뒤에 스페이스바로 한 칸 띄어주면 된다.   
- 조건: `#`으로 제목처리된 곳으로만 갈 수 있다 `#`이나 `###` 처럼 개수는 상관없음. (단, ### 으로 갈때도 목차명에 #한개만 써야함)  

<br>

## 3. \`\`\`java 이런 식으로 코드 블럭 처음부분에 언어를 입력하면 언어별 코드 블럭 만들 수 있음.   
- 대문자 안되는 걸로 알고 있음 c, c++, c#, python, java 이런 키워드 스페이스 없이 붙이면 됨.  

ex) \`\`\`c#   

```c#
using System;

public class A {
  private int a;
}
```

ex) \`\`\` 언어 입력 안했을 경우  
```
using System;

public class A {
  private int a;
}
```

<br>

## 4. 한 줄이상 자동으로 안 띄어지는데 이 때 `<br>`을 사용하면 됨  

<br>
<br>
<br>

## 5. Typora 무료 마크 다운 편집기와 같이 쓰면 실시간 모습이 직접보이고 단축키와 띄어쓰기 줄 고려 안해도돼 유용해 추천  

## 6. 텍스트 가운데/오른쪽 정렬 하고싶을 때  
```
<div dir='rtl'>테스트1</div> or
<p align="right">테스트2</p>  

<p align="center">테스트3</p>
```
<div dir='rtl'>테스트1</div>
<p align="right">테스트2</p>  
  
<p align="center">테스트3</p>  

<br><br><br>
## References

> [https://gist.github.com/Curookie/2b7c110e23955b7131afbc76ffd2f724#file-gist-md](https://gist.github.com/Curookie/2b7c110e23955b7131afbc76ffd2f724#file-gist-md)