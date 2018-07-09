---
layout: post
title:  "RxJava2"
date:   2018-07-06 15:40:56
categories: java rxJava2
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
just는 가장 간단하게 Observable을 생성하는 방법입니다. 입력받은 값을 다시 방출하는 Observable이 리턴합니다.
just는 null을 입력을 받으면 null을 방출하기 때문에 주의가 필요합니다.
```java
Observable.just("Hello", "World").subscribe(System.out::println);

Observable.create((ObservableOnSubscribe<String>)
        subscriber -> {        
          subscriber.onNext("Hello");
          subscriber.onNext("World");
          subscriber.onComplete();})
        .subscribe(System.out::println);
```
just는 한줄로 정의가 가능하지만, 실제 내부에서는 onNext가 2번 호출이 되고, onComplete가 호출이 되면서 종료가 됩니다.
### Observable.array
List를 받아서 List의 element들을 순서대로 방출하는 Observable을 생성을 합니다.
fromArray는 array를 파라미터로 받기 때문에 아래 코드는 생각대로 동작 하지 않습니다.
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
            "dog");

Observable.fromArray(words).subscribe(System.out::println);
```
위의 코드 수행결과는 [the, quick, brown, fox, jumped, over, the, lazy, dog]가 나오게 됩니다.
만약 위의 코드처럼 list를 넘겨주고 싶다면 fromIterable을 사용해야 합니다. fromIterable은 Iterable개체를 순서대로 방출을 하는 Observable을 반환을 합니다.

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
            "dog");

Observable.fromIterable(words).subscribe(System.out::println);
```
위의 코드는 list의 element가 순서대로 방출이 되어 콘솔에 찍히게 됩니다.

### Observable.range
특정 범위의 숫자값을 순서대로 방출하는 Observable을 반환을 합니다.
```java
Observable.range(1, 5).subscribe(System.out::println);
```

### Observable.map
map은 하나의 입력을 다른(같거나) 타입으로 변환을 하는 기능입니다.
Observable.map의 파라미터는 io.reactivex.functions.Function입니다.
Optional에서 사용하는 Function과는 패키지가 다르지만, 동작은 같습니다.
```java
Function<String, String> getCount = ball -> ball + " 개";
String[] balls = {"1", "2", "3", "4", "5"};
Observable<String> source = Observable.fromArray(balls).map(getCount);
source.subscribe(System.out::println);
```
위의 코드는 balls의 순서대로 방출하는 Observable을 fromArray로 생성을 하고, 방출된 결과는 map으로 보내서 Function이, 입력으로 보내고 결과는 "개"라는 글자가 붙어서 반환이 됩니다.
따라서 결과는 다음과 같습니다.
```java
1 개
2 개
3 개
4 개
5 개
```
### Observable.flatMap
flatMap은 map과 다르게 다른 타입을 반환하는게 아니라 같은 방출되는 타입이 다른 Observable 개체를 반환을 합니다.
따라서 방출된 결과는 Observable이기 때문에 다시 subscribe을 해야합니다.

```java
List<String> sentences = new ArrayList<>();
sentences.add("A B");
sentences.add("C D E F");

Observable.fromIterable(sentences)
  .flatMap(s -> Observable.fromArray(s.split(" ")))
  .blockingSubscribe(System.out::println);
```
위의 코드는 fromIterable을 하게 되면 "A B"와 "C D E F" 2개의 결과를 순서대로 Observable을 flatMap의 파라미터로 넘겨주고 Observable.fromArray가 "A B"를 "A", "B"로 다시 방출되는 Observable개체를 반환하고 그 결과를 출력을 하게 됩니다.
이 코드에서는 subscribe를 한번만 했고, 두개의 Observable이 모두 수행이 되는 것을 알 수 있습니다.

### Observable.sort
sort는 결과를 List를 받아서 정렬 한 결과를 다시 Observable에 담아 반환하여 정렬된 값이 방출됩니다.
```java
List<String> sentences = new ArrayList<>();
sentences.add("A C");
sentences.add("D B F E");

Observable.fromIterable(sentences)
    .flatMap(s -> Observable.fromArray(s.split(" ")))
    .sorted()
    .blockingSubscribe(System.out::println);
```

### Observable.zip
두개 이상의 Observable를 수행하여 그 결과를 combine한 결과를 반환하는 Observable을 다시 반환하는 기능입니다.
zip은 코드만 보면 마치 병렬로 처리 되는것으로 생각하기 쉬운데 기본적으로는 모두 같은 쓰레드에서 동작합니다.
```java
List<String> letters = Arrays.asList("A", "B", "C", "D", "E");
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Observable.zip(
    Observable.fromIterable(letters),
    Observable.fromIterable(numbers),
    (string, index) -> index + "-" + string
).subscribe(System.out::println);
```
Observable은 2개가 있고, 결과는 서로 합쳐서 하나의 문자를 반환합니다.
결과는 아래와 같습니다.
```java
1-A
2-B
3-C
4-D
5-E
```
만약 letters가 "A", "B" 2개가 있다면 결과가 어떻게 나올까요?
이런 경우 1-A, 2-B만 나오게 됩니다.

반대로 numbers가 1, 2만 있고, letters는 A부터 E까지 있는 경우는 결과가 어떻게 나올까요?
마찬가지로 1-A, 2-B만 나오게 됩니다.

### Observable.zipWith
위와 동일한 결과가 나옵니다.
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
