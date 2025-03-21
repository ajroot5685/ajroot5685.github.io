---
title: 스프링 로깅 도입과 슬랙 연동으로 실시간 서버 오류 전달받기
date: 2024-08-23 23:01:00 +09:00
categories: [spring]
tags:
  [
    spring,
    logging
  ]
---

## 로깅이란?

<div class="spotlight1">
애플리케이션이 실행되는 동안 발생하는 다양한 이벤트, 에러, 경고 및 기타 정보를 기록하여 디버깅, 성능 모니터링, 문제 분석 등에 도움을 주는 기능이다.
</div>

> 로깅에 대한 개념은 많이 나와있으므로 다루지 않겠다.

<br>

## Slf4j
- **S**imple **L**ogging **F**acade **for** **J**ava
- 자바 애플리케이션에서 여러 로깅 구현체들을 통합하여 일관된 인터페이스를 제공하는 로깅 파사드이다.
    > 파사드 패턴이 궁금하다면 [여기](https://ajroot5685.github.io/posts/Facade-Pattern/) 를 참고하자.
- `Logback`, `Log4j`, `java.util.logging` 등과 결합하여 사용할 수 있도록 설계되었다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log {
    private static final Logger logger = LoggerFactory.getLogger(Log.class);

    public void main() {
        logger.info("INFO 로그");
        logger.debug("DEBUG 로그");
        logger.error("ERROR 로그");
    }
}
```

<br>

#### @Slf4j
- Lombok 라이브러리에서는 로깅 객체를 자동으로 생성해주는 애노테이션을 제공하여 더욱더 편리하게 사용할 수 있게 해준다.

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Log {

    public void main() {
        log.info("INFO 로그");
        log.debug("DEBUG 로그");
        log.error("ERROR 로그");
    }
}
```

<br>

## 로깅 설정 커스터마이징
- 기본적으로 스프링부트에는 로깅 기본설정이 존재하여 빌드할 때 다음 콘솔 화면을 볼 수 있다.
    <img src="/assets/img/240823/logging1.png" alt="logging1" width="800">
- 여기서 더 나아가 우리가 원하는대로 로깅 설정을 하려면 `logback-spring.xml` 파일을 생성하여 직접 설정을 해줘야 한다.

<br>

#### [주요 태그]
- 설정하는 데 필요한 주요 태그들을 간단하게 설명하겠다.

<br>

**\<configuration>**
- Logback 설정 파일의 루트 태그로, 모든 설정이 이 태그 안에 포함된다.

<br>

**\<property>**
- 로깅 패턴이나 파일 경로 등의 값을 변수로 정의하는 데 사용된다.

<br>

**\<springProperty>**
- 스프링 환경 변수나 프로퍼티 값을 로깅 설정을 위한 변수로 사용할 수 있게 한다.

<br>

**\<include>**
- 다른 Logback 설정 파일을 포함할 때 사용된다.

<br>

**\<appender>**
- 로그 메세지를 출력할 대상을 정의한다.
- 이 태그 안에서 <layout> 은 로그 메세지의 형식을 의미한다.

<br>

**\<root>**
- 전체 애플리케이션에 적용되는 기본 레벨을 정의한다.
- `level="INFO"` 로 설정하면 INFO 레벨 이상의 로그들만 나온다.
- \<appender-ref> 로 정의한 \<appender>의 로깅 레벨을 설정한다.
    > 설정하지 않으면 해당 \<appender>는 적용되지 않는다.

<br>

#### [logback-spring.xml]
- 경로는 `src-main-resources` 바로 아래에 두자.
    > 폴더 안에 따로 관리하고 싶다면 인식을 위한 세팅이 추가로 필요하다.
- 슬랙에 대한 설정을 추가하기 전에 기본적인 로깅이 잘 동작하도록 설정하겠다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 변수 설정 -->
    <property name="LOG_PATTERN" value="[%d{yyyy-MM-dd HH:mm:ss}:%-3relative][%thread]" />
    <property name="LOG_PATTERN_CONSOLE" value="${LOG_PATTERN} %highlight(%-5level) %logger{36} - %msg%n" />

    <!-- 콘솔 출력 설정 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN_CONSOLE}</pattern>
        </encoder>
    </appender>

    <!-- 로깅 레벨 설정 -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>

