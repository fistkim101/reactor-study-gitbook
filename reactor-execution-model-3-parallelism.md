# Reactor execution model 3 - parallelism

### Parallelism using parallel() and runOn() operator

```java
public class PararellTest {

    @Test
    void pararellTest_1() {
        Flux<Integer> numbers = Flux.range(1, 10)
                .map(this::stop1Second)
                .log();

        StepVerifier.create(numbers)
                .expectNextCount(10)
                .verifyComplete();
    }

    @Test
    void pararellTest_2() {
        Flux<Integer> numbers = Flux.range(1, 10)
                .publishOn(Schedulers.parallel())
                .map(this::stop1Second)
                .log();

        StepVerifier.create(numbers)
                .expectNextCount(10)
                .verifyComplete();
    }

    @Test
    void pararellTest_3() {
        int availableProcessors = Runtime.getRuntime().availableProcessors();
        System.out.println("availableProcessors : " + availableProcessors);
        
        ParallelFlux<Integer> numbers = Flux.range(1, 10)
                .parallel()
                .runOn(Schedulers.parallel())
                .map(this::stop1Second)
                .log();

        StepVerifier.create(numbers)
                .expectNextCount(10)
                .verifyComplete();
    }

    private Integer stop1Second(int value) {
        try {
            System.out.println("current value : " + value);
            Thread.sleep(1000L);
            return value;
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

}

```

이전에 publishOn()과 subscribeOn()으로 작업 스레드를 제어하여 원하는 작업을 병렬 수행하는 방법에 대해서 실습을 해보았는데, 강의에서 소개하는 또 다른 방식으로 pararell을 이용해서 로직을 병렬처리하는 방식을 소개하고 있었다.

pararellTest\_1, pararellTest\_2는 모두 10초 이상이 걸리며 pararellTest\_3은 모든 작업을 수행하는데 1초 남짓이 걸린다. 참고로 내가 작업한 컴퓨터의 코어 수는 10개라서 availableProcessors 가 10이다.





### Parallelism using flatMap() operator

```java
    @Test
    void pararellTest_4() {
        Flux<Integer> numbers = Flux.range(1, 10)
                .flatMap(number -> Mono.just(number)
                        .map(this::stop1Second)
                        .subscribeOn(Schedulers.parallel()))
                .log();

        StepVerifier.create(numbers)
                .expectNextCount(10)
                .verifyComplete();
    }
```

flatMap()은 inner-publisher 들을 eagarly 하게 모두 subscribe()해서 각기 다른 lifecycle에 따라 도착하는 순서대로 down-stream 으로 보내주는 operator 이다. 위 코드는 이를 이용해서 병렬 수행하는 예제코드이다. 위에서 다룬  pararellTest\_3 과 결과는 비슷하지만 flatMap()을 이용했다는 측면에서 방식이 다르다고 할 수 있다.

여기서 up-stream의 순서를 그대로 유지하여 down-stream 으로 보내주고 싶으면 flatMap을 flatMapSequantial()로 바꿔주기만 하면 된다.
