---
layout: post
title:  "Map과 Flatmap의 차이 #2"
date:   2019-07-17 15:40:56
categories: java
---
## 예제
Flatmap이 가장 빛을 발하는 샘플 코드를 설명드리겠습니다.

조급 복잡한 문자열로 "홍길동:100#이순신:90"라는 문자열을 파싱을 하면서 이순신이 몇점인지를 반환을 하는 코드를 작성을 해보겠습니다.

만약 동일 요건을 과거의 방식으로 푼다면 다음처럼 루프를 돌아야 합니다.

```java
//Java7
Arrays.asList(samples).forEach(firstString -> {
    String[] value = firstString.split("#");
    Arrays.asList(value).forEach(secondString -> {
        String[] record = secondString.split(":");
        if ("이순신".equals(record[0])) {
            System.out.println(record[1]);
        }
    });
});

--> 90
```
우선 '#'으로 문자를 나누게 되면 "홍길동:100"과 "이순신:90"으로 문자가 구성이 됩니다. 이후 ':'으로 나누게 되면 비로서 우리가 찾으려는 값을 찾을 수 있게 됩니다.

```java
//Java8
Arrays.stream(samples)
      .flatMap(v -> Stream.of(v.split("#")))
      .filter(v -> v.indexOf("이순신") == 0)
      .map(v -> Integer.parseInt(v.replaceAll("이순신:", "")))
      .findFirst()
      .ifPresent(System.out::println);

--> 90
```
Java8로 다시 작성을 하면 위의 코드처럼 작성이 가능합니다. flatmap은 다시 리턴을 Stream을 함으로써 이중 for문이 동작을 하듯이 "홍길동:100#이순신:90"을 "홍길동:100"과 "이순신:90"으로 분리가 되어 그 이후 map으로 넘어가게 됩니다. 기존 if문으로 작성된 코드는 filter로 대체가 되어 이순신인경우에만 다음으로 넘어가게 됩니다.

Stream.map의 경우 리턴이 하나만 해야하는 것과는 매우 다릅니다.

### Checked Exception
만약 만약 map이나 filter에서 사용하는 특정 함수가 Checked Exception이 있는 경우 코드가 많이 이상해집니다. Integer.parseInt 부분을 강제로 Exception이 발생하는 메소드로 변경을 해보겠습니다.



```java
//Java8
@Test
public void test() {
    String[] samples = {"홍길동:100#이순신:90"};

    Arrays.stream(samples)
          .flatMap(v -> Stream.of(v.split("#")))
          .filter(v -> v.indexOf("이순신") == 0)
          .map(v -> parse(v.replaceAll("이순신:", ""))) -> Compile Error
          .findFirst()
          .ifPresent(System.out::println);

}

private int parse(String v) throws Exception{
    return Integer.parseInt(v);
}
```
parse 메소드가 명시적으로 Exception을 발생시키고 있어서 map이 있는 라인에서 컴파일 에러가 발생을 합니다. 만약 try문을 사용한다면 다음과 같은 코드가 됩니다.

```java
//Java8
String[] samples = {"홍길동:100#이순신:90"};

Arrays.stream(samples)
      .flatMap(v -> Stream.of(v.split("#")))
      .filter(v -> v.indexOf("이순신") == 0)
      .map(v -> {
          try {
              return parse(v.replaceAll("이순신:", ""));
          } catch (Exception e) {
              e.printStackTrace();
          }
          return 0;
      })
      .findFirst()
      .ifPresent(System.out::println);
```
위와 같은 코드를 개선을 하기 위해서 여러가지 방법이 있지만, 그중 한가지 방법으로 수정을 해보겠습니다.

```java
//Java8
@FunctionalInterface
public interface FunctionWithException<T, R, E extends Exception> {
    R apply(T t) throws E;
}

private <E extends Exception>
Function<String, Integer> hideException(FunctionWithException<String, Integer, E> fe) {
    return arg -> {
        try {
            return fe.apply(arg);
        } catch (Exception e) {
            return 0;
        }
    };
}

@Test
public void test() {
   String[] samples = {"홍길동:100#이순신:90"};

   Arrays.stream(samples)
         .flatMap(v -> Stream.of(v.split("#")))
         .filter(v -> v.indexOf("이순신") == 0)
         .map(hideException(v -> parse(v)))
         .findFirst()
         .ifPresent(System.out::println);

}

```
hideException은 FunctionWithException를 파라미터로 받아서 호출 결과를 리턴하는 Function 클래스를 리턴을 합니다.

hideException(v -> parse(v)) 은 FunctionWithException의 구현체 입니다.
실제 클래스를 풀어서 쓰면 아래 코드 처럼 정의가 됩니다.

```java
hideException(new FunctionWithException<String, Integer, Exception>() {
    @Override
    public Integer apply(String v) throws Exception {
        return parse(v);
    }
})
```


## 결론
하나의 for문은 map이 적당하고, 이중 for문과 같은 경우 flatMap으로 변환이 적합합니다.

## 참조
* https://www.baeldung.com/java-lambda-exceptions
* https://stackoverflow.com/questions/19757300/java-8-lambda-streams-filter-by-method-with-exception