</configuration>
```
- \<property>로 로그의 패턴을 정의했다.
    - `%highlight()` 를 사용하여 로그 레벨 별로 다른 색깔로 출력하도록 설정하였다.
        <img src="/assets/img/240823/logging2.png" alt="logging2" width="600">

    > property는 다른 property의 값도 사용 가능하다.
- \<appender>에서 정의한 패턴을 이용하면서, 출력 대상을 콘솔로 설정했다.
- \<root>에서 레벨을 INFO로 설정하여 INFO 레벨 이상의 로그들이 콘솔에 출력될 수 있도록 정의했다.

<br>

## 슬랙 연동 준비
- 슬랙으로 알람을 보내기 위해 슬랙에서 `Webhook URL`을 생성해야 한다.

1. 채널의 통합 탭에서 **앱 추가**를 누른다.
    <img src="/assets/img/240823/slack1.png" alt="slack1" width="600">

2. `incoming webhooks` 를 검색하고 설치한다.
    <img src="/assets/img/240823/slack2.png" alt="slack2" width="600">

3. 웹페이지에서 연결할 채널을 선택하고 추가한다.
    <img src="/assets/img/240823/slack3.png" alt="slack3" width="600">

4. 원하는대로 웹훅에 대한 세팅을 마친 뒤 **Weghook URL**을 기록해둔다.
    <img src="/assets/img/240823/slack4.png" alt="slack4" width="600">

<br>

## 슬랙 연동을 위한 설정 추가
- 이제 애플리케이션에서 슬랙과 연동을 할 차례다.

<br>

#### [외부 라이브러리 사용]
- 연동을 편하게 할 수 있는 외부 깃헙 라이브러리를 사용한다.
- `build.gradle` 에 다음을 추가한다.
    ```groovy
    implementation 'com.github.maricn:logback-slack-appender:1.6.1'
    ```

<br>

#### [application.yml 설정 정보 추가]
- 알림 설정에 사용할 변수들을 선언한다.
- 웹훅 url, 채널명, 메세지를 보낸 이름, 이모지를 설정한다.
    ```yml
    logging:
      slack:
        webhook-uri: https://hooks.slack.com/services/T05...
        channel: error
        username: logging
        emoji: rotating_light
    ```
- 결과를 먼저 보여주자면 다음과 같다.
    <img src="/assets/img/240823/slack5.png" alt="slack5" width="800">

<br>

#### [logback-spring.xml 설정 정보 추가]
- \<springProperty> 를 통해 `application.yml` 로부터 설정 정보를 가져왔다.
    > `application.yml` 파일 하나를 숨김으로써 안전하게 보안을 지킬 수 있다.
- 전역적인 로깅 레벨을 `INFO`지만, `filter` 프로퍼티를 통해 `ERROR` 레벨의 로그가 발생했을 때에만 슬랙에 알림이 발송된다.
- 알림 메세지는 입맛에 맞게 바꾸면 좋다.
    - `%msg`에 발생한 로그의 메세지가 들어간다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 변수 설정 -->
    <property name="LOG_PATTERN" value="[%d{yyyy-MM-dd HH:mm:ss}:%-3relative][%thread]" />
    <property name="LOG_PATTERN_CONSOLE" value="${LOG_PATTERN} %highlight(%-5level) %logger{36} - %msg%n" />
    <property name="LOG_PATTERN_SLACK" value="${LOG_PATTERN} %-5level %logger{36} %msg%n" />

    <springProperty name="SLACK_WEBHOOK_URI" source="logging.slack.webhook-uri"/>
    <springProperty name="SLACK_CHANNEL" source="logging.slack.channel"/>
    <springProperty name="SLACK_USERNAME" source="logging.slack.username"/>
    <springProperty name="SLACK_EMOJI" source="logging.slack.emoji"/>

    <!-- 콘솔 출력 설정 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN_CONSOLE}</pattern>
        </encoder>
    </appender>

    <!-- 슬랙 알림 전송 -->
    <appender name="SLACK" class="com.github.maricn.logback.SlackAppender">
        <webhookUri>${SLACK_WEBHOOK_URI}</webhookUri>
        <channel>#${SLACK_CHANNEL}</channel>
        <username>${SLACK_USERNAME}</username>
        <iconEmoji>:${SLACK_EMOJI}:</iconEmoji>
        <colorCoding>true</colorCoding>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>${LOG_PATTERN_SLACK}</pattern>
        </layout>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 로깅 레벨 설정 -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SLACK"/>
    </root>

</configuration>
```

<br>

## Error 로그 설정
- 이제 로직들 사이에서 에러 로그들을 설정할 차례이다.

<br>

#### [테스트용 컨트롤러]
- 테스트용 컨트롤러를 만들어서 일부러 에러를 발생시키도록 하겠다.

```java
@RestController
@Slf4j
public class TestController {

    @GetMapping("/")
    public ResponseEntity<?> home() {
        String test = "테스트용 인자";
        log.error("에러 로그 테스트 = {}", "테스트용 인자");
        return ResponseEntity.ok("hello world");
    }
}
```
- `@Slf4j`를 통해 쉽게 에러 로그를 찍을 수 있다.
    <img src="/assets/img/240823/slack6.png" alt="slack6" width="800">
