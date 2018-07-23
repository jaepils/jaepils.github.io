---
layout: post
title:  "Kotlin Part#2"
date:   2018-07-10 15:40:56
categories: kotlin
---
## Basic Types
Kotlin에서 사용 가능한 데이터 타입을 알아보록 하겠습니다.

### Var vs Val
kotlin에서는 변수 정의시 2가지 타입이 있습니다.

- var : 여러번 값을 할당하여 쓰는 변수의 경우에 사용됩니다.
- val : 한번 값이 정해지면 바뀌지 않는 상수에 해당합니다.

### Int
Integer는 최소 -2147483648에 최대 2147483647까지 이고, 동작 방식은 Primitive int 보다는 java.lang.Integer와 유사합니다.
Integer 변수 생성 방식은 다음과 같습니다.

```java
fun main(args: Array<String>) {
    val intValue: Int = 10000
    val intValue2 = 10000

    println(intValue)
    println(intValue2)
}
--> 10000
--> 10000
```
숫자 타입을 println에 넣어도 정상적으로 출력이 됩니다. Int를 명시하지 않아도 Integer임을 알고 있어서 intValue와 intValue2는 모두 같은 동작을 합니다.

다음처럼 이미 정의된 상수에 다른 값을 넣는 것은 오류가 발생합니다. 변경하고 싶은 경우 var로 변수로 정의해야합니다.

```java
fun main(args: Array<String>) {
    val intValue1 = 10000
    intValue1 = 1 --> reassigned error

    println(intValue1)
}
--> 10000
```

다른 타입으로의 캐스팅 즉 int를 float로 전환하고 싶을때도 (float) 가 아닌 내부 제공해주는 함수를 사용해야 합니다.
```java
fun main(args: Array<String>) {
    val intValue: Int = 10000
    val flatValue: Float = intValue.toFloat()

    println(flatValue)
}
--> 10000.0
```
다음과 같이 기본적인 연산은 가능합니다.
```java
fun main(args: Array<String>) {
    val intValue1: Int = 10000
    val intValue2: Int = intValue1 - 100
    val intValue3: Int = intValue1 + 100
    val intValue4: Int = intValue1 * 100
    val intValue5: Int = intValue1 / 100
    val intValue6: Int = intValue1 % 100

    println(intValue1)
    println(intValue2)
    println(intValue3)
    println(intValue4)
    println(intValue5)
    println(intValue6)
}
--> 10000
--> 9900
--> 10100
--> 1000000
--> 100
--> 0
```

혹은 내부 함수를 이용할 수 있습니다.
```java
fun main(args: Array<String>) {
    val intValue1: Int = 10000
    val intValue2: Int = intValue1.minus(100)
    val intValue3: Int = intValue1.plus(100)
    val intValue4: Int = intValue1.times(100)
    val intValue5: Int = intValue1.div(100)
    val intValue6: Int = intValue1.mod(100)

    println(intValue1)
    println(intValue2)
    println(intValue3)
    println(intValue4)
    println(intValue5)
    println(intValue6)
}
--> 10000
--> 9900
--> 10100
--> 1000000
--> 100
--> 0
```
연산을 수행할때 타입이 다른 경우 더 큰 변수의 타입으로 캐스팅이 됩니다. 즉 Int보다 작은 byte나 short은 int가 되고, Long을 minus에 파라미터로 전달이 되면 리턴은 Long이 됩니다. 마찬가지로 Float는 Float로 Double은 Double로 리턴타입이 달라지게 됩니다.

이외에도 비트 연산자 등을 제공하고 있습니다.
Float, Long, Double은 모두 동일하기 때문에 바로 문자로 넘어가도록 하겠습니다.

### Characters
Java에서 char라고 생각하면 됩니다. 내부 함수는 java.lang.Character처럼 숫자로 변환정도 제공을 해주고 있습니다. java.lang.Character에서 처럼 대소문자 전환 등은 제공이 되고 있지 않습니다. 사실 많이 쓰이는 기능이 아닙니다.
```java
fun main(args: Array<String>) {
    val charValue1: Char = 'A'
    val charValue2 = 'B'

    println(charValue1)
    println(charValue2)
}
--> A
--> B
```
val로 정의를 했기 때문에 상수입니다. 상수는 숫자 타입과 마찬가지로 reassign은 되지 않습니다.
```java
fun main(args: Array<String>) {
    val charValue1: Char = 'A'

    println(charValue1.toInt())
    println(charValue1.inc())
}
--> 65
--> B
```
문자 A의 Ascii value는 65입니다.
Char는 제공해주는 기능이 별로 없어서 넘어가겠습니다.

### Boolean
boolean은 다른 프로그래밍 언어처럼 매우 간단합니다. 단 두가지 값중 하나를 가지게 됩니다.
제공해주는 기능도 java.lang.Boolean과 마찬가지로 거의 없습니다.

```java
fun main(args: Array<String>) {
    val booleanValue1: Boolean = false

    println(booleanValue1)
}
--> false
```
설명할게 없네요.

