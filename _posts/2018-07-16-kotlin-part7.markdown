---
layout: post
title:  "Kotlin Part#7"
date:   2018-07-16 15:40:56
categories: kotlin
---
## Lambda Function
kotlin은 java처럼 lambda function을 제공합니다.

### Function
Java8에 추가된 Function은 input을 하나 받아서 출력으로 하나를 반환합니다.
코드를 작성하기 위해서는 실제 함수와는 다르게 인라인 클래스 처럼 작성을 해야합니다.

```java
Function<String, String> mylambda = s -> {
    System.out.println("Hello," + s);
    return s;
};

String value ="test";

mylambda.apply(value);
```
apply를 호출해야 인라인의 람다가 실행이 됩니다.

이와 다르게 kotlin의 경우 함수 자체를 변수로 만들수가 있습니다.

```java
val mylambda = { s: String ->
   println("Hello, $s!")
}
mylambda("test")
```

mylambda라는 함수 자체를 사용하기 위해서는 실제 함수처럼 사용을 하면 됩니다.
비슷하지만, apply를 안쓴다는 점이 틀립니다.
위의 kotlin코드는 .class 파일이 2개가 생성이 됩니다.

HelloworldKt$main$mylambda$1.class 이렇게 람다에 관련된 함수가 클래스로 생성이 되고, 실제 호출하는 클래스는 다음과 같습니다.

```java
Function1 mylambda = (Function1)HelloworldKt.main.mylambda.1.INSTANCE;
mylambda.invoke("test");
```
mylambda 인스턴스를 받아서 invoke로 호출을 합니다.

이런 방식으로 클래스이기 때문에 다른 함수에 다음처럼 파라미터 전달도 가능합니다. 함수의 파라미터로 인스턴스가 넘어가게 됩니다.

```java
fun main(args: Array<String>) {
    val mylambda = { s: String ->
        println("Hello, $s!")
    }

    val v:String="test"

    myFun(v,mylambda)

}

fun myFun(a :String, action: (String)->Unit) {
    action(a)
}
```
위의 코드는 mylambda라는 function을 myFun이라는 함수의 파라미터로 전달해서 다시 mylambda가 동작을 하게됩니다.
즉 main -> myFun -> mylambda가 실행이 됩니다.

```java
public final class HelloworldKt {
  public static final void main(@org.jetbrains.annotations.NotNull String[] args) {
    Function1 mylambda = (Function1)HelloworldKt.main.mylambda.1.INSTANCE;

    String v = "test";

    myFun(v, mylambda);
  }

  public static final void myFun(String a, Function1<? super String, kotlin.Unit> action)
  {
    action.invoke(a);
  }
}
```
디컴파일 코드를 보면 mylambda라는 Function1 인스턴스를 myFun 함수의 파라미터로 전달을 하고, Unitinvoke를 호출해서 인스턴스내에서 정의된 로직인 print가 동작을 하게 됩니다.

### return
lambda funtion을 {} 없이 한줄로 표현을 하기 위해서는 다음 방식으로 작성을 해야합니다.
```java
val mylambda1 = { s: String ->
    println("$s!")
}

val mylambda2 :(String)->Unit ={s:String->print(s)}
```
mylambda1과 mylambda2는 완전히 동일합니다. 다만 Unit이라는 단어가 쓰임을 알수 있습니다.
Unit은 void라고 생각을 하면 될거 같습니다. 만약 lambda function이 리턴값을 가진다면 다음처럼 코드를 작성하면 됩니다.

```java
fun main(args: Array<String>) {

    val fun1: (Int,Int)->Int = { a,b -> Math.max(a,b) }
    var result = fun1(1, 2)
    println(result)

}
```
fun1이라고 된 lambda function은 숫자 a, b를 받아서 Math.max(a, b)의 결과를 리턴을 합니다.

## 결론
java8과 유사하지만, 조금 더 자유로운게 kotlin의 장점인거 같습니다.

## 참조
* https://marcin-chwedczuk.github.io/lambda-expressions-in-kotlin
