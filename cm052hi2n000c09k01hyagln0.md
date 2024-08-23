---
title: "ParallelStream을 활용한 병렬 처리로 배치 성능 개선하기"
datePublished: Thu Aug 22 2024 09:15:16 GMT+0000 (Coordinated Universal Time)
cuid: cm052hi2n000c09k01hyagln0
slug: parallelstream
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/nVqNmnAWz3A/upload/ed0307db8ba5cf243a9fc3890287d2a5.jpeg
tags: redis, batch, parallelstream

---

1. ## 발단 👣
    

테이블에 담긴 대용량 데이터를 레디스에 적재하는 배치 코드를 수정할 사항이 있었다.

가장 기본적인 reader, processor, writer 구성이며 테이블 크기는 100만건 정도이다.

read하는 테이블만 수정하고 코드를 돌려보니 무척이나 오래 걸리는 구간들이 보이는것이다.

**병목 구간 분석**

1\. **데이터 매핑 및 Map 저장**: 테이블에서 가져온 데이터를 원하는 클래스로 변환한 후 Map에 저장하는 과정이 시간이 오래 걸림.

2\. **Redis Insert**: 변환된 데이터를 Redis에 삽입하는 부분에서도 시간이 많이 소요됨.

오래 걸리는 구간을 파악했으니 해결해보자

---

2. ## 시도 🤔
    

### **parallelStream 도입**

성능 개선을 위해 parallelStream을 도입해보았다.

parallelStream을 사용하면 기존 stream이랑 달리 병렬로 작업을 처리할 수 있기 때문에 전체 처리 시간이 줄어드는 효과를 기대해볼 수 있다. 실제로 도입 후 성능이 개선되는 것을 확인할 수 있었다.

코드 수정도 간단하다

```java
items.forEach(item -> { do something});

items.parallelStream().forEach(item -> { do something});
```

하지만 다시 찾아보니 parallelStream을 막 도입하기에는 주의사항이 많았다

---

3. ## 문제 😵‍💫
    

### **parallelStream 사용 시 주의사항**

1\. **공유 자원 동기화 문제**: parallelStream을 사용할 때, 여러 스레드가 동시에 접근하는 공유 자원에 대해 동기화 문제가 발생할 수 있다. 이로 인해 데이터 불일치나 예외 상황이 발생할 수 있음.

2\. **오버헤드 증가**: 모든 경우에 parallelStream이 성능을 개선하는 것은 아니다. 특히 작은 데이터셋이나 간단한 작업에서는 병렬 처리로 인한 오버헤드가 성능을 오히려 저하시킬 수 있음.

3\. **ForkJoinPool 제한**: parallelStream은 기본적으로 ForkJoinPool.commonPool()을 사용한다. 이 풀의 스레드 수는 JVM의 가용 CPU 코어 수에 의해 제한되기 때문에, 예상보다 병렬 처리 성능이 나오지 않을 수 있음.

만능처럼 보이던 `parallelStream` 은 동기화 문제, 오버헤드, 그리고 스레드를 직접적으로 건들기 때문에 수십개의 배치가 동시에 돌아가는 서버에서 어떠한 영향을 줄지 몰랐다...

하지만 이것도 해결책을 찾아보자~

---

4. ## 해결 🤩
    

### **ForkJoinPool 설정으로 해결**

커스텀 ForkJoinPool을 설정하는 방법을 통해 해당 `parallelStream` 에서 사용하는 쓰레드 수를 수동으로 제어할 수 있다.

이를 통해 병렬 작업의 스레드 수를 조절하여 과도하게 사용하는 것을 방지하고 성능을 더욱 최적화할 수 있었다.

```java
private final ForkJoinPool forkJoinPool = new ForkJoinPool(6);

try {
  forkJoinPool.submit(() ->
    keyInItems.parallelStream().forEach(item ->
      keyInItemMap.putIfAbsent(item.getQuery(), item)
    )
  ).get();
} catch (InterruptedException | ExecutionException e) {
  log.error("Error during parallel processing", e);
  throw new RuntimeException("Error during parallel processing", e);
}
```

### **Redis 성능 개선 - executePipelined 도입**

Redis의 경우, parallelStream을 사용하는 대신 executePipelined를 사용하여 성능을 개선했다.

executePipelined를 사용하면 여러 명령을 한 번에 전송하여 네트워크 비용을 줄일 수 있다.

db insert 쪽의 성능 향상을 위해서는 보통 오퍼레이션 수를 늘리는 것보다 batch insert를 하는 것을 권장한다.

```java
try {
  redisTemplate.executePipelined((RedisCallback<Object>) connection -> {
    items.forEach(item -> {
      try {
        String key = REDIS_TYPE_NAME + item.getQuery();
        byte[] value = objectMapper.writeValueAsBytes(item);
        connection.set(key.getBytes(), value);
        connection.expire(key.getBytes(), TimeUnit.DAYS.toSeconds(2));
      } catch (JsonProcessingException e) {
        log.error("{}", e.getMessage());
        throw new RuntimeException("Redis Pipeline 중 에러 발생", e);
      }
    });
    return null;
  });
} catch (Exception e) {
  log.error("Error during Redis Pipeline operation", e);
  throw new RuntimeException("Redis Pipeline 중 에러 발생", e);
}
```

---

5. ## 결과 🤝
    

**성능 개선 결과**

이제 개선된 배치의 소요시간을 살펴보자

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724316152854/515cbc69-3139-4224-879e-cd1c38766425.png align="center")

• **개선 전**: 병목 구간 약 6~7분 소요

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1724316208900/557eb969-89dc-47c8-b6c8-497b7af1c59e.png align="center")

• **개선 후**: 해당 구간 약 30초로 단축

parallelStream과 Redis의 executePipelined를 도입한 결과, 병목 구간의 개선을 이루어내면서 배치 수행 시간이 획기적으로 줄었다.

대용량 데이터를 처리하는 이런 배치 잡들은 작은 튜닝으로도 엄청난 성능 향상을 가져올 수 있다.

하지만 특정 기술들(이번 경우에는 parallelStream)을 사용할 때는 주의가 필요하며, 상황에 따라 적절한 적용이 중요해보인다.

무엇을 사용하든, 무엇인지 제대로 알고 쓰자..!