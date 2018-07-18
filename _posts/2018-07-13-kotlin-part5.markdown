---
layout: post
title:  "Kotlin Part#5"
date:   2018-07-13 15:40:56
categories: kotlin
---
## OOP
다른 OOP 언어와 마찬지로, Kotlin에서도 일반적인 모든 기능을 구현이 가능합니다. 이번에는 아래 기능들이 어떻게 구현을 하는지를 살펴보겠습니다.

 - Class
 - Object
 - Inheritance
 - Polymorphism
 - Abstraction
 - Interface

### Class
Class를 정의하는 것을 보도록 하겠습니다. 기본적인 사항은 Java와 동일합니다.
Java와 마찬가지로 class는 attribute를 가질 수 있고, private와 같이 외부 노출 여부를 정의할 수 있습니다.

```java
fun main(args: Array<String>) {

    val obj = Student()
    obj.printMe()
}

class Student {

    private var id:Int = 1
    private var name: String = "ABC"

    fun printMe() {
        print("My Id is $id and Name is $name")
    }
}

--> My Id is 1 and Name is ABC
```        
우선 Student 클래스를 정의하였습니다. private 속성으로 id와 name을 가지고 있습니다.
메소드는 리턴이 fun인 것을 제외하고는 java와 유사한거 같습니다.
main에서는 인스턴스를 생성하는데, new 연산자 없이 생성을 하고 있는 부분이 다른거 같습니다.

#### Non-Static Nested Classes
Class내의 중첩 클래스를 생성하는 방법을 보겠습니다.
Nested class는 특정 class 내에 정의된 class로 java에서는 컴파일이 되면 부모 클래스이름에 Outer클래스명$Inner클래스명.class로 생성이 되고, 인스턴스 생성시 아래 코드와 같이 생성이 가능합니다.

```java
@Test
public void test() {
    Outer outer = new Outer();

    Outer.Inner b = outer.new Inner();
}


public class Outer {
    public class Inner {
    }
}
```  
Outer 클래스내에 Inner 클래스 정의시 java에서는 outer.new 라는 방식으로 생성이 가능해집니다.

kotlin의 경우를 보겠습니다.




#### Static Nested Classes

```java
public class Sample {

    public static class Inner {
    }

    @Test
    public void test() {
        Inner inner = new Sample.Inner();

    }

}
```


## 결론
Streams는 Java8에 비해 kotlin이 비교 우위에 있는거 같습니다.

## 참조
* https://www.tutorialspoint.com/kotlin
* https://blog.plan99.net/kotlin-fp-3bf63a17d64a
