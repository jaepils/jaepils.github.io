---
layout: post
title:  "Kotlin Part#5"
date:   2018-07-13 15:40:56
categories: kotlin
---
## OOP
다른 OOP 언어와 마찬지로, Kotlin에서도 일반적인 모든 기능을 구현이 가능합니다. 이번에는 아래 기능들이 어떻게 구현을 하는지를 살펴보겠습니다.

 - Class
 - Inheritance
 - Polymorphism
 - Abstraction
 - Interface
 - Data

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
메소드는 java에서는 리턴 타입부분이 fun인 것을 제외하고는 java와 유사한거 같습니다.
main에서는 인스턴스를 생성하는데, new 연산자 없이 생성을 하고 있는 부분이 다른거 같습니다.

#### Non-Static Nested Classes
Class내의 중첩 클래스를 생성하는 방법을 보겠습니다. Nested Class는 다른 클래스 내에 정의된 클래스를 의미합니다.
컴파일이 되면 부모 클래스이름에 OuterClass$InnerClass.class로 생성이 됩니다.
java에서는 인스턴스 생성시 아래 코드와 같이 생성이 됩니다.

```java
@Test
public void test() {
    OuterClass outer = new OuterClass();

    OuterClass.Inner b = outer.new InnerClass();
}


public class OuterClass {
    public class InnerClass {
    }
}
```  
Outer 클래스내에 Inner 클래스 정의시 java에서는 outer.new 라는 방식으로 생성이 가능해집니다.

kotlin의 경우에서는 class 정의시 inner를 붙여서 생성을 해야합니다.

```java
fun main(args: Array<String>) {

    val inner =  OuterClass().InnerClass()
}

class OuterClass {
    inner class InnerClass {

    }
}
```
Inner class는 Outer 클래스의 데이터 멤버인것처럼 접근을 해야합니다.
실제 디컴파일된 코드를 보면 다음과 같이 나옵니다.

```java
OuterClass.InnerClass inner = new OuterClass.InnerClass(new OuterClass());
```

#### Static Nested Classes
Nested class를 static으로 정의하게 되면 상위 클래스의 멤버가 아닌 일반적인 클래스 처럼 접근이 가능해집니다. 다만 패키지처럼 상위 클래스를 참조해서 생성해야합니다.

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
Sample 클래스가 static inner class인 Inner 를 가지고 있는 경우 외부에서는 Sample.Inner()로 생성을 해야합니다. Sample의 경우 static import로 정의시 뺄수 있습니다.

kotlin의 경우를 살펴 보겠습니다.

```java
fun main(args: Array<String>) {
    val inner = OuterClass.InnerClass()
}

class OuterClass {
    class InnerClass {

    }
}
```
OuterClass는 생성하지 않고, InnerClass 만 생성을 하고 있습니다.
디컴파일 결과는 다음과 같습니다.

```java
OuterClass.InnerClass inner = new OuterClass.Inner();
```

#### constructors
생성자는 생각외로 다릅니다. 클래스에 name이라는 속성을 가지고 있어서 생성자에서 매핑을 해주는 작업 자체가 필요가 업습니다.

```java
fun main(args: Array<String>) {

    val person = Person("홍길동", 15)

    println("Name = ${person.name}")
}

class Person(val name: String, var age: Int) {

}
```
디컴파일을 하면 아래와 같이 클래스가 생성이 됩니다.
```java
public final class Person
{
  @NotNull
  private final String name;
  private int age;

  public Person(@NotNull String name, int age)
  {
    this.name = name;
    this.age = age;
  }

  public final void setAge(int <set-?>) { age = <set-?>; }
  public final int getAge() { return age; }

  @NotNull
  public final String getName() { return name; }
}
```
디컴파일 결과를 보면 name은 immutable하게 생성이 되었고, age는 mutable 하게 생성이 되어 getter와 setter가 둘다 생성이 되어있습니다.
타입을 val로 하는 경우에만 final이 붙고, setter가 생성이 되지 않습니다.

```java
class Person(var ssn:Int, val name: String, var age: Int) {

    fun test() {
        name = "길동" --> reassign 에러
        age = 1      --> 성공
    }
}
```
클래스 내부에 임의의 함수를 만들어도 상수이기 때문에 name은 다른 변수 할당이 되지 않습니다.

mutable한 String 속성을 위해서는 멤버 변수를 다시 정의를 하는 방법이 있습니다.

```java
fun main(args: Array<String>) {

    val person = Person("홍길동", 15)

    println("Name = ${person.name}")

    person.name = "홍길동1"
    println("Name = ${person.name}")

}

class Person(val _name: String = "", var _age: Int) {
    var name: String = _name
    var age: Int = _age

}
```
이렇게 하게 되면, 멤버변수가 2개가 아닌 4개가 나오게 되어 좋은 방법은 아닌거 같습니다.
클래스가 immutable이어야하는 경우에만 val을 사용을 하면 됩니다.

#### Secondary Constructor
위의 경우를 해결하기 위해서는 다음과 같은 방식으로 Secondary Constructor를 생성을 해야 합니다.

```java
fun main(args: Array<String>) {

    val person = Person("홍길동", 15)

    println("Name = ${person.name}")

    person.name = "홍길동1"
    println("Name = ${person.name}")
}

class Person {
    var name: String
    var age: Int

    // Secondary Constructor
    constructor(name: String, age: Int)  {
        this.name = name
        this.age = age
    }
}

--> Name = 홍길동
--> Name = 홍길동1
```
constructor를 명시해서 생성자를 만들고, java처럼 직접 매핑을 하면 됩니다.

