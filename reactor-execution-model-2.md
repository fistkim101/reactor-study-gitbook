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



이 부분에 대해서 그림으로 너무 잘 설명해놓은 자료가 있어 첨부한다. 모든 그림은 이 유투브([https://www.youtube.com/watch?v=hfupNIxzNP4\&t=2194s](https://www.youtube.com/watch?v=hfupNIxzNP4\&t=2194s))에서 가져왔다. 20분 40초 부터 해당 부분에 관련된 내용이 나온다.

![](<.gitbook/assets/image (4).png>)

보면 subscribe() 를 시작한 thread가 op1, op2 까지 처리를 하고 publishOn 이후부터 thread 가 바뀌는 것을 볼 수 있다.





![](<.gitbook/assets/image (5).png>)

subscribeOn 은 동작하는 것이 약간 다르다. 위 publishOn과는 달리 숫자의 순서가 subscribe가 1이 아닌 것을 알 수 있다. Flux를 구성할때 안에 subscribeOn 이 있으면 무조건 subscribe 를 시작할때 subscribeOn이 지정해준 thread 로 시작을 한다는 의미이다.

그래서 subscribeOn은 reactive chain 어디에 있든지 간에 전체 pipe에 영향을 줄 수 밖에 없는 것이다.





![](<.gitbook/assets/image (6).png>)

이건 subscribeOn 과 publishOn이 섞인 경우이다. 이거 그림의 숫자가 좀 잘못된 것 같은데 영상에서는 이렇게 나오긴 했다. 아무튼 subscribeOn 이 op1의 앞에 있든, op2의 뒤에 있든, op1과 op2의 중간에 있든 publishOn을 만나기 전까지의 chain 전체에는 subscribeOn에서 명시한 thread로 동작하고 publishOn이후는 publishOn에서 명시한 thread가 처리를 하며 subscribe 내부의 로직은 결국 구독이 되어 나온 시점이므로 publishOn에서 명시한 thread가 이를 처리하는게 맞다.



publishOn() 활용 예제

```java
public class ExecutionModelTest {

    @Test
    void publishOnTest() {

        Flux<String> alphabet1 = Flux.fromIterable(List.of("a", "b", "c"))
                .map(this::toUpperCase);
        Flux<String> alphabet2 = Flux.fromIterable(List.of("d", "e", "f"))
                .map(this::toUpperCase);

        StepVerifier.create(alphabet1.mergeWith(alphabet2).log())
                .expectNextCount(6)
                .verifyComplete();
    }

    @Test
    void publishOnTestAsync() {

        Flux<String> alphabet1 = Flux.fromIterable(List.of("a", "b", "c"))
                .publishOn(Schedulers.parallel())
                .map(this::toUpperCase);
        Flux<String> alphabet2 = Flux.fromIterable(List.of("d", "e", "f"))
                .publishOn(Schedulers.parallel())
                .map(this::toUpperCase);

        StepVerifier.create(alphabet1.mergeWith(alphabet2).log())
                .expectNextCount(6)
                .verifyComplete();
    }

    private String toUpperCase(String value) {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return value.toUpperCase();
    }

}
```

```bash
/Users/jungkwonkim/Library/Java/JavaVirtualMachines/corretto-11.0.15/Contents/Home/bin/java -ea -Didea.test.cyclic.buffer.size=1048576 -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=54498:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-launcher/1.8.2/junit-platform-launcher-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-engine/1.8.2/junit-platform-engine-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-commons/1.8.2/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/apiguardian/apiguardian-api/1.1.2/apiguardian-api-1.1.2.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit5-rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit-rt.jar:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/test:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/main:/Users/jungkwonkim/Lab/App/reactor-study/build/resources/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-test/3.4.0/ad22a711534639a5f34dd696594c7a1ade5f8e83/reactor-test-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-core/3.4.0/683c9c676c438e5945b0f12808575f22160c5e54/reactor-core-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-test/2.6.7/8fe7761bc8609b832d8cc1104ff59a0512ef9aaf/spring-boot-starter-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter/2.6.7/59bd62cd60e6e7bb2a525931947c13548b207288/spring-boot-starter-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.7.0-Beta/836df62ad6bfc8a2ed7d4ba59a49bf2d251a75b5/kotlin-stdlib-jdk8-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter/5.5.1/4e3393c2238d1cf48d29b3c79f0c0928e873bc11/junit-jupiter-5.5.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.reactivestreams/reactive-streams/1.0.3/d9fb7a7926ffa635b3dcaa5049fb2bfa25b3e7d0/reactive-streams-1.0.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test-autoconfigure/2.6.7/b95d7d094604e315cbd0c7746d700acdddeee95d/spring-boot-test-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test/2.6.7/78741985ff50aecc5cd2776c74804a288ffc208d/spring-boot-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-test/5.3.19/84d0f06bfe2433173880240808203b69bc40244/spring-test-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-core/5.3.19/344ff3b291d7fdfdb08e865f26238a6caa86acc5/spring-core-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.jayway.jsonpath/json-path/2.6.0/67f565b424f7903a12d4f5b9361b11462ecacdac/json-path-2.6.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.xml.bind/jakarta.xml.bind-api/2.3.3/48e3b9cfc10752fba3521d6511f4165bea951801/jakarta.xml.bind-api-2.3.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.assertj/assertj-core/3.21.0/27a14d6d22c4e3d58f799fb2a5ca8eaf53e6942a/assertj-core-3.21.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.hamcrest/hamcrest/2.2/1820c0968dba3a11a1b30669bb1f01978a91dedc/hamcrest-2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-junit-jupiter/4.0.0/b76de25bd6e5d8f7924d0536729c0076e37e9396/mockito-junit-jupiter-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-core/4.0.0/f5195e0c4a45716bbd2d1d29173adbd148acce3a/mockito-core-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.skyscreamer/jsonassert/1.5.0/6c9d5fe2f59da598d9aefc1cfc6528ff3cf32df3/jsonassert-1.5.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.xmlunit/xmlunit-core/2.8.4/35be57989ca80eefa03161b211630e319a8f36c6/xmlunit-core-2.8.4.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-autoconfigure/2.6.7/d96e51e7a13f16c07f60654071f157f99a0b99be/spring-boot-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot/2.6.7/aef2fff50d49feb43a0726ab7e4945914186e06/spring-boot-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-logging/2.6.7/b49b159bb93086c76fa42555ca074c7ebe9d5fe6/spring-boot-starter-logging-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.annotation/jakarta.annotation-api/1.3.5/59eb84ee0d616332ff44aba065f3888cf002cd2d/jakarta.annotation-api-1.3.5.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.yaml/snakeyaml/1.29/6d0cdafb2010f1297e574656551d7145240f6e25/snakeyaml-1.29.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.7.0-Beta/ab6a690623c06efecd79fd7774de41a29c792d9d/kotlin-stdlib-jdk7-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.7.0-Beta/fb95185efb95996ea0f51dd7aa3e746f5f697236/kotlin-stdlib-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-params/5.8.2/ddeafe92fc263f895bfb73ffeca7fd56e23c2cce/junit-jupiter-params-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-api/5.8.2/4c21029217adf07e4c0d0c5e192b6bf610c94bdc/junit-jupiter-api-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-jcl/5.3.19/eae3d8b8728133782d9e41f9b3ecbd19a4146114/spring-jcl-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/json-smart/2.4.8/7c62f5f72ab05eb54d40e2abf0360a2fe9ea477f/json-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/slf4j-api/1.7.36/6c62681a2f655b49963a5983b8b0950a6120ae14/slf4j-api-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.activation/jakarta.activation-api/1.2.2/99f53adba383cb1bf7c3862844488574b559621f/jakarta.activation-api-1.2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy/1.11.22/8b4c7fa5562a09da1c2a9ab0873cb51f5034d83f/byte-buddy-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy-agent/1.11.22/2fbcf3210dfc09b42242e3b66a5281cc5b9adb80/byte-buddy-agent-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.vaadin.external.google/android-json/0.0.20131108.vaadin1/fa26d351fe62a6a17f5cda1287c1c6110dec413f/android-json-0.0.20131108.vaadin1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-context/5.3.19/d663259767f8fb66229209db0422d65978235525/spring-context-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-classic/1.2.11/4741689214e9d1e8408b206506cbe76d1c6a7d60/logback-classic-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-to-slf4j/2.17.2/17dd0fae2747d9a28c67bc9534108823d2376b46/log4j-to-slf4j-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/jul-to-slf4j/1.7.36/ed46d81cef9c412a88caef405b58f93a678ff2ca/jul-to-slf4j-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.7.0-Beta/5339108aef3a69a5030701ff3ebe9b811dfc54ac/kotlin-stdlib-common-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apiguardian/apiguardian-api/1.1.2/a231e0d844d2721b0fa1b238006d15c6ded6842a/apiguardian-api-1.1.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-commons/1.8.2/32c8b8617c1342376fd5af2053da6410d8866861/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.opentest4j/opentest4j/1.2.0/28c11eb91f9b6d8e200631d46e20a7f407f2a046/opentest4j-1.2.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/accessors-smart/2.4.8/6e1bee5a530caba91893604d6ab41d0edcecca9a/accessors-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-aop/5.3.19/339b0ad295a7af51ac716bd89d36a4fb0febaf5d/spring-aop-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-beans/5.3.19/4bc68c392ed320c9ab5dc439d7f2deb83f03fe76/spring-beans-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-expression/5.3.19/14346b7b84721f61d2b23d3c8baa60c6655527a6/spring-expression-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-core/1.2.11/a01230df5ca5c34540cdaa3ad5efb012f1f1f792/logback-core-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-api/2.17.2/f42d6afa111b4dec5d2aea0fe2197240749a4ea6/log4j-api-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.ow2.asm/asm/9.1/a99500cf6eea30535eeac6be73899d048f8d12a8/asm-9.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-engine/5.8.2/c598b4328d2f397194d11df3b1648d68d7d990e3/junit-jupiter-engine-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.objenesis/objenesis/3.2/7fadf57620c8b8abdf7519533e5527367cb51f09/objenesis-3.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-engine/1.8.2/b737de09f19864bd136805c84df7999a142fec29/junit-platform-engine-1.8.2.jar com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit5 com.fistkim.reactorstudy.execution.ExecutionModelTest,publishOnTest
07:06:44.856 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
07:06:44.889 [main] INFO reactor.Flux.Merge.1 - onSubscribe(FluxFlatMap.FlatMapMain)
07:06:44.895 [main] INFO reactor.Flux.Merge.1 - request(unbounded)
07:06:45.906 [main] INFO reactor.Flux.Merge.1 - onNext(A)
07:06:46.912 [main] INFO reactor.Flux.Merge.1 - onNext(B)
07:06:47.915 [main] INFO reactor.Flux.Merge.1 - onNext(C)
07:06:48.921 [main] INFO reactor.Flux.Merge.1 - onNext(D)
07:06:49.924 [main] INFO reactor.Flux.Merge.1 - onNext(E)
07:06:50.930 [main] INFO reactor.Flux.Merge.1 - onNext(F)
07:06:50.933 [main] INFO reactor.Flux.Merge.1 - onComplete()

Process finished with exit code 0

```

```bash
/Users/jungkwonkim/Library/Java/JavaVirtualMachines/corretto-11.0.15/Contents/Home/bin/java -ea -Didea.test.cyclic.buffer.size=1048576 -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=54501:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-launcher/1.8.2/junit-platform-launcher-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-engine/1.8.2/junit-platform-engine-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-commons/1.8.2/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/apiguardian/apiguardian-api/1.1.2/apiguardian-api-1.1.2.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit5-rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit-rt.jar:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/test:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/main:/Users/jungkwonkim/Lab/App/reactor-study/build/resources/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-test/3.4.0/ad22a711534639a5f34dd696594c7a1ade5f8e83/reactor-test-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-core/3.4.0/683c9c676c438e5945b0f12808575f22160c5e54/reactor-core-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-test/2.6.7/8fe7761bc8609b832d8cc1104ff59a0512ef9aaf/spring-boot-starter-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter/2.6.7/59bd62cd60e6e7bb2a525931947c13548b207288/spring-boot-starter-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.7.0-Beta/836df62ad6bfc8a2ed7d4ba59a49bf2d251a75b5/kotlin-stdlib-jdk8-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter/5.5.1/4e3393c2238d1cf48d29b3c79f0c0928e873bc11/junit-jupiter-5.5.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.reactivestreams/reactive-streams/1.0.3/d9fb7a7926ffa635b3dcaa5049fb2bfa25b3e7d0/reactive-streams-1.0.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test-autoconfigure/2.6.7/b95d7d094604e315cbd0c7746d700acdddeee95d/spring-boot-test-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test/2.6.7/78741985ff50aecc5cd2776c74804a288ffc208d/spring-boot-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-test/5.3.19/84d0f06bfe2433173880240808203b69bc40244/spring-test-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-core/5.3.19/344ff3b291d7fdfdb08e865f26238a6caa86acc5/spring-core-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.jayway.jsonpath/json-path/2.6.0/67f565b424f7903a12d4f5b9361b11462ecacdac/json-path-2.6.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.xml.bind/jakarta.xml.bind-api/2.3.3/48e3b9cfc10752fba3521d6511f4165bea951801/jakarta.xml.bind-api-2.3.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.assertj/assertj-core/3.21.0/27a14d6d22c4e3d58f799fb2a5ca8eaf53e6942a/assertj-core-3.21.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.hamcrest/hamcrest/2.2/1820c0968dba3a11a1b30669bb1f01978a91dedc/hamcrest-2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-junit-jupiter/4.0.0/b76de25bd6e5d8f7924d0536729c0076e37e9396/mockito-junit-jupiter-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-core/4.0.0/f5195e0c4a45716bbd2d1d29173adbd148acce3a/mockito-core-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.skyscreamer/jsonassert/1.5.0/6c9d5fe2f59da598d9aefc1cfc6528ff3cf32df3/jsonassert-1.5.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.xmlunit/xmlunit-core/2.8.4/35be57989ca80eefa03161b211630e319a8f36c6/xmlunit-core-2.8.4.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-autoconfigure/2.6.7/d96e51e7a13f16c07f60654071f157f99a0b99be/spring-boot-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot/2.6.7/aef2fff50d49feb43a0726ab7e4945914186e06/spring-boot-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-logging/2.6.7/b49b159bb93086c76fa42555ca074c7ebe9d5fe6/spring-boot-starter-logging-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.annotation/jakarta.annotation-api/1.3.5/59eb84ee0d616332ff44aba065f3888cf002cd2d/jakarta.annotation-api-1.3.5.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.yaml/snakeyaml/1.29/6d0cdafb2010f1297e574656551d7145240f6e25/snakeyaml-1.29.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.7.0-Beta/ab6a690623c06efecd79fd7774de41a29c792d9d/kotlin-stdlib-jdk7-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.7.0-Beta/fb95185efb95996ea0f51dd7aa3e746f5f697236/kotlin-stdlib-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-params/5.8.2/ddeafe92fc263f895bfb73ffeca7fd56e23c2cce/junit-jupiter-params-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-api/5.8.2/4c21029217adf07e4c0d0c5e192b6bf610c94bdc/junit-jupiter-api-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-jcl/5.3.19/eae3d8b8728133782d9e41f9b3ecbd19a4146114/spring-jcl-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/json-smart/2.4.8/7c62f5f72ab05eb54d40e2abf0360a2fe9ea477f/json-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/slf4j-api/1.7.36/6c62681a2f655b49963a5983b8b0950a6120ae14/slf4j-api-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.activation/jakarta.activation-api/1.2.2/99f53adba383cb1bf7c3862844488574b559621f/jakarta.activation-api-1.2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy/1.11.22/8b4c7fa5562a09da1c2a9ab0873cb51f5034d83f/byte-buddy-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy-agent/1.11.22/2fbcf3210dfc09b42242e3b66a5281cc5b9adb80/byte-buddy-agent-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.vaadin.external.google/android-json/0.0.20131108.vaadin1/fa26d351fe62a6a17f5cda1287c1c6110dec413f/android-json-0.0.20131108.vaadin1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-context/5.3.19/d663259767f8fb66229209db0422d65978235525/spring-context-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-classic/1.2.11/4741689214e9d1e8408b206506cbe76d1c6a7d60/logback-classic-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-to-slf4j/2.17.2/17dd0fae2747d9a28c67bc9534108823d2376b46/log4j-to-slf4j-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/jul-to-slf4j/1.7.36/ed46d81cef9c412a88caef405b58f93a678ff2ca/jul-to-slf4j-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.7.0-Beta/5339108aef3a69a5030701ff3ebe9b811dfc54ac/kotlin-stdlib-common-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apiguardian/apiguardian-api/1.1.2/a231e0d844d2721b0fa1b238006d15c6ded6842a/apiguardian-api-1.1.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-commons/1.8.2/32c8b8617c1342376fd5af2053da6410d8866861/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.opentest4j/opentest4j/1.2.0/28c11eb91f9b6d8e200631d46e20a7f407f2a046/opentest4j-1.2.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/accessors-smart/2.4.8/6e1bee5a530caba91893604d6ab41d0edcecca9a/accessors-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-aop/5.3.19/339b0ad295a7af51ac716bd89d36a4fb0febaf5d/spring-aop-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-beans/5.3.19/4bc68c392ed320c9ab5dc439d7f2deb83f03fe76/spring-beans-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-expression/5.3.19/14346b7b84721f61d2b23d3c8baa60c6655527a6/spring-expression-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-core/1.2.11/a01230df5ca5c34540cdaa3ad5efb012f1f1f792/logback-core-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-api/2.17.2/f42d6afa111b4dec5d2aea0fe2197240749a4ea6/log4j-api-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.ow2.asm/asm/9.1/a99500cf6eea30535eeac6be73899d048f8d12a8/asm-9.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-engine/5.8.2/c598b4328d2f397194d11df3b1648d68d7d990e3/junit-jupiter-engine-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.objenesis/objenesis/3.2/7fadf57620c8b8abdf7519533e5527367cb51f09/objenesis-3.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-engine/1.8.2/b737de09f19864bd136805c84df7999a142fec29/junit-platform-engine-1.8.2.jar com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit5 com.fistkim.reactorstudy.execution.ExecutionModelTest,publishOnTestAsync
07:07:03.163 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
07:07:03.206 [main] INFO reactor.Flux.Merge.1 - onSubscribe(FluxFlatMap.FlatMapMain)
07:07:03.209 [main] INFO reactor.Flux.Merge.1 - request(unbounded)
07:07:04.241 [parallel-1] INFO reactor.Flux.Merge.1 - onNext(A)
07:07:04.250 [parallel-2] INFO reactor.Flux.Merge.1 - onNext(D)
07:07:05.244 [parallel-1] INFO reactor.Flux.Merge.1 - onNext(B)
07:07:05.256 [parallel-2] INFO reactor.Flux.Merge.1 - onNext(E)
07:07:06.250 [parallel-1] INFO reactor.Flux.Merge.1 - onNext(C)
07:07:06.261 [parallel-2] INFO reactor.Flux.Merge.1 - onNext(F)
07:07:06.267 [parallel-2] INFO reactor.Flux.Merge.1 - onComplete()

Process finished with exit code 0

```

각각 publishOnTest(), publishOnTestAsync() 에 대한 로그이다. 일부러 toUpperCast()에서 delay를 1초씩 발생시켰다.

먼저 publishOnTest() 는 6초 정도 걸렸다. 왜냐하면 각 알파벳을 처리하는데 1초 이상이 소요되는데 publishOnTest()는 따로 스레드를 할당해주지 않아서 subscribe()를 시작한 main 스레드 하나가 이를 모두 처리해야하기 때문에 모든 delay를 main 스레드 하나가 모두 감내해야 하기 때문이다.

반면에  publishOnTestAsync()는 3초 정도 걸렸다. 왜냐하면 alpabet1, alpabet2 각각에 .publishOn(Schedulers.parallel())을 설정해줌으로써 각각 병렬적으로 다른 스레드로 처리하도록 지시해뒀기 때문에 두 flux가 병렬처리 되도록 했다. 이를 subscribeOn()으로 바꿔줘도 동일한 결과를 볼 수 있다. 로그를 보면 request() 까지 main 스레드가 처리하고 그 뒤 작업부터 parallel 스레드가 처리한 것을 확인할 수 있다.
