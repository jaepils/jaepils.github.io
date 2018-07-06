---
layout: post
title:  "RxJava2"
date:   2018-07-06 15:40:56
categories: java rxjava2
---
## RxJava
> RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences.

Reactive Extensions (ReactiveX이라고 불리기도함)이 명령형 프로그래밍 언어가 데이터가 동기식인지 비동기식인지에 관계없이 데이터 시퀀스에서 작동 할 수있게 해주는 도구 모음을 의미합니다.
RxJava는 Reactive Extensions을 구현체로 이벤트의 변화를 처리를 비동기적 처리를 지원하는 라이브러리입니다. 비동기적인 프로그래밍이란 함수를 호출하고, 완료가 되면 그 결과를 콜백에서 받아 처리하는 방식입니다. ReactiveX는 Observer pattern, Iterator pattern, functional programming이 조합되어 구현이 되어 있습니다.

### Dependency
```java
dependencies {
	compile "io.reactivex.rxjava2:rxjava:2.1.3"
	compile "org.apache.commons:commons-lang3:3.7"
}
```

### the Basics
* Observable은 Observer 패턴 구현체.
* Observable은 아이템들을 발행(emit)한다. (방출로 번역하기도함) 
* Subscriber는 이 아이템들을 소비한다.

ReactiveX는 Observable을 subscribe를 시작함으로써 동작을 합니다. 즉 subscriber없이는 아무런 동작을 하지 않습니다. Observer는 Observable이 발행하는 아이템을 순서대로 처리를 하게 됩니다.
Observable이 가지고 있는 각각의 Subscriber마다 Observable은 Subscriber.onNext()을 몇 번이나 호출하다가 완료가 되면 Subscriber.onComplete() 혹은 에러가 발생하면 Subscriber.onError()이 호출이 됩니다.


### Hello world
```java
Observable.create((ObservableOnSubscribe<String>)
	  subscriber -> {	    
        subscriber.onNext("Hello world");	})
   .subscribe(System.out::println);
```
위의 코드는 Observable을 하나 만들고, Observer가 subscribe 하면 Hello world를 발행하는 예제입니다. 여기서 Observer는 아래 클래스를 람다로 축약을 해서 정의를 하였습니다.
.subscribe를 하는 시점에 Observable이 방출을 시작을 하게 됩니다.

```java
Subscriber<String> simpleSubscriber = new Subscriber<String>() {
  @Override
    public void onNext(String s) { System.out.println(s); }
  @Override
    public void onCompleted() { }
  @Override
    public void onError(Throwable e) { }
};
```

### Subscribe

```java
Observable.create((ObservableOnSubscribe<String>)
    subscriber -> {    
        subscriber.onNext("1");    
        subscriber.onNext("2");    
        subscriber.onComplete();})
    .subscribe(System.out::println);
```
위의 코드는 1과 2과 순서대로 방출이 되고 Observable이 subscriber에게 이제 끝났다고 알려주고 전체 플로우가 끝이 나게 됩니다. Subscriber에서는 onComplete에서 아무런 행동도 취하지 않았기 때문에 콘솔에 1과 2가 찍히게 됩니다.

```java
Observable.create((ObservableOnSubscribe<String>)
          subscriber -> {
              subscriber.onNext("1");
              subscriber.onNext("2");
              subscriber.onError(new Throwable("3"));})
          .onErrorResumeNext(throwable -> {
              return Observable.fromArray("4", "5","6");})
          .subscribe(System.out::println, error -> log.error(error.getMessage()));
```
이제 에러가 나는 케이스를 보도록 하겠습니다. 위의 코드는 1과 2를 정상적으로 방출을 했지만, 3의 경우 에러를 강제로 만들었습니다. Observable은 에러가 나는 경우 4, 5, 6을 다시 순서대로 방출하도록 하였습니다. 여기서 subscriber는 onNext이외에도 onError의 경우 에러 로그를 남기도록 했습니다.

결과는 1, 2, 4, 5, 6이 찍힐뿐 에러를 찍지 않습니다. onErrorResumeNext을 제거를 하면 1, 2 하고 나서 에러 메세지가 나오게 됩니다.


```java
Observable.create((ObservableOnSubscribe<String>)
        subscriber -> {
            subscriber.onNext("1");
            subscriber.onNext("2");
            subscriber.onComplete();})
        .subscribe(
                System.out::println, //consumer
                error -> log.error(error.getMessage()),  //error
                () -> 	System.out.println("Completed")); //action
```
위의 코드는 subscriber가 onNext가 되면 출력을 하고, 에러 나오면 에러 로그, 완료되면 Completed를 찍도록 했습니다. 결과는 당연하게도 1, 2, Completed가 나오게 됩니다.

## Observable
### Observable.just
```java
Observable.just("Hello", "World").subscribe(System.out::println);

Observable.create((ObservableOnSubscribe<String>)
        subscriber -> {        
          subscriber.onNext("Hello");
          subscriber.onNext("World");
          subscriber.onComplete();})
        .subscribe(System.out::println);
```

### Observable.array

```java
List<String> words = Arrays.asList(        
      "the",        
      "quick",        
      "brown",        
      "fox",        
      "jumped",        
      "over",        
      "the",        
      "lazy",        
      "dog");

Observable.just(words).subscribe(System.out::println);
```

### Observable.fromIterable
```java
List<String> words = Arrays.asList(        
      "the",        
      "quick",        
      "brown",        
      "fox",        
      "jumped",        
      "over",        
      "the",        
      "lazy",        
      "dog");

Observable.fromIterable(words).subscribe(System.out::println);
```

### Observable.range
```java
Observable.range(1, 5).subscribe(System.out::println);
```

### Observable.map
```java
Function<String, String> getCount = ball -> ball + " 개";
String[] balls = {"1", "2", "3", "4", "5"};
Observable<String> source = Observable.fromArray(balls).map(getCount);
source.subscribe(System.out::println);
```

### Observable.flatMap
```java
List<String> sentences = new ArrayList<>();
sentences.add("A B");
sentences.add("C D E F");

Observable.fromIterable(sentences).blockingSubscribe(System.out::println);

Observable.fromIterable(sentences)
          .flatMap(s -> Observable.fromArray(s.split(" ")))      
          .blockingSubscribe(System.out::println);
```

### Observable.sort
```java
List<String> sentences = new ArrayList<>();
sentences.add("A C");
sentences.add("D B F E");

Flowable.fromIterable(sentences)
        .flatMap(s -> Flowable.fromArray(s.split(" ")))
        .sorted()
        .blockingSubscribe(System.out::println);
```

### Observable.zip
```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Observable.zip(
      Observable.fromIterable(letters),
      Observable.fromIterable(numbers),  
      (string, index) -> index + "-" + string
).subscribe(System.out::println);
```

### Observable.zipWith
```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Observable<String> observable = Observable.fromIterable(letters)
                        .zipWith(Observable.fromIterable(numbers),
                             (string, index) -> index + "-" + string);

observable.subscribe(System.out::println);
```

### Observable.error
```java
Observable.just(1, 2, 3, 4, 5)
          .subscribe(integer -> {
              if (integer == 3) {
                throw new RuntimeException();
              }
              log.info("integer : " + integer);
          }, error -> log.error(error.getMessage()));
```


## 참조
* https://en.wikipedia.org/wiki/Reactive_programming
* http://reactivex.io/
* https://github.com/ReactiveX/RxJava/wiki
