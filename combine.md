---
description: 조합에 관한 연산자 정리
---

# Combine

### Flux.concat vs Flux.concatWith

![](<.gitbook/assets/image (7).png>)

![](<.gitbook/assets/image (5).png>)

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





### Flux.merge vs Flux.mergeWith

![](<.gitbook/assets/image (4).png>)

![](.gitbook/assets/image.png)

기본적으로 merge, mergeWith은 복수의 publisher 에 대해서 이를 합쳐서 하나의 down-stream으로 제공하되 이를 병렬처한다는 특징이 있다. flatMap과 유사하지만 element에 변형을 가하지 않는다는 점이 다르다고 생각하면 될 것 같다.

merge, mergeWith의 차이는 concat과 concatWith과 유사하게 static 이면서 가변인자로 parameter 들을 받아서 merge 해주는 것과 특정 publisher 와 parameter로 받은 특정 publisher 와 merge 하는 차이가 있었다.

```java
    @Test
    void mergeTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3)).delayElements(Duration.ofMillis(100));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(4, 5, 6)).delayElements(Duration.ofMillis(110));

        Flux<Integer> merged = Flux.merge(numbers_1, numbers_2).log();

        StepVerifier.create(merged)
                .expectNext(1, 4, 2, 5, 3, 6)
                .verifyComplete();
    }

    @Test
    void mergeWithTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3)).delayElements(Duration.ofMillis(100));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(4, 5, 6)).delayElements(Duration.ofMillis(110));

        Flux<Integer> merged = numbers_1.mergeWith(numbers_2).log();

        StepVerifier.create(merged)
                .expectNext(1, 4, 2, 5, 3, 6)
                .verifyComplete();
    }
```



















































