---
layout: post
title:  "Single"
date:   2018-06-14 15:40:56
categories: java RxJava2
---
### 정의
Single 클래스는 하나의 값만을 응답하는 Reactive Pattern의 구현체입니다.

Single은 Observable과 유사하며, Observable과 다르게 onSuccess와 onError만을 가지고 있습니다. 즉 하나의 값을 리턴하고, Obserable에서 제공하는 일부 기능을 사용할 수 없습니다.

간단한 차이점 하나를 보면 다음과 같습니다.

{% highlight java %}
Single.just("Hello", "World").subscribe(System.out::println);
{% endhighlight %}

만약 위와 같이 하게 되면, 하나의 입력이 들어와서 하나의 응답이 나가야하는데, 입력이 2개가 되어 컴파일 오류가 나게 됩니다.

마찬가지로 fromIterable도 제공되지 않습니다.

{% highlight java %}
List<String> words = Arrays.asList("AA", "BB");
Single.fromIterable(words).subscribe(System.out::println);
{% endhighlight %}

제공되지 않기 때문에 위의 코드도 마찬가지로 컴파일 오류가 납니다.

Single은 다음 기능만을 제공을 합니다.

{% highlight java %}
interface SingleObserver<T> {
    void onSubscribe(Disposable d);
    void onSuccess(T value);
    void onError(Throwable error);
}
{% endhighlight %}
onNext나 onComplete는 onSuccess로 합쳐져서 단 하나의 값만 리턴이 됩니다.

Single의 경우 DBMS의 데이터를 조회한 결과를 한건을 반환하는데 적합합니다.

따라서 다음과 같은 코드 작성이 가능합니다.

{% highlight java %}
public Single<Person> findPerson(long ssn) {
    return Single.fromCallable(() -> personMapper.findPerson(ssn));
}
{% endhighlight %}
personMapper는 db를 조회하여 Person개체를 반환하는 myBatis 코드 샘플입니다. RxJava 1.2 부터는 비동기로 작업을 처리하고 그 결과를 리턴을 편리하게 만들기 위해서

Observable.fromCallable() 이 추가되었습니다. defer를 대체할 수 있습니다.

{% highlight java %}
Single.fromCallable(dao::findPerson)
  .subscribe(person -> {
    // Next Step
  }, throwable -> {
    // Error handling
  });
{% endhighlight %}
fromCallable은 subsribe될때는 아무것도 하지 않으며, 호출(call)이 되는 시점에 onSuccess가 실행되어 personMapper가 호출이 됩니다.

### 참조
*  http://reactivex.io/RxJava/javadoc/io/reactivex/Single.html
