---
description: Reactor 스레드 동작 탐구
---

# Reactor execution model 2

### Reactor Threading and Schedulers

Reactor의 개요에서도 이미 확인했듯이 Reactor의 실행 모델을 이해하는 것의 핵심은 내부적인 스레드 동작에 대한 파악에 있다고 생각한다.

마침 이에 대해서 정리가 잘된 글([https://alegrucoding.com/reactor-execution-model-threading-and-schedulers/](https://alegrucoding.com/reactor-execution-model-threading-and-schedulers/))을 발견하여 이를 보면서 이론적인 정리를 하고, 카카오 Tech 에 올라온 Reactor 관련 포스팅([https://tech.kakao.com/2018/05/29/reactor-programming/](https://tech.kakao.com/2018/05/29/reactor-programming/))을 통해서 실습하며 이론을 확인해본다.





### 기본적으로 subscribe()한 thread가 whole pipeline execution을 수행한다

위 소제목이 핵심적인 원리이다. 아래 코드를 보자.

```java
class ReactiveJavaTutorial {

  public static void main(String[] args) {

    Flux<String> cities = Flux.just("New York", "London", "Paris", "Amsterdam")
            .map(String::toUpperCase)
            .filter(cityName -> cityName.length() <= 8)
            .map(cityName -> cityName.concat(" City"))
            .log();

    cities.subscribe();

  }
}
```

```bash
INFO 14040 --- [main] reactor.Flux.MapFuseable.1  : | onSubscribe([Fuseable] FluxMapFuseable.MapFuseableSubscriber)
INFO 14040 --- [main] reactor.Flux.MapFuseable.1  : | request(unbounded)
INFO 14040 --- [main] reactor.Flux.MapFuseable.1  : | onNext(NEW YORK City)
INFO 14040 --- [main] reactor.Flux.MapFuseable.1  : | onNext(LONDON City)
INFO 14040 --- [main] reactor.Flux.MapFuseable.1  : | onNext(PARIS City)
INFO 14040 --- [main] reactor.Flux.MapFuseable.1  : | onComplete()
```

위 예제코드를 보면 main thread가 구독부터 발행까지 전체 파이프라인을 책임지고 수행하고 있다.

_**"The same**_** thread **_**that performs a subscription will be used for the whole pipeline execution."**_

이미 앞서 살펴 보았듯이 비동기, 논블로킹의 동작을 통해 얻을 수 있는 장점을 고려해본다면, 위와 같이 하나의 스레드가 모든 것을 처리하는 경우에는 성능에 이점이 없다.

이러한 구독의 흐름에서 특정한 Flux 또는 Mono에 대해 처음 구독을 시작한 thread가 아니라 이를 처리할  thread를 따로 분기하여 담당하게 만들면 수행 속도가 더 빨라질 수 있다. 이 때 특정한 pool을 지정해주고 subscribe 혹은 publish시 thread를 거기서 꺼내오게 지정해줄 수 있다.

이 thread pool에 대해서 Reactor 에서는 Schdulers 라는 Factory 클래스를 제공한다. 이를 이용하면 Flux 또는 Mono 수행에 사용되는 thread 를 switch 할 수 있다. 아래는 Schedulers의 종류들이다.



* **Schedulers.parallel()** – It has a fixed pool of workers. The number of threads is equivalent to the number of CPU cores.
* **Schedulers.boundElastic()** – It has a bounded elastic thread pool of workers. The number of threads can grow based on the need. The number of threads can be much bigger than the number of CPU cores. \
  Used mainly for making blocking IO calls.
* **Schedulers.single()** –  Reuses the same thread for all callers.

그리고 아래 method 들을 이용해서 Reactor 가 어떤 Schedulers 들을 쓸지를 명령 할 수 있다.

* The **publishOn** method
* The **subscribeOn** method





### subscribeOn() vs publishOn()



#### subscribeOn()

```java
class ReactiveJavaTutorial {

  public static void main(String[] args) {

    Flux<String> cities = Flux.just("New York", "London", "Paris", "Amsterdam")
            .subscribeOn(Schedulers.boundedElastic())
            .map(String::toUpperCase)
            .filter(cityName -> cityName.length() <= 8)
            .map(cityName -> cityName.concat(" City"))
            .log();

    cities.subscribe();

  }
}

```

```bash
Output:
INFO 7500 --- [main] reactor.Flux.Map.1  : onSubscribe(FluxMap.MapSubscriber)
INFO 7500 --- [main] reactor.Flux.Map.1  : request(unbounded)
INFO 7500 --- [boundedElastic-1] reactor.Flux.Map.1  : onNext(NEW YORK City)
INFO 7500 --- [boundedElastic-1] reactor.Flux.Map.1  : onNext(LONDON City)
INFO 7500 --- [boundedElastic-1] reactor.Flux.Map.1  : onNext(PARIS City)
INFO 7500 --- [boundedElastic-1] reactor.Flux.Map.1  : onComplete()
```

위와 같이 subscribe()를 하고 onSubscribe() 를 수행하며 Subscription을 넘기면서 request()를 수행하는 것 까지 구독을 시작한 thread인 main 이 수행하고, 그 이후부터는 다른 thread가 처리를 담당한 것을 확인할 수 있다.



#### publishOn()

```java
class ReactiveJavaTutorial {

  public static void main(String[] args) {

    Flux.just("New York", "London", "Paris", "Amsterdam")
            .map(ReactiveJavaTutorial::stringToUpperCase)
            .publishOn(Schedulers.boundedElastic())
            .map(ReactiveJavaTutorial::concat)
            .subscribe();
  }

  private static String stringToUpperCase(String name) {
    System.out.println("stringToUpperCase: " + Thread.currentThread().getName());
    return name.toUpperCase();
  }

  private static String concat(String name) {
    System.out.println("concat: " + Thread.currentThread().getName());
    return name.concat(" City");
  }
}
```

```bash
stringToUpperCase: main
stringToUpperCase: main
stringToUpperCase: main
concat: boundedElastic-1
concat: boundedElastic-1
concat: boundedElastic-1
```

publishOn()을 만나기 직전까지는 구독을 시작한 thread에 의해서 처리되다가, publishOn을 만난 직후부터 처리를 담당해주는 thread 가 변경된 것을 확인할 수 있다.

publishOn()은 operator와 유사하게 수행이 된다고 보면 된다. up-stream과 down-stream 사이에 존재하면서 명시된 Schedulers의 pool 에서 thread를 꺼내와 이를 처리하도록 thread 를 switch 해준다.

subscribeOn()과 publishOn()의 가장 큰 차이점은 publishOn()은 operator 처럼 pipe line 어디에든 원하는 곳에 넣어서 사용할 수 있고, subscribeOn()은 어디에 위치하든 whole reactive chain 에 적용이 된다는 것이다.

"That is the main difference between the **subscribeOn** and **publishOn** operators since the **subscribeOn** will apply the provided Scheduler to the whole reactive chain, no matter where we placed it."









* 웹클라이언트, repository 의 경우 내부적으로 실제로 딜레이 발생시켜서 스레드 갈아타게 만드는지 살펴보기.
* 한다면 응답이 와서 push 수행을 하는 스레드는 애초에 요청을 한 동일한 스레드인가?