#### Function
함수 정의 방법을 알아 보겠습니다. 함수는 리턴을 마지막에 명시하는 것이 조금 다른거 같습니다.
```java
fun main(args: Array<String>) {

    val value = calculateValues(1)

    println("value = $value")

}

fun calculateValues(input:Int) : Int {
    return input * 2
}
```
calculateValues의 Input으로 Integer 타입인 input을 받아서 Output으로 Integer를 반환을 하고 있습니다.

두개의 output을 받고 싶을때는 다음과 같은 방식도 가능합니다.

```java
fun foo(): Pair<String,Int>{
    return Pair("bar",5)
}
```
두개 일때는 pair, 세개를 리턴하고 싶을때는 Triple을 사용하면 됩니다.

### Inheritance
상속은 개념은 비슷하지만, 코딩 방법이 많이 다릅니다.

```java
open class Computer {
    open fun doWork() {
        print("work")
    }
}

// Derived class (Sub class)
class Laptop: Computer() {

    override fun doWork() {
        print("work on laptop")
    }
}
```
우선 상속이 가능하게 하려면 class 앞에 open이라고 명시를 해야합니다. 이렇게 하지 않으면 디컴파일된 java 코드에서보면 final로 클래스가 생성이 되어 상속이 불가능합니다.

Computer라는 클래스에 명시된 메소드 역시 기본적으로는 private이고 override가 불가능합니다. 자식 클래스에서 override하게 하려면 메소드 역시 open을 명시를 해야 합니다.

### Abstraction
abstract 클래스에 대해 알아보겠습니다. abstract는 당연하지만, 초기화를 하지 못하고, 상속을 할수 있습니다.
java에서는 당연하겠지만, kotlin의 경우 기본적으로 class는 final이기 때문에 open 을 명시하지 않으면 상속이 되지 않습니다. 다만 abstract 클래스의 경우는 open을 명시하지 않아도 상속이 가능합니다.
이외에는 클래스와 모두 동일합니다.

```java
fun main(args: Array<String>) {

    var student = Student("홍길동", 15)

    student.print()
}

abstract class Person {
    open fun print() {
        println("Person")
    }
}


class Student(val name: String, var age: Int) : Person() {
    override fun print() {
        println("Name = $name")
    }
}
```


### Interface
interface는 java와 완전이 동일한거 같습니다. 변수 정의도 가능하고, 인터페이스내에 구현을 해야할 메소드 정의도 가능합니다. 한가지 특이한 것은 interface내에서 구현까지 된 메소드의 경우 java로 전환시 default 메소드로 생성이 되는게 아닙니다.
아래 예를 보면서 설명하겠습니다.
```java
fun main(args: Array<String>) {

    val computer = Computer()

    computer.print()
    computer.printMe()
}

interface printable {

    fun print()

    fun printMe() {
        println("print ? ")
    }
}


open class Computer : printable {
    open fun doWork() {
        print("work")
    }

    override fun print() {
        println("print 1 ")
    }

    override fun printMe() {
        println("print 2 ")
    }
}
```
printable 인터페이스는 print와 printMe 메소드 2개를 정의하고 있고, printMe는 실제 로직을 가지고 있습니다.
이 경우 디컴파일 결과를 보면 다음과 같습니다.

```java
public abstract interface printable
{
  public abstract void print();

  public abstract void printMe();
}

public class Computer implements printable
{
  public void doWork()
  {
    String str = "work";System.out.print(str);
  }

  public void print() {
    String str = "print 1 ";System.out.println(str);
  }

  public void printMe() {
    String str = "print 2 ";System.out.println(str);
  }

  public Computer() {}
}
```
디컴파일된 코드를 보면 printable은 abstract 메소드 2개를 가지고 있습니다. default 메소드는 보이지 않습니다.
Computer 클래스를 보면 메소드 2개를 implements하였고, kotlin에서 인터페이스 정의할때 구현 로직이 Computer 클래스가 가지고 있음을 알 수 있습니다.

### Data
Data class는 생성자로 기본적으로 클래스와 유사하지만, toString, equals등이 이미 정의가 되어있 다른 기능을 제공하지 않는 클래스라고 kotlin에서 정의하고 있습니다.
```java
fun main(args: Array<String>) {

    val person = Person("홍길동", 15)

    println("Name = ${person.name}")

    person.age = 20
    println("age = ${person.age}")

    println("ToString = " + person.toString())
    person.print()
}

data class Person(val name: String, var age: Int) {
    fun print() {
        println("print 1 ")
    }
}

--> Name = 홍길동
--> age = 20
--> ToString = Person(name=홍길동, age=20)
--> print 1
```
우선 Person 클래스는 data로 정의를 하면, 하나 이상의 primary constructor를 정의를 해야합니다.
kotlin에서 class와 마찬가지로 String 은 기본적으로 final이고, Int는 수정이 가능합니다.
내부 함수 역시 동일하게 가질 수 있습니다. 차이점은 abstract, open, inner 클래스가 아니어야 합니다.
기본적으로 equals, toString, copy등을 제공을 하고 있습니다.


## 결론
kotlin도 java와 비슷한 것 같지만, attribute가 기본적으로 immutable인점, 클래스는 final로 생성이 되고, 메소드 역시 private하기 때문에 java에 비해 제한이 많이 하려한거 같습니다.

## 참조
* https://www.tutorialspoint.com/kotlin
* https://blog.plan99.net/kotlin-fp-3bf63a17d64a
