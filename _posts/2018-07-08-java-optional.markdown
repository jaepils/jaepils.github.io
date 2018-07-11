---
layout: post
title:  "java.util.Optional"
date:   2018-07-07 15:40:56
categories: java
---
## 정의
자바 개발을 하면서 수많은 NPE에러를 보았고, 이를 막기 위해 아직도 많은 코드에서 if null 체크 로직이 많이 사용이 되고 있습니다. 구글에서 Optional을 설명하는 articles대부분이 이런 문장으로 시작을 하고 있습니다.
> If you’re a Java programmer, then you must have heard about or experienced NullPointerExceptions in your programs.

그만큼 많은 개발자들이 NPE 에러로 인해서 고통을 겪었기에 이런 문구가 많다고 생각합니다.
저는 개인적으로 Optional을 좋아하고, 자주 사용합니다. 하지만 Optional이 NPE를 해결해줄수 있을지는 모르겠지만, 코드 자체에 불필요한 코드 자체를 다 없애준다고 생각하지는 않습니다. 클래스의 속성이 Optional인 것과 아닌 것이 섞이거나, 중첩된 Optional등은 아직 개선점이 있었으면 하고 생각을 합니다.

그럼에도 기존보다 좋기 때문에 java8에서 추가된 Optional이 어떻게 기존의 코드를 개선하는지 보도록 하겠습니다. 설명 순서는 Optional의 클래스 속성과 중요한 메소드등을 먼저 보고, 실제 사용예제를 보도록 하겠습니다.


## Class
```java
public final class Optional<T> {
    private static final Optional<?> EMPTY = new Optional<>();

    private final T value;

    private Optional() {
        this.value = null;
    }

	public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

	public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }

	public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
...
}		
```

Optional클래스는 많은 메소드를 가지고 있고, static으로 정의된 기능도 많습니다.
우선 Optional은 내부적으로 값을 하나를 가지고 있고, 이 값을 있거나 혹은 없더라도, 오류가 나지 않게 하는 역할을 하게 됩니다.

static 메소드중 of 혹은 ofNullable은 Optional개체를 생성하는 방법입니다. of의 경우는 반드시 null이 아닌 개체를 담고자 할때 사용이 되고, ofNullable은 널일수 있는 개체를 담을때 사용을 합니다.
만약 null 개체를 of로 담으려고 하면 오류가 발생하게 됩니다.

우선 간단한 문자를 담는 Optional 예제를 보도록 하겠습니다.

### Optional.of
```java
Optional stringOptional = Optional.of("test");
System.out.println(stringOptional);
```
이 상태에서 콘솔에는 해쉬코드가 아닌 Optional[test]이 찍힙니다.

만약 null개체를 of에 담은 결과를 보도록 하겠습니다.
```java
Optional stringOptional = Optional.of(null);
System.out.println(stringOptional);

-> java.lang.NullPointerException
	at java.util.Objects.requireNonNull(Objects.java:203)
```
우리가 피하려고 했던 NPE가 나오는 것을 알 수 있습니다.

### Optional.ofNullable
이런 경우 ofNullable을 사용하면 다음과 같은 결과를 얻을 수 있습니다.
```java
Optional stringOptional = Optional.ofNullable(null);
System.out.println(stringOptional);

 -> Optional.empty
```
Optional이 내용이 없는 경우 empty라고 나오게 됩니다.

### Optional.get
Optional이 내부적으로 가지고 있는 값을 꺼내는 방법은 다음과 같습니다.
```java
Optional stringOptional1 = Optional.of("test");
Optional stringOptional2 = Optional.ofNullable(null);

System.out.println(stringOptional1.get());
System.out.println(stringOptional2.get());

-> test

java.util.NoSuchElementException: No value present
```
test를 담고 있는 Optional의 경우 get을 호출하면 정상적으로 test를 반환하지만, null을 가진 emtpy인 경우에는 get을 호출하게 되면 NoSuchElementException 에러가 나게 됩니다.

Optional 개체는 이외에도 map과, flatMap을 제공을 합니다.
이전 다른 글에서도 설명했듯이 map은 입력을 받아서 다시 출력으로 다른 타입 혹은 값으로 반환을 할때 사용이 되고, flatMap은 입력을 다시 Optional 개체로 반환을 해야합니다. 둘 간의 차이를 보도록 하겠습니다.

### Optional.map
map의 파라미터는 function입니다. 이는 입력과 출력을 반환을 유도하는 apply 메소드만을 가진 FunctionalInterface 개체입니다. Function의 주석은 다음과 같습니다.

> Represents a function that accepts one argument and produces a result.

```java
Optional stringOptional1 = Optional.of("test");
System.out.println(stringOptional1.map(v -> v + "2").get());

-> test2
```
따라서 test가 function의 입력이 되었고, 출력으로 문자 "2"를 붙여서 반환이 됩니다.

### Optional.flatMap
map과 flatMap은 다릅니다. 따라서 다음처럼 코드를 작성하면 오류가 발생을 합니다.

