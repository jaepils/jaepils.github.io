---
layout: post
title:  "Sidecar Pattern"
date:   2018-09-28 15:40:56
categories: pattern
---
## 정의
Sidecar는 오토바이 옆에 한사람을 더 태울 수 있는 조수석을 의미합니다.
![sidecar](https://user-images.githubusercontent.com/23305428/46193806-59fbe500-c33a-11e8-9277-3891988f37f6.jpg)

소프트웨어 패턴에서 Sidecar pattern은 두가지이상의 독립된 기능을 묶어 하나처럼 동작을 하게 하는 패턴을 의미합니다. 또한 다른 기술 즉 Java와 Node처럼 기술과 lifecycle이 다른 컨테이너를 묶어서 하나의 어플리케이션으로 구성을 할 수 있습니다.

### 장점
기존 어플리케이션과 분리된 별도의 기능을 다른 컨테이너에서 개발을 할 수 있다는 것입니다. 컨테이너가 다를 수 있기 때문에 하나의 기능을 가진 응용 프로그램처럼 동작을 하지만, 개벌 구성 요소를 별도로 배포 할 수 있는 패턴입니다.

### 단점
만약 별도의 분리된 컨테이너에 들어갈 기능이 너무 작다면 굳이 분리를 해야할지 의문이 들 것입니다. 서로 다른 기술과 서로 다른 배포 방법이 부담이 될 수 있습니다.
또한 하나의 어플리케이션이지만, 한 컨테이너의 부하가 너무 커서 스케일을 늘리고 싶을때는 하나의 어플리케이션이 아닌 별도 서비스로 분리를 하고 싶을 수 있습니다.

## Usecase
Spring cloud sidecar를 보도록 하겠습니다. Spring cloud는 여러 서비스를 하나로 묶어서 동작을 하게 합니다. zuul이라고 불리는 Gateway에서는 외부 http요청을 라우팅을 하는 역할을 하는 입구 역할을 수행하고, Eureka는 비즈니스 로직을 수행할 어플리케이션 서버들을 관리하는 역할을 합니다. 어플리케이션 서버가 기동이 되면 Eureka에 등록을 하고, 동작 가능한 서버들 목록을 GW가 조회하여 실제 서비스 요청이 오면 해당 서버로 라우팅합니다. 여기서 한가지 제약은 Spring cloud는 Spring framework기반으로 작성된 어플리케이션만이 동작이 가능합니다. 따라서 Non-JVM으로 어플리케이션을 만들고 싶을때 spring cloud를 사용하면 구현하지 않아도 되는 기능들을 다 구현하지 않으면 spring cloud와 엮어서 동작을 할 수 없습니다.  Spring cloud sidecar를 이용하면 Non-JVM Application에서도 Eureka를 이용할 수 있습니다.

![sidecar-implementation-using-spring-cloud-netflix-postgres-and-docker](https://user-images.githubusercontent.com/23305428/46193805-59fbe500-c33a-11e8-8c22-13604414dc7b.png)

위의 그림에서 가운데에 있는 non-JVM microservices를 보면 네모는 서버를 의미하고, sidecar는 java로 작성된 Spring cloud Application입니다. non-JVM은 node라고 가정을 하면 하나의 서버에 노드와 jvm이 같이 떠 있는 상태입니다. 서로 접근은 localhost로 하게 됩니다.

처음 설명한대로 비즈니스 로직을 담은 어플리케이션 서버가 기동을 하게 되면 Eureka에 등록을 하는 과정을 거쳐야만 Client에서 접근이 가능해지는데, non-JVM의 경우 등록을 할 방법이 없는 상태입니다.
이때 sidecar의 역할을 하는 어플리케이션이 기동하게 되면 기동시 non-JVM의 동작여부를 체크를 한 후 Eureka에 본인이 아닌 non-JVM 어플리케이션을 등록을 합니다.
이렇게 하는 이유는 Client가 http 요청이 오게 되면 Gateway에서 그 요청을 포워드를 하게 되믄데, sidecar가 받으면 다시 non-JVM에 전달을 해야하는 불필요한 과정이 있게 됩니다. 따라서 Eureka에 non-JVM어플리케이션을 등록 (실제 서버와 포트만 등록함)을 하면 gw가 node로 바로 전달이 됩니다.
만약 node가 죽게 되면 sidecar는 주기적으로 ping을 보내다가 실패시 Eureka에 이 서버를 빼라는 동작ㅇ르 하게 됩니다. 그러면 gw에서는 이 서버가 죽었다고 판단을 하고 이 서버로 요청을 포워드 하지 않게 됩니다.

![non-jvm](https://user-images.githubusercontent.com/23305428/46193874-94658200-c33a-11e8-823d-573f4233a249.png)

위의 그림에서 다시 요약해서 설명을 하면 Eureka (Service Discovery)에 sidecar가 port 3000으로 등록을 하게 됩니다. node는 독립적인 어플리케이션이기 때문에 sidecar의 존재를 모릅니다. 단지 sidecar가 기동이 되면서 node의 상태를 조회해서 Eureka에 등록을 하는 과정과 내리는 과정이 동작을 하게 됩니다.

이렇게 함으로써 node는 실제 어플리케이션이 동작을 해야하는 비즈니스 로직만을 담을 수 있게 됩니다.


## 결론
Sidecar Pattern는 많은 곳에서 효과적으로 사용되고 있으며, 기능의 분리가 가장 큰 장점입니다.

## 참조
* https://blog.davemdavis.net/2018/03/13/the-sidecar-pattern/
* https://www.voxxed.com/2015/01/use-container-sidecar-microservices/
* https://tech.asimio.net/2018/02/20/Microservices-Sidecar-pattern-implementation-using-Postgres-Spring-Cloud-Netflix-and-Docker.html
