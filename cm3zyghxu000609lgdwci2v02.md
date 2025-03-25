---
title: "광고 업체 유효성 검사 시스템 개선"
datePublished: Wed Nov 27 2024 14:02:29 GMT+0000 (Coordinated Universal Time)
cuid: cm3zyghxu000609lgdwci2v02
slug: kafka
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/GC_MO-vOa3s/upload/ffd9d482dd74ccbf2be1f63a058b4815.jpeg
tags: mysql, mongodb, kafka, grpc, webflux

---

오늘은 광고 업체 유효성 검사 시스템을 어떻게 발전시켜 왔는지 과정을 정리해봤다.

처음에는 단순한 API 호출로 시작했지만, 지금은 Kafka 기반의 실시간 이벤트 처리 시스템으로 진화를 거듭했으며

이 포스트는 해당 여정을 정리한 글이다🚀

## Version 1.0: "단순하게 시작해보자" - 직접 API 호출 방식

### 시스템 구성

처음에는 정말 단순했다

**광고 파트가 사용하는 광고 검사 URL을 서버에서 호출한다.**

광고 파트에서 제공하는 API를 직접 호출하고, MySQL 데이터베이스를 실시간으로 조회하는 방식이다

```java
// 초기 버전의 pseudocode
public List<String> checkVendorValidity(List<String> venIdList) {
    return adApiClient.call("/addpapi/dp/cpcPlusDispTgtVenChk.ssg", venIdList);
}

```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732715550685/33a44349-14f3-489a-8b48-f3d4660ed8fe.png align="center")

### 장점

* 실시간으로 데이터를 조회하니 정확성이 높음
    
* 변경된 업체 상태가 바로바로 반영됨
    
* 구현이 단순해서 이해하기 쉽다
    

### 단점

처음엔 괜찮았지만, 트래픽이 늘어나면서 문제가 하나둘씩 나타나기 시작했다:

* API를 동기식으로 호출하다 보니 응답 시간이 늘어남
    
* MySQL 서버가 점점 힘들어하기 시작했고 결국에는 뻗어버렸다…
    
* 광고 구좌가 늘어날수록 시스템이 버거워함 → 확장성 떨어짐
    

### 개선이 필요했던 부분

* MySQL 서버를 직접 호출하는 방식은 확장성에 한계가 있다
    
* 시스템 간 결합도가 너무 높음 → 의존도 감소 필요
    

## Version 2.0: "캐시를 써보자" - 배치 기반 캐싱 시스템

### 시스템 구성

MySQL 부하를 줄이기 위해 배치 처리와 캐싱을 도입했습니다.

```java
// 배치 처리 pseudocode
@Bean
public Step advertVenIdAggrStep() {
  return stepBuilderFactory.get("advertVenIdAggrStep").tasklet((stepContribution, chunkContext) -> {
    mysqlJdbcTemplate = new JdbcTemplate(advertDataSouce);
    Set<String> venIdList = new HashSet<>(getAdvertSql("select.advert.all"));
    log.info(">>> 최종 유효 업체 {}개 저장", venIdList.size());

    saveAdvertVenIds(venIdList, "advertVenIds");
    insertRedisStep(venIdList);
    return RepeatStatus.FINISHED;
  }).build();
}

// 검증 url
@PostMapping("/aiad/check")
public Mono<List<String>> checkVenId(@Valid @RequestBody List<String> venId) {
  return aiadVenService.findValidVenId(venId);
}

```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732715858692/9b7a6854-ba4d-4f6d-ac8b-a57befa732a3.png align="center")

기존의 유효한 업체들을 미리 가이드쿼리 기준으로 몽고 컬렉션으로 떨궈서 해당 컬렉션을 조회하는 방식

### 장점

* MySQL 서버 부하 대폭 감소
    
* webflux 기반의 aggregate API를 사용하면서 성능 향상
    
* 응답 속도 대폭 개선
    
* 시스템 안정성이 전반적으로 향상
    

### 단점

하지만 새로운 문제들이 등장했습니다:

* 업체 상태 변경을 실시간으로 추적하기 어려움 (로깅 시스템의 부재)
    
