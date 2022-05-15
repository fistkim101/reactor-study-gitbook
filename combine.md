---
description: 조합에 관한 연산자 정리
---

# Combine

### Flux.concat vs Flux.concatWith

![](<.gitbook/assets/image (7) (1).png>)

![](<.gitbook/assets/image (5) (1).png>)

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

![](<.gitbook/assets/image (4) (1).png>)

![](<.gitbook/assets/image (1) (1).png>)

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





### Flux.mergeSequantial

![](<.gitbook/assets/image (5).png>)

static 함수로 가변인자로 받은 source들을 eagerly 하게 subsribe() 하지만 최종적으로 반환해주는 down-stream은 호출을 시작한 순서대로 조합하여 구성해준다.

flatMapSequantial() 은 element 들을 변형하여 호출 순서에 맞게 down-stream을 구성하는데 여기서 element 들을 변형하는 것을 빼면 mergeSequantial() 이라고 할 수 있을 것 같다.

```java
    @Test
    void mergeSequentialTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3)).delayElements(Duration.ofMillis(100));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(4, 5, 6)).delayElements(Duration.ofMillis(110));

        Flux<Integer> merged = Flux.mergeSequential(numbers_1, numbers_2).log();

        StepVerifier.create(merged)
                .expectNext(1, 2, 3, 4, 5, 6)
                .verifyComplete();
    }
```





### Flux.zip vs Flux.zipWith

![](<.gitbook/assets/image (2) (1).png>)

zip 은 두 publisher 를 하나로 묶어줄 때 사용한다. concat은 단순히 publisher 를 이어 붙여줬고, merge 가 병렬적으로 subscribe 하여 하나의 down-stream 으로 합쳐줬다면 zip은 publisher 들을 조합함에 있어서 좀더 세밀하게 이를 컨트롤하여 down-stream 자체가 이미 원하는 처리가 되어서 나오도록 구현이 가능하다.

```java
    @Test
    void zipSameCountTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(4, 5, 6));

        Flux<Integer> ziped = Flux.zip(numbers_1, numbers_2, (num1, num2) -> num1 + num2).log();
        StepVerifier.create(ziped)
                .expectNext(5, 7, 9)
                .verifyComplete();
    }

    @Test
    void zipNotSameCountTest() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 1, 1));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(1, 1));

        Flux<Integer> ziped = Flux.zip(numbers_1, numbers_2, (num1, num2) -> num1 + num2).log();
        StepVerifier.create(ziped)
                .expectNext(2, 2)
                .verifyComplete();
    }

```



위 예시는 합쳐줌과 동시에 원하는 형태로 변형까지 한 것인데, 적어도 나의 경험상 실무에서는 단순히 tuple 형태로 반환하도록 사용하는 방법을 더 많이 썼던 것 같다.(내가 못해서 그런가!?)

지금 생각해보니 함수에서 의도를 잘 드러낼 수 있다면 tuple로 가져와서 밑에서 원하는 로직을 넣는 것 보다는 위 예시처럼 함수 자체를 parameter로 넣어줘서 zip operator 가 자체적으로 개발자가 원하는 변형 로직까지 수행하도록 하는 것도 괜찮겠다.



아래 document와 예제코드는 단순히 결합만 해주는 zip 활용에 대한 내용이다.

![](<.gitbook/assets/image (6) (1).png>)

```java
    @Test
    void simpleZipTest() {
        Flux<String> names = Flux.fromIterable(List.of("leo", "siri", "jbl"));
        Flux<String> capital = Flux.fromIterable(List.of("A", "B"));

        Flux<String> ziped = Flux.zip(names, capital)
                .map(t2 -> t2.getT1() + t2.getT2())
                .log();
        StepVerifier.create(ziped)
                .expectNext("leoA", "siriB")
                .verifyComplete();
    }
```



zipWith 도 이미 concat과 merge에서 다뤘던 내용과 유사하다. zip 이 static 메소드라면 zipWith은 publisher 의 메소드로 특정 publisher 를 parameter로 받아서 tuple형태로 down-stream 에 내려줄 수 있고, 람다로 함수를 parameter로 더 받아서 변형까지 가능하다.

![](<.gitbook/assets/image (3) (1).png>)

```java
    @Test
    void zipWithTest() {
        Flux<String> names = Flux.fromIterable(List.of("leo", "siri", "jbl"));
        Flux<String> capital = Flux.fromIterable(List.of("A", "B"));

        Flux<String> ziped = names.zipWith(capital, (str1, str2) -> str1 + str2).log();

        StepVerifier.create(ziped)
                .expectNext("leoA", "siriB")
                .verifyComplete();
    }

```



zip, zipWith 이 내부적으로 구독되는 방식에 대해서 정확하게 어떻게 작동하는지 의문이 많았는데(아직 레퍼런스를 찾지는 못했음) 마블 다이어그램 그대로로 일단 이해하면 될 것 같다.

zip 이 되는 두 Flux 가 있을때 이것이 모두다 쭉 구독이 일단 완료되고(병렬적으로 둘을 동시에) 내부적으로 이를 결합 해주 는 것으로 보인다. 왜냐하면 마블 다이어그램 자체가 둘 다 complete 까지 도달하는 것을 보면 알 수 있다. 따라서 개수를 맞춰주기 위해서 하나 emit하고 다른 대상 Flux의 emit을 기다리는 것이 아니라 일단 쭉 구독은 구독대로 진행하고, 결합은 결합대로 수가 맞을때 맞춰서 emit 해주는 것으로 보인다.
