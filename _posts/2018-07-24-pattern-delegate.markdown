---
layout: post
title:  "Business delegate pattern"
date:   2018-07-24 15:40:56
categories: pattern
---
## 정의
Business delegate pattern은 Java EE design pattern의 하나로 특정 케이스마다 처리해야하는 로직이 틀린 경우에 사용이 되는 패턴입니다.
구글에서 검색해서 나오는 예제들을 보면 대부분 EJB를 설명을 하고 있습니다. EJB의 경우 lookup이 일반적으로 사용이 됩니다. 특정 일을 수행할 서비스를 찾고 싶으면 EJB server가 제공해주는 lookup에게 물어보면 EJB server가 가지고 있던 정보를 찾아서  담당 서비스를 리턴을 해주어서, 이 서비스에게 일을 시키면 됩니다. 하지만, 굳이 EJB server가 아니더라도, 이 패턴은 매우 유용하게 사용이 됩니다.

### Structure
* Business delegate - client가 특정 서비스를 요청할 클래스 (비즈니스 로직 안가지고 있음)
* Lookup Service - delegate가 호출하는 클래스로 특정 케이스마다 처리를 담당하는 실제 구현체를 반환
* Business Service - 실제 사용될 서비스 인터페이스로 실제 비즈니스 로직은 이 클래스를 구현을 해야함

### Implementation

#### Business Service
```java
public interface DrawService {
   public void draw();
}
```
예제로 설명할 서비스는 draw를 구현하면 됩니다.

#### Concrete Classes
DrawService를 구현한 클래스는 2개를 정의합니다. 사각형과 원을 담당하는 클래스 두개를 정의합니다.
당연하지만 두 클래스 모두 DrawService를 implements를 해야합니다.
```java
public class RectangleDrawService implements DrawService {

   @Override
   public void draw() {
      System.out.println("Drawing rectangle");
   }
}

public class CircleDrawService implements DrawService {

   @Override
   public void draw() {
      System.out.println("Drawing circle");
   }
}
```

#### Lookup Service
실제 두개의 서비스 중 하나를 리턴하는 클래스입니다.
```java
public class DrawServiceLookUp {
   public DrawService getDrawService(String drawType){

      if(drawType.equalsIgnoreCase("CIRCLE")){
         return new CircleDrawService();
      }
      else {
         return new RectangleDrawService();
      }
   }
}
```

#### Business delegate
client는 이 클래스에게 모든 것을 위임을 합니다. 이 클래스는 lookup을 해서 리턴된 서비스에게 일을 전달을 합니다.
```java
public class DrawDelegate {
   private DrawServiceLookUp lookupService = new DrawServiceLookUp();

   public void draw(String serviceType){
      DrawService drawService = lookupService.getBusinessService(serviceType);
      businessService.draw();		
   }
}
```
#### client
```java
public class Demo {
   public static void main(String[] args) {
     DrawDelegate drawDelegate = new DrawDelegate();
     drawDelegate.draw("CIRCLE");
   }
}
```

## 결론
EJB가 유행?하던 시기에서는 많이 쓰이던 패턴이지만, Spring이 유행하는 지금은 많이 사용이 되지 않고 있습니다.

## 참조
* https://en.wikipedia.org/wiki/Business_delegate_pattern
* https://www.tutorialspoint.com/design_pattern/business_delegate_pattern.htm
