---
layout: post
title:  "Introduction to KafkaStreams"
date:   2018-08-08 15:40:56
categories: kafka
---
## KafkaStreams
KafkaStreams은 Apache Kafka에서 만든 데이트 흐름을 정의할 수 있는 라이브러리 입니다.
KafkaStreams은 kafka 토픽을 consume해서, 분석 혹은 데이터 변환을 하여 다른 토픽에 전달 할 수 있습니다.
KafkaStreams은 KStream과 KTable로 크게 나눌 수 있는데, KStream은 데이터가 흘러가는 Java에서 Stream과 비슷한 역할이고, KTable은 change log의 기준으로 변경된 데이터를 반영한 스토리지로 이해하시면 편할거 같습니다. Ktable은 다시 KStream으로 변환이 가능합니다. 변경하게 되면 Ktable의 변경된 데이터가 다시 Stream으로 전달이 됩니다.

데모로 사용자가 입력한 글자를 받아서 단어별로 나누고 단어가 몇번 반복이 되는지를 보여주는 샘플을 작성하겠습니다. 사용자가 입력한 전체 글자는 토픽으로 전달이 되고, 토픽에 데이터가 들어오면 전체 문장을 단어로 쪼개고 단어를 키로 해서 반복 횟수를 저장을 합니다. 저장된 데이터는 다시 콘솔에 몇번 반복이 되는지를 출력을 하게 됩니다.

이번 작업 순서는 다음과 같습니다.
1. Kafka 실행
2. Topic 생성
3. gradle 설정
4. Topology 생성
5. 테스트


### Kafka 실행
kafka 설치는 https://kafka.apache.org/ 에서 다운로드 받아서 압축만 풀면 됩니다. Java 어플리케이션이기 때문에 Java는 사전에 설치되어있어야 합니다.

실행순서는 다음과 같습니다.
1. zookeeper 실행
2. broker 실행

bin 에서 다음 명령어를 치면 됩니다.

zookeeper 실행방법은 아래와 같습니다.
```java
./zookeeper-server-start.sh ../config/zookeeper.properties
```

broker 실행방법은 아래와 같습니다.
```java
./kafka-server-start.sh ../config/server.properties
```

### Topic 생성
Topic은 옵션이 많습니다. 지금 구현을 해볼 데모는 클러스터 구성이 필요없기 때문에 replication을 1로 설정해서 데이터 중복 저장을 하지 않습니다. replication을 2로 하려면 broker가 2개 있어야 합니다. 파티션은 kafka에서 가장 중요한 개념으로 하나의 토픽이 여러 파티션을 가질 수 있습니다. 각 파티션은 키에 의해서 데이터가 나누어지게 되고, 각 파티션단위로 순서를 보장하게 됩니다. 하나의 쓰레드에서 여러 파티션을 처리할 수도 있는데, 이 경우 한 파티션 처리후 다른 파티션을 처리합니다.

여기서는 샘플이기 때문에 파티션을 1로 만들겠습니다. 마지막으로 input은 토픽명입니다.

```java
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic input
```

토픽이 정상적으로 만들어졌는지 보려면 아래 명령어를 치면 됩니다.
```java
./kafka-topics.sh --list --zookeeper localhost:2181
```

### gradle 설정
Spring boot 어플리케이션을 https://start.spring.io/ 에서 생성을 하면 됩니다.
Dependencies는 아무것도 추가하지 않아도 됩니다.

```java
ext {
    springCloudVersion = 'Finchley.RELEASE'
    kafkaVersion = '1.1.0'
    testJunitVersion = '1.0.3'
    junitVersion = '5.1.0'
    junitPlatformVersion = '1.1.0'
}

buildscript {
    ext {
        springBootVersion = '2.0.3.RELEASE'
    }
    repositories {
        mavenCentral()
        maven { url "https://plugins.gradle.org/m2/" }
        maven { url 'http://repo.spring.io/plugins-release' }
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group 'sample'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/milestone" }
    maven { url 'http://packages.confluent.io/maven/' }
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
    compile 'org.springframework.kafka:spring-kafka'

    compile "org.apache.kafka:kafka-streams:$kafkaVersion"
    compile "org.apache.kafka:kafka-clients:$kafkaVersion"

    testCompile group: 'junit', name: 'junit', version: '4.12'
    testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```
