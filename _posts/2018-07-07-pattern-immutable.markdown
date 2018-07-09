---
layout: post
title:  "Immutable Design Pattern"
date:   2018-07-07 15:40:56
categories: pattern
---
## 정의
Immutable class는 수정할 수 없는 클래스를 의미합니다. 반대 개념으로 수정할 수 있는 클래스는 mutable 이라고 합니다. 각 인스턴스에 포함 된 모든 속성등은 객체가 생성 될 때 제공되며 객체의 수명 동안 변경이 되지 않습니다. 따라서 클래스를 생성할때는 수정 가능하게 할지 아니면 수정불가능하게 할지 고려해서 설계를 해야 합니다.

Immutable Pattern은 동일한 객체에 대한 참조를 공유하는 객체의 견고성이 증가하고 객체에 대한 동시 액세스의 오버 헤드를 줄일 수 있습니다. 예를 들어 특정 클래스에 파라미터로 새로 생성된 클래스를 전달하게 되면 reference로 전달되어 내부적으로 수정이 될 여지가 있습니다. 이 것을 막기 위해서 방어적 코드로 복사를 하는 경우가 있지만, Immutable로 생성되면 이런 걱정을 할 필요가 없게 됩니다.

이 클래스는 생성 후 동시에 몇개의 쓰레드가 동기화(Synchronized)없이 동시에 액세스 할 수 있습니다.

## Immutable Pattern in Java
* java.lang에 있는 클래스 String, Integer, Boolean등은 immutable입니다.
* enum class는 immutable입니다.
* java.math.BigInteger, Bigdecimal은 immutable입니다.
* Java8에 추가된 Date and Time, Optionals and Streams 등은 immutable입니다.

## How to Create an Immutable Object?
* 인스턴스의 상태를 변경할 수 있는 setter를 생성하면 안됩니다.
* 클래스를 final로 선언하여 서브 클래스가 생성됨을 막아야 합니다.
* 모든 속성은 private 와 final로 정의해야 합니다.

## Sample
```java
public final class ImmutableDate {
	private final long date;

	public ImmutableDate (long date) {
		this.date = date;
	}
	public final long getDate(){
		return date;	// We don't have to clone this either.
	}
}
```

## 결론
Immutable Pattern은 여러 쓰레드간의 경합에서 락을 걸어야 할 필요성을 제거를 해주 패턴입니다.

## 참조
* http://lkumarjain.blogspot.com/2016/02/immutable-design-pattern.html
* https://www.javalobby.org/articles/immutable/index.jsp
* https://dzone.com/articles/the-importance-of-immutability-in-java
