---
title: "Data Binding \uC608\uC81C"
date: 2021-03-16 14:27:15+09:00
description: 'Android '
categories: android
tags: android databinding AAC
---

Android Data Binding 관련 간단한 예제
<!--more-->

## 간단한 예제 작성
Data Binding 관련해서 아무래도 간단한 예제가 필요할 듯 하여 참고할만한 예제를 작성.
Recycler view 에도 붙여보려고 했는데, 따로 추가하여 작성할 예정
전체 코드는 
[https://github.com/ssadakk/SampleDataBinding](https://github.com/ssadakk/SampleDataBinding) 에서 확인 가능

## 전체 화면 개요
![1.png](1.png)   
진짜 간단하다. 버튼을 누르면 viewmodel 의 카운터를 증가시킬 것이고 이것을 바로 textview 와  progress bar 에 적용시키도록 만들것. 대충만 이해하기 위한 것이므로 이정도면 충분.

## Layout 작성
```html
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="viewmodel"
            type="com.ssadakk.sampledatabinding.LiveDataViewModel" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView
            android:id="@+id/txtLike"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{Integer.toString(viewmodel.counter)}"
            />
        <ProgressBar
            android:id="@+id/progressBar"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:max="@{100}"
            app:progressScaled="@{viewmodel.counter}"
            app:hideIfZero="@{viewmodel.counter}"
            />
        <Button
            android:id="@+id/btnLike"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="@{() -> viewmodel.addCounter()}"
            android:text="BtnLike"/>
    </LinearLayout>
</layout>
```
Variable 을 선언하여 view model 에 접근하여 데이터 세팅 할 수 있다.
TextView 에는 counter 값을 표시하도록 했고, Progress bar 에는 progress 를 설정했는데 이거는 뒤에 다시 설명   
버튼을 눌렀을 경우에는 viewModel 의 addCounter() 메서드를 통해 counter 증가 시켜줬다.

## BindingAdapters 생성
ProgressBar 에 보면 android:max="@{100}", app:progressScaled="@{viewmodel.counter}", app:hideIfZero="@{viewmodel.counter}" 이런것 들이 보이는데, 데이터 바인딩 어댑터를 이용해 적절한 값을 설정하도록 설정 할 수 있다.

BindingAdapters.kt 파일은 아래처럼 설정했다.
```kotlin
object BindingAdapters {

    @BindingAdapter("app:hideIfZero")
    @JvmStatic fun hideIfZero(view: View, number: Int) {
        if(number === 0) view.visibility = View.INVISIBLE else view.visibility = View.VISIBLE
    }

    @BindingAdapter(value = ["app:progressScaled", "android:max"], requireAll = true)
    @JvmStatic fun setProgress(progressBar: ProgressBar, counter: Int, max: Int)  {
        Log.d("hmjoo", "counter : $counter , max : $max")
        progressBar.progress = (counter)
    }
}
```
BindingAdapters 는 class 가 아니라 object 로 정의됨에 주의.   
app:hideIfZero 라는 이름으로 정의한 녀석은, counter 를 전달받아 0 인경우 투명하게 처리   

그 아래는 progress 를 세팅하도록 설정


## Run~
위 처럼 설정을 하면 버튼을 클릭하면 counter 가 증가하며 UI 에 실시간 반영되는 것을 볼 수 있다.
예전에 비해 상당히 간편해졌으며, 문득 예전에 진행했던 프로젝트가 생각난다. Databinding 을 활용했다면 UI 이슈가 상당량 줄어들었을 것으로 생각된다. 다음에는 RecyclerView를 이용해 DataBinding 을 사용해봐야지.



