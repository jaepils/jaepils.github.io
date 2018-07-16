---
layout: post
title:  "Kotlin Part#3"
date:   2018-07-11 15:40:56
categories: kotlin
---
## Basic Types
Kotlin의 기본문법을 알아보도록 하겠습니다.

### if
if문의 경우 자바와 동일합니다. 두개 이상의 연산을 체크하는 방식도 && 혹은 || 을 사용하고 있습니다.

```java
val a:Int=1
val b:Int=2

if (a > b) {
    print("a is bigger than b")
} else {
    print("b is bigger than a")
}
--> b is bigger than a
```

### when
Java와 유사한 if와는 달리 switch문의 경우 정말 많이 다릅니다. 우선 switch대신에 when만을 사용하며 케이스에 해당하는 표현 역시 다릅니다.
예제를 보도록 하겠습니다.
```java
fun main(args: Array<String>) {
    val x:Int=1
    when (x) {
        1 -> {
            print("case : ")
            print("x == 1")
        }
        2 -> print("case : x == 2")
        else -> {
            print("x is neither 1 nor 2")
        }
    }
}
```
case 문도 없고, break도 없으며 default은 else로 처리가 됩니다.
한줄의 경우는 {} 없이 코드 작성이 가능합니다. 개인적으로는 break를 안써서 버그를 많이 만드는 편이라서 Java보다 나은거 같습니다.

여러 케이스를 한번에 처리하고 할때는 다음의 방식도 가능합니다.
```java
fun main(args: Array<String>) {
    val x:Int=1
    when (x) {
        1,2 -> print(" case : x == 1 or 2")
        else -> {
            print("x is neither 1 nor 2")
        }
    }
}
```

### for
for의 경우 Java도 개선이 되어서 큰 차이가 나지는 않습니다. Java에서 : 대신에 in을 사용을 합니다.

```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)
    for (i in items) {
        println("values of the array : "+ i)
    }
}
```

loop를 돌게 되면 보통 몇번째 index에 있는지에 따라 로직을 분리하고 싶을 수 있습니다.
Java에서는 이런 경우 다음과 같이 코드를 작성을 하게 됩니다.

```java
List<String> values= Arrays.asList("1", "2");
for (int i = 0 ; i < values.size() ; i++) {
    if(i == 2) {
        //do something
    }
}
values.forEach(v -> {
    int index = values.indexOf(v);
    if(index == 2) {
        //do something
    }
});
```
예전 방식으로 for문을 작성해서 index를 구하던가 아니면 루프중에 index를 꺼내야합니다.

kotlin에서는 이런 경우 다음과 같은 코드 작성이 가능합니다.
```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)
    for ((index, value) in items.withIndex()){
        if(index == 2) {
            println(value)
        }
    }
}
--> 3
```
withIndex로 루프를 돌리게 되면 index와 value를 구할 수 있습니다.
index는 0부터 시작을 하기 때문에 2에 해당하는 값은 3입니다.

디컴파일 결과는 다음과 같습니다.
```java
for (IndexedValue localIndexedValue : kotlin.collections.CollectionsKt.withIndex((Iterable)items)) {
  int index = localIndexedValue.component1();
  int value = ((Number)localIndexedValue.component2()).intValue();

  if (index == 2) {
      System.out.println(value);
  }
}
```
IndexedValue는 index와 value를 가진 개체입니다.


### While
while은 if와 마찬가지로 기존과 완전히 동일합니다.
```java
fun main(args: Array<String>) {
    var x:Int=0

    while(x<=10){
        println(x)
        x++
    }
}
```

Java와 크게 다르지는 않아서 문법을 새로 배우는 느낌은 없는거 같습니다.

## 참조
* https://www.tutorialspoint.com/kotlin
