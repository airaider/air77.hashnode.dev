---
title: "gRPC가 뭔데?"
datePublished: Mon Aug 05 2024 13:35:14 GMT+0000 (Coordinated Universal Time)
cuid: clzh1abv9000009mdfqsxalq3
slug: grpc
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/T9rKvI3N0NM/upload/d7a0dc26fafb96ec2377220ffbd55a43.jpeg
tags: grpc, grafana, microservice-architecture

---

gRPC: 현대적인 마이크로서비스 통신의 핵심

# gRPC: MSA 시대의 효율적인 통신 솔루션

마이크로서비스 아키텍처(MSA) 시대에 주목받고 있는 gRPC에 대해 알아보려고 합니다.

최근 많은 기업들이 MSA를 도입하면서, 서비스 간 효율적인 통신의 중요성이 크게 부각되고 있습니다. 이런 상황에서 gRPC는 MSA 환경에 최적화된 강력한 통신 솔루션으로 주목받고 있다. 그럼 gRPC가 무엇이고, 왜 MSA 환경에서 강점을 가지는지 자세히 살펴보자

## 1\. gRPC란? 🤔

gRPC는 Google이 개발한 오픈소스 RPC(Remote Procedure Call) 프레임워크이다.

간단히 말하면, 서로 다른 환경에 있는 서비스들이 효율적으로 통신할 수 있게 해주는 도구이다.

gRPC의 주요 특징:

* Protocol Buffers를 사용한 데이터 구조 정의
    
* HTTP/2 기반의 고성능 통신
    
* 다양한 프로그래밍 언어 지원
    
* 양방향 스트리밍 지원
    

## 2\. gRPC의 미래 전망 🚀

gRPC의 미래는 매우 밝아 보인다.

특히 모노 시스템에서 MSA 환경이 각광받는 요즘에는 그 중요성이 더욱 커질 것으로 예상됩니다.

* 클라우드 네이티브 환경에서의 채택 증가
    
* IoT 및 모바일 애플리케이션에서의 활용 확대
    
* 대규모 분산 시스템의 핵심 통신 프로토콜로 자리매김
    

## 3\. gRPC의 장점 👍

1. **높은 성능**: HTTP/2 기반으로 빠른 통신이 가능하다.
    
2. **강력한 타입 안정성**: Protocol Buffers를 통해 명확한 인터페이스를 정의할 수 있다.
    
3. **다중 언어 지원**: 다양한 프로그래밍 언어로 구현이 가능해 MSA 환경에 적합.
    
4. **양방향 스트리밍**: 실시간 데이터 처리에 적합.
    
5. **효율적인 직렬화**: 작은 메시지 크기로 네트워크 사용량을 줄일 수 있다.
    

## 4\. gRPC 간단한 예제

Kotlin을 사용한 간단한 gRPC 서버와 클라이언트 예제를 살펴보자

먼저, Protocol Buffers 정의 파일 일명 proto 단위로 데이터를 주고받게 된다

```plaintext
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

서버코드

요청이 들어오는 dto를 Proto에 명시된 HelloReply로 매핑하여 리턴한다

```kotlin
import io.grpc.ServerBuilder
import io.grpc.stub.StreamObserver

class GreeterService : GreeterGrpcKt.GreeterCoroutineImplBase() {
    override suspend fun sayHello(request: HelloRequest): HelloReply {
        return HelloReply.newBuilder()
            .setMessage("안녕하세요, ${request.name}님!")
            .build()
    }
}

fun main() {
    val server = ServerBuilder
        .forPort(50051)
        .addService(GreeterService())
        .build()

    server.start()
    println("서버가 포트 50051에서 시작되었습니다.")
    server.awaitTermination()
}
```

클라이언트 코드

서버단에 보낼코드를 Proto에 명시된 HelloRequest로 매칭하여 요청한다

```kotlin
import io.grpc.ManagedChannelBuilder

suspend fun main() {
    val channel = ManagedChannelBuilder.forAddress("localhost", 50051)
        .usePlaintext()
        .build()

    val stub = GreeterGrpcKt.GreeterCoroutineStub(channel)

    val request = HelloRequest.newBuilder()
        .setName("MSA 개발자")
        .build()

    try {
        val response = stub.sayHello(request)
        println("서버의 응답: ${response.message}")
    } finally {
        channel.shutdown()
    }
}
```

## 5. 현업 적용 예시

gRPC를 적용한 서비스와 기존 http 요청을 보내는 서비스를 비교해보자

nGrinder를 통한 부하테스트를 실행하고 grafana를 통해 성능을 각각 체크했다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723606236707/563c5d12-a557-40fc-bce4-728a59210793.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723606246331/f65f2c99-2746-420c-8c25-6b0193121ea2.png align="center")

* 평균 TPS: 기존 http 99.6 → gRPC 169.2, 70% 증가
    
* 평균 테스트 시간(ms): 4,960 ms → 2,939 ms, 40% 감소
    

그 외에도 CPU, 응답시간, open files, 트래픽 관련해서 좋은 성능 향상이 관찰되었다!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723606744802/ccd64fb1-a380-49ec-9e28-45b4be799339.png align="center")

## 6\. gRPC의 단점 👎

물론 gRPC에도 몇 가지 단점이 있다:

1. **학습 곡선**: Protocol Buffers와 gRPC 개념을 이해하는 데 시간이 필요할 수 있다. (하지만 대체적으로 낮은 편이다)
    
2. **브라우저 지원 제한**: 웹 브라우저에서 직접 사용하기 어렵다.
    
3. **디버깅의 어려움**: 바이너리 형식으로 인해 가독성이 떨어질 수 있다.
    
4. **동적 타입 언어에서의 이점 감소**: 정적 타입 언어에 비해 장점이 줄어들 수 있다.
    
5. **서비스 검색 메커니즘 부재**: 별도의 구현이 필요하다.
    

## 7\. 결론 🤩

gRPC는 MSA 환경에서 서비스 간 효율적인 통신을 위한 강력한 도구입니다. 높은 성능, 강력한 타입 안정성, 다중 언어 지원 등의 장점으로 인해 많은 기업과 개발자들이 채택하고 있다.

하지만 학습 곡선과 브라우저 지원 제한 등의 단점도 있다. 따라서 프로젝트의 요구사항과 개발 팀의 역량을 고려하여 gRPC 도입을 결정해야 합니다.

MSA의 확산과 함께 gRPC의 중요성은 계속 커질 전망입니다. 특히 클라우드 네이티브 환경과 IoT 분야에서의 성장이 기대되므로, 개발자들은 gRPC에 대한 이해와 활용 능력을 키우는 것이 좋을 것 같다.

gRPC는 현대적인 서비스 간 통신의 핵심 기술로 자리잡아가고 있으며, MSA 시대에 더욱 빛을 발할 것으로 보입니다. 기존 레거시 프로젝트에 gRPC를 도입하여 성능 향상을 이루어내보자.