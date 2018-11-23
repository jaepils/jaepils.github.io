---
layout: post
title:  "Redis SCAN"
date:   2018-11-20 15:40:56
categories: redis
---
## 정의
Redis를 사용하다보면 key를 검색을 하고 싶을때가 있습니다. 기본적으로 제공하는 keys 명령어는 redis의 기본 싱글 쓰레드를 멈추고 해당 커맨드를 수행하기 때문에 사용을 금지하다시피 하고 있습니다.
이 글에서는 keys 명령어를 대신하여 SCAN을 사용하여 원하는 작업을 수행해보도록 하겠습니다.


### SCAN
SCAN은 커서 기반의 데이터를 탐색 기능을 제공 합니다. 이 의미는 페이징으로 생각하면 쉬운데, 한번 호출을 하면 다음 커서를 반환을 조회할때 인수로 사용할 커서를 반환합니다.

우선 Redis의 데이터를 임의로 넣고, keys 명령어로 보도록 하겠습니다.

(1 A) (2 B)(3 C)(4 D)(5 E)(6 F)(7 G)(8 H) 를 넣고 keys * 을 하면 아래와 같이 조회가 됩니다.

<img width="253" alt="1-1" src="https://user-images.githubusercontent.com/23305428/48819760-f23fa480-ed94-11e8-8394-d92ae7287b74.png">

순서는 생각과 달리 꼬여있지만, 8개 모두 나오는 것을 알 수 있습니다.

Scan의 기본 사용법은 scan 0입니다. 결과를 keys와 비교를 해보겠습니다.

<img width="201" alt="1-2" src="https://user-images.githubusercontent.com/23305428/48819999-29fb1c00-ed96-11e8-89e1-e4939ba9ae6b.png">

keys와는 다르게 첫번째줄의 0은 다음 데이터가 없다는 의미입니다. 두번째부터 다시 8개의 데이터가 있고, 그 순서는 keys와는 다른것을 알 수있습니다. 데이터 탐색방식이 다른것으로 보입니다.

만약 scan 0 count 5로 하면 다음과 같은 결과가 나옵니다.

<img width="286" alt="1-3" src="https://user-images.githubusercontent.com/23305428/48820556-ca524000-ed98-11e8-8148-27308f23aac1.png">

count의 의미는 보여줄 데이터의 사이즈입니다. 따라서 scan 0 count 5로 하면 처음부터 조회해서 5개의 데이터를 반환하게 됩니다.
그 결과로 커서를 1로 반환을 하고, 데이터는 4, 6, 2, 5, 7을 반환을 하는데, 그 다음 데이터를 조회하기 위해서는 반환받은 커서번호를 사용을 해서 scan 1 count 5를 호출해야합니다.

반환되는 커서는 일정 규칙을 보이지 않으며 0을 반환하게 되면 더이상 데이터가 없음을 의미합니다.

기본적인 count의 값은 10입니다.

SCAN은 여러 클라이언트가 동시에 호출을 하더라도, 상태를 가지고 있지 않기 때문에 모든 클라이언트가 다 개별 결과를 받게 됩니다.

### Spring data redis
Redis를 사용하여 데이터를 조회하기 위해서 가장 편한 방법인 Spring을 사용해보겠습니다.
아래처럼 spring-boot-starter-data-redis를 추가하면 간단하게 redis를 사용할 수 있습니다.

```java
compile('org.springframework.boot:spring-boot-starter-data-redis')
```
Java에서 SCAN을 사용하는 방법은 다음과 같습니다.

```java
RedisConnection redisConnection = redisProductTemplate.getConnectionFactory().getConnection();
ScanOptions options = ScanOptions.scanOptions().match("*").count(5).build();

Cursor<byte[]> c = redisConnection.scan(options);
while (c.hasNext()) {
    log.info(new String(c.next()));
}
```
ScanOptions 사용하면 SCAN이 옵션으로 받을 수 있는 패턴과 count를 넘겨줄 수 있습니다.
특정 패턴이 없이 전체 데이터를 조회하기 위해서 * 를 넘겨주고, count 5로 해서 실행하면 결과에서는 마치 redis-cli의 결과처럼 커서와 5개의 데이터만 넘겨줄것 같지만, 실제로는 모든 데이터가 넘어오게 됩니다.

이유는 Spring에서 redis의 scan을 리턴값을 보고 루프를 돌아 모든 데이터를 반환하도록 구현을 했기 때문입니다.

<img width="880" alt="1-4" src="https://user-images.githubusercontent.com/23305428/48821668-91689a00-ed9d-11e8-9091-a51ac7503f9e.png">

를 보면 nextCursorId에 1을 받고, 이를 ScanIteration 리턴하고 있습니다.



## 결론
같은 항목이 여러번 나올 가능성이 있고, 중간에 추가된 데이터의 경우 누락이 발생할 수 있지만, keys대비 여러가지 강점을 가지는 거 같습니다.

그리고 Scan은 Redis Cluster 환경에서 동작하지 않습니다.
> Scan is not supported across multiple nodes within a cluster.

## 참조
* https://redis.io/commands/scan
* http://tech.kakao.com/2016/03/11/redis-scan/
