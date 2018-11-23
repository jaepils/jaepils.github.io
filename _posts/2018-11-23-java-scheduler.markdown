---
layout: post
title:  "Spring Scheduler"
date:   2018-11-20 15:40:56
categories: java, spring
---
## Scheduler
주기적으로 반복된 일을 수행할때 스케쥴러를 많이 사용을 합니다. 간단하고, 정확하기 때문인데요. 사용하면서 매번 궁금했던 내용이 있었습니다.
만약 1초마다 수행이 되도록 설정을 했는데, 이 task가 1분이 걸린다면 어떻게 될까? 였습니다.
저는 1초마다 수행이 스케쥴링의 중요하기 때문에 task의 수행시간이 상관없이 호출이 된다고 가정을 했고, 이전 task의 내용이 다른 thread에서 중복처리 되지 않도록 설계를 하면서 복잡하게 코딩을 했었습니다.

이번에 스케쥴링을 다시 작성을 하면서 이부분에 대해 명확히 알고 싶었습니다.

### TaskScheduler
우선 스케쥴링을 수행하는 thread를 제가 임의로 만든 threadPool에서 수행이 되도록 하였습니다.

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();

        threadPoolTaskScheduler.setPoolSize(10);
        threadPoolTaskScheduler.setThreadNamePrefix("scheduled-task-pool-");
        threadPoolTaskScheduler.initialize();

        scheduledTaskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}
```
threadPool을 만들고 사이즈를 10으로 설정을 하였습니다. 만약 해당 작업이 제 생각대로 동작을 한다면 10개 이내에서 병렬로 처리가 되고, 10개가 넘는다면 thread가 얻을 될때까지 대기를 할것으로 생각됩니다.

### Task
```java
@Scheduled(fixedRate = 1000)
public void run() {
    System.out.println("Fixed Rate Task :: execution Time - "+ dateTimeFormatter.format(LocalDateTime.now()));


    try {
        TimeUnit.MINUTES.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
run이라는 메소드를 Schedule을 설정했고, fixedRate을 1초로 설정하였습니다.
정상적인 경우 1초마다 수행이 되어야 합니다. 초기 가정으로 해당 task가 1초를 넘어서 어떤일을 수행해야하기 때문에 sleep 설정을 하였습니다.

### 실행 결과
```java
Fixed Rate Task :: execution Time - 10:16:52
Fixed Rate Task :: execution Time - 10:17:52
Fixed Rate Task :: execution Time - 10:18:52
```
결과는 thread가 틀림에도 1분단위로 실행이 됩니다.

## 결론
Spring document에서는 다음과 같이 적혀있습니다.
>If a fixed rate execution is desired, simply change the property name specified within the annotation. The following would be executed every 5 seconds measured between the successive start times of each invocation.

각 호출의 연속 시작 시간 사이에 측정 된 5 초마다 실행된다라고만 되어있습니다. thread간의 동작에 관해서는 아무런 언급이 없습니다.

제가 좋아하는 baeldung 블로그에서 보니 다음과 같이 적혀있었습니다.
> Note that the beginning of the task execution doesn’t wait for the completion of the previous execution.

태스크 실행의 시작은 이전 실행의 완료를 기다리지 않는다고 되어있습니다. 테스트 결과가 다르네요.

마지막으로 stackoverflow에서는 어느 친절한 분이 자세히 적어 놓은게 있었습니다.

>This correct and it is the intended behaviour. Each scheduled task, irrespective of fixedRate or fixedDelay, will never run in parallel. This is true even if the invocation takes longer than the configured fixedRate.

이것은 정확하고 의도 된 행동입니다. fixedRate 또는 fixedDelay 상관없이 절대 병렬로 실행이 되지 않습니다.
설정된 fixedRate보다 더 오래 걸리더라도 마찬가지입니다.

**task가 오래걸리게 되면 fixedRate가 원하는 주기로 실행이 되지 않는게 결론입니다.**

## 참조
* https://www.baeldung.com/spring-scheduled-tasks
* https://stackoverflow.com/questions/49671257/spring-scheduled-fixedrate-not-working-properly
* https://docs.spring.io/spring/docs/4.1.5.RELEASE/spring-framework-reference/html/scheduling.html
