---
layout: post
title:  "java.util.function :: JD"
date:   2018-07-02 15:40:56
categories: java
---
이제는 흔해서 다 알만한 Stream에 관해 정리하고자 합니다. 사실 저는 SI 프로젝트를 오래해서 Stream의 존재를 최근에야 알게 되었습니다.

아시다시피 Stream은 Java8에서 추가된 가장 강력한 기능중에 하나입니다. Stream을 논하기 전에 우선 Functional Interface을 알아야합니다.

Functional Interface는 구현되어야 할 하나의 추상 메소드만을 가진 간단한 인터페이스입니다. 또한 아무런 기능이 포함되어있지 않습니다.

간단한 인터페이스이지만, Stream에서는 이 Functional interface없이 설명을 할 수가 없습니다.


### Predicate
Predicate는 주어진 파라미터가 어떤 예상했던 값인지 체크를 하는 인터페이스입니다.
Predicate가 가진 하나의 추상메소드는 test입니다. 이외에 default로 and나 or를 가지고 있지만 이는 조금 후에 설명하도록 하겠습니다.

{% highlight java %}
Predicate<String> predicateString = s -> s.equals("test");
System.out.println(predicateString.test("test"));

Predicate<String> predicateString2 = new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.equals("test");
            }
        };
{% endhighlight %}
Predicate의 test 메소드는 s가 test 문자인지 체크해서 결과를 반환하고 있습니다. 람다 방식이 아닌 기존 방식으로 풀면 predicateString2가 됩니다.

test 메소드는 equals를 수행한 결과를 반환합니다. 기존 방식에 비해 람다가 코드 라인수가 적음을 알수 있습니다.

보시다시피 predicate는 아주 단순합니다. true or false를 반환을 하면 됩니다.

이제 같이 제공되는 default 메소드를 설명하겠습니다. default 메소드는 and , or가 제공이 되는데 샘플을 보면 다음과 같습니다.

{% highlight java %}
Predicate<String> predicateString = s -> s.equals("test");
Predicate<String> predicateInt = s -> s.length() > 0;

System.out.println(predicateString.and(predicateInt).test("test"));
System.out.println(predicateString.or(s -> s.length() > 0).test("test"));
{% endhighlight %}
Predicate를 2개를 정의를 했습니다. 입력받은 글자가 test인지 확인하는 것과 입력받은 문자의 사이즈가 0보다 큰지를 비교를 했습니다.

사용하는 방법이 default로 제공하는 and나 or를 사용하는 것입니다.

두개의 Predicate가 다 만족하는지를 확인하는 and나 둘중에 하나가 true인지 확인하는 or를 위의 4, 5라인처럼 정의하면 됩니다.

두개 모두 결과는 true입니다.



### Supplier
supplier는 하나의 오브젝트를 결과로 리턴하는 인터페이스입니다. supplier가 가진 하나의 인터페이스는 get입니다. get이외에 제공되는 default 메소드는 없습니다.

{% highlight java %}
Supplier<String> stringSupplier = () -> "test";
System.out.println(stringSupplier.get());
{% endhighlight %}
Supplier는 test라는 메소드를 반환을 하고 있습니다. () ->는 람다에서 파라미터가 없는 경우를 의미합니다. 결과는 test가 됩니다.

Optional의 get과 같다고 생각하면 될거 같습니다.


### Function
function은 입력값에 따라 결과를 생성하여 반환하는 인터페이스입니다. function이 가진 하나의 인터페이스는 apply입니다.
apply이외에 제공되는 default 메소드는 compose, andThen등이 있습니다.

{% highlight java %}
Function<String, Integer> stringIntegerFunction = s -> s.length();
System.out.println(stringIntegerFunction.apply("test"));
{% endhighlight %}
입력으로 받은 문자의 길이를 반환하는 Function을 정의하였습니다. 이 Function에 test를 넘겨주면 결과로 4가 나오게 됩니다.

{% highlight java %}
System.out.println(stringIntegerFunction
                .compose((Function<String, String>) value -> value + "_function")
                .apply("test"));
{% endhighlight %}

두개의 Function을 Compose하는 예제입니다. 이 경우 순서가 매우중요하게 됩니다. 우선 첫번째 function은 입력받은 문자의 길이를 반환을 하고, 두번째 function은 입력받은 글자에 _function을 붙여서 반환을 합니다.

위의 코드를 실행하면 13이 나오는데, 우선 두번째 function이 수행이 되어 test_function이 되고, 첫번째 function이 수행되어 길이가 리턴이 되어 13이 나오게 됩니다. compose의 리턴은 우선 실행될 function입니다.

두번째 default 메소드는 andThen입니다. compose와 반대가 됩니다.

{% highlight java %}
System.out.println(((Function<String, String>) value -> value + "_function")
                .andThen(stringIntegerFunction)
                .apply("test"));
{% endhighlight %}
위와 같은 결과를 얻기 위해서는 function의 순서를 바꿔야 합니다. 만약 andThen의 function의 순서를 바꾸면 stringIntegerFunction은 Integer를 반환함으로 String을 받아야하는 첫번째 function의 파라미터 타입이 안맞아서 오류가 납니다.

결과는 마찬가지로 13입니다.

### Consumer
consumer는 입력은 있지만 결과가 없는 Function입니다. Consumer가 가진 메소드는 accept입니다.
consumer는 accept이외에 andThen을 가지고 있습니다. 리턴이 없는 경우에 andThen이 어떻게 동작하는지 보겠습니다.

{% highlight java %}
Consumer<String> stringConsumer1 = s -> System.out.println("consumer1 : "+s);
Consumer<String> stringConsumer2 = s -> System.out.println("consumer2 : "+s);
stringConsumer1.andThen(stringConsumer2).accept("test");
{% endhighlight %}
결과는 accept에서 consumer1을 호출하고, 같은 파라미터를 consumer2에 넘겨주기 때문에 위의 결과는 모두 test가 찍히게 됩니다.



### 결론
java.util.function 패키지에는 이외에 많은 클래스가 있지만, 대부분 위의 클래스들의 확장입니다.
따라서 앞으로 Stream을 설명할때는 이정도 function만 알고 있어도 무리없을것으로 보입니다.
