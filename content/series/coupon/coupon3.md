---
emoji: 3️⃣
title: 쿠폰 발급 (3) - 레디스 적용
date: '2023-05-23 00:00:00'
author: wooobo
tags: coupon
categories: 시리즈
---

## 레디스를 적용해보자

---

## 시나리오

1\. Coupon 이 생성 될때, 발행갯수만큼 CouponTicket(userId가 null 상태)을 DB에 저장한다.

2\. 쿠폰 발급, CouponTicket의 ID를 Redis에 쌓는다.

3\. 쿠폰 배급 Redis에 쌓이 CouponTicket을 가져온다.

- 가져온 CouponTicket이 이미 userId가 정해져있는지 DB에 확인하고 정해져 있지 않다면 userId를 맵핑하고 저장한다.

- 쿠폰티켓 결과를 반환해준다.

### 구현

1\. Coupon 이 생성 될때, 발행갯수만큼 CouponTicket(userId가 null 상태)을 DB에 저장한다.

```java
@RestController
public class CouponController {
	
    ...
    
    @PostMapping("api/create")
    public ResponseEntity<Void> create(@RequestBody int limit) {
        couponGenerateService.generate(limit);

        return ResponseEntity.ok().build();
    }
}

---

@Service
public class CouponGenerateService {
    private final CouponRepository couponRepository;
    private final CouponTicketRepository couponTicketRepository;

    public CouponGenerateService(CouponRepository couponRepository, CouponTicketRepository couponTicketRepository) {
        this.couponRepository = couponRepository;
        this.couponTicketRepository = couponTicketRepository;
    }

    @Transactional
    public void generate(int limit) {
        Coupon coupon = Coupon.limitOf(limit);
        couponRepository.save(coupon);

        List<CouponTicket> couponTickets = coupon.allPublish();
        couponTicketRepository.saveAll(couponTickets);
    }
}
```

2\. 쿠폰 발급, CouponTicket의 ID를 Redis에 쌓는다.

여러 방법이 있겠지만, 스케쥴러를 적용해보기로 했다.

일정 갯수 이하일때 Redis에 쌓기.

**Redis 설정하기**

```java
@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }
}
```

**스케쥴러 적용**

-   1초에 한번씩 체크
-   2000개 이하일때 10000개씩 밀어 넣게 구현

```java
@Service
public class CouponTicketRedisScheduler {

    private final RedisTemplate<String, Object> redisTemplate;
    private final CouponTicketRepository couponTicketRepository;

    public CouponTicketRedisScheduler(RedisTemplate<String, Object> redisTemplate, CouponTicketRepository couponTicketRepository) {
        this.redisTemplate = redisTemplate;
        this.couponTicketRepository = couponTicketRepository;
    }

    @Scheduled(fixedDelay = 1000)
    public void run() {
        Long couponId = 2L;

        ListOperations<String, Object> valueOperations = redisTemplate.opsForList();
        Long remainCount = valueOperations.size(generateKey(couponId));

        generateCouponTicket(remainCount, couponId);

        System.out.println("finished");
    }

    private synchronized void generateCouponTicket(Long remainCount, Long couponId) {
        Pageable page = Pageable.ofSize(10000);

        Stream<CouponTicket> couponTickets = couponTicketRepository.findByCouponIdAndUserIdNull(couponId, page).get();

        if (remainCount < 2000) {
            ListOperations<String, Object> valueOperations = redisTemplate.opsForList();
            List<Long> couponTicketIds = couponTickets.map(CouponTicket::getId).toList();

            valueOperations.rightPushAll(generateKey(couponId), couponTicketIds.toArray());
        }
    }

    private String generateKey(Long couponId) {
        return "couponTickets:" + couponId;
    }
}
```

3\. 쿠폰 배급 Redis에 쌓이 CouponTicket을 가져온다.

\- 컨트롤러

```java
    @GetMapping("api/publish/{userId}/{couponId}")
    public ResponseEntity<TicketDto> publish(@PathVariable Long userId, @PathVariable Long couponId) {
        CouponTicket couponTicket = couponTicketRedis.publish(couponId, userId);
        if (couponTicket != null) {
            return ResponseEntity.ok().body(TicketDto.of(couponTicket.getCouponId()));
        }

        return ResponseEntity.ok().body(TicketDto.fail());
    }
```

\- Reids 에서 쿠폰 티켓 가져오기 서비스

```java
@Component
public class CouponTicketRedis {
    private final RedisTemplate<String, Object> redisTemplate;
    private final CouponTicketRepository couponTicketRepository;

    public CouponTicketRedis(RedisTemplate<String, Object> redisTemplate, CouponTicketRepository couponTicketRepository) {
        this.redisTemplate = redisTemplate;
        this.couponTicketRepository = couponTicketRepository;
    }

    public CouponTicket publish(Long couponId, Long userId) {
        ListOperations<String, Object> valueOperations = redisTemplate.opsForList();

        // 티켓 POP
        CouponTicket couponTicket = (CouponTicket) valueOperations.leftPop(generateKey(couponId));
        if (couponTicket == null) {
            throw new NoSuchElementException("쿠폰이 존재하지 않습니다.");
        }

        // 유저 세팅
        couponTicket.owner(userId);

        // 유저 세팅 후 저장
        couponTicketRepository.save(couponTicket);

        return couponTicket;
    }

    private String generateKey(Long couponId) {
        return "couponTickets:" + couponId;
    }

}
```

