---
title: Kotlin Expert Syntax
date: 2021-05-11 10:47:11+09:00
description: ''
categories:
- kotlin
tags:
- kotlin
- Lamda
type: post
draft: false
---

Kotlin 관련 조금더 심화된 사용법 정리

<!--more-->
### Lamda

람다식 -> Value 처럼 다룰 수 있는 익명 함수

- 형식 : val lamdaName: Type = {argumentList -> codebody}
사용 예

```kotlin
val square: (Int) -> (Int) = {number -> number * number} //Int 받아서 제곱

val nameAge = {name: String, age:Int ->
    "my name is ${name}, I'm ${age}"    
}
```

람다의 리턴

```kotlin
val calculateGrade : (Int) -> (String) = {
    when (it) {
        in 0..40 -> "fail"
    }
} 
```

람다를 표현하는 방법

```kotlin
fun invokeLamda(lamda : (double) -> Boolean) : Boolean {
    return lamda(5.2343)
}

val lamda = {number: Double ->
    number == 4.3213    
}

println(invokeLamda(lamda)) // 결과는 false, lamda 를 인자로 받고 lamda 에 5.2343을 인자로 넘긴다
println(invokeLamda({})) // invokeLamda 에서 lamda 호출되고 it 에 값이 5.2343으로 넘어온다, 값은 true

invokeLamda() {it > 3.22} // 이 두개가 똑같다
invokeLamda {it > 3.22} // function 의 마지막 파라미터가 람다인경우, 생략 가능
```

익명 내부 함수

```kotlin
button.setOnClickListener(object : View.OnClickListener {
    override fun onClick(p0: View?) {
    }
})

//kotlin interace 가 아닌 자바 인터페이스여야 한다
// 그 인터페이스는 딱 하나의 메소드만 가져야 한다
// 예를 들면 아래처럼 setOnClickListener 를 사용 가능
button.setOnClickListener{ //it: View
    // to do..
}
```

### 확장함수

```kotlin
val addEndString : String.() -> String = {
    this + "END"
}

val a = "Start "

println(a.addEndString()) // 결과는 "START END"

------------------------------------------------------

fun extendString(name: String, age: Int) : String {
    val introduceMyself : String.(Int) -> String = {
        "I'm ${this} and ${it} years old"
    }

    return name.introduceMyself(age)
}

println(extendString("Kim", 20)) // I'm Kim and 20 years old
```

### Data Class

```kotlin
data class Ticket(val company: String, val name: String, var date: String, var seat : Int)

val ticketA = Ticket("CompanyA", "Kim", "2020-12-12", 12)
```

Data class 의 선언 및 사용은 위와 같으며, toString(), hashCode(), equals(), copy() 메소드를 만들어줘 보일러플레이트 줄이는 데 유용하다.

### Companion object

```kotlin
class Book private constructor(val id:Int, val name:String) { //다른곳에서는 객체를 생성하지 못함
    companion object BookFactory { // private property, object 를 읽어올 수 있게 해준다
        override fun getId(): Int {
            return 444
        }
    
        val myBook = "name"
        fun create() = Book(0, myBook)
    }
}

fun main(){
    val book = Book.Companion.create() //객체 생성 된다 (Companion 생략 가능)
    val bookId = Book.BookFactory.getId() // BookFactory 생략 가능
}
```

### Object class

Object class 의 객체는 한번만 만들어진다(Singleton Pattern)  
아래처럼 사용 가능

```kotlin
object CarFactory{
    val cars = mutableListOf<Car>()
    fun makeCar(hp: Int): Car {
        val car = Car(hp)
        cars.add(car)
        return car
    }

}

data class Car(val hp: Int) 

fun main() {
    val car = CarFactor.makeCar(10)
    val car2 = CarFactor.makeCar(100)

    println(car)
    println(car2) //dataclass 이기 때문에 toString() 값이 출력
    println(CarFactory.cars.size.toString()) // 2 출력
}
```
