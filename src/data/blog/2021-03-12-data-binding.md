---
author: HM Joo
pubDatetime: 2021-03-12T06:47:02.000Z
title: Data Binding
slug: data-binding
featured: false
draft: false
description: Data Binding
---

Data Binding 에 대한 정리

<!--more-->

[참고사이트](https://developer.android.com/topic/libraries/data-binding/expressions)   


## Data Binding
간단히 말해, 기존 아래와 같은 코드를
```kotlin
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}
```
아래처럼 쓸 수 있는것
```html
<TextView
        android:text="@{viewmodel.userName}" />
```

## What to do to use?
Gradle 에 아래와 같이 추가 (View binding 과 같다)
```gradle
android {
        ...
        dataBinding {
            enabled = true
        }
    }
```

## Data Binding
View binding 과 마찬가지로, layout name + Binding 으로 데이터 바인딩 클래스 생성됨.   
e.g) activity_main.xml -> ActivityMainBinding   

## Layout binding & expression
```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"/>
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"/>
       </LinearLayout>
    </layout>
```
User 라는 클래스가 있다고 가정, TextView 에서 User 의 firstName, lastName 을 갖다쓰는걸 볼 수 있다.   
@{} 구문을 사용.

그렇다면 데이터 결합은? 아래처럼   
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = DataBindingUtil.setContentView(
            this, R.layout.activity_main) // View binding 에서처럼 inflate() 를 이용할 수도 있다.

    binding.user = User("Test", "User")
}
```
binding.user 로 Test user 가 들어가겠지   

Fragment, listView, RecytclerView 같은 어댑터 내에서 Binding 하는 경우는 DataBindingUtil.Inflate() 사용할 수 있음
```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
    // or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
```

## 표현식
```html
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```
위처럼 각 연산자 사용 가능. 

## 뷰 참조
아래처럼 ID로 다른 뷰 참조 가능
```html
<EditText
        android:id="@+id/example_text"
        android:layout_height="wrap_content"
        android:layout_width="match_parent"/>
    <TextView
        android:id="@+id/example_output"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{exampleText.text}"/>
```

그 외도 다양한 참조 방법 가능.

## Method 참조
핸들러를 정의하고, 이벤트에 핸들러를 결합하여 사용할 수 있음
아래처럼 사용하는 경우의 장점은 컴파일 타임에 처리가 된다. 따라서 메서드가 없거나 서명이 올바르지 않으면 컴파일 에러 발생할 수 있음.
```kotlin

    class MyHandlers {
        fun onClickFriend(view: View) { ... }
    }
```

```html
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="handlers" type="com.example.MyHandlers"/>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"
               android:onClick="@{handlers::onClickFriend}"/>
       </LinearLayout>
    </layout>
```

## Listener 결합
이벤트가 발생할때 실행되는 결합 표현식. 메서드 참조와 비슷하지만 임의의 데이터 표현식을 실행할 수 있음.

```kotlin
class Presenter {
        fun onSaveClick(task: Task){}
    }
```

```html
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
        <data>
            <variable name="task" type="com.android.example.Task" />
            <variable name="presenter" type="com.android.example.Presenter" />
        </data>
        <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
            <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:onClick="@{() -> presenter.onSaveClick(task)}" />
        </LinearLayout>
    </layout>
```


## Import (가져오기)
Layout  내에서 클래스를 쉽게 참조할 수 있음.
```html
<data>
    <import type="android.view.View"/>
</data>
```
```html
<TextView
       android:text="@{user.lastName}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

클래스 이름이 충도랗면 이름을 별칭으로 바꿀수 있음. 
```html
<import type="android.view.View"/>
    <import type="com.example.real.estate.View"
            alias="Vista"/>
```

다른 클래스 가져오기
```html
<data>
        <import type="com.example.User"/>
        <import type="java.util.List"/>
        <variable name="user" type="User"/>
        <variable name="userList" type="List&lt;User>"/>
    </data>
    
```

아래처럼 Type convert도 가능
```html
<TextView
       android:text="@{((User)(user.connection)).lastName}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
```

static field, method 도 사용 가능
```html
<data>
        <import type="com.example.MyStringUtils"/>
        <variable name="user" type="com.example.User"/>
    </data>
    …
    <TextView
       android:text="@{MyStringUtils.capitalize(user.lastName)}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
```

변수도 사용 가능
```html
<data>
        <import type="android.graphics.drawable.Drawable"/>
        <variable name="user" type="com.example.User"/>
        <variable name="image" type="Drawable"/>
        <variable name="note" type="String"/>
    </data>
```








