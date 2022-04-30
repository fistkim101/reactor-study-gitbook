---
description: 변환에 관한 연산자 정리
---

# Transform

### Flux.flatMap

![](.gitbook/assets/image.png)

up-stream 이 emit 하는 각각의 element 들을 비동기적으로 모두 Publisher로 만들고, 이렇게 만들어진 multiple 한 Publisher 들을 모두 subscribe 해서 하나의 Flux로 merge 한 down-stream 을 반환한다.

여기서 up-stream의 element 들을 inner publisher 로 만드는 과정에서 각각의 lifecycle을 가진 여러 publisher 가 layer 처럼 쌓이고 이를 flatten 해서 하나의 single flow 로 merge 하기 때문에 "flat" + "map" 이라고 할 수 있다.

각각의 inner publisher 들은 독자적인 life cycle을 지니므로 flatMap 이 최종적으로 반환하는 down-stream인 하나의 flux는 최초 up-stream에 있던 순서를 보장하지 않는다.

parameter 로는 element 들을 inner publisher로 변환해줄 mapper function을 받는다.

```java
        Flux<Integer> numbers = Flux.fromIterable(List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10));
        numbers.flatMap(number -> {
                    int delay = new Random().nextInt(10);
                    return Mono.just(number + 1).delayElement(Duration.ofMillis(delay));
                })
                .log()
                .subscribe();
```





