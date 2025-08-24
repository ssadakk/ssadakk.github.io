---
author: HM Joo
pubDatetime: 2021-03-11T07:23:00.000Z
title: View Binding
slug: view_binding
featured: false
draft: false
tags:
  - android
  - AAC
description: View Binding
---

View binding 에 대해 알아볼것   

<!--more-->

## View Binding
이전까진, 대부분의 경우 View 를 결합하기 위해 findViewById() 를 사용했음.   
View Binding 을 사용하면 좀더 쉽게 View 를 컨트롤 할 수 있다.   
코틀린이 익숙하지 않아 앞으로의 스터디는 코틀린 위주로 진행.

## Gradle 설정
View binding 사용하기 위해선, 먼저 viewBinding 요소를 gradle 에 추가해야됨.
```gradle
android{
    ...
    viewBinding{
        enabled = true
    }
}
```

## Usage
위처럼 View Binding 이 설정되면, XML layout 파일의 Binding class가 생성됨.   
형식은 XML 이름 + Binding.   
예를들어, result_profile.xml 파일이 있는경우 Binding class 의 이름은 **ResultProfileBinding**   
```xml
<LinearLayout ... >
    <TextView android:id="@+id/name" />
    <ImageView android:cropToPadding="true" />
    <Button android:id="@+id/button"
        android:background="@drawable/rounded_button" />
</LinearLayout>
```
이런 Layout 이 있을 경우, TextView, Button 은 ID 가 있기 때문에 Binding class 통해 참조 가능하지만 ImageView 의 경우에는 ID 가 없어 참조 불가하다.   

### Usage in Activity
inflate() 호출해 getRoot() 메소드로 root view 참조
```kotlin
override fun onCreate(savedInstanceState: Bundle) {
    super.onCreate(savedInstanceState)
    binding = ResultProfileBinding.inflate(layoutInflater) // XML 파일로 class 결정
    val view = binding.root
    setContentView(view)
}
```
각 child view 는 아래처럼 참조 가능
```kotlin
binding.name.text = viewModel.name
binding.button.setOnClickListener { viewModel.userClicked() }
```   

### Usage in Fragment
#### Inflate Fragment
```kotlin
override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        val binding = FragmentBlankBinding.inflate(inflater, container, false) //use inflat
        fragmentBlankBinding = binding
        binding.textViewFragment.text = getString(R.string.hello_from_vb_inflatefragment) //get info from view
        return binding.root
    }
```

#### Bind Fragment
```kotlin
class BindFragment : Fragment(R.layout.fragment_blank) { //Layout 명시

    // Scoped to the lifecycle of the fragment's view (between onCreateView and onDestroyView)
    private var fragmentBlankBinding: FragmentBlankBinding? = null

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val binding = FragmentBlankBinding.bind(view) // Bind !!!!
        fragmentBlankBinding = binding
        binding.textViewFragment.text = getString(string.hello_from_vb_bindfragment) //view control
    }

    override fun onDestroyView() {
        // Consider not storing the binding instance in a field, if not needed.
        fragmentBlankBinding = null
        super.onDestroyView()
    }
}
```

## findViewById 와의 차이점
1. Null Safe
    직접 참조를 생성하지 않으므로 , Null exception 발생의 위험이 적음 (기존에는 작은 실수에 의해 생각보다 많이 발생 )
2. Type safe
    Type 이 xml의 정의와 일치하므로, 예외 발생 위험 없음.
위와 같은 장점으로, 호환 안되는 경우 컴파일에서 에러 걸러낼수 있음

## Data binding 과의 비교
둘 다 View 를 직접 참조하는데 사용할 수 있는 바인딩 클래스 생성. 하지만 View binding은 보다 단순한 사용 사례 를 위한것이며 아래와 같은 장점이 있음.
- 빠른 컴파일
- 사용 편의성
하지만 제약사항으로는
- 레이아웃 변수, 레이아웃 표현식 지원하지 않음
- 양방향 데이터 결합 지원하지 않음

위의 상황을 고려해 두가지를 적절히 사용하는것이 바람직.






