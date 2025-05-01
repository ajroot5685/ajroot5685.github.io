---
title: TaskScheduler로 배치 작업 개선하기(1/3) - 주요 개념 정리
date: 2024-11-25 23:44:00 +09:00
categories: [spring]
tags:
  [
    spring,
    batch,
    taskScheduler
  ]
---

## 들어가며

[2. 시나리오 설계 및 가정](https://ajroot5685.github.io/posts/TaskScheduler2/)

[3. TaskScheduler 도입](https://ajroot5685.github.io/posts/TaskScheduler3/)

특정 시간에 알림을 발송하는 작업을 위해 `@Scheduled`가 아닌 `TaskScheduler`를 사용하여 단일 인스턴스의 자원 사용량을 줄여보았다.
<br><br>
다음과 같은 순서로 나눠서 포스팅을 하려 한다.

1. `@Scheduled`와 `TaskScheduler` 개념 및 주요 기능
    > `TaskScheduler`가 더 복잡하고 많은 기능을 가지고 있기에 중점으로 다룬다.
2. `@Scheduled`사용을 지양하게 된 시나리오 및 `TaskScheduler`를 사용해야 했던 이유
3. `TaskScheduler`를 도입하면서 고려되었던 요소들

<br>

## Spring Batch?

당연히 별도의 배치 서버를 구성하여 배치 작업을 수행하는 것이 가장 효율적이고, 안전한 방법이다.
<br>
그러나 비용 등의 내부 사정으로 고려하지 않는다.

<br>

## @Scheduled 개념

- 주기적인 작업이 필요한 경우 간편하게 활용 가능하다.
- 날짜,시간 기반의 `cron 표현식`이나 간격 설정을 통해 주기적으로 특정 메서드를 실행할 수 있다.

#### [예시]

```java
@Component
public class ScheduledTask {

    // 5초마다 실행
    @Scheduled(fixedRate = 5000)
    public void fixedRateTask() {
        System.out.println("FixedRate 작업: " + LocalDateTime.now());
    }

    // 이전 작업 종료 후 10초 후 실행
    @Scheduled(fixedDelay = 10000)
    public void fixedDelayTask() {
        System.out.println("FixedDelate 작업: " + LocalDateTime.now());
    }

    // 초 분 시 일 월 요일
    // 매일 오전 9시에 실행
    @Scheduled(cron = "0 0 9 * * *")
    public void cronTask() {
        System.out.println("Cron Task executed at: " + LocalDateTime.now());
    }
}
```

## TaskScheduler 개념

- 스케줄링을 관리하기 위한 추상화된 인터페이스다.
    > 기본 구현체 : `ThreadPoolTaskScheduler`
- 스케줄링 작업 시간이 동적일 때 유용하다.
- 병렬 처리 지원하여, 여러 개의 쓰레드가 작업을 병렬로 처리할 수 있다.
- 작업을 설정하고, 취소할 수 있다.
<br>

<div class="spotlight1" markdown="1">
기존 `@Scheduled`에서는 한 번에 모든 데이터를 불러와서 작업했지만, `TaskScheduler`에서는 사용자가 알림을 생성할 때 작업을 예약한다.
</div>

#### [예시]

- `@Scheduled`와 동일하게 cron 표현식으로 반복 작업을 설정할 수 있다.
- `Instant`로 작업이 수행될 시간을 예약할 수 있다.

```java
@Component
@RequiredArgsConstructor
public class SimpleTaskScheduler {

    private final ThreadPoolTaskScheduler taskScheduler;

    // @Scheduled와 동일하게 cron 표현식으로 설정
    public void startSimpleTask() {
        Runnable task = () -> System.out.println("작업 수행: " + LocalDateTime.now());
        CronTrigger cronTrigger = new CronTrigger("0/5 * * * * *"); // 5초마다 실행
        taskScheduler.schedule(task, cronTrigger);
        System.out.println("반복 작업 설정됨");
    }

    // 특정 Instant로 작업 예약
    public void scheduleTaskAtInstant() {
        Runnable task = () -> System.out.println("작업 수행: " + Instant.now());

        // 10초 뒤 실행
        Instant futureTime = Instant.now().plusSeconds(10);
        taskScheduler.schedule(task, Date.from(futureTime));

        System.out.println("작업 예약 시간: " + futureTime);
    }
}
```

#### [병렬 처리]

별도의 쓰레드 풀 설정 없이 `ThreadPoolTaskScheduler`를 사용하면 **단일 쓰레드**가 작업 시간을 체크하면서 등록된 작업들을 처리한다.

<img src="/assets/img/24/11/25/defaultPoolSize.png" alt="defaultPoolSize" width="500">

<br>
각각 등록되는 작업들에 대해서 단일 쓰레드가 예약 시간을 체크하고, 실행되어야 하는 작업까지 처리한다면 병목 현상이 발생할 것이다.
<br>
쓰레드 풀을 설정하여 멀티스레드 기반으로 처리하도록 개선해보자.
<br>

```java
@Configuration
public class SchedulerConfig {

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5); // 스레드 풀 크기 설정
        scheduler.setThreadNamePrefix("TaskScheduler-"); // 쓰레드 이름 설정
        scheduler.initialize();
        return scheduler;
    }
}
```

```java
@Component
@RequiredArgsConstructor
public class MyTaskScheduler {

    private final TaskScheduler taskScheduler;

    public void startTask() {
        Runnable task = () -> System.out.println("작업 수행: " + Thread.currentThread().getName());
        
        Instant futureTime = Instant.now().plusSeconds(10);
        taskScheduler.schedule(task, Date.from(futureTime));
    }
}
```

#### [schedule 메서드 뜯어보기]

아래는 `ThreadPoolTaskScheduler` 클래스의 `schedule` 메서드다.
<br>
`ScheduledExecutorSevice`가 실제 쓰레드풀을 관리하고 작업을 실행시키는 주체이다.
<br><br>
내부적으로 `DelayQueue`를 확장한 우선순위 큐를 사용하여, 예약된 작업들을 실행 시간 기준으로 정렬하고, 가장 빨리 실행될 작업부터 순차적으로 실행한다.

```java
public ScheduledFuture<?> schedule(Runnable task, Instant startTime) {
    ScheduledExecutorService executor = this.getScheduledExecutor();
    Duration delay = Duration.between(this.clock.instant(), startTime);

    try {
        return executor.schedule(this.errorHandlingTask(task, false), NANO.convert(delay), NANO);
    } catch (RejectedExecutionException var6) {
        RejectedExecutionException ex = var6;
        throw new TaskRejectedException("Executor [" + executor + "] did not accept task: " + task, ex);
    }
}
```

<br>

아래는 실제로 우선순위 큐 정렬기준으로 사용된 메서드다

<img src="/assets/img/24/11/25/method.png" alt="method" width="500">

<br>

#### [등록된 작업 취소]

바로 위의 메서드에서 반환 값이 `ScheduledFuture`를 반환하고 있다.
<br>
이 객체를 사용하여 작업을 취소하거나 상태를 확인할 수 있다.
<br>
** 메모리를 사용하여 객체를 저장해놓기 때문에 생명주기를 잘 관리해야한다.

```java
private final TaskScheduler taskScheduler;
private ScheduledFuture<?> scheduledTask;

public void startTask() {
    Runnable task = () -> System.out.println("작업 수행: " + Thread.currentThread().getName());
    
    Instant futureTime = Instant.now().plusSeconds(10);
    scheduledTask = taskScheduler.schedule(task, Date.from(futureTime));
}

public void cancelTask() {
    if (scheduledTask != null && !scheduledTask.isCancelled()) {
        scheduledTask.cancel(false); // 실행 전 작업만 취소
        System.out.println("작업이 취소되었습니다.");
    } else {
        System.out.println("취소할 작업이 없습니다.");
    }
}
```