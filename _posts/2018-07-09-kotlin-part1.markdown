---
layout: post
title:  "Kotlin Part#1"
date:   2018-07-08 15:40:56
categories: kotlin
---
## Kotlin이란?
Kotlin은 JVM, Android, 브라우저를 위한 정적 타입의 프로그래밍 언어입니다. 정적 타입의 프로그래밍 언어란 런타임이 아닌 컴파일 타입에 타입에 대한 체크를 수행하는 것을 의미합니다.
대표적으로 Java, C등이 해당을 합니다. 정적으로 타입이 지정되므로 Java와 같은 동일한 타입 안정성을 갖습니다.
Kotlin compiler은 byte code로 컴파일하기 때문에 jvm에서 동작이 가능합니다. 또한  Java script 코드로도 컴파일이 됩니다. 메인 개발은 JetBrain의 팀에서 이루어졌습니다. 그래서 그런지 IntelliJ에 기본이 포함이 되어있습니다.
Kotlin은 Java, Scala, Groovy등에 영향을 받았습니다. 하지만, Java와 조금 틀리지만, 기존 Java library를 사용할 수 있습니다.

## History
* In July 2011 JetBrains unveiled Project Kotlin, a new language for the JVM, which had been under development for a year.
* Kotlin v1.0 was released on February 15, 2016.
* At Google I/O 2017, Google Announced First-class Support For Kotlin On Android.
* Kotlin v1.2 was released on November 28, 2017

## Advantages
* Easy Language :  Kotlin은 Functional language이고, 배우기가 매우 쉽습니다. 문법은 Java와 유사해서 기억하기가 쉽습니다. Kotlin은 표현력(expressive)이 좋아서 조금 더 이해하기 쉽고, 읽기 편하게 작성이 가능합니다.
* Runtime and Performance : 성능이 좋습니다.
* Null 안정성 : Optional과 유사한 기능으로, ?만 있으면 null여부를 체크하지 않아도 됩니다. 뒤에서 코드로 보도록 하겠습니다.

## Environment Setup
IntelliJ기준으로 설명을 하겠습니다. 우선 엔터프라이즈 버전에서는 kotlin plugin이 이미 설치가 되어있어 설치과정은 빼도록 하겠습니다.
* New Project
새 프로젝트에서 Kotlin을 선택을 하면 JVM or JS등 환경을 선택하는 화면이 나옵니다. 우선 여기서는 JVM을 선택하고 다음을 누릅니다.
<img width="600" alt="kotlin-ex1-step-1" src="https://user-images.githubusercontent.com/23305428/42615524-9077debe-85e5-11e8-8da8-f9607b7e8a80.png">

* Project location
프로젝트명 및 소스 코드 위치는 임의로 지정을 하면 됩니다.
<img width="600" alt="kotlin-ex1-step-2" src="https://user-images.githubusercontent.com/23305428/42615523-904a7794-85e5-11e8-8e8d-047e3701f51b.png">

* Hello world
모든 프로그래밍은 Hello world부터 시작을 해야겠죠?
New File에서 kotlin file을 선택한 후 helloworld를 입력을 하면 kotlin editor가 화면에 나옵니다.
여기에 다음과 같이 코드를 넣으면 됩니다.
## helloworld.kt
```java
fun main(args: Array<String>) {
    println("Hello World!")
}
```
실행시켜보면 결과는 콘솔에 Hello World!가 나오게 됩니다.
<img width="400" alt="kotlin-ex1-step-3" src="https://user-images.githubusercontent.com/23305428/42615522-90216264-85e5-11e8-8193-0ff80c32d304.png">


## Decompilation
Kotlin compiler에 의해 컴파일이 되면 byte code가 생성이 된다고 이전에 설명을 했습니다. 따라서 실제 생성된 코드는 java와 동일한 .class 파일이 생성이 됩니다.
이코드의 디컴파일한 결과를 보면 다음과 같습니다.
```java
public final class HelloworldKt {
  public static final void main(@NotNull String[] args) {
    Intrinsics.checkParameterIsNotNull(args, "args");
    String str = "Hello World!";
    System.out.println(str);
  }
}
```
기본적인 자바 코드로 생성한 것과 거의 유사한 코드임을 알 수 있습니다.

다음에는 타입과 기본 문법에 대해 알아보도록 하겠습니다.

## 참조
* https://www.tutorialspoint.com/kotlin
