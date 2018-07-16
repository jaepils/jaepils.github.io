---
layout: post
title:  "Kotlin Part#4"
date:   2018-07-12 15:40:56
categories: kotlin
---
## Streams
Java8에서 추가된 Streams가 Kotlin에서는 어떻게 다른지 보도록 하겠습니다.
Java8에서는 Streams를 사용하기 위해서는 List를 .streams()로 변환을 해서 사용해야하지만, kotlin은 기본적으로 제공이 된다는게 가장 큰 차이점 같습니다.

### forEach
우선 루프를 도는 forEach가 어떤 차이가 있는지 보도록 하겠습니다.

```java
List<String> values = Arrays.asList("1", "2", "3");
values.forEach(v -> System.out.println(v));
```        
문자 List를 루프를 돌면서 람다 표현식에 의해서 임의의 글자 위의 예제에서는 v로 정의되는 글자를 출력을 하는 예제입니다.

```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)

    items.onEach { println(it) }
    items.onEach { v -> println(v) }
}
```
마찬가지로 루프를 돌면서 element를 하나씩 출력을 합니다. 틀린점 kotlin의 경우 예약어로 it가 있어서 람다 표현식을 사용하지 않고도 해당 변수 출력이 가능합니다. Java8과 동일한 방식 역시 가능합니다.

### map
Streams의 대표적인 기능인 map의 경우 Java와 kotlin이 크게 다르지는 않습니다. 위에서 언급한대로 List를 Streams으로 변환은 필요 없고, mapToInt와 같은 기능은 map으로만 해도 가능합니다.
map은 하나의 입력을 다른 출력으로 변환을 하는 기능 자체는 크게 변화가 없습니다.

```java
List<Integer> values = Arrays.asList(1, 2, 3, 4);
        int sum = values.stream().mapToInt(v -> v * 2).sum();
        System.out.println(sum);
```
Java에서는 stream()로 변환을 호출하여야 streams의 기능을 이용할 수 있습니다.

```java
fun main(args: Array<String>) {
    val items = listOf(1, 2, 3, 4)

    var sum = items.map { it * 2 }.sum()
    print(sum)
}
```
kotlin의 경우 list에서 바로 map을 호출이 가능하며 sum과 같은 기능도 간단히 사용이 가능합니다.

### chaining of Streams
데이터 소스의 요소에 대해 연산을 순차적으로 수행하고 결과를 집계하려면 소스, 중간 연산 및 터미널 연산의 세 부분으로 구성이 되어 순차적으로 수행이 됩니다.

문자 List를 받아서 다른 문자로 변환후, 그 중 1이 들어간 문자열을 삭제한 결과를 다시 문자 list로 변환하는 조금은 복잡한 예제를 보도록 하겠습니다.
이 예제의 중간 연산자는 map, filter이고, 터미털 연산자는 collect입니다.

```java
List<String> input = Arrays.asList("1", "2", "3");
List<String> result = input.stream()
        .map(x -> x + "-" + x)
        .filter(y -> ! y.startsWith("1"))
        .collect(Collectors.toList());

System.out.println(result);
```
위의 코드는 java8로 작성되어, map으로 변환, filter로 특정 문자 여기서는 "1"이 있는 문자열을 빼고 다시 List로 변환을 하고 있습니다. Streams을 설명할때 보통 많이 설명하는 패턴중에 하나입니다.

```java
fun main(args: Array<String>) {
    val input = Arrays.asList("1", "2", "3")
    val result = input.stream()
            .map { x -> "$x-$x" }
            .filter { y -> !y.startsWith("1") }
            .collect(Collectors.toList())

    println(result)
}
```
위의 자바 코드를 intelliJ에 붙여넣기를 하면 코드를 자동 kotlin 코드로 변환을 해줍니다.
물론 100%는 아니고, 약간의 오류는 있습니다.

우선 외관상 모습을 보더다도, java8과 kotlin 코드는 차이가 보지지 않습니다.

