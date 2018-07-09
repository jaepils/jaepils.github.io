---
layout: post
title:  "리스코프 치환 원칙 :: JD"
date:   2018-07-04 15:40:56
categories: pattern
---
SOLID 객체지향원칙중에 하나인 리스코프 치환 원칙에 대해 알아보도록 하겠습니다.
바바라 리스코프라는 사람의 이름으로 패턴을 만들어서 쉽게 이해가 가지 않습니다.


### 정의
위키 백과에서 설명을 보면 다음과 같습니다.

> 컴퓨터 프로그램에서 자료형 S가 자료형 T의 하위형이라면 필요한 프로그램의 속성(정확성, 수행하는 업무 등)의 변경 없이 자료형 T의 객체를 자료형 S의 객체로 교체(치환)할 수 있어야 한다는 원칙이다.

영문으로 보면 다음과 같습니다.
> in a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.)

쉽게 이해가 가지는 않지만 특정 클래스를 사용하는 클래스에서는 사용하려는 클래스를 모르더라도 그 클래스의 상위 클래스 개체로 치환해서 사용가능해야한다고 말을 하는 거 같습니다. 즉 하위 클래스에 대한 속성을 모르게 해야한다는걸 설명하려는거 같습니다.

### 잘못된 예제
설명이 쉽지 않으니 예제 코드를 보도록 하겠습니다. 예제는 사실 현실적이지 않지만, 대표적인 케이스로 설명하고 있는 넓이를 구하는 샘플을 보도록 하겠습니다.

{% highlight java %}
@Test
public void test () {
    Rectangle rectangle = new Rectangle();
    Rectangle square = new Square();

    assertThat(getArea(rectangle)).isEqualTo(200);
    assertThat(getArea(square)).isEqualTo(100);  //--> 에러 400이 나옴
}

private int getArea(Rectangle rectangle) {
    rectangle.setHeight(10);
    rectangle.setWidth(20);

    return rectangle.getArea();
}

@Data
class Rectangle {
    protected int width;
    protected int height;

    public int getArea() {
        return width * height;
    }
}

@Data
class Square extends Rectangle {
    @Override
    public int getArea() {
        return width * width;
    }
}
{% endhighlight %}

사각형을 부모 클래스로 정의하고, 정사각형을 하위 클래스로 정의를 하였습니다.
이를 사용하는 곳에서 높이와 길이를 주고 넓이를 계산하는 간단한 코드입니다.
여기서 리스코프 치환 원칙을 위배한 부분은 직사각형을 정사각형의 하위 클래스로 정의를 한 부분입니다.
정사각형의 경우 길이이든 높이이든 하나의 속성만 있으면 되는데 높이와 길이를 세팅하게 되어 불필요한 로직이 수행이 되었고, 결과적으로 잘못된 값으로 인해 오류가 발생하였습니다.
정의된 Rectange을 사용하는 클래스의 경우 이 하위의 속성이 어떤것이 있든지 동일한 처리를 해야하는데, setWidth와 setHeight을 함으로써 하위 클래스에서 올바른 결과가 나오지 못하게 된 것입니다.

따라서 위의 코드는 상속관계를 재정의가 필요합니다.

### 수정된 코드
{% highlight java %}
@Test
public void test () {
    Shape rectangle = new Rectangle(10, 20);
    Shape square = new Square(10);

    assertThat(getArea(rectangle)).isEqualTo(200);
    assertThat(getArea(square)).isEqualTo(100);
}

private int getArea(Shape shape) {
    return shape.getArea();
}

abstract class Shape {
    public abstract int getArea();
}

@Data
class Rectangle extends Shape {
    protected int width;
    protected int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

@Data
class Square extends Shape {
    protected int width;

    public Square(int width) {
        this.width = width;
    }

    @Override
    public int getArea() {
        return width * width;
    }
}
{% endhighlight %}

### 결론
상위 클래스는 하위 클래스에서 공통적으로 가지고 있거나 추상화해야하는 기능만을 가지고 있고 하위 클래스는 자기만의 독작적인 성격을 가지고 있는게 맞는거 같습니다.

### 참조
* https://ko.wikipedia.org/wiki/%EB%A6%AC%EC%8A%A4%EC%BD%94%ED%94%84_%EC%B9%98%ED%99%98_%EC%9B%90%EC%B9%99
* https://dzone.com/articles/the-liskov-substitution-principle-with-examples
