---
author: HM Joo
pubDatetime: 2021-04-28T04:49:56.000Z
title: Kotlin syntax \uC815\uB9AC (Basic)
slug: basic_syntax
featured: false
draft: false
description: Kotlin syntax \uC815\uB9AC (Basic)
---

안드로이드 개발을 위한 기본적인 Kotlin 문법 정리.  
기본적인 언어의 문법은 알고 있다는 가정 하에, Java와 다른 부분 위주로 정리
<!--more-->

### 상수와 변수
val, var 두 가지로 상수와 변수를 정의한다.
val : value, 변하지 않는 상수
var : variable, 변수

### String template
Kotlin 에서는 String 표시중 $표시를 이용해 변수를 직접 이용할 수 있다.  
```java
println("my name is $name ")
println("my name is ${name}")
println("my name is ${firstname + lastname}")
println("is this true? ${1==0}")
```

### 조건식
#### if
java 와 거의 비슷, 일반적인 사용은 아래처럼
```java
if( a > b ) {
	return a
} else {
	return b
}
```

(a>b)? a : b; 같은 표현은 kotlin 에서는 아래처럼 표현
```java
if(a>b) a else b // (a>b)? a : b 와 동일. 
```

#### when
java 의 switch 로 보면 된다.   
기본적인 사용은 아래와 같다  
```java
when(score) {// 일반적인 사용, else 사용되지 않아도 상관없음
    0 -> println("0")
    1 -> println("1")
    2,3 -> println("2 or 3")
    else -> println("something else")
}
```
각각 값에 따라서 다른 부분으로 분기   
   
아래 처럼 범위를 지정해 사용할 수도 있다
```java
when(score) { // 범위를 지정하여 사용하는 경우
    in 90..100 -> println("90 ~ 100")
    in 10..90 -> println("10 ~ 90")
    else -> println("otehr value")
}
```
**중요** 아래 처럼 값을 할당하는데 when 을 사용하는 경우, else 를 반드시 사용해야 한다.   
```java
var b = when(score){
	1-> 1
	2-> 2
	else -> 3 // score 에 따라 값을 리턴하여 b 에 할당, else 없으면 에러
}
```
#### Expression & Statement
> 어떤 값을 생성하는것 = Expression
> 예를 들어 위에 when 을 이용해 값을 할당하는 경우에는 Expression, 아무것도 하지 않은 경우는 Statement
> Kotlin에서는 아무것도 하지 않는 경우도 Unit을 리턴하기 때문에 모든 함수는 기본적으로 Expression
> 자바에서는 Void 형이 존재하므로 Statement 함수 사용 가능.
> 자바에서는 if문이 Statement 로 밖에 사용되지 않는 반면 코틀린에서는 원하는 방향으로 사용 가능

#### Array, List
자바와 동일하게 Array는 사이즈 지정되어 있다   
List 는 일반 ImmutableList 와 MutableList 로 구분
```java
val array = arrayOf(1, 2, 3)
val list = listOf(1, 2, 3)

val array2 = arrayOf(1, "D", 3.4f)
val list2 = listOf(1, "D", 3L)
```
List는 값을 참조만 가능하고 변경은 불가능. 값 변경을 위해서는 Mutable list 사용해야 한다. Mutable list 의 하나가 ArrayList
```java
val arayList = arrayListOf<Int>()
arrayList.add(10)
arrayList.add(20)
```
   

#### for, while
사용법은 아래와 같다
```kotlin
val students = arrayListOf("Alice", "James", "Roy") // 기본 arrayList 생성

for(name in students) { // 리스트 반복
	println("${name}")
}

for((index, name) in students.withIndex()) { //index 와 value 를 함께 사용
	println("${index}, ${name}")
}

for(i in 1..10) { // 1부터 10까지 반복
}

for( i in 1..10 step 2 ) { // 1부터 10까지 2씩 증가 (1, 3, 5, 7, 9)
}

for( i in 10 downTo 1 ){ // 10 부터 1 까지 반복
}

for( i in 1 until 10) { // 1 부터 9 까지 반복 (10 포함되지 않음)
}

var index = 0
while(index < 10) {
	index++
}

```













