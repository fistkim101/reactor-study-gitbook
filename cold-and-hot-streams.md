# Cold & Hot Streams

### Cold Streams Overview

* Cold Stream is a type of Stream which emits the elements from beginning to end for every new subscription.
* ex. HTTP Cal with similar request, DB call with similar request



### Hot Streams Overview

* Data is emitted continuously
* Any new subscriber will only get he current state of the Reactive Stream
  * Type 1 : Waits for the first subscription from the subscriber and emits the data continuously
  * Type 2 : Emits the data continuously without the need for subscription
* ex. Stock Tickers - Emits stock updates continuously as the change,  Uber Driver Tracking - Emits the of the current position of the Driver Continuously.



### [공식문서 가이드](https://projectreactor.io/docs/core/release/reference/#reactor.hotCold)

> Hot publishers, on the other hand, do not depend on any number of subscribers. They might start publishing data right away and would continue doing so whenever a new `Subscriber` comes in (in which case, the subscriber would see only new elements emitted _after_ it subscribed). For hot publishers, _something_ does indeed happen before you subscribe.

* Hot publisher는 subscriber에 의존하지 않는다. subscriber 가 언제 붙어서 언제 구독을 하든 상관없다.
* subscribe전에 이미 publisher는 뭔가를 한다.



> Sometimes, you may want to not defer only some processing to the subscription time of one subscriber, but you might actually want for several of them to rendezvous and then trigger the subscription and data generation.

* 멀티 프로세싱을 해야할 때 어떤 것을 잠시 멈추게 한다던가 하지 않고 각자 알아서 병렬수행을 하고, 각각의 처리결과를 한데 모아서 재처리를 해야할 경우가 있다는 뜻.



> This is what `ConnectableFlux` is made for. Two main patterns are covered in the `Flux` API that return a `ConnectableFlux`: `publish` and `replay`.

* ConnectableFlux 를 쓰면 위와 같은 경우를 해결할 수 있다.



```java
package com.fistkim.reactorstudy.hotcold;

import org.junit.jupiter.api.Test;
import reactor.core.publisher.ConnectableFlux;
import reactor.core.publisher.Flux;
import reactor.test.StepVerifier;
import reactor.test.scheduler.VirtualTimeScheduler;

import java.time.Duration;

public class HotColdTest {

    @Test
    void hotTest_1() {
        Flux<Integer> numbers = Flux.range(1, 10)
                .delayElements(Duration.ofSeconds(1));
        ConnectableFlux<Integer> connectableNumbers = numbers.publish();
        connectableNumbers.connect();
        connectableNumbers.subscribe(num -> System.out.println("subscriber 1 : " + num));
        stopSeconds(5);
        connectableNumbers.subscribe(num -> System.out.println("subscriber 2 : " + num));

        stopSeconds(10);
    }

    @Test
    void autoConnectTest() {
        Flux<Integer> numbers = Flux.range(1, 10)
                .delayElements(Duration.ofSeconds(1));
        Flux<Integer> numbersFlux = numbers.publish().autoConnect(2);
        numbersFlux.subscribe(num -> System.out.println("subscriber 1 : " + num));
        stopSeconds(5);
        numbersFlux.subscribe(num -> System.out.println("subscriber 2 : " + num));

        stopSeconds(10);
    }

    @Test
    void refConnectTest() {
        Flux<Integer> numbers = Flux.range(1, 10)
                .delayElements(Duration.ofSeconds(1));
        Flux<Integer> numbersFlux = numbers.publish().refCount(2);
        var a = numbersFlux.subscribe(number -> System.out.println("subscriber 1 : " + number));
        var b = numbersFlux.subscribe(number -> System.out.println("subscriber 2 : " + number));
        a.dispose();
        b.dispose();

        stopSeconds(10);
    }

    @Test
    void virtualTimerTest() {
        VirtualTimeScheduler.getOrSet();
        Flux<Integer> numbers = Flux.range(1, 10)
                .delayElements(Duration.ofSeconds(1));

        StepVerifier.withVirtualTime(() -> numbers)
                .thenAwait(Duration.ofSeconds(12))
                .expectNextCount(10)
                .verifyComplete();
    }

    private void stopSeconds(Integer second) {
        try {
            Thread.sleep(Long.parseLong(second.toString()) * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```