### Strings
문자는 char의 배열입니다. 기본적으로 java와 마찬가지로 많은 함수를 제공합니다.

```java
fun main(args: Array<String>) {
    var stringValue: String = "ABC"

    println(stringValue)
    println(stringValue[0])
    println(stringValue.length)

    stringValue = "DE"
    println(stringValue)
    println(stringValue[0])
    println(stringValue.length)
}
--> ABC
--> A
--> 3
--> DE
--> D
--> 2
```
변수로 정의를 했기 때문에 수정이 가능합니다.
조금전에 설명한대로 String은 char의 배열이기 때문에 배열처럼 순서대로 꺼낼수 있습니다. 첫번째 char를 꺼내면 A가 됩니다.
문자의 길이를 리턴하는 length를 호출하면 3이 나옵니다.

만약 다음처럼 코드를 작성하면 결과가 어떻게 나올까요?

```java
fun main(args: Array<String>) {
    val stringValue: String = "ABC"

    stringValue.plus("DEF")
    println(stringValue)
}
```
결과는 ABC가 나오게 됩니다. DEF가 더해진 문자는 새로운 변수입니다. 타입이 val 에서 var로 변경이 되었습니다.

```java
fun main(args: Array<String>) {
    val stringValue: String = "ABC"

    var stringValue2 = stringValue.plus("DEF")
    var stringValue3 = stringValue + "DEF"

    println(stringValue2)
    println(stringValue3)
}
--> ABCDEF
--> ABCDEF
```

### Arrays
Arrays는 생성 방법이 다릅니다. 게다가 배열의 타입도 명시를 해야하기 때문에 이부분이 기존 자바와 많이 다른거 같습니다.

우선 기본적인 Integer 배열을 생성하는 코드를 보도록 하겠습니다.
```java
fun main(args: Array<String>) {
    val intArray: IntArray = intArrayOf(1, 2, 3, 4, 5)

    println(intArray)
    println(intArray.size)
    println(intArray.component5())
}
--> [I@49476842
--> 5
--> 5
```
배열을 생성할때는 타입별로 intArrayOf, booleanArrayOf 등을 제공을 합니다.
생성된 배열을 바로 출력을 해보면 배열이기 때문에 인스턴스 hashCode가 찍히게 됩니다.

배열의 크기는 size로 알 수 있습니다. 문자에서는 length로 한 것과 다르게 size 로 네이밍을 한게 흥미롭습니다. 자바에서는 둘다 legth이기 때문입니다. 자바에서는 List 타입에서만 size 를 사용을 합니다.

특이하고, 이해안가는 것은 component1 ~ component5 인데, 1부터 5까지의 요소는 별로 api로 값을 알수 있습니다. 왜 5까지 인지 진심 궁금하네요.

### Collections
Kotlin은 collection을 immutable 아니면 mutable 하게 생성이 가능합니다. 참고적으로 java9에서는 immutable collection이 추가되었습니다.

기본적인 List를 보도록 하겠습니다.
```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)

    println(items)
    println(items.size)
    println(items.component2())

}
--> [1, 2, 3, 4]
--> 4
--> 2
```
다행히 배열과 다르게 출력시 hashCode가 아닌 element들이 보여집니다. 길이는 size로 알수가 있고, 배열처럼 component1~5까지 제공이 됩니다. 이 메소드는 내부적으로 get(index)를 호출이 됩니다.

java.util.List와 다르게 제공되는 함수가 get, iterator정도뿐이여서 많이 다릅니다.

#### MutableList
listOf로 생성된 List는 element에 접근을 할 뿐 수정할 방법을 제공하고 있지 않습니다. 내부 element를 컨트롤 하기 위해서는 MutableList로 생성을 해야합니다.

```java
fun main(args: Array<String>) {
    val mutableList: MutableList<Int> = mutableListOf(1, 2, 3) //mutable List

    println(mutableList)
    println(mutableList.size)
    println(mutableList.component2())

    mutableList.add(4)
    println(mutableList.size)

    mutableList.removeAt(3)
    println(mutableList.size)

    mutableList.set(1, 7)
    println(mutableList)
}
--> [1, 2, 3]
--> 3
--> 2
--> 4
--> 3
--> [1, 7, 2, 3]
```
이제서야 java.util.List와 유사한거 같습니다.

mutableList의 디컴파일 결과는 다음과 같습니다.
```java
List items = kotlin.collections.CollectionsKt.listOf(new Integer[] { Integer.valueOf(1), Integer.valueOf(2), Integer.valueOf(3), Integer.valueOf(4) });

System.out.println(items);
int i = items.size();System.out.println(i);
List localList1 = items;
int j = ((Number)localList1.get(1)).intValue();
System.out.println(j);
```

MutableList를 생성하는 방법이 있었으니 당연히 immutableList를 생성하는 방법이 있을거 같아서 찾아봤는데, 없었습니다. 이유는 List 자체가 이미 immutable이기 때문이였습니다.

다음에는 기본 문법을 보도록 하겠습니다.

## 참조
* https://www.tutorialspoint.com/kotlin