### associate
List element들을 Map으로 변환은 매번 쉽지 않습니다. 아래 예제는 list를 루프를 돌면서 결과를 Map으로 전환하는 방법이 java8와 kotlin이 어떻게 다른지를 쉽게 보여줍니다.

```java
List<String> input = Arrays.asList("1", "2", "3");
Map<String, String> map = input.stream().collect(Collectors.toMap(i -> i, i -> i + "개"));

System.out.println(map);
```
우선 list를 stream으로 변환후 다시 하나의 결과를 모으기 위해서 collect를 호출을 합니다. Collector는 입력받은 문자를 키로 넣고, value에서는 입력받은 문자와 "개"를 더해서 넣고 있습니다.

```java
val input = Arrays.asList("1", "2", "3")

val map = input.associate { n -> n to n + "개" }
println(map)
```
kotlin의 경우는 associate를 사용해서 키와 value를 한번에 정의가 가능합니다.
java8에 비해 코드가 매우 간결한 것을 알 수 있습니다.

kotlin코드를 디컴파일을 하면 아래와 같이 됩니다. 디컴파일된 코드 자체는 streams보다는 예전처럼 iterator가 루프를 돌면서 Map에 넣고 있습니다.

```java
int capacity$iv = kotlin.ranges.RangesKt.coerceAtLeast(kotlin.collections.MapsKt.mapCapacity(kotlin.collections.CollectionsKt.collectionSizeOrDefault($receiver$iv, 10)), 16);

Iterable localIterable1 = $receiver$iv;Map destination$iv$iv = (Map)new java.util.LinkedHashMap(capacity$iv);
int $i$f$associateTo; Iterable $receiver$iv$iv; Map localMap1; kotlin.Pair localPair;

for (java.util.Iterator localIterator = $receiver$iv$iv.iterator(); localIterator.hasNext(); localMap1.put(localPair.getFirst(), localPair.getSecond()))
{
  Object element$iv$iv = localIterator.next();
  localMap1 = destination$iv$iv;String n = (String)element$iv$iv;
  int $i$a$1$associate;
  localPair = kotlin.TuplesKt.to(n, n);
}

Map map = destination$iv$iv;
System.out.println(map);

```

### zip
Streams에서는 두개의 streams을 하나로 묶는 것은 guava없이는 안됩니다.

```java
List<Integer>  numbers = Arrays.asList(1, 2, 3, 4);
List<String>  words = Arrays.asList("one", "two", "three");

int shortestLength = Math.min(numbers.size(),words.size());
List<String> result = new ArrayList<>();
for(int i=0 ; i < shortestLength ; i++) {
    result.add(numbers.get(i) + words.get(i));
}

System.out.println(result);
```
다른 오픈소스를 이용하면 좀더 쉬운 방법은 있겠지만, 제공되는 기능만으로는 쉽게 표현을 할 수 없습니다.

kotlin 의 경우는 이런 경우 쉽게 구현이 가능합니다.

```java
fun main(args: Array<String>) {
    val numbers = listOf(1, 2, 3, 4)
    val words = listOf("one", "two", "three")

    var result = numbers.zip(words)

    print(result)
}

--> [(1, one), (2, two), (3, three)]
```

디컴파일 코드를 보면 다음과 같습니다.
```java
kotlin.jvm.internal.Intrinsics.checkParameterIsNotNull(args, "args");
java.util.List numbers = CollectionsKt.listOf(new Integer[] { Integer.valueOf(1), Integer.valueOf(2), Integer.valueOf(3), Integer.valueOf(4) });
java.util.List words = CollectionsKt.listOf(new String[] { "one", "two", "three" });

java.util.List result = CollectionsKt.zip((Iterable)numbers, (Iterable)words);

System.out.print(result);
```

## 결론
Streams는 Java8에 비해 kotlin이 비교 우위에 있는거 같습니다.

## 참조
* https://www.tutorialspoint.com/kotlin
* https://blog.plan99.net/kotlin-fp-3bf63a17d64a