```java
Optional stringOptional1 = Optional.of("test");

System.out.println(stringOptional1.flatMap(v -> new Opv + "2").get());

-> java.lang.ClassCastException: java.lang.String cannot be cast to java.util.Optional
```
재미나게도 컴파일 오류가 아닌 런타입 오류가 발생을 합니다.
만약 map과 동일 결과를 원한다면 다음과 같이 코드를 작성을 해야합니다.
```java
Optional stringOptional1 = Optional.of("test");
System.out.println(stringOptional1.flatMap(v -> Optional.of(v + "2")).get());

-> test2				
```
단이 이런 이유만으로 map 대신에 flatMap을 사용하지는 않을거 같습니다. 다만 flatMap은 다음과 같이 복잡한 예제에서 매우 큰 힘을 발휘를 합니다.

```java
public class Computer {
  private Optional<Soundcard> soundcard;  
  ...
}

public class Soundcard {
	private String name;
}

Optional.of(computer).flatMap(Computer::getSoundcard).map(Soundcard::name)

```
Computer의 속성을 soundcard를 가지고 있지만, Optional인 경우 두번 get을 해야하지만,
flatMap을 사용하게 되면 리턴이 Optional이기 때문에 Optional<Soundcard>가 반환이 되고, 여기서는 Optional개체가 아닌 실제 값이 필요함으로 map을 사용하여 내부에 값을 꺼내게 되었습니다.

## Optional.orElse
값이 없는 경우는 피하기 어렵기 때문에 해당 값이 없는 경우 기본값으로 사용하는 방법을 알아보겠습니다.

```java
public T orElse(T other) {
    return value != null ? value : other;
}
```
orElse의 경우 내부에서 null 체크를 하고 null인 경우 other를 리턴하는 간단한 메소드입니다.
orElse의 리턴은 optional이 아닌 실제 값이기 때문에 get과 같은 역할을 수행합니다.
```java
Optional stringOptional1 = Optional.of("test");
System.out.println(stringOptional1.orElse("test2"));
```
위의 코드처럼 orElse를 사용을 하면 되는데, 따로 get을 안해도 됩니다. 위 코드 결과는 당연히 test가 됩니다. test 문자가 null이 아니기 때문입니다.

## Optional.orElseGet
orElse와 유사하지만, Supplier를 사용하여 조금 더 복잡한 동작을 한 결과를 리턴하는 orElseGet을 알아보도록 하겠습니다.
```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```
Optional이 가진 값이 null 인경우 Supplier의 get을 반환을 합니다.
예제를 보도록 하겠습니다.
```java
Optional stringOptional1 = Optional.ofNullable(null);
System.out.println(stringOptional1.orElseGet(() -> {
    StringBuilder sb = new StringBuilder();
    sb.append("test2");

    return sb.toString();
}));
```
물론 test2를 반환을 해도 되지만, StringBuilder를 사용해서 문자를 생성해서 반환을 했습니다.
primitive type의 경우 그냥 반환을 해도 되지만, Object를 반환하는 경우 초기값등의 세팅이 필요한 경우 사용을 하면될거 같습니다.

## Optional.orElseThrow
값이 없는 경우 에러를 발생하는 예제입니다.

```java
Optional stringOptional1 = Optional.ofNullable(null);
stringOptional1.orElseThrow(() -> new RuntimeException());
stringOptional1.orElseThrow(RuntimeException::new);
```

## Optional.isPresent
기존 null 체크 로직은 대부분 다음처럼 작성을 합니다.
```java
@Test
public String test () {

    Computer computer = new Computer();

    if(computer.getSoundcard() != null) {
        return computer.getSoundcard().getName();
    }
    return "NONE";

}

@Data
public class Computer {
    private Soundcard soundcard;
}

@Data
public class Soundcard {
    private String name;
}
```
위의 코드는 설명을 위한 코드이기는 하지만, NPE 없이 정상적으로 동작을 합니다. 이미 null을 체크를 했기 때문입니다.  이 코드를 Optional로 전환을 하면 다음처럼 할 수 있습니다.

```java
@Test
public String test () throws Throwable {

    Computer computer = new Computer();

    if(computer.getSoundcard().isPresent()) {
        return computer.getSoundcard().get().getName();
    }
    return "NONE";

}

@Data
public class Computer {
    private Optional<Soundcard> soundcard;
}

@Data
public class Soundcard {
    private String name;
}
```
Soundcard를 Optional로 전환을 하였고, 제공해주는 null 체크를 하는 isPresent를 사용하여 null이 아닌 경우에만 get을 호출하였습니다.
물론 이렇게 코딩하는 방식은 java8에서 추구하려는 방향과 맞지 않습니다. 따라서 다음처럼 작성을 할 수 있을거 같습니다.

```java
Computer computer = new Computer();

Optional.of(computer).flatMap(Computer::getSoundcard)
                     .map(Soundcard::getName)
                     .orElse("NONE");
```


## 결론
java9에서는 ifPresentOrElse 와 같이 조금 더 편리한 방법도 추가가 되었습니다.
isPresent가 필요한 경우에는 잘못된 코드를 작성하고 있을 수 있으니 다른 대안을 찾아보는게 좋을거 같습니다.

## 참조
* http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html
* https://www.javadevjournal.com/java/java-8-optional/
