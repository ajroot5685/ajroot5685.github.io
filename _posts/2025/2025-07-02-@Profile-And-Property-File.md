---
title: Profile 애노테이션과 프로필 별 yml 파일 구성 시 주의점
date: 2025-07-02 19:22:00 +09:00
categories: [spring]
tags:
  [
    spring,
    profile,
    yml
  ]
---

## 들어가며

스프링 프로젝트를 진행할 때 보통 `application.yml` 파일을 프로필 별로 구성하여 환경 별로 다른 파일을 사용한다.
<br>
추가로 나는, `@Profile`을 사용하여 특정 Config 클래스와 그 안에 속해 환경 변수를 불러오는 `@Value`를 비활성화하여 운영 환경에 맞는 설정을 구성하려 했다.
<br>
미숙함에서 발생했던 문제를 소개하며, `@Profile` 사용법과 설정 파일 구성 시 주의점에 대해 소개하려 한다.

<br>

## 구조

구조는 `resources` 폴더에 환경 변수 파일들을 모아놨다.

```text
src
└── main
    └── resources
        ├── application.yml
        ├── application-prod.yml
        ├── application-local.yml
        └── application-test.yml
```

<br>

`application.yml`
- `spring.profiles.active`로 활성화 시킬 프로필을 선택한다.
- 기본적으로 이 파일에 있는 모든 설정을 사용하며, 활성화 시킨 프로필에서 추가하거나 재정의 할 수 있다.

```yaml
spring:
  # 프로필 선택
  profiles:
    active: dev

  # 모든 프로필에 적용되는 설정
  web:
    resources:
      add-mappings: false

  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 20MB

  # test 프로필에는 필요하지 않음
  data:
    redis:
      host: ${REDIS_HOST}
      port: ${REDIS_PORT}
      timeout: 5000
```

<br>

`application-prod.yml`
- 모든 공통 설정을 사용하며, 스웨거 springdoc만 비활성화한다.
- redis의 변수들은 `.env`를 통해 주입된다.

```yaml
# 설정 추가
springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false
```

<br>

`application-local.yml`
- 모든 공통 설정을 사용하며, 스웨거에 들어갈 환경 변수는 하드코딩으로 정의한다.
- 환경변수들은 로컬에 존재하는 `.env` 파일을 사용한다.

```yaml
# 로컬에 있는 .env 파일 사용
spring:
  config:
    import: optional:file:.env[.properties]

# @Value로 들어갈 스웨거 환경 변수
swagger:
  server-url: http://localhost:8080
```

<br>
s
`application-test.yml`
- 인수테스트 시에는 메모리 H2 DB를 사용하며, 스웨거와 Redis는 사용하지 않는다.
- **그럼에도 불구하고 Redis 환경변수들은 더미데이터라도 넣어주어야 한다.**

```yaml
spring:
  data:
    redis:
      host: test
      port: 1234

springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false
```

<br>

## 문제

위의 예시에서는 최소한의 값만 넣었지만, 실제 프로젝트에서 `application-test.yml`은 필요없는 각종 더미데이터들이 들어가 매우 지저분했다.
<br>
그 이유는 테스트 컨벤션에서 다른 모든 프로필들이 사용하는 외부 서비스를 테스트하지 않고 mocking했기 때문이다.
<br>
외부 의존성은 메모리 H2 DB 단 하나밖에 없었다.

<br>

## @Profile을 사용해보자

그래서 지저분한 test.yml 파일을 개선하기 위해 환경 변수를 사용하는 config 클래스들을 비활성화 시켜보자 생각했다.
<br>
`@Profile`를 사용했는데 사용법부터 간단하게 알아보겠다.

<br>

### 프로필 하나 지정

```java
@Profile("local")
public class SwaggerConfig { ... }
```

프로필명을 쌍따옴표에 감싸 value로 전달한다.

<br>

### 프로필 여러개 지정

```java
@Profile({"local", "prod"})
public class SwaggerConfig { ... }
```