* 배치 주기만큼 데이터 반영이 지연
    
* 가이드 쿼리가 변경될 때마다 배치 로직을 수정
    

### 개선이 필요했던 부분

* 모니터링 체계가 부족하다 → 업체의 상태 변경을 확인하기 어렵다
    
* 가이드 쿼리에 대한 의존성을 줄일 필요
    

## Version 3.0: "실시간으로 가자" - 이벤트 기반 시스템

### 시스템 구성

Kafka를 도입하여 실시간 이벤트 처리 시스템으로 진화 → 현재 운영 시스템에 도입한 방식

```java
// Kafka Consumer pseudocode
@KafkaListener(topics = TOPIC_NAME, groupId = GROUP_ID)
  public void consume(ConsumerRecord<String, String> record) throws Exception {
    try {
      ObjectMapper objectMapper = originalObjectMapper.copy();
      objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.LOWER_CAMEL_CASE);
      String message = record.value();
      AiadVenDispStatMessageDto msg = objectMapper.readValue(message, AiadVenDispStatMessageDto.class);
      LocalDateTime messageTimestamp = LocalDateTime.ofInstant(Instant.ofEpochMilli(record.timestamp()),
        ZoneId.systemDefault());
      boolean isUseYn = isValidVen(msg);
      AiadVenStat stat = ...;
      aiadVenStatRepository.save(stat);
      AiadValidVen validVen = AiadValidVen.builder()
        ._id(msg.getKey())
        .build();
      if (isUseYn) {
        aiadValidVenRepository.save(validVen);
      } else {
        aiadValidVenRepository.delete(validVen);
      }
    } catch (Exception e) {
      log.error("AiadVenDispStatMessageDto Exception : {}", record);
      throw e;
    }

    boolean isValidVen(AiadVenDispStatMessageDto msg) {
        return ...;
      }
  }

// gRPC 서비스
@GrpcService
public class VendorValidityService extends VendorValidityServiceGrpc.VendorValidityServiceImplBase {
    @Override
    public void checkValidity(VendorRequest request, StreamObserver<VendorResponse> observer) {
        boolean isValid = checkVendorCriteria(request.getVendorId());
        observer.onNext(VendorResponse.newBuilder().setIsValid(isValid).build());
        observer.onCompleted();
    }
}

```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732715975996/855c1508-7378-48ba-9044-bbf2d07d4689.png align="center")

카프카 기반의 업체 상태 변경 토픽을 consume 하여 몽고 컬렉션에 이력 저장

유효한 업체들을 판별하여 별도의 유효 컬렉션 생성

통시 프로토콜에 gRPC를 도입하여 한 단계 더 향상

### 장점

* 실시간으로 업체 정보가 동기화
    
* 상태 변경 이력을 추적 가능
    
* gRPC 도입으로 성능이 더욱 개선
    
* 시스템 확장이 한결 수월
    

### 현재 시스템의 특징

* 표준화된 필드(`adug_critn_disp_yn`, `ven_critn_disp_yn`)를 사용하여 유지보수가 편하다
    
* 분산 시스템 아키텍처로 안정성이 크게 향상
    
* AI추천광고 시스템과의 연동도 자연스럽게 가능
    

## 마치며

이렇게 우리 시스템은 단순한 API 호출부터 시작해서 실시간 이벤트 처리 시스템으로 성장했습니다. 각 단계마다 새로운 도전과 문제가 있었지만, 그때그때 적절한 기술을 도입하고 아키텍처를 개선하면서 해결하고 개선해 나감

현재의 시스템은 이전보다 훨씬 더 안정적이고 확장 가능하며 유지보수하기 쉬워졌습니다. 추후 개선이 필요한 부분이 생기면 보다 수월하게 적용이 가능할 것으로 보입니다!

### 핵심 개선 포인트 요약

1. 성능: API 직접 호출 → 캐싱 → 이벤트 기반 처리
    
2. 확장성: 단일 DB → 분산 아키텍처
    
3. 유지보수성: 임시 쿼리 → 표준화된 필드
    
4. 안정성: 동기식 → 비동기식 처리
    
5. 모니터링: 제한적 → 상세 이력 관리