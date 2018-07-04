---
layout: post
title:  "Map과 Flatmap의 차이"
date:   2018-06-27 15:40:56
categories: java
---
### 정의
java 8에서 추가된 funtional languages에서 시작이 되었습니다.



### Map
많이 사용이 되는 Optional과 Stream에서 map의 함수정의를 보면 다음과 같습니다.

{% highlight java %}
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
<U> Optional<U> map(Function<? super T, ? extends U> mapper)
{% endhighlight %}

첫번째 라인은 Stream에서 map의 정의이고 두번째 라인은 Optional에서의 map입니다. 둘다 차이가 없기 때문에 두번째 Optional로 설명을 하면 파라미터로 받은 Function에서 extends U를 리턴이 되는 간단한 기능입니다.

이 의미를 살펴보면 다음과 같습니다.



{% highlight java %}
Optional<String> stringOptional = Optional.ofNullable("hi");
Optional<Integer> integerOptional = stringOptional.map(s -> s.length());
Optional<Integer> integerOptional2 = stringOptional.map(new Function<String, Integer>() {
            @Override
            public Integer apply(String s) {
                return s.length();
            }
        });
Optional<String> stringOptional2 = stringOptional.map(s -> " java");
{% endhighlight %}

우선 Optional 개체를 하나 생성을 합니다. Optional은 of메소드나 ofNullable를 사용하여 생성이 가능합니다.

두번째 라인을 보면 map 함수를 이용해서 입력받은 문자의 길이를 반환을 하고 있습니다. 두번째 map코드만 보면 Funtion이 보이지 않는다고 생각할 수 있는데, s -> s.length()가 function이고, s는 정의에서 T가 되고, s.length는 정의에서 R이 됩니다. 이를 예전 방식으로 풀어서 정의를 하면 3번째 라인이 됩니다.

3번째 라인을 보면, new Function을 정의를 했고, 타입으로 String, Integer를 정의를 했습니다. 반환은 반드시 Integer이어야 하고, 왜냐면 map의 파라미터인 Funtion의 두번째 타입으로 리턴을 해야하기 때문입니다.

리턴된 타입은 Optional이 되는데 Optional.map의 리턴타입은 무조건 Optional입니다. Stream에서는 Stream이 됩니다. 문자를 받아서 길이라는 숫자를 반환을 했지만, 다시 Optional에 담겨져서 반환이 됩니다.

결론적으로는 map은 앞에서 넘겨 받은 데이터를 가지고 한번 가공해서 다시 넘겨 받는 간단한 기능을 제공을 합니다.



### FlatMap
map과 마찬가지로 flatMap도 Optional과 Stream에서 제공합니다. 이둘은 비슷하면서 조금 다릅니다.

우선 정의를 보면 다음과 같습니다.

{% highlight java %}
<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper);
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);
{% endhighlight %}
Funtion의 두번째 파라미터가 Optional이나 Stream으로 map과 다름을 알수가 있습니다.

map에서 했던 코드를 flatMap으로 전환하면 다음과 같이 됩니다.

{% highlight java %}
Optional<String> stringOptional = Optional.ofNullable("hi");
Optional<Integer> integerOptional = stringOptional.flatMap(s -> Optional.of(s.length()));
{% endhighlight %}
map과 같은 Optional<Integer>로 리턴하기 위해서는 flatMap의 경우 s.length를 반환하는게 아닌 Optional개체를 반환을  해야합니다.

즉 이렇게 코딩을 해야하는 경우 flatMap보다 map이 적합하다고 할 수 있습니다. 하지만 flatMap은 다음과 같은 코딩이 가능합니다.

{% highlight java %}
String[][] data = new String[][]{ {"1", "2"}, {"3", "4"} };

Stream<Stream<String>> map = Arrays.stream(data).map(x -> Arrays.stream(x));
map.forEach(s -> s.forEach(System.out::println));

Stream<String> flatmap = Arrays.stream(data).flatMap(x -> Arrays.stream(x));
flatmap.forEach(System.out::println);
{% endhighlight %}

Map의 경우 이중 for로 해결을 못하는 이슈가 있습니다. 즉 data와 같이 그룹핑이 되어있을 경우 map으로하게 되면 for 문을 2번 돌리는 듯하게 코드를 작성을 해야하는데 비하여 flatmap은 첫번째 받은 데이터를 다시 stream으로 리턴해서 for를 한번만 수행해도 같은 결과를 얻을 수 있습니다.

### 결론
map과 flatMap 모두 쉽게 사용이 가능하며 차이점은 하나의 입력을 다시 하나의 리턴인 경우에는 map을 하나의 입력을 다시 stream 혹은 Optional로 변환을 하고자할때는 flatmap이 맞습니다.