## 부하 테스트

k6 부하 테스트

\- script.js

```java
import http from 'k6/http';
import {check, sleep} from 'k6';

export const options = {
    stages: [
        {duration: "10s", target: 500},
        {duration: "10s", target: 3000},
        {duration: "10s", target: 3000},
        {duration: "10s", target: 0},
    ],
    thresholds: {
        http_req_duration: ['p(99)<100'],
    },
};

export default function () {
    const res = http.get(`http://localhost:8080/api/publish/${getRandomInt(0, 1000000)}/2`);

    check(res, {'status was 200': (r) => r.status === 200});
    sleep(1);
}

function getRandomInt(min, max) {
    min = Math.ceil(min);
    max = Math.floor(max);
    return Math.floor(Math.random() * (max - min)) + min; //최댓값은 제외, 최솟값은 포함
}
```

**테스트 결과**

![](../content/series/coupon/coupon-3-01.png)

평균 100ms 이하로 나왔다.

지연되는 부분은

```java
        // 유저 세팅 후 저장
        couponTicketRepository.save(couponTicket);
```

## DB POOL SIZE 조정해보면 어떨까?

문득 DB 커넥션 POOL을 늘리면 빨라지지 좀 더 최적화 할 수 있지 않을까 생각이 되었다.

설정을 추가했다.
```yml
spring.datasource.hikari.maximum-pool-size: 100
```

그리고 pool size 를 나누어 테스트 해보았다.
```md
# DB pool size 25
avg=403.13ms min=2.63ms med=521.26ms max=999.98ms p(90)=712.01ms p(95)=755.75ms
avg=453.87ms min=2.79ms med=584.58ms max=1.17s   p(90)=808.83ms p(95)=942.38ms
avg=430.86ms min=2.79ms med=564.97ms max=1.24s    p(90)=861.28ms p(95)=947.41ms

# DB pool size 50
avg=327.05ms min=2.65ms med=354.36ms max=1.62s   p(90)=634.81ms p(95)=1.07s
avg=167.88ms min=2.33ms med=192.51ms max=580.31ms p(90)=329.93ms p(95)=395.59ms
avg=229.95ms min=2.6ms  med=312.7ms  max=686.1ms  p(90)=431.14ms p(95)=460.81ms

# DB pool size 75
avg=295.76ms min=3.05ms med=327.56ms max=927.11ms p(90)=614.81ms p(95)=676.6ms

# DB pool size 100
avg=178.74ms min=2.55ms med=208.44ms max=560.36ms p(90)=342.46ms p(95)=376.09ms
avg=163.61ms min=2.71ms med=133.92ms max=626.94ms p(90)=374.09ms p(95)=451.05ms
avg=205.32ms min=1.11ms med=215.16ms max=708.67ms p(90)=426.21ms p(95)=503.23ms
```

나름 유의미한 결과가 나왔다.  
현재 로컬 구조는 `docker Mysql`, `docker Redis`, `Local Spring boot Server` 이다.  
PC는 맥북 에어 m2이다.

현재 사양에서는 DB pool size 100 일때 가장 최적화된것으로 확인되었다.  
pool size 를 더 늘려보았지만, DB 사양 한계치가 있어서인지 그이상은 오히려 느려졌다.

**서버 스레드 늘리면?**
```md
server.tomcat.threads.max=20

DB pool size 50, tomcat threads 20
avg=464.32ms min=2.6ms  med=623.78ms max=882.89ms p(90)=821.75ms p(95)=838.93ms
avg=496.68ms min=2.68ms med=649.88ms max=987.8ms  p(90)=866.58ms p(95)=934.47ms
avg=539.1ms  min=2.77ms med=553.2ms  max=1.35s   p(90)=1.08s p(95)=1.15s
```

어차피 DB의 한계 때문에 `input` 을 마냥 많이 받는게 답은 아닌것 같다.


## 정리
(1) 에서는 로컬 시스템 만으로 쿠폰 도메인 로직과 동시성 이슈에 대해서 알아보았고,  
(2) 에서는 웹 서버로 올려서 부하테스트를 해보았다. 이벤트 기반으로 DB save 로직을 분리해보았다.  
(3) 다중 서버일때의 환경을 고려해서 Redis 를 적용해보고 테스트 까지 진행해보았다.  

--- 

> 여러가지 시나리오를 생각하며 진행해본 작업이었는데 재미있었다.


```toc

```