여러 개의 프로필을 지정할 땐 중괄호에 각 프로필을 따옴표에 감싸 전달한다.

<br>

### 하나의 프로필만 제외

```java
@Profile("!local")
public class SwaggerConfig { ... }
```

프로필 하나만 제외하는 경우 해당 프로필 앞에 !를 붙인다.

<br>

### 비활성화된 클래스의 @Value

비활성화된 클래스에 속한 `@Value`도 비활성화되고, yml 파일로부터 값을 읽어오지 않는다.

```java
@Profile({"local", "prod"})
@Configuration
public class SwaggerConfig {

    // 만약 프로필이 local이나 prod가 아니라면 @Value도 비활성화된다.
    @Value("${swagger.server-url}")
    private String swaggerServerUrl;

}
```

<br>

### 안되는 경우

아래는 내가 실제로 실수했던 사용방식이다.

```java
// 여러 개의 프로필을 지정할 때는 !를 사용할 수 없다.
@Profile({"!local", "!prod"})
public class SwaggerConfig { ... }

// 반드시 프로필마다 따옴표로 감싸줘야 한다.
// 아래의 경우 (local, prod) 라는 프로필만 선택된다.
@Profile({"local, prod"})
public class SwaggerConfig { ... }
```

<br>

## 애플리케이션 실행 실패 상황

이제 내가 겪었던 잘못 설정한 경우를 보겠다.
<br>
스웨거와 Redis에 대한 Config 클래스는 프로필마다 활성화 여부가 다르다.
<br>
> 가독성을 위해 yml 파일은 `---`을 이용하여 하나의 파일에 구성한 형태로 제시하겠다.

<br>

### @Profile 설정 예시

```java
@Profile({"local"})
@Configuration
public class SwaggerConfig {
    @Value("${swagger.server-url}")
    private String swaggerServerUrl;
}

@Profile("!test")
@Configuration
public class RedisConfig {
    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;
}
```

<br>

### 로컬에서 정상 작동하지만 test 환경에서 작동하지 않음

local 프로필은 스웨거와 Redis 둘 다 사용하며, 제대로 변수를 주입하여 정상 작동한다.
<br>
반면에 test 프로필은 스웨거와 Redis 둘 다 사용하지 않는데, Redis 환경 변수를 불러오지 못했다는 오류가 발생한다.

```yaml
spring:
  profiles:
    active: test

  data:
    redis:
      host: ${REDIS_HOST}
      port: ${REDIS_PORT}
      timeout: 5000
---
spring:
  profiles: local
  config:
    import: optional:file:.env[.properties] # redis에 대한 환경 변수가 로컬 .env 파일에 존재

swagger:
  server-url: http://localhost:8080
---
spring:
  profiles: test

# redis에 대한 환경 변수 정보가 없음
```

<br>

### 올바르게 작동시키려면?

redis의 설정 정보를 더미데이터라도 넣어주어야 한다.
<br>
왜 스웨거는 설정 안해도 되는데, redis는 꼭 넣어주어야 할까?

```yaml
spring:
  profiles:
    active: test

  data:
    redis:
      host: ${REDIS_HOST}
      port: ${REDIS_PORT}
      timeout: 5000
---
spring:
  profiles: local
  config:
    import: optional:file:.env[.properties]

swagger:
  server-url: http://localhost:8080
---
spring:
  profiles: test
  data:
    redis:
      host: test
      port: 1234
      timeout: 5000
```

<br>

### 문제의 원인

애플리케이션 빈에서 환경 변수를 불러오지 않더라도, 공통적으로 사용하면 `application.yml`에 `${}`으로 넣어줘어야 하는 변수가 있다면, 반드시 환경 변수 또는 활성화된 프로필에서 재정의 해줘야 한다.
<br>
그래서 스웨거에 대한 환경 변수는 생략할 수 있더라도, Redis에 대한 환경 변수는 넣어주어야 한다.
<br>
하지만 테스트는 말그대로 테스트기 때문에, 가독성이 떨어지더라도 필요 시 더미 변수값을 넣기로 했다.