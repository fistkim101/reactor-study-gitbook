---
description: 조합에 관한 연산자 정리
---

# Combine

### Flux.concat vs Flux.concatWith

![](<.gitbook/assets/image (4).png>)

![](<.gitbook/assets/image (3).png>)

static 으로 제공되는 함수다. concatWith은 그걸 사용하는 publisher 에 parameter로 받는 publisher를 이어 붙여서 하나의 down-stream을 만들어 주 반면에 concat 은 이어 붙이기위한 여러 element 들을 가변인자로 받아 줄 수 있다. 또 재미있는 점은 concat의 경우 down-stream 을 구성할 element type 에 대해 조금 더 자유롭다는 것이다.

```java
    @Test
    void concatWithTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(4, 5, 6));

        StepVerifier.create(numbers_1.concatWith(numbers_2).log())
                .expectNext(1, 2, 3, 4, 5, 6)
                .verifyComplete();
    }

    @Test
    void concatTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(4, 5, 6));

        StepVerifier.create(Flux.concat(numbers_1, numbers_2, Mono.just("string")).log())
                .expectNext(1, 2, 3, 4, 5, 6, "string")
                .verifyComplete();
    }

```

