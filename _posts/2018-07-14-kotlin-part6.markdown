---
layout: post
title:  "Kotlin Part#6"
date:   2018-07-13 15:40:56
categories: kotlin
---
## NULL
kotlin의 타입 시스템은 코드에서의 null 참조의 위험을 제거할 수 있도록 설계되었습니다.
kotlin에서는 null (nullable 참조)을 가질 수 있는 참조와 (null이 아닌 참조) 가질 수 없는 참조를 구별합니다.

### nullable
기본적으로 변수는 null일수가 없습니다.
다음 코드를 보면 java와 달리 컴파일 오류가 발생합니다.

```java
var a: String = "abc"
a = null -> compile error

print(a)
```    

같은 변수(var)이더라도, nullable을 만들고 싶은 경우 타입에 ?을 붙이면 됩니다.
```java
var b: String? = "abc"
b = null // ok
print(b)

val c: String? = null
print(c)

--> null
--> null
```

따라서 null 일수가 없는 변수의 경우 null체크 없이 length와 같은 기능을 바로 사용을 해도 절대 NPE가 발생하지 않습니다.
```java
fun main(args: Array<String>) {

    var a: String = "abc"
    a = "abcd"

    assertEquals(a.length, 4)
}
```
기본적으로, kotlin은 해당 변수가 null일수 없다고 컴파일 타임에 판단을 내립니다.
nullable 변수의 경우는 다릅니다.
```java
fun main(args: Array<String>) {

    var b: String? = "abc"
    b = null // ok
    print(b)

    assertEquals(b.length, 4) -> compile error
}
```
위의 코드는 해당 변수는 nullable이기 때문에 컴파일 오류가 발생합니다.

### Safe Calls
nullable인 변수의 길이를 출력을 하기 위해서는 기존 java의 경우 다음처럼 null을 한번 체크를 한 후 사용이 일반적인 방법입니다.

```java
var b: String? = "abc"
b = null // ok
print(if (b != null) b.length else -1)
```
if문으로 b가 null인지를 체크를 하고, b.length를 호출하면 컴파일 오류가 발생하지 않습니다.
다만 이렇게 코드를 작성하는것은 매번 귀찮은 일입니다.

```java
var b: String? = "abc"
b = null // ok
print(b?.length)
print(b?.length ?: -1)
```
kotlin에서는 b?.length를 사용하게 되면, null인 경우 null을 리턴하고 아니면 길이를 리턴할 수 있습니다.
만약 null인 경우 다른 값으로 치환 하고 싶은 경우는 b?.length ?: -1 로 하면 됩니다.
java8에 추가된 Optional과 매우 유사한거 같습니다.

### Nullable Unsafe Get
위에서 안전하게 값을 꺼내는 방식이 아닌 NPE를 감수하고 변수를 사용하는 방법이 있습니다. 사실 개인적으로는 왜 이런 기능을 제공해야했는지 이해가지는 않습니다. 사용에 대단히 조심을 해야할거 같습니다.
```java
fun main(args: Array<String>) {
    var b: String? = "value"
    b = null

    print(b!!.length)
}

--> Exception in thread "main" kotlin.KotlinNullPointerException
```
예상했던 대로 에러가 발생합니다.


## 결론
kotlin도 java8에서 추가된 Optional과 매우 유사한 기능을 제공하고 있습니다.

## 참조
* https://kotlinlang.org/docs/reference/null-safety.html
