---
layout: post
title:  "hashTable VS hashMap VS ConcurrentHashMap"
subtitle:   "hashTable VS hashMap VS ConcurrentHashMap"
date:   2021-06-25 01:31:27 +0900
categories: spring
tags: spring java springboot hashTable hashMap ConcurrentHashMap
comments: true
---

## hashTable VS hashMap VS ConcurrentHashMap

- 목차
	- [HashMap](#HashMap) 
	- [ConCurrentHaspMap](#ConCurrentHaspMap)
	- [HashTable](#HashTable) 
	- [HashMap VS HashTable](#HashMap-VS-HashTable) 
	- [HashTable VS ConcurrentHashMap](#HashTable-VS-ConcurrentHashMap) 
	- [정리](#정리) 

<br>

### HashMap
   HashMap은 synchronized 키워드가 없기 때문에 동기화가 보장되지 못한다.<br>
(싱글 스레드 환경에서 사용하길) 따라서 동기화처리를 하지 않기 때문에 값을 찾는 속도가 상당히 빠르다.<br>
또한 HashTable과 다르게 key,value null값을 허용한다. 즉 속도가 빠르지만, 신뢰성 안정성은 떨어진다고 생각하면 된다.
<br><br>

### ConCurrentHaspMap
   HashMap의 멀티스레드 환경에서의 동기화처리로 인한 문제점을 보완한 것이 ConCurrentHashMap이다. <br>
하지만 HashMap과 다르게 key,value에 null을 허용하지 않는다.
<br><br>

### HashTable
   HashTable의 메서드는 전부 synchronized 키워드가 붙어있기 때문에 메서드 호출 전 쓰레드간 동기화 락을 통해 멀티 쓰레드 환경에서 data의 무결성을 보장해준다.<br>
   또한 key,value값의 null을 허용하지 않는다. <br>
   즉 동기화 락때문에 속도는 느리지만, data의 안정성이 높고 신뢰가 높은 컬렉션이다.
<br><br>

### HashMap VS HashTable
   Hashtable은 모든 데이터 변경 메소드가 synchronized로 선언되어 있어서, 메소드 호출 전 스레드 간의 동기화 lock을 통해 멀티 스레드 환경에서 data의 무결성을 보장해준다. (thread-safe)<br>
하지만 HashMap은 그런 장치가 1도 없기 때문에, 동기화 문제가 발생할 수 있다.(non-thread-safe)<br>
물론, HashMap도 synchronized로 잘 잡아주면 동기화 문제가 해결될 수도 있긴 하다.
>ex. Map m = Collections.synchronizedMap(new HashMap(...));

동기화 lock이 엄청 느린 동작이기 때문에 성능은 HashMap이 훨씬 우수하다.
<br><br>

### HashTable VS ConcurrentHashMap
   동기화를 지원한다는 것이 Hashtable과 같지만 성능은 ConcurrentHashMap이 더 우수하므로 그냥 이걸 사용하는 쪽을 추천한다.<br>
한 자바 관련 서적에 의하면 Vector의 상위호환(?)개념인 ArrayList의 사용을 권장하듯 새로운 버전인 HashMap을 활용하고 동기화가 필요한 시점에서는 Java 5부터 제공하는 ConcurrentHashMap을 사용하는 것이 더 좋은 방법이라 표현한다.
<br><br>


## 정리
|  | HashTable | HashMap | ConCurrentHashMap |
|---|:---|---|---:|
|thread-safe<br>(synchronized)| O | X | O |
|반환 값| Enumeration | Fail-Fast Iterator | Fail-Fast Iterator |
|성능 순위| 3 | 1 | 2 |
|key, value null허용 | X | O | X |
|사용할 상황| X | 단일 스레드 환경에서<br> map을 사용하는 경우 | 멀티 스레드 환경에서<br> map을 사용하는 경우 |

> Enumeration 이란?<br>
> Enumeration은 순차적 접근 시 콜렉션 객체에 변경이 일어나도 이를 무시하고, 끝까지 동작한다.


> fail-fast 이란?<br>
> Iterator는 순차적 접근이 모두 끝나기 전에 콜렉션 객체에 변경이 일어날 경우 순차적 접근이 실패되면서 예외를 return하게 되는데 이를 fail-fast 방식이라고 부른다.

<br><br><br>
## References

> [https://limkydev.tistory.com/40](https://limkydev.tistory.com/40)

> [https://j-i-y-u.tistory.com/30](https://j-i-y-u.tistory.com/30)

> [https://shlee0882.tistory.com/44](https://shlee0882.tistory.com/44)