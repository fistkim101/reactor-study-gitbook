---
description: Reactor execution model 이해를 바탕으로 실습해보기
---

# Reactor execution model 3

카카오Tech 에 이에 관해서 매우 정리가 잘된 글([https://tech.kakao.com/2018/05/29/reactor-programming/](https://tech.kakao.com/2018/05/29/reactor-programming/))을 발견해서 똑같이 실습해보면서 코드를 점점 개선해 나가보았다.

위 포스팅 전반에서 개선 포인트들을 짚어주고 설명해주는데, 이 과정 그대로 따라하면서 실제로 실행해서 로그를 관찰해보았다.



```java
public class FruitExample1 {
    public static void main(String[] args) {
        final List<String> basket1 = Arrays.asList(new String[]{"kiwi", "orange", "lemon", "orange", "lemon", "kiwi"});
        final List<String> basket2 = Arrays.asList(new String[]{"banana", "lemon", "lemon", "kiwi"});
        final List<String> basket3 = Arrays.asList(new String[]{"strawberry", "orange", "lemon", "grape", "strawberry"});
        final List<List<String>> baskets = Arrays.asList(basket1, basket2, basket3);
        final Flux<List<String>> basketFlux = Flux.fromIterable(baskets);

        basketFlux.concatMap(basket -> {
            final Mono<List<String>> distinctFruits = Flux.fromIterable(basket).distinct().collectList();
            final Mono<Map<String, Long>> countFruitsMono = Flux.fromIterable(basket)
                    .groupBy(fruit -> fruit) // 바구니로 부터 넘어온 과일 기준으로 group을 묶는다.
                    .concatMap(groupedFlux -> groupedFlux.count()
                            .map(count -> {
                                final Map<String, Long> fruitCount = new LinkedHashMap<>();
                                fruitCount.put(groupedFlux.key(), count);
                                return fruitCount;
                            }) // 각 과일별로 개수를 Map으로 리턴
                    ) // concatMap으로 순서보장
                    .reduce((accumulatedMap, currentMap) -> new LinkedHashMap<String, Long>() {
                        {
                            putAll(accumulatedMap);
                            putAll(currentMap);
                        }
                    }); // 그동안 누적된 accumulatedMap에 현재 넘어오는 currentMap을 합쳐서 새로운 Map을 만든다. // map끼리 putAll하여 하나의 Map으로 만든다.
            return Flux.zip(distinctFruits, countFruitsMono, (distinct, count) -> new FruitInfo(distinct, count));
        }).subscribe(System.out::println);
    }
}

```

위 코드는 각 basket의 과일의 종류를 distincti 처리해주고, 각 종류마다 개수를 파악해주는 일을 수행한다. 각각 distinctFruits, countFruitsMono가 그 역할을 하는데, 위 코드의 문제점은 두 가지이다.

* reactor의 장점인 비동기, 논블로킹이 전혀 쓰이지 않고 있다. 특별히 병렬처리를 지시하지 않았기 때문에 기본적인 execution 원칙에 의해서 구독을 시작한 thread가 모든 것을 처리할 것이기 때문이다.
* basketFlux가 emit하는 basket 을 distinctFruits, countFruitsMono 가 각각 순회한다. 어차피 basket 을 순회하며 무엇인가 처리를 할 것이라면 한번만 순회하면 되는데 두번씩 돌리는 것이다.(비효율)



먼저 아래와 같은 코드로 두 작업을 각각 비동기로 병렬 수행시켜서 로직을 개선할 수 있다.

```java
public class FruitExample2 {
    public static void main(String[] args) throws InterruptedException {
        final List<String> basket1 = Arrays.asList(new String[]{"kiwi", "orange", "lemon", "orange", "lemon", "kiwi"});
        final List<String> basket2 = Arrays.asList(new String[]{"banana", "lemon", "lemon", "kiwi"});
        final List<String> basket3 = Arrays.asList(new String[]{"strawberry", "orange", "lemon", "grape", "strawberry"});
        final List<List<String>> baskets = Arrays.asList(basket1, basket2, basket3);
        final Flux<List<String>> basketFlux = Flux.fromIterable(baskets);

        basketFlux.concatMap(basket -> {
            final Mono<List<String>> distinctFruits = Flux.fromIterable(basket).log().distinct().collectList().subscribeOn(Schedulers.parallel());
            final Mono<Map<String, Long>> countFruitsMono = Flux.fromIterable(basket).log().groupBy(fruit -> fruit) // 바구니로 부터 넘어온 과일 기준으로 group을 묶는다.
                    .concatMap(groupedFlux -> groupedFlux.count().map(count -> {
                                final Map<String, Long> fruitCount = new LinkedHashMap<>();
                                fruitCount.put(groupedFlux.key(), count);
                                return fruitCount;
                            }) // 각 과일별로 개수를 Map으로 리턴
                    ) // concatMap으로 순서보장
                    .reduce((accumulatedMap, currentMap) -> new LinkedHashMap<String, Long>() {
                        {
                            putAll(accumulatedMap);
                            putAll(currentMap);
                        }
                    }) // 그동안 누적된 accumulatedMap에 현재 넘어오는 currentMap을 합쳐서 새로운 Map을 만든다. // map끼리 putAll하여 하나의 Map으로 만든다.
                    .subscribeOn(Schedulers.parallel());
            return Flux.zip(distinctFruits, countFruitsMono, (distinct, count) -> new FruitInfo(distinct, count));
        }).subscribe(System.out::println);

        Thread.sleep(5000L);
    }

}

```

별달리 해준 것은 없고 각각의 distinctFruits, countFruitsMono 에 대해서 subscribeOn()을 통해 thread를 별도로 지정해주었다. 로그를 확인해보면 main thread가 아니라 parallel 에서 꺼내온 thread가 처리에 개입하고 있음을 확인할 수 있다.

Thread.sleep()을 해준 이유는, 초기 코드와는 달리 병렬 수행을 위해서 데몬 thread에게 처리를 지시함으로써 데몬 thread 가 백그라운드로 작업을 처리해주는데, 이와중에 main thread가 종료됨으로써 applicaion이 끝나버리기 때문이다.

이 부분에 대한 이해를 하는 과정에서 이 글([https://www.geeksforgeeks.org/difference-between-daemon-threads-and-user-threads-in-java/](https://www.geeksforgeeks.org/difference-between-daemon-threads-and-user-threads-in-java/)) 이 도움이 좀 되었다. 애초에 JVM 에서 thread의 구분을 두고 있고 각 thread 에 대해서 JVM이 인식하고 처리해주는 방식이 다르다. 여기서 적용되는 원칙은 아래와 같다.

**JVM doesn’t wait for daemon thread to finish but it waits for User Thread**



이제 비동기 문제는 개선을 해보았으니 동일한 source 에 대해서 두 번 순회하는 비효율을 거둬보자.

```java

public class FruitExample3 {

    public static void main(String[] args) throws InterruptedException {

        final List<String> basket1 = Arrays.asList(new String[]{"kiwi", "orange", "lemon", "orange", "lemon", "kiwi"});
        final List<String> basket2 = Arrays.asList(new String[]{"banana", "lemon", "lemon", "kiwi"});
        final List<String> basket3 = Arrays.asList(new String[]{"strawberry", "orange", "lemon", "grape", "strawberry"});
        final List<List<String>> baskets = Arrays.asList(basket1, basket2, basket3);
        final Flux<List<String>> basketFlux = Flux.fromIterable(baskets);


        basketFlux.concatMap(basket -> {
            final Flux<String> source = Flux.fromIterable(basket).log().publish().autoConnect(2);
            final Mono<List<String>> distinctFruits = source.distinct().collectList().log().subscribeOn(Schedulers.parallel());
            final Mono<Map<String, Long>> countFruitsMono = source
                    .groupBy(fruit -> fruit) // 바구니로 부터 넘어온 과일 기준으로 group을 묶는다.
                    .concatMap(groupedFlux -> groupedFlux.count()
                            .map(count -> {
                                final Map<String, Long> fruitCount = new LinkedHashMap<>();
                                fruitCount.put(groupedFlux.key(), count);
                                return fruitCount;
                            }) // 각 과일별로 개수를 Map으로 리턴
                    ) // concatMap으로 순서보장
                    .reduce((accumulatedMap, currentMap) -> new LinkedHashMap<String, Long>() {
                        {
                            putAll(accumulatedMap);
                            putAll(currentMap);
                        }
                    }) // 그동안 누적된 accumulatedMap에 현재 넘어오는 currentMap을 합쳐서 새로운 Map을 만든다. // map끼리 putAll하여 하나의 Map으로 만든다.
                    .log()
                    .subscribeOn(Schedulers.parallel());
            return Flux.zip(distinctFruits, countFruitsMono, (distinct, count) -> new FruitInfo(distinct, count));
        }).subscribe(e -> System.out.println(Thread.currentThread() + " || " + e.toString()));

        Thread.sleep(5000L);
    }

}
```

distinctFruits, countFruitsMono 두 작업은 데몬 thread를 할당해주어 병렬 수행하면서도 source를 hot으로 변경하여 한번만 순회하도록 처리된 코드이다.

즉, 이전에는 cold 로 관리가 되어 구독자가 구독을 시작하는 시점에 각각 스트림을 발생 시켰다면,  이번에는 hot으로 전환하여 특정 조건, 특정 시점이 되었을때 스트림을 발생시켜서 이를 구독하는 구독자들에게 동일한 데이터를 일괄적으로 흘려준 것이다. 이렇게 쓸데없이 동일한 스트림을 두 번 발생시키던 비효율을 한 번만 발생시키도록 개선할 수 있다.

hot과 cold 에 대해서는 udemy 강의를 듣고 더 정리할 생각이다.