- 지금까지 잘 설정되었다면 슬랙 채널에도 알림이 정상적으로 발송되었을 것이다.
    <img src="/assets/img/240823/slack7.png" alt="slack7" width="800">

<br>

#### [에러 핸들링 처리하기]
- 그럼 에러가 날 수 있는 곳에 일일이 로깅 처리를 다 할 것인가? 유지보수는 어디갔는가?
- `@ControllerAdvice`를 사용하여 발생한 에러를 컨트롤러 레이어에서 자동으로 처리하자
- 아래 코드에서 `ErrorCode`는 규격화된 에러 객체이다.

```java
@RestControllerAdvice
@Slf4j
public class WebExceptionHandler {

    @ExceptionHandler(ErrorException.class)
    public ResponseEntity<ErrorResponse> handlerTokenException(ErrorException ex) {
        ErrorCode errorCode = ex.getErrorCode();
        errorLogging(ex.getStackTrace(), errorCode, 0);

        return ResponseEntity
                .status(HttpStatus.valueOf(errorCode.getStatus()))
                .body(ErrorResponse.of(errorCode));
    }

    @ExceptionHandler(Exception.class)
    protected ResponseEntity<ErrorResponse> handleRemainException(Exception ex) {
        ErrorCode errorCode = ErrorCode.SERVER_ERROR;
        errorLogging(ex.getStackTrace(), errorCode, 1);

        return ResponseEntity
                .status(errorCode.getStatus())
                .body(ErrorResponse.of(errorCode));
    }
}
```
- 이렇게 처리하면 어느 레이어에서 발생한 오류가 Controller까지 전파되었을 때 정의된 처리 로직에 따라 에러가 처리된다.
- 에러를 던지는 예시는 다음과 같다.

```java
@RestController
@Slf4j
public class TestController {

    @GetMapping("/")
    public ResponseEntity<?> home() {
        throw new ErrorException(ErrorCode.OBJECT_NOT_FOUND);
    }
}
```

<br>

#### [로깅 로직 구성하기]
- 이제 어떻게 로그를 구성할 건지 고민할 차례다.
- 나는 `어느클래스의 어디메소드에서 발생했는지`, `에러코드는 무엇인지`, `알 수 없는 에러의 경우에는 전체 에러 메세지까지 출력`하고 싶었다.
- 먼저 어디서 발생했는지는 인자로 넘어온 Exception 객체 속의 `stackTrace`를 살펴보면 된다.

```java
StackTraceElement[] stackTraceElements = e.getStackTrace();

// 각 스택 트레이스 요소를 순회하면서 정보를 출력
for (int i = 0; i < stackTraceElements.length; i++) {
    StackTraceElement element = stackTraceElements[i];
    System.out.println("Index " + i + ": " + element.getClassName() + "." + element.getMethodName() 
            + " (Line: " + element.getLineNumber() + ")");
}
```
<img src="/assets/img/240823/stacktrace1.png" alt="stacktrace1" width="600">

- 확인을 해보니 2번째 인덱스에 내가 원하는 클래스와 메서드가 나열되어 있다.
- 서비스 레이어에서만 에러를 발생시키고, 컨트롤러와 서비스 레이어 간 중간레이어가 없으므로 2번 인덱스의 데이터를 그대로 사용하기로 했다.
- 알 수 없는 에러가 발생한 경우에는 위치를 정확히 파악할 수 없으므로 전체 에러 메세지를 그대로 로그에 넣기로 하였다.
- 이를 반영한 `errorLogging()` 메서드는 다음과 같다.

```java
private void errorLogging(StackTraceElement[] stackTrace, ErrorCode errorCode, Integer type) {
    String callerClassName = "Unknown";
    String callerMethodName = "Unknown";

    if (stackTrace.length > 2) {
        callerClassName = stackTrace[2].getClassName();
        callerMethodName = stackTrace[2].getMethodName();
    }

    String stackTraceString = "";
    if (type != 0) {
        StringBuilder sb = new StringBuilder();
        sb.append(Arrays.toString(stackTrace)).append("\n");
        for (StackTraceElement element : stackTrace) {
            sb.append("\tat ").append(element.toString()).append("\n");
        }
        stackTraceString = sb.toString();
    }

    if (type == 0) {
        log.error("\n에러 발생 위치: {}.{}\n에러 코드: {}", callerClassName, callerMethodName, errorCode);
    } else {
        log.error("\n에러 발생 위치: {}.{}\n에러 코드: {}\n에러 메세지: {}", callerClassName, callerMethodName, errorCode, stackTraceString);
    }
}
```

- 이렇게 에러가 발생하면 바로 나의 개인채널로 알림이 오도록 설정이 완료됐다.
    <img src="/assets/img/240823/slack8.png" alt="slack8" width="800">
