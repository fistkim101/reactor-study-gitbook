---
description: doOn~ 종류의 operator 들을 통칭하여 Side Effect Method 라고 부른다. 이를 알아본다.
---

# Side Effect Methods

side effect method(operator) 들은 Publisher에 의해서 emit 되는 모든 event 들에 대해서 기존의 up-stream sequence에 영향을 주지 않으면서 이를 들여다 볼 수 있도록 해준다.

reactor에서는 여러가지 side effect method 들을 제공한다.

![](<.gitbook/assets/image (1).png>)

내가 잘못 알고 있었던 부분은 이러한 side effect method 들의 실행 순서이다. 왜 그렇게 생각하고 있었는지 이유는 정확히 모르겠지만 모든 doOn\~ operator 들이 target event의 직후에 발생한다고 생각했었다. 하지만 오히려 반대로 target event 바로 직전에 호출되는 것이 일반적인 흐름이었다.

단어를 봐도 사실 do on 이라는 의미가 '실행한다. 붙어서. \~에' 이기 때문에 순서상 target event 앞이 맞다. (cf. doOnComplete 은 onComplete() 직후 실행된다)

```java
    @Test
    void sideEffect() {
        Flux<Integer> numbers = Flux.fromIterable(List.of(1, 2, 3))
                .doOnEach(number -> System.out.println("doOnEach : " + number))
                .doOnNext(number -> System.out.println("doOnNext : " + number))
                .doOnSubscribe(number -> System.out.println("doOnSubscribe : " + number))
                .log()
                .doOnComplete(() -> System.out.println("doOnComplete"));

        StepVerifier.create(numbers)
                .assertNext(number -> {
                    Assertions.assertEquals(1, number);
                })
                .assertNext(number -> {
                    Assertions.assertEquals(2, number);
                })
                .assertNext(number -> {
                    Assertions.assertEquals(3, number);
                })
                .verifyComplete();
    }
```

```bash
10:55:49.883 [Test worker] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
doOnSubscribe : reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber@2b5f4d54
10:55:49.921 [Test worker] INFO reactor.Flux.PeekFuseable.1 - | onSubscribe([Fuseable] FluxPeekFuseable.PeekFuseableSubscriber)
10:55:49.927 [Test worker] INFO reactor.Flux.PeekFuseable.1 - | request(unbounded)
doOnEach : doOnEach_onNext(1)
doOnNext : 1
10:55:49.928 [Test worker] INFO reactor.Flux.PeekFuseable.1 - | onNext(1)
doOnEach : doOnEach_onNext(2)
doOnNext : 2
10:55:49.933 [Test worker] INFO reactor.Flux.PeekFuseable.1 - | onNext(2)
doOnEach : doOnEach_onNext(3)
doOnNext : 3
10:55:49.934 [Test worker] INFO reactor.Flux.PeekFuseable.1 - | onNext(3)
doOnEach : onComplete()
10:55:49.935 [Test worker] INFO reactor.Flux.PeekFuseable.1 - | onComplete()
doOnComplete

```

강의 자료가 좀 오래된 것이다 보니 강의에서는 소개 되었지만 실습환경의 reactor 버전에는 없는 것들도 있었다. doOnEach는 굳이 쓸 일이 있을까 싶다.