현재 기준으로 최신인 Spring boot2와 kafka 1.1.0을 추가를 했습니다.

### Topology 생성
Topology는 Stream이 흘러가는 흐름을 의미합니다.

```java
@Component
public class StreamInitializingBean implements InitializingBean, DisposableBean {

    protected KafkaStreams kafkaStreams;

    @Override
    public void afterPropertiesSet() throws Exception {
        StreamsBuilder streamsBuilder = new StreamsBuilder();
        KStream<String, String> textLines = streamsBuilder
                                                .stream("input", Consumed.with(Serdes.String(), Serdes.String()));

        Pattern pattern = Pattern.compile("\\W+", Pattern.UNICODE_CHARACTER_CLASS);
        KTable<String, Long> wordCounts = textLines
                .flatMapValues(value -> Arrays.asList(pattern.split(value.toLowerCase())))
                .groupBy((key, word) -> word, Serialized.with(Serdes.String(), Serdes.String()))
                .count();

        wordCounts.toStream()
                .foreach((w, c) -> System.out.println("word: " + w + " -> " + c));

        Topology topology = streamsBuilder.build();
        this.kafkaStreams = new KafkaStreams(topology, getStreamConfig());

        kafkaStreams.start();
    }

    @Override
    public void destroy() throws Exception {
        this.kafkaStreams.close();
    }

    private StreamsConfig getStreamConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "sample");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        return new StreamsConfig(props);
    }
}
```
kafkaStreams를 만들기 위해서는 Topology와 kafka서버 정보가 필요합니다.
우선 input 토픽에 데이터가 들어오게 되면 Streams에서 데이터를 받아야합니다. kafka 데이터는 Key와 value로 구성이 되는데, key는 널일 수가 있습니다. kafka의 데이터는 기본적으로 byte[]로 관리를 하는데, 데이터가 들어오면 String으로 변환하기 위해서는 Serde를 추가를 해야합니다.

아래 코드는 textLines가 input토픽에 데이터가 들어오면 Serdes.String()으로 deserialize를 수행하는 KStream을 생성하게 됩니다.

```java
KStream<String, String> textLines = streamsBuilder
                                        .stream("input", Consumed.with(Serdes.String(), Serdes.String()));
```

문장을 단어로 쪼개는 것은 Pattern을 이용하고, 단어를 기준으로 그룹을 만듭니다.
마지막으로 모든 단어의 값을 집계하고 특정 단어의 수를 계산하는 count()를 호출합니다.
```java
KTable<String, Long> wordCounts = textLines
                .flatMapValues(value -> Arrays.asList(pattern.split(value.toLowerCase())))
                .groupBy((key, word) -> word, Serialized.with(Serdes.String(), Serdes.String()))
                .count();
```    
여기서 groupBy에서 다시 Serialized를 정의하는 것은 데이터가 다시 kafka로 저장이 되기 때문에 문자를 byte[]로 바꿀 방법을 알려줘야 합니다.

```java
wordCounts.toStream()
        .foreach((w, c) -> System.out.println("word: " + w + " -> " + c));
```
저장된 KTable의 데이터를 KStream으로 변환하게 되면 KTable의 변경된 데이터가 foreach에 넘어오게 됩니다. 여기서 화면에 글자의 반복 회수를 콘솔에 출력을 합니다.

## 테스트
Kafka에 데이터를 전송하는 가장 간단한 방법은 console producer를 사용하는 것입니다.
```java
./kafka-console-producer.sh --broker-list localhost:9092 --topic input
>"this is a pony"
>"this is a horse and pony"
```
console producer에서 위의 2문장을 입력하면 아래와 같은 결과가 찍히게 됩니다.

```java
word:  -> 1
word: this -> 1
word: is -> 1
word: a -> 1
word: pony -> 1
word:  -> 2
word: this -> 2
word: is -> 2
word: a -> 2
word: horse -> 1
word: and -> 1
word: pony -> 2
```

## 참조
* https://www.baeldung.com/java-kafka-streams
