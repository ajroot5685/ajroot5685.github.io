---
title: JWT RTR 도입 (2/2) - 스프링에서 RTR 로직 적용
date: 2024-07-20 20:21:00 +09:00
categories: [backend]
tags:
  [
    backend,
    spring,
    redis,
    jwt,
    refresh token rotation
  ]
---

> `JWT`, `RTR`에 대한 개념이 부족하다면 [이 글](https://ajroot5685.github.io/posts/JWT/)을 보고 오기 바란다.

## Refresh Token Rotation
- 다중 토큰을 사용하는 JWT에서 `Refresh Token`(RT)의 탈취 위험을 줄이고자 도입하는 기술이다.
- 하나의 RT만 계속 사용한다면 공격자는 회원의 신분(`Access Token`)으로 계속 서비스를 이용할 수 있게 된다.
- 간단하게 RTR를 설명하면 아래와 같다.

<div class="spotlight2" markdown="1">
1. `reissue` 때마다 새로운 RT로 갱신한다.
2. 반드시 하나의 RT만 사용한다.
3. 현재 사용중인 RT 외에 다른 RT(무효화된 RT)로 `reissue`가 시도될 경우 모든 RT를 무효화시킨다.
4. 위 경우 서비스를 이용하려면 다시 로그인해야 한다.
</div>

<br>

## RTR을 적용할 때 추가되는 로직
- 기존의 로직에서는 유저 정보가 들어오면 DB에서 회원 정보를 본 다음 토큰을 생성하고 넘겨주기만 했다.
- 조금 더 나아간다면 RT를 DB 또는 세션에 저장했을 것이다.
- `reissue` 에서는 서버에 저장된 RT와 요청으로 들어온 RT를 대조해서 같으면 새로운 `Access Token`을 넘겨준다.

#### [로그인]
1. 생성한 `refresh Token` 을 DB에 저장한다.

#### [Reissue]
1. DB에 저장된 해당 회원의 RT를 가져온다.
2. 요청으로 들어온 RT와 저장된 RT를 비교한다.
    - 같으면, 새로운 RT를 생성하고 DB에 저장 및 회원에게 전달한다.
    - 다르면, 회원의 모든 RT를 무효화한다.

> 원래는 추가 정보를 수집하기 위해 블랙리스트로 이전의 모든 RT들을 저장하지만, 정보가 필요하지 않으므로 DB에는 하나의 회원이 현재 사용하는 RT만 저장하도록 설계하겠다.

#### [로그아웃]
1. 로그아웃 시 저장된 회원의 RT를 삭제한다.

<br>

## 왜 Redis를 사용하는가?

<img src="/assets/img/24/07/20/redis.png" alt="redis" width=600>
- 물론 RDB(`Relational DataBase`)를 사용해도 되지만 Redis를 사용하는 것이 더 좋고 편하다.

1. 성능이 빠르다.
- 로그인도 보통 RDB를 통해 인증을 하는데, `reissue`도 RDB를 쓴다면 복잡한 `reissue`가 아니라 그냥 로그인을 하면 될 것이다.

2. Redis의 단점이 치명적이지 않다.
- Redis는 오류 발생 시 기존의 데이터들이 날라가고, 복구가 힘들 수도 있다는 단점이 있다.
- 그러나 데이터가 날아가더라도 다시 로그인을 하면 되므로 단점이 치명적이지 않다.

3. Time To Live 기능
- Redis에는 유효기간이 지나면 자동으로 데이터를 삭제하는 TTL 기능이 내장되어 있다.
- 이 편리한 기능으로 서버에서 직접 유효기간을 확인할 필요 없이 자동으로 유효기간이 지난 토큰이 삭제된다.

<br>

## 스프링 Redis 설정
- 사실 이번이 처음으로 Redis를 써보는 것이었기에 조금 겁이 나기도 했지만 생각보다 별게 없다. RDB를 써본 적이 있다면 정말 쉽게 사용할 수 있으니 겁먹지 말고 도전해보자

#### [Redis 인스턴스 생성]
- 서버 컴퓨터에 Redis를 직접 설치하는 방법이 아닌 Docker 이미지를 통해 Redis를 구축하였다.
- 여기에 더해 좀 더 간편한 docker-compose 로 사용하여 다른 도커 인스턴스와 함께 관리하였다.

```yml
version: '3.8'

services:
  redis:
    image: redis:latest
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - ./redis-data:/data  # 호스트 경로 : 인스턴스 내부 경로
```

- 사실 도커를 사용하면 경로를 지정하는 `volumes` 설정을 통해 오류가 발생하더라도 데이터가 사라지지 않고 복구가 가능하다.
- 이외에도 여러 기능을 통해 데이터가 사라질 수 있다는 단점을 극복할 수 있어서 사실상 단점이 없다고 볼 수 있다.

#### [스프링 Redis 설정]

**build.gradle**
- redis의 인기 덕분에 스프링 부트에서도 레디스에 대한 종속 패키지를 제공하고 있다.
- 덕분에 별다른 버전 지정 없이 아래와 같이 명시해 주면 된다.
    > 이것이 바로 스프링이 아닌 스프링 부트의 장점이다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

<br>

**application.yml**
- 여러 가지 설정을 할 수 있지만 간단하게 `host`와 `port`만 지정해주겠다.

```yml
spring:
  redis:
    host: 인스턴스 주소
    port: 6379
```

<br>

**RedisConfig**
- `@EnableRedisRepositories` 를 사용하여 `스프링 데이터 JPA`와 연동한다. 이는 자동으로 Redis 리포지토리를 스캔하고 구현체를 생성하게 한다.
- Redis 서버에 연결을 위해 `Lettuce` 클라이언트를 사용한다.
- `RedisTemplate`을 통해 Redis 명령을 추상화한다.
    - 타입인 `<Long, String>`은 `<Key, Value>` 를 나타낸다.
    - key값에는 Long 타입인 `userId`를 사용한다.

```java
@Configuration
@EnableRedisRepositories
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(host, port);
    }

    @Bean
    public RedisTemplate<Long, String> redisTemplate() {
        RedisTemplate<Long, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        return redisTemplate;
    }
}
```

<br>

## 엔티티, Repository 설정
- Redis의 엔티티를 설정할 때는 `@RedisHash`를 지정해주어야 같은 타입의 키값으로 여러 엔티티를 사용할 때 충돌이 발생하지 않는다.
    - 즉, 테이블의 개념과 유사하다.
- `@TimeToLive`로 Redis 내장 기능인 TTL을 사용할 수 있다. 초단위로 설정된 시간이 지나면 자동으로 데이터가 삭제된다.

```java
@RedisHash("refresh")
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class RefreshToken {

    @Id
    private Long id;

    private String token;

    @TimeToLive
    private Long expiration;
}
```

```java
@Repository
public interface RefreshTokenRepository extends CrudRepository<RefreshToken, Long> {
}
```

<br>

## 비즈니스 로직
- 실제 service단 로직을 살펴보며 어떻게 Redis를 사용하여 RTR을 적용하는지 살펴보자.
- 모든 코드를 올리는 것은 양이 너무 많으므로 주석으로 간단하게 설명하겠다.

#### [로그인]
- 사실 이 로직이 적용되는 프로젝트는 소셜 로그인으로만 인증이 진행되고, 프론트에서 인증 후 회원정보만을 넘겨준다.
- 때문에 로그인에서는
    1. 회원 정보를 DB에 저장(회원가입)한다.
    2. 회원의 `userId`를 통해 토큰 세트를 생성한다.
- 여기서는 2번에 해당하는 토큰 세트를 생성하는 로직만 보겠다.

```java
public TokenDTO generateAndSaveNewToken(Long userId) {
    String accessToken = jwtUtil.createAccessToken(userId); // 액세스 토큰 생성
    String refreshToken = jwtUtil.createRefreshToken();     // 리프레시 토큰 생성

    Date refreshExpiration = jwtUtil.getExpiration(refreshToken);   // 미리 정의된 함수를 통해 Date 타입의 만료일자를 가져옴
    Long expirationSecond = refreshExpiration.getTime() / 1000; // 초단위, Long 타입으로 변경

    /**
     * RTR
     * 엔티티를 생성하여 Redis에 저장한다.
     * 새로 로그인하는 것이므로 기존에 RT가 있더라도 덮어씌워진다. -> 하나의 회원은 하나의 RT만 가진다.
     **/
    RefreshToken rt = new RefreshToken(userId, refreshToken, expirationSecond);
    refreshTokenRepository.save(rt);

    return new TokenDTO(accessToken, refreshToken); // 생성된 토큰 set 반환
}
```

<br>

#### [reissue]
- 원래는 저장된 RT와 동일한지, 유효한지만 확인한다.
- 그러나 RTR 기법에서는 무효화된 토큰이 들어온 것인지 확인하고, RT도 새로 발급한다.
- 사용했던 RT는 무효화시킨다.

```java
public TokenDTO reissue(String accessToken, String refreshToken, Long userId) {
    RefreshToken rt = refreshTokenRepository.findById(userId)   // 해당 회원의 RT 조회
            .orElseThrow(() -> new ErrorException(ErrorCode.INVALID_REFRESH_TOKEN));    // 못찾은 경우는 RT를 저장한 적이 없거나, RT의 유효기간이 만료되어 삭제된 경우다.

    if (!rt.getToken().equals(refreshToken)) {  // 저장된 RT와 다른 경우, 즉 무효화된 RT인 경우
        refreshTokenRepository.deleteById(userId);  // 저장된 RT도 삭제
        throw new ErrorException(ErrorCode.INVALID_REFRESH_TOKEN);  // 오류 반환
    }

    // 새로운 토큰 set 생성
    String newAccessToken = jwtUtil.createAccessToken(userId);
    String newRefreshToken = jwtUtil.createRefreshToken();

    Date refreshExpiration = jwtUtil.getExpiration(refreshToken);
    Long expirationSecond = refreshExpiration.getTime() / 1000;

    // 새로운 RT는 Redis에 저장, 기존 RT는 덮어씌워진다.
    RefreshToken newRt = new RefreshToken(userId, newRefreshToken, expirationSecond);
    refreshTokenRepository.save(newRt);

    return new TokenDTO(newAccessToken, newRefreshToken);
}
```

<br>

#### [로그아웃]
- 로그아웃하면 어떤 RT로도 `reissue` 를 진행할 수 없게 만들어야 하므로 RT를 삭제시킨다.

```java
public Long logout(Long userId) {
    refreshTokenRepository.deleteById(userId);

    // AccessToken 무효화시키는 로직 생략

    return userId;
}
```

<br>

## Redis 데이터 확인
- 실제 인스턴스에 접근해서 잘 저장되는지 살펴보자

- `docker exec -it <컨테이너 id> bash` 명령어로 인스턴스 접속
- `redis-cli` 로 redis 관리모드 진입

#### [기본적인 명령어들]

```bash
- set : 데이터 쓰기
set KEY VALUE
# ex. set 1 kim

- get : 데이터 읽기
get KEY
# ex. get 1

- scan : 단위 키 조회
scan 단위숫자
# ex. scan 0

- keys : 특정 키 조회
keys 패턴
# ex. keys *

- del : 키 삭제
del KEY
# ex. del 1

- hgetall : 해시 객체 조회 # 엔티티를 저장했을 때에는 이 명령어로 객체 조회를 해야 한다.
hgetall KEY
# ex. hgetall refresh:9
```

#### [로그인 후]
- `userId` 가 9인 회원 로그인

<img src="/assets/img/24/07/20/login.png" alt="login" width=300>
<img src="/assets/img/24/07/20/login2.png" alt="login2" width=600>

<br>

#### [reissue]
- reissue시 새로운 RT가 저장된다.

<img src="/assets/img/24/07/20/reissue.png" alt="reissue" width=600>

<br>

#### [로그아웃]
- 로그아웃 또는 유효하지 않은 RT가 들어온 경우 무효화, 여기서는 삭제된다.

<img src="/assets/img/24/07/20/logout.png" alt="logout" width=400>

<br>

## 누군가가 물어본다면
<div class="spotlight1" markdown="1">
`Refresh Token Rotation` 기법을 통해 `Refresh Token`의 취약점을 보완할 수 있습니다.
<br>
또한 `Redis`를 사용하여 `Refresh Token`을 빠르고 안전하게 보관할 수 있습니다.
</div>