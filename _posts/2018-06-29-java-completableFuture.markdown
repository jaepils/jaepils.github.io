---
layout: post
title:  "CompletableFuture"
date:   2018-06-29 15:40:56
categories: java
---
쉽지 않은 CompletableFuture를 정리해보겠습니다.

비동기 프로그래밍의 의미는 메인 쓰레드에서 분리된 쓰레드에서 동작하며 처리 완료후 성공 혹은 실패를 메인쓰레드에 알려주는 non-blocking 코드를 의미합니다.

우선 CompletableFuture는 Future와 CompletionStage를 구현한 클래스입니다. Future 인터페이스 자체는 Java5에서 제공이 되었지만, computation들을 중첩 시키거나 에러를 처리하는 방법을 제공하지 않았다고 합니다. java1.8에서 이에 여러 computation을 처리하기 위해서 CompletionStage가 추가되었고, 이를 구현한 CompletableFuture는 여러가지 손쉬운 기능들을 제공하고 있습니다. 물론 손쉽다고해서 쉬운건 아니기 때문에 샘플을 통해 기능들을 확인해보겠습니다.

### RunAsync
가장 간단한 비동기적으로 완료가 되는 CompletableFuture를 생성하는 방법은 runAsync입니다.  
runAsync 메소드는 supplyAsync와 달리 파라미터가 Runnable 이기 때문에 반환값이 없습니다.
{% highlight java %}
static CompletableFuture <Void> runAsync(Runnable runnable)
{% endhighlight %}
CompletableFuture 에 타입이 Void로 생성이 되고 있습니다.
예제를 보면 다음과 같습니다.

{% highlight java %}
CompletableFuture.runAsync(() -> {
            log.info("test");
        }).get();
{% endhighlight %}
리턴값이 없기 때문에 get()에서 아무것도 반환이 되지 않습니다.

{% highlight java %}
14:28:15.873 [ForkJoinPool.commonPool-worker-1] INFO Sample - test
{% endhighlight %}
실행결과는 ForkJoinPool에서 얻은 분리된 쓰레드에서 Runnable이 수행이 됨을 알 수 있습니다.

SupplyAsync
supplyAsync는 runAsync와 달리 결과를 리턴할 수 있습니다.  파라미터로는 Supplier을 받는데 이는 단순히 하나의 값만 리턴하는 인터페이스입니다.

supplyAsync 정의를 보면 다음과 같습니다.

{% highlight java %}
static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)
{% endhighlight %}
파라미터로 Supplier를 받아서 CompletetableFuture를 반환을 하고, 사용하는 타입은 한가지 타입만이 사용이 됩니다.

샘플코드를 보면 좀더 이해가 쉬울거 같습니다.

{% highlight java %}
log.info(CompletableFuture.supplyAsync(() -> {
            log.info("test");
            return "test";
        }).get());
{% endhighlight %}
이해를 돕기 위해서 로그를 남겼습니다. 로그는 CompletableFuture를 get한 이후에 남기는 것과 Supplier에서 남겨서 두군데에 로그를 남기고 있습니다.
결과를 보면 다음과 같습니다.

{% highlight java %}
13:11:07.754 [ForkJoinPool.commonPool-worker-1] INFO Sample - test
13:11:07.757 [main] INFO Sample - test
{% endhighlight %}
처음 로그는 Supplier에서 찍혔고, 두번째는 CompletableFuture를 get 한 이후 로그가 찍혔습니다. 로그를 보면 쓰레드가 다른 것을 알수 있습니다.
Supplier는 supplyAsync가 실행하면서 ForkJoinPool에서 얻은 분리된 쓰레드에서 수행을 하고 있고, 두번째는 로그는 메인 쓰레드에서 동작하고 있습니다.
즉 간단하게 쓰레드가 분리가 되어 수행이 되는 것을 알수 있습니다.


### Completed CompletableFuture
입력받은 값으로 이미 완료된 CompletableFuture를 생성하는 가장 간단한 api입니다.

{% highlight java %}
CompletableFuture cf = CompletableFuture.completedFuture("done");
assertTrue(cf.isDone());
assertEquals("done", cf.getNow("failed"));
{% endhighlight %}

### thenAccept
thenAccept메소드는 파라미터로 Consumer를 받습니다. Completion stage에서 어떤 completion이 발생하면 thenAccept는 CompletableFuture에서 받은 결과를 Consumer에게 넘겨줍니다.

예제를 보도록 하겠습니다.

{% highlight java %}
CompletableFuture.supplyAsync(() -> {
        log.info("test1");
        return "test1";
 }).thenAccept(s -> log.info("complete1 {}", s));

 CompletableFuture.runAsync(() -> {
        log.info("test2");
 }).thenAccept(s -> log.info("complete2 {}", s));
{% endhighlight %}

당연하게도 supplyAsync와 runAsync는 CompletableFuture를 반환하고, 반환받은 CompletableFuture에 callback을 등록하는 것이기 때문에 둘간의 차이는 없습니다.

다만 supplyAsync는 리턴값이 있지만, runAsync는 리턴값이 없기 때문에 complete2의 경우 null이 찍히게 됩니다.

수행 결과를 보도록 하겠습니다.

{% highlight java %}
14:50:49.923 [ForkJoinPool.commonPool-worker-1] INFO Sample - test1
14:50:49.923 [ForkJoinPool.commonPool-worker-2] INFO Sample - test2
14:50:49.925 [ForkJoinPool.commonPool-worker-1] INFO Sample - complete1 test1
14:50:49.925 [ForkJoinPool.commonPool-worker-2] INFO Sample - complete2 null
{% endhighlight %}
complete2는 언급된 대로 null이 찍힙니다.

위의 결과중 중요한 것은 supplyAsync와 runAsync를 순서대로 수행하지만, 둘간의 쓰레드는 worker-1과 worker-2로 틀리고, 결과가 나오는 것도 순서가 코드 순이 아닙니다.

비동기 프로그래밍의 경우 순서를 보장하지 않기 때문에 주의가 필요합니다.



### thenApply
thenAccept는 리턴이 CompletableFuture이기 때문에 중첩이 가능합니다. 따라서 다음처럼 작성이 가능합니다.

{% highlight java %}
CompletableFuture.supplyAsync(() -> {
            log.info("test1");
            return "test1";
        }).thenAccept(s -> log.info("complete1 {}", s))
          .thenAccept(s -> log.info("complete2 {}", s));
{% endhighlight %}

위의 결과는 complete1은 test1이 찍히지만, complete2는 null이 찍힙니다. thenAccept는 리턴이 completableFuture이지만 타입이 void여서 결과가 전달을 할 수 없습니다.

이런 경우 callback처리를 중첩시키고 싶다면 thenApply를 사용하면 됩니다.

{% highlight java %}
CompletableFuture.supplyAsync(() -> {
    log.info("test1");
    return "test1";
}).thenApply(s -> {
    log.info("complete1 {}", s);
    return s;
}).thenAccept(s -> log.info("complete2 {}", s));
{% endhighlight %}

이렇게 하면 complete1과 complete2에 모두 test1이 찍힙니다. 다만 thenApply의 경우 반환값이 있어야 하기 때문에 입력받은 s를 다시 리턴을 했습니다.

{% highlight java %}
16:05:56.412 [ForkJoinPool.commonPool-worker-1] INFO Sample - test1
16:05:56.414 [ForkJoinPool.commonPool-worker-1] INFO Sample - complete1 test1
16:05:56.416 [ForkJoinPool.commonPool-worker-1] INFO Sample - complete2 test1
{% endhighlight %}
thenApply와 thenAccept는 모두 같은 쓰레드에서 수행이 됩니다.


### thenCompose
이제 여러 Future들을 묶는 케이스를 알아보도록 하겠습니다. 두개의 서로 연관성이 없는 Future가 있다면, 그리고 그 두개를 모두 수행한 결과를 얻고 싶은 경우 하나의 Future가 끝나고 두번째 Future가 수행이 되는 즉 Chain처럼 연결되어 줄지어서 수행이 되는 케이스와, 두개의 Future가 동시에 수행이 되고, 즉 Parallel하게 수행하여 그 결과가 머지되는 케이스가 있습니다.

우선 Cain처럼 줄지어서 수행이 되는 케이스에서는 thenCompose가 사용이 됩니다.

{% highlight java %}
CompletableFuture.supplyAsync(() -> {
    log.info("test1");
    return "test1";
}).thenCompose(s -> CompletableFuture.completedFuture(s + "2"))
  .thenAccept(s -> log.info("complete {}", s));
{% endhighlight %}
위의 코드는 두개의 CompletetableFuture가 사용이 됩니다. 첫번째는 test1을 리턴하고, 두번째 Future에서는 입력받은 값에 문자 2를 추가하고 있습니다.
이를 수행한 결과는 test12가 됩니다. 첫번째가 수행이되고, 그 결과를 두번째가 받아서 결과가 나오게 됩니다.

즉 콜백함수 thenAccept에서 CompletableFuture를 사용하게 된 경우 thenAccept말고 thenCompose를 사용하면 됩니다.



### thenCombine
thenCompose와 달리 thenCombine은  서로 다른  Future가 동시에 수행이 되어 그 결과가 머지되는 됩니다.

{% highlight java %}
CompletableFuture anotherCompletableFuture = CompletableFuture.supplyAsync(() -> {
            log.info("future-2");
            return "2";
        });

        CompletableFuture.supplyAsync(() -> {
            log.info("future-1");
            return "test1";
        }).thenCombine(anotherCompletableFuture, (s1, s2) -> s1 + s2)
          .thenAccept(s -> log.info("complete {}", s));
{% endhighlight %}
코드가 이전보다는 조금 복잡해졌습니다. 우선 CompletableFuture는 compose와 동일하게 2개가 사용이 되었습니다. 다만 사용하는 방법이 다르고 서로 다른 쓰레드에서 수행이 되기 때문에 로그를 남기기 위해서 CompletableFurture가 사용하지 않았습니다.
 마찬가지로 첫번째 Future에서는 test1을 반환을 하고, 두번째 anotherCompletableFurture에서는 문자 2를 반환을 합니다.

두개를 머지하는 방법은 thenCombine의 첫번째 파라미터에 Future를 넣고, 두번째 파라미터에 두개의 값을 받아서 하나의 값을 리턴하는 BiFunction을 사용하여 결과를 머지한 후 콜백에 넘겨주게 됩니다.

결과를 수행하면 다음과 같이 쓰레드가 분리됨을 알 수 있습니다.

{% highlight java %}
08:35:11.132 [ForkJoinPool.commonPool-worker-2] INFO Sample - future-2
08:35:11.132 [ForkJoinPool.commonPool-worker-1] INFO Sample - future-1
08:35:11.135 [ForkJoinPool.commonPool-worker-1] INFO Sample - complete test12
{% endhighlight %}
worker-1, worker-2로 서로 다른 쓰레드가 분리되어 동시에 수행한 후 결과가 머지가 됩니다.

만약 2개 이상의 CompletableFuture가 사용이 되는 경우는 어떻게 해야하는지 알아보도록 하겠습니다.


### CompletableFuture.allOf()
2개 이상의 독립적인 Future들을 모두 동시에 수행한 후 머지 끝난 경우 머지가 되는 경우 사용이 됩니다. CompletableFuture.allOf는 리턴값이 void입니다.

따라서 콜백이 아니면 결과를 얻을 수가 없습니다. 이 경우 콜백이 어떻게 수행이 되는지 보도록 하겠습니다.

{% highlight java %}
CompletableFuture completableFuture1 = CompletableFuture.supplyAsync(() -> {
    log.info("future-1");
    return "1";
});

CompletableFuture completableFuture2 = CompletableFuture.supplyAsync(() -> {
    log.info("future-2");
    return "2";
});

CompletableFuture completableFuture3 = CompletableFuture.supplyAsync(() -> {
    log.info("future-3");
    return "3";
});

CompletableFuture.allOf(completableFuture1, completableFuture2, completableFuture3)
        .thenAccept(s -> log.info("complete {}", s));
{% endhighlight %}
CompletableFuture는 3개가 정의 되어 있습니다.  CompletableFuture.allOf는 3개의 Future를 동시에 수행을 하고 모두 완료가 된 후 콜백이 수행이 됩니다.

콜백에서는 thenCombine처럼 데이터를 머지하는 Function을 쓰지 않고 있습니다. 이 경우 어떤 값이 나오게 될까요?

값은 널입니다.  값을 얻기 위해서는 다음처럼 수정을 해야 합니다.



{% highlight java %}
List<completablefuture> futures = Arrays.asList(completableFuture1,
                      completableFuture2,
                      completableFuture3);
CompletableFuture.allOf(completableFuture1, completableFuture2, completableFuture3)
       .thenAccept(s -> {
            List<object> result = futures.stream()
                    .map(pageContentFuture -> pageContentFuture.join())
                    .collect(Collectors.toList());
            log.info(result.toString());
        });
{% endhighlight %}

### CompletableFuture.anyOf()
CompletableFuture.allOf는 모든 CompletableFuture가 끝나서 머지가 되는 경우라면 CompletableFuture.anyOf는 CompletableFuture가 먼저 종료되면 다른 Future에 상관없이 완료가 되며 먼저 종료된 CompletableFuture가 CompletableFuture.anyOf()의 리턴이 되며 타입은 CompletableFuture<Object>입니다. 중요한 것은 모두 한번씩은 수행을 하기 때문에 다른 Future가 먼저 완료가 되어도 모든 Future들은 수행이 됩니다. 가장 먼저 종료가 되지 못한 CompletableFuture의 결과는 무시가 됩니다.

{% highlight java %}
CompletableFuture completableFuture1 = CompletableFuture.supplyAsync(() -> {
    log.info("future-1");
    return "1";
});

CompletableFuture completableFuture2 = CompletableFuture.supplyAsync(() -> {
    log.info("future-2");
    return "2";
});

CompletableFuture completableFuture3 = CompletableFuture.supplyAsync(() -> {
    log.info("future-3");
    return "3";
});

CompletableFuture.anyOf(completableFuture1, completableFuture2, completableFuture3)
                .thenAccept(s -> log.info(s.toString()));
{% endhighlight %}
결과는 다음과 같습니다.

{% highlight java %}
09:27:42.774 [ForkJoinPool.commonPool-worker-1] INFO Sample - future-1
09:27:42.774 [ForkJoinPool.commonPool-worker-3] INFO Sample - future-3
09:27:42.777 [ForkJoinPool.commonPool-worker-1] INFO Sample - 1
09:27:43.776 [ForkJoinPool.commonPool-worker-2] INFO Sample - future-2
{% endhighlight %}
가장 먼저 완료가 된 Future-1의 결과가 콜백에 넘겨져서 로그에 찍히게 됩니다.


### 참조
https://www.callicoder.com/java-concurrency-multithreading-basics/
https://www.callicoder.com/java-8-completablefuture-tutorial/
