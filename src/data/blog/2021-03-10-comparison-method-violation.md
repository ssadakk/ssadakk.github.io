---
author: HM Joo
pubDatetime: 2021-03-10T05:55:19.000Z
title: Comparison Method Violation
slug: comparison-method-violation
featured: false
draft: false
tags:
  - android
  - trouble_shotting
description: Comparison Method Violation
---

Android 사용 중 object 값 비교를 위해 Comparator.compare 를 override 하는 중 발생하는 Timsort 에서 Comparation method vioration 에 대한 정리
<!--more-->

## 참고 동영상
- [https://www.youtube.com/watch?v=Enwbh6wpnYs](https://www.youtube.com/watch?v=Enwbh6wpnYs)
- [https://www.youtube.com/watch?v=bvnmbRo7a1Y](https://www.youtube.com/watch?v=bvnmbRo7a1Y)

위 영상을 보면 여러가지 문제되는 케이스들이 제기되기는 하지만, 나의 경우 명확히 찾아내지 못해 발생했던 문제 케이스 몇가지를 listup 함.

## Comparison method violates it's genral contract!
> The implementor must ensure that sgn(compare(x, y)) == -sgn(compare(y, x)) for all x and y. (This implies that compare(x, y) must throw an exception if and only if compare(y, x) throws an exception.)
>The implementor must also ensure that the relation is transitive: ((compare(x, y)>0) && (compare(y, z)>0)) implies compare(x, z)>0.
>Finally, the implementor must ensure that compare(x, y)==0 implies that sgn(compare(x, z))==sgn(compare(y, z)) for all z.
결국    
A > B and B > C then for any A, B and C: A > C   
가 성립해야 된다는 말.

## 발생하는 문제
일반적으로 적당한 값들의 비교에서는 별다른 문제가 생지기 않지만 특정 경우 문제가 발생하는 경우가 있다.
실제로 나의 경우 아래와 같은 코드에서 문제가 생겼다.

```java
public int compare(Object o1, Object o2) {
    return (int) (o1.getLongValue() - o2.getLongValue());
}
```

### 1. Integer.MIN_VALUE 문제
- (int) (o1.getLongValue() - o2.getLongValue()) 로 비교문 쓰는 곳이 많은데, 둘 다 long value 인 상태.   
이 때, compare() 의 return value 가 Integer.MIN_VALUE() 인 경우가 문제 될수 있음.   
o1, o2 parameter 순서를 바꿔도 Integer.MIN_VALUE() 가 나오는 경우가 발생할 수 있다.  
이렇게 되는 경우 sgn(compare(x,y)) == -sgn(compare(y,x)) 규칙에 위반되므로 Exception 발생 

### 2. Integer overflow
나의 경우 위의 compare() 를 시간을 비교하기 위한 경우로 사용했다.   
이런 경우에 생길 수 있는 문제.   
Integer.MAX_VALUE 값은 환산하면 24일.  즉 24일 이상 차이나는 아이템이 문제가 될 수 있음.   
예) 각 아이템에 저장된 날짜가 1월 1일, 15일, 30일 인 경우,    
compare(1일, 15일) ⇒ 14일   
compare(15일, 30일) ⇒ 15일,    
compare(1일, 30일) ⇒ 29일, Overflow 발생, 마이너스 값 리턴 발생.   
정리하면    
compare(1일, 15일)  > 0   
compare(15일, 30일) > 0   
compare(1일, 30일) > 0 이어야 하지만 마이너스 값 리턴되기 때문에  < 0   
Long.compare() 를 사용하거나 Comparator.comparing(RecExtProgram::getStartTimeUtcMillis) 를 사용해 회피함   

### 3. String compare
이건 String compare 중 발생한 문제인데, Object Null , compareTo() 메소드를 바로 사용하는 경우 문제 발생하더라.
한 4000개 정도 되는 String 비교중 특정 상황에서만 발생하는데, 나는 compareTo() 메소드가 1, 0, -1 만 리턴하는줄 알고 있었는데 그게 아니었다. 또 Null 과의 compare 문제도 발생하고. 그래서 아래처럼 Null check 와 함께 return value 를 1, 0, -1로 만들어주니 문제가 해결됨

```java
programList.sort((p1, p2) -> {
    int res = 0;
    if(p1.getString() != null && p2.getString() != null) {
        res = p1.getString().compareTo(p2.getString());
    }
    return Integer.compare(res, 0);
});
```


