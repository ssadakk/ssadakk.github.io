---
title: Live Data
date: 2021-03-16 23:22:56+09:00
description: ''
categories:
- android
tags:
- android
- AAC
type: post
draft: false
---


<!--more-->

## Live Data?
LiveData 는 데이터 홀더 클래스. 특이한 점은 LiveData 는 수명 주기를 인식한다. Activity, fragment, service 의 라이프사이클을 모두 인식. 이를 통해 활둥 상태에 있는 Observer 만 update 한다.

## Live Data의  이점
- UI 와 Data 상태 일치 보장
- Memory leak 없음
- Stop 상태로 인한 비정상 종료 없음
- LifeCycle 을 수동으로 처리하지 않는다.
- 최신 데이터 유지
- 적절한 구성 변경
- 리소스 공유

## LiveData 사용
대략적인 LiveData 사용 순서는 아래와 같다.
1. LiveData instance 생성. 일반적으로 ViewModel 클래스 내에 생성
2. onChanged() 메서드를 정의하는 Observer instance 생성, LiveData 데이터 변경시 발생하는 작업을 제어. 일반적으로 액티비티, 프래그먼트 같은 UI controller 에 생성 
3. observe() 메서드로 LiveData 에 Observer 연결

대략적으로 보자면 아래와 같이 사용할 수 있다.
```
class LiveDataViewModel : ViewModel() {
    private val _counter = MutableLiveData(0)
    val counter: LiveData<Int> = _counter
}
```
```
val dataObserver: Observer<ArrayList<User>> = Observer { livedata ->
    Log.e("hmjoo", "in data observer")
    data.value = livedata
    var newAdapter = UserProfileAdapter(data)
    binding.recyclerView.adapter = newAdapter
}
viewModel.liveData.observe(this, dataObserver)
```
아래는 좀더 자세한 설명   

### LiveData instance 생성
LiveData 는 Collection 객체를 비롯해 모든 데이터와 함께 사용 가능한 래퍼. ViewModel 내에 저장.   
ViewModel 객체의 UI 를 업데이트하는 LiveData 를 저장해야 한다. 
- Activity, Fragment 가 지나치게 커지지 않게 하기 위해.
- LiveData 객체를 특정 Activity, Fragment 에서 분리, 구성 변경에도 LiveData 객체가 유지되도록 하기 위해.

## LiveData Observe
Observe 의 경우, onCreate() 에서 observe 하는것이 적합. 이유는, 
- onResume() 에서 중복 호출을 하지 않도록 하기 위해,
- Activity, Fragment 가 active 되자마자 표시될 데이터를 포함할 수 있도록 하기 위해. 

## LiveData Update
LiveData는 저장된 데이터를 업데이트하는 public method 가 없다. MutableLiveData 클래스는 setValue(), postValue() 를 public method 로 한다. LiveData 에 저장된 값을 수정하려면 위 메서드 사용해야 됨. 

```
button.setOnClickListener {
    val anotherName = "John Doe"
    model.currentName.setValue(anotherName)
}
```
위 예제를 보면 setValue()를 호출하면 onChanged() 메서드 함께 호출됨.   
네트워크 요청, DB 로드 완료 등에 호출하게 되면 Observer 트리거 될것이고 UI 업데이트 될 것.
** 메인 스레드에서 LiveData 를 업데이트 하려면 setValue() 를 사용해야 됨. 워커스레드에서 실행한다면 postValue(*) 를 사용**


## Room 으로 LiveData 사용
Room 에서 쿼리를 사용하면 LiveData 를 리턴해준다.   
DB 업데이트 될 때 Room 에서는 LiveData 객체를 업데이트하는데 필요한 모든 코드를 생성. 생성된 코드는 백그라운드 스레드에서 비동기적으로 쿼리를 실행. 이 패턴은 UI 와 DB의 데이터의 동기 유지에 유용.

## LiveData 확장
```
class StockLiveData(symbol: String) : LiveData<BigDecimal>() {
    private val stockManager = StockManager(symbol)

    private val listener = { price: BigDecimal ->
        value = price
    }

    override fun onActive() {
        stockManager.requestPriceUpdates(listener)
    }

    override fun onInactive() {
        stockManager.removeUpdates(listener)
    }
}
```

- onActive() : LiveData 에 Active 상태의 Observer 가 있을 때 호출.
- onInactive() : LiveData 에 Active  상태의 Observer 가 없을 때 호출. 

사용하는 부분에서는 아래 처럼 사용 할 수 있다. 
```
public class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val myPriceListener: LiveData<BigDecimal> = ...
        myPriceListener.observe(viewLifecycleOwner, Observer<BigDecimal> { price: BigDecimal? ->
            // Update the UI.
        })
    }
```
observe() 에서 LivecycleOwner 를 첫째 인수로 전달, 이렇게 되면 이 observer 는 Fragment lifecycle 에 결합된다.   
이렇게 되면 즉,   
- Lifecycle 객체가 활성 상태 아니면 값이 변경 되더라도 observer  호출 되지 않음
- Lifecycle 개체가 제거되면 observer 는 자동으로 삭제.
- LiveData 가 Lifecycle 을 인식한다는 것은, 여러 activity, fragment, service 간에 객체를 공유할수 있다는 뜻. Singletone 으로 LiveData 클래스를 구현하면 간단.


## LiveData Transformation
Lifecycle 패키지는 LiveData 값을 변경하거나 하기 위한 Transformations 클래스를 제공   
- Transformations.map()
LiveData 객체에 저장된 값에 함수를 적용하고 결과를 다운스트림으로 리턴
```
val userLiveData: LiveData<User> = UserLiveData()
val userName: LiveData<String> = Transformations.map(userLiveData) {
    user -> "${user.name} ${user.lastName}"
}
```

- Transformations.switchMap()
map()과 마찬가지로 LiveData 객체에 저장된 값에 함수를 적용하고 결과를 래핑 해제하여 다운스트림으로 전달. switchMap()에 전달된 함수는 다음 예와 같이 LiveData 객체를 리턴해야 함.
```
private fun getUser(id: String): LiveData<User> {
  ...
}
val userId: LiveData<String> = ...
val user = Transformations.switchMap(userId) { id -> getUser(id) }
```
Transformatiosns 클래스를 이용하면 lifecycle 전반에 걸쳐 정보를 전달할 수 있음. 


