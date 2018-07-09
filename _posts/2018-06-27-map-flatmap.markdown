---
layout: post
title:  "java.time.Instant :: JD"
date:   2018-06-26 15:40:56
categories: java
---
### 정의
java 8에서 추가된 Instant class는 타임라인의 특정 순간을 나타낼때 사용이 됩니다. 즉 어플리케이션에서 이벤트 타임 기록을 남길때 사용이 될 수 있습니다.

우선 이 함수를 사용하기 앞서 epoch라는 용어를 알아야 하는데, 의미는 1st 1970 - 00:00 - Greenwhich mean time (GMT) 이후의 offset을 의미합니다.



Epoch Time
Epoch time을 구하는 간단한 방법은 다음과 같습니다.

{% highlight java %}
System.out.println(System.currentTimeMillis() / 1000);
System.out.println(Instant.now().getEpochSecond());
System.out.println(org.joda.time.Instant.now().getMillis());
{% endhighlight %}
위의 결과는 다음과 같습니다.

물론 수행시간에 따라 결과가 다르기 때문에 같은 결과가 나오진 않습니다.

{% highlight java %}
1529997409
1529997409
1529997409387
{% endhighlight %}

1번과 2번은 결과가 같고 3번은 다르게 나오는데, 이유는 1번과 2번은 초가 기준이고, 3번은 밀리세컨드 기준으로 수행이 되어 약간 틀린 결과가 나옵니다.

Instant.now().toEpochMilli을 수행하면 joda와 같은 결과가 나옵니다. 다만 joda에서는 초단위로는 구할수가 없습니다.

이 클래스를 사용하면 직관적으로 시간을 구할 수 있습니다.


### Instant Calculations
현재 기준으로 시간을 더하거나 빼는 방법은 다음과 같습니다.

{% highlight java %}
Instant now = Instant.now();

Instant later = now.plusSeconds(10);
Instant earlier = now.minusSeconds(10);

System.out.println("now : "+ now.toString());
System.out.println("later : "+ later.toString());
System.out.println("earlier : "+ earlier.toString());
{% endhighlight %}

{% highlight java %}
now : 2018-06-26T07:30:00.029Z
later : 2018-06-26T07:30:10.029Z
earlier : 2018-06-26T07:29:50.029Z
{% endhighlight %}

결과는 코드 처럼 현재 순간에서 10초를 추가되거나, 10초가 줄어든 결과가 나오게 됩니다. 매우 직관적인 코드 작성이 가능해진게 장점으로 보입니다.
기존 방식으로 보면 다음과 같이 작성을 했어야 했습니다.

{% highlight java %}
Calendar calendar = Calendar.getInstance();
calendar.add(Calendar.SECOND, 5);
System.out.println(calendar.getTime());
{% endhighlight %}

위의 코드와 비교를 해보면 직관적인지 않은 불친절한 코드로 보입니다.


### Compare
비교 역시 기존대비 매우 직관적인 방식을 제공하고 있습니다.

{% highlight java %}
Instant current = Instant.now();
Instant afterTenSec = current.plusSeconds(10);

System.out.println("before : " + current.isBefore(afterTenSec));
System.out.println("after : " + current.isAfter(afterTenSec));
{% endhighlight %}

결과는 before는 true이고, after는 false가 나옵니다.

이 클래스는 immutable 하면서 thread safe하기 때문에 쓰레드간 경합 상황에서도 대해서 사용이 가능합니다.
