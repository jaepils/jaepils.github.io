---
layout: post
title:  "TimeWindows in kafka"
date:   2018-08-08 15:40:56
categories: kafka
---
## TimeWindows
KafkaStreams에서는 고정된 시간을 기준으로 입력값의 집합을 처리할 수 있습니다. 즉 특정 토픽에 입력이 발생하면 5초 범위안에 있는 데이터들을 모아서 스트림을 생성할 수 있습니다.
이렇게 데이터를 모으는 방법을 TimeWindows라고 합니다.

이번에 만들 예제는 입력을 받아서 5초간 가장 큰 온도를 찾고 그 데이터를 다른 토픽에 전달을 하는 간단한 예제입니다.

이번 작업 순서는 다음과 같습니다.
1. Topic 생성
2. gradle 설정
3. Topology 생성
4. 테스트

### Topic 생성
이번 예제는 토픽이 2개가 필요합니다. 입력을 받는 토픽과 출력을 하는 토픽이 필요합니다.
```java
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic input
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic temp
```

토픽이 정상적으로 만들어졌는지 보려면 아래 명령어를 치면 됩니다.
```java
./kafka-topics.sh --list --zookeeper localhost:2181
```

### gradle 설정
이전 글에서는 kafka stream 1.1을 사용을 했습니다. TimeWindows를 사용하기 위해서는 kafka stream 2로 변경을 해야합니다.

```java
ext {
    springCloudVersion = 'Finchley.RELEASE'
    kafkaVersion = '2.0.0'
    testJunitVersion = '1.0.3'
    junitVersion = '5.1.0'
    junitPlatformVersion = '1.1.0'
}
...
```

### Topology 생성
Topology는 Stream이 흘러가는 흐름을 의미합니다.

```java
@Component
public class StreamInitializingBean implements InitializingBean, DisposableBean {

    // threshold used for filtering max temperature values
    private static final int TEMPERATURE_THRESHOLD = 20;
    // window size within which the filtering is applied
    private static final int TEMPERATURE_WINDOW_SIZE = 5;

    protected KafkaStreams kafkaStreams;

    @Override
    public void afterPropertiesSet() {
        StreamsBuilder streamsBuilder = new StreamsBuilder();

        final KStream<String, String> source = streamsBuilder.stream("input", Consumed.with(Serdes.String(), Serdes.String()));

        final KStream<Windowed<String>, String> max = source
                .selectKey((key, value) -> "temp")
                .groupByKey(Serialized.with(Serdes.String(), Serdes.String()))
                .windowedBy(TimeWindows.of(TimeUnit.SECONDS.toMillis(TEMPERATURE_WINDOW_SIZE)))
                .reduce((value1, value2) -> {
                    if (Integer.parseInt(value1) > Integer.parseInt(value2))
                        return value1;
                    else
                        return value2;
                })
                .toStream()
                .filter((key, value) -> Integer.parseInt(value) > TEMPERATURE_THRESHOLD);

        final Serde<Windowed<String>> windowedSerde = WindowedSerdes.timeWindowedSerdeFrom(String.class);

        // need to override key serde to Windowed<String> type
        max.to("temp", Produced.with(windowedSerde, Serdes.String()));


        Topology topology = streamsBuilder.build();
        this.kafkaStreams = new KafkaStreams(topology, getStreamConfig());
        this.kafkaStreams.setStateListener((newState, oldState) -> {
            System.out.printf("newState :" + newState);
            System.out.printf("oldState :" + oldState);
        });

        kafkaStreams.start();
    }

    @Override
    public void destroy() {
        this.kafkaStreams.close();
    }

    private Properties getStreamConfig() {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "sample");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");

        return props;
    }
}
```
이번 토폴로지는 input 토픽을 KStream를 생성하고, 입력이 들어오면 키를 같게 한 후 그룹으로 묶어서 5초간의 데이터를 모은후 가장 큰 값을 찾게 됩니다.

스텝별로 나누어서 설명하겠습니다.

```java
final KStream<String, String> source = streamsBuilder.stream("input", Consumed.with(Serdes.String(), Serdes.String()));
```
input 토픽으로 KStream을 생성합니다. consume할때는 키와 값 모두 String으로 deserialize을 합니다.

```java
final KStream<Windowed<String>, String> max = source
                .selectKey((key, value) -> "temp")
                .groupByKey(Serialized.with(Serdes.String(), Serdes.String()))
                .windowedBy(TimeWindows.of(TimeUnit.SECONDS.toMillis(TEMPERATURE_WINDOW_SIZE)))
                .reduce((value1, value2) -> {
                    if (Integer.parseInt(value1) > Integer.parseInt(value2))
                        return value1;
                    else
                        return value2;
                })
                .toStream()
                .filter((key, value) -> Integer.parseInt(value) > TEMPERATURE_THRESHOLD);
```    
selectKey는 KeyValueMapper 인터페이스로 키와 값을 넘겨주고, 새로운 값을 넘겨줍니다. 여기서는 어떤 값이 오더라도 무조건 temp를 반환해서 이후 groupBy에서 키로 사용이 되게 합니다. 즉 토픽의 모든 데이터의 키를 temp로 바꾸게 됩니다. 여기서 groupBy에서 다시 Serialized를 정의하는 것은 데이터가 다시 kafka로 저장이 되기 때문에 문자를 byte[]로 바꿀 방법을 알려줘야 합니다.

windowedBy에서 TimeWindows가 사용이 됩니다.TimeWindows.of의 리턴값은 5초간의 데이터를 aggregations하기 위한 TimeWindows입니다.

reduce는 5초간의 데이터의 크기를 비교하는 Reducer의 구현체이고, 크기가 큰 값을 반환합니다. reduce의 리턴값은 KTable입니다. KTable을 다시 toStream를 사용하여 KStream로 변환하고, 특정 값 20 이하는 거르기 위한 필터를 추가를 합니다.

```java
final Serde<Windowed<String>> windowedSerde = WindowedSerdes.timeWindowedSerdeFrom(String.class);
max.to("temp", Produced.with(windowedSerde, Serdes.String()));
```
5초간의 데이터중 가장 큰 값이면서 20보다 큰 값이 발생하면 temp 토픽으로 보내지게 됩니다.

## 테스트
Kafka에 데이터를 전송하는 가장 간단한 방법은 console producer를 사용하는 것입니다.
```java
./kafka-console-producer.sh --broker-list localhost:9092 --topic input
>21
>22
>23
>24
```
임의의 숫자 값을 여러개 보내겠습니다. 보낸 값중에 가장 큰 값은 24입니다.

```java
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic temp --from-beginning
21
24
```
결과를 보면 21이 혼자 TimeWindows로 동작을 하고, 22, 23, 24가 그 이후 동작을 해서 2개가 나온 것을 알 수 있습니다. 이 결과는 입력 속도에 따라 다르게 나옵니다.

## 참조
* https://github.com/apache/kafka/blob/trunk/streams/examples/src/main/java/org/apache/kafka/streams/examples/temperature/TemperatureDemo.java
