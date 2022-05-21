# retry, repeat

### **retry()**

* Use this operator to retry failed exceptions
* When to use it?
  * Code interacts with external systems through network
    * Examples are : RestFul API calls, DB Calls
  * these calls may fail intermittently



![](<.gitbook/assets/스크린샷 2022-05-21 오후 9.44.02.png>)

에러가 발생하면 무한히 다시 subscribe()를 시도한다. onComplete() 을 받지 못했다면 끝없이 subscribe()를 다시 시도한다.

```java
    @Test
    void retryTest() throws InterruptedException {
        AtomicInteger index = new AtomicInteger();
        Flux<Integer> numbersWithError = Flux.fromIterable(List.of(1, 2, 3))
                .concatWith(Mono.error(new RuntimeException()))
                .onErrorResume(exception -> {
                    if (index.get() == 5) {
                        System.out.println("index equals 5");
                        return Mono.just(10);
                    } else {
                        System.out.println("index < 5");
                        return Mono.error(new RuntimeException());
                    }
                })
                .doOnError(ex -> {
                    index.getAndIncrement();
                })
                .retry()
                .log();

        numbersWithError.subscribe();
        Thread.sleep(10000L);
    }
```

```bash
22:04:52.564 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
22:04:52.602 [main] INFO reactor.Flux.Retry.1 - onSubscribe(FluxRetry.RetrySubscriber)
22:04:52.606 [main] INFO reactor.Flux.Retry.1 - request(unbounded)
22:04:52.607 [main] INFO reactor.Flux.Retry.1 - onNext(1)
22:04:52.607 [main] INFO reactor.Flux.Retry.1 - onNext(2)
22:04:52.607 [main] INFO reactor.Flux.Retry.1 - onNext(3)
index < 5
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(1)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(2)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(3)
index < 5
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(1)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(2)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(3)
index < 5
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(1)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(2)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(3)
index < 5
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(1)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(2)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(3)
index < 5
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(1)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(2)
22:04:52.608 [main] INFO reactor.Flux.Retry.1 - onNext(3)
index equals 5
22:04:52.609 [main] INFO reactor.Flux.Retry.1 - onNext(10)
22:04:52.609 [main] INFO reactor.Flux.Retry.1 - onComplete()

```



반면에 아래와 같이 parameter로 retry count를 넣어주면 정해준 count 만큼만 retry를 시도한다.

![](<.gitbook/assets/image (4).png>)



### retryWhen()

![](<.gitbook/assets/image (5).png>)

retryWhen은 공식 문서의 설명이 너무 길어서 마블 다이어그램만 발췌해 왔다. retry를 무작정 하지 않고 retrySpec에 의거하여 retry 를 한다는 것이 특징이다. 사실 실무에서는 retry 보다 retryWhen 을 쓸 가능성이 크다고 판단된다. 특히 정상적인 요청에 대해서 상대 서버가 간헐적으로 이상한 값을 내려준다면 이를 조건적으로 판단해서 retry 해주는 로직이 필요하므로 그 때 사용하면 좋다.



d
