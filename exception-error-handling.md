---
description: Flux, Mono 의 에러처리
---

# Exception/Error handling

reactor의 에러처리에서 대전제로 항상 염두하고 있어야 할 사항은 에러가 발생하면 그 즉시 onError 가 call 되며 stream 의 구독이 중단 된다는 것이다.(Any Exception will terminate the reactive stream)

```java
    @Test
    void terminateCheck() {
        Flux<Integer> numbers = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbersWithException = numbers
                .concatWith(Mono.error(RuntimeException::new))
                .concatWith(Mono.just(4))
                .log();

        StepVerifier.create(numbersWithException)
                .expectNext(1, 2, 3)
                .expectError()
                .verify();
    }
```

```bash
12:07:19.510 [Test worker] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
12:07:19.582 [Test worker] INFO reactor.Flux.ConcatArray.1 - onSubscribe(FluxConcatArray.ConcatArraySubscriber)
12:07:19.588 [Test worker] INFO reactor.Flux.ConcatArray.1 - request(unbounded)
12:07:19.590 [Test worker] INFO reactor.Flux.ConcatArray.1 - onNext(1)
12:07:19.590 [Test worker] INFO reactor.Flux.ConcatArray.1 - onNext(2)
12:07:19.591 [Test worker] INFO reactor.Flux.ConcatArray.1 - onNext(3)
12:07:19.592 [Test worker] ERROR reactor.Flux.ConcatArray.1 - onError(java.lang.RuntimeException)
12:07:19.595 [Test worker] ERROR reactor.Flux.ConcatArray.1 - 
java.lang.RuntimeException: null
	at reactor.core.publisher.MonoErrorSupplied.subscribe(MonoErrorSupplied.java:70)
	at reactor.core.publisher.Mono.subscribe(Mono.java:3987)
	at reactor.core.publisher.FluxConcatArray$ConcatArraySubscriber.onComplete(FluxConcatArray.java:208)
	at reactor.core.publisher.FluxConcatArray.subscribe(FluxConcatArray.java:80)
	at reactor.core.publisher.Flux.subscribe(Flux.java:8095)
	at reactor.test.DefaultStepVerifierBuilder$DefaultStepVerifier.toVerifierAndSubscribe(DefaultStepVerifierBuilder.java:868)
	at reactor.test.DefaultStepVerifierBuilder$DefaultStepVerifier.verify(DefaultStepVerifierBuilder.java:824)
	at reactor.test.DefaultStepVerifierBuilder$DefaultStepVerifier.verify(DefaultStepVerifierBuilder.java:816)
	at com.fistkim.reactorstudy.exception.ExceptionTest.terminateCheck(ExceptionTest.java:23)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.junit.platform.commons.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:725)
	at org.junit.jupiter.engine.execution.MethodInvocation.proceed(MethodInvocation.java:60)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation.proceed(InvocationInterceptorChain.java:131)
	at org.junit.jupiter.engine.extension.TimeoutExtension.intercept(TimeoutExtension.java:149)
	at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestableMethod(TimeoutExtension.java:140)
	at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestMethod(TimeoutExtension.java:84)
	at org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall.lambda$ofVoidMethod$0(ExecutableInvoker.java:115)
	at org.junit.jupiter.engine.execution.ExecutableInvoker.lambda$invoke$0(ExecutableInvoker.java:105)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation.proceed(InvocationInterceptorChain.java:106)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.proceed(InvocationInterceptorChain.java:64)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.chainAndInvoke(InvocationInterceptorChain.java:45)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.invoke(InvocationInterceptorChain.java:37)
	at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:104)
	at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:98)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$invokeTestMethod$7(TestMethodTestDescriptor.java:214)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.invokeTestMethod(TestMethodTestDescriptor.java:210)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:135)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:66)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:151)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1541)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1541)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:35)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:54)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:108)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:88)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.lambda$execute$0(EngineExecutionOrchestrator.java:54)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.withInterceptedStreams(EngineExecutionOrchestrator.java:67)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:52)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:96)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:75)
	at org.gradle.api.internal.tasks.testing.junitplatform.JUnitPlatformTestClassProcessor$CollectAllTestClassesExecutor.processAllTestClasses(JUnitPlatformTestClassProcessor.java:99)
	at org.gradle.api.internal.tasks.testing.junitplatform.JUnitPlatformTestClassProcessor$CollectAllTestClassesExecutor.access$000(JUnitPlatformTestClassProcessor.java:79)
	at org.gradle.api.internal.tasks.testing.junitplatform.JUnitPlatformTestClassProcessor.stop(JUnitPlatformTestClassProcessor.java:75)
	at org.gradle.api.internal.tasks.testing.SuiteTestClassProcessor.stop(SuiteTestClassProcessor.java:61)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:36)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:24)
	at org.gradle.internal.dispatch.ContextClassLoaderDispatch.dispatch(ContextClassLoaderDispatch.java:33)
	at org.gradle.internal.dispatch.ProxyDispatchAdapter$DispatchingInvocationHandler.invoke(ProxyDispatchAdapter.java:94)
	at com.sun.proxy.$Proxy2.stop(Unknown Source)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker$3.run(TestWorker.java:193)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.executeAndMaintainThreadName(TestWorker.java:129)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.execute(TestWorker.java:100)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.execute(TestWorker.java:60)
	at org.gradle.process.internal.worker.child.ActionExecutionWorker.execute(ActionExecutionWorker.java:56)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:133)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:71)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:69)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:74)
BUILD SUCCESSFUL in 2s
```

publisher - subscription - subscriber 원형에서 보면 subscribe에 따른 onSubscribe(new Subscription(\~))내 에서 request(int n) 내부에서 onNext() 수행 전체를 try\~catch 로 감싸놓고 여기서 에러가 발생시 바로 catch 부분이 실행되며 구독이 끊기는 것이라 할 수 있다.



강의에서는 reactor가 제공하는 exception/error 를 핸들링하는 operator 를 크게 두 부류로 나눠서 알려주고 있다.

![](<.gitbook/assets/스크린샷 2022-05-08 오후 5.03.17.png>)

![](<.gitbook/assets/스크린샷 2022-05-08 오후 5.03.39.png>)

로직 처리중 에러가 발생했을시 에러에 기반하여 응답을 해줘야 하는 경우가 있고, 에러가 났다고 해도 이를 무시하고 처리하던 flow를 이어서 처리를 해야할 경우가 있는데 이 두 경우를 고려하여 적절한 operator 를 사용해야 한다.





### onErrorReturn

![](<.gitbook/assets/image (7) (1) (1).png>)

up-stream 을 구독중에 에러가 발생하면 그 즉시 구독이 멈추는 것이 대전제임을 항상 명심한다.(위에 정리했다시피 new Subscription() 내부에서 try\~catch 방식이라는 점을 기억) onErrorReturn operator는 catch 에서 정해준 특정한 값을 onNext() 를 통해서 보내주는 역할을 수행한다.

```java
    @Test
    void onErrorReturnTest() {
      Flux<Integer> numbers = Flux.fromIterable(List.of(1, 2, 3))
                .concatWith(Mono.error(new RuntimeException()))
                .onErrorReturn(4)
                .log();

        StepVerifier.create(numbers)
                .expectNext(1, 2, 3, 4)
                .verifyComplete();
    }
```

```java
    @Test
    void onErrorReturnTest_2() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(5, 6, 7));

        Flux<Integer> mergedNumbers = numbers_1
                .concatWith(Mono.error(new RuntimeException()))
                .onErrorReturn(4)
                .concatWith(numbers_2)
                .log();

        StepVerifier.create(mergedNumbers)
                .expectNext(1, 2, 3, 4, 5, 6, 7)
                .verifyComplete();
    }
```





### onErrorResume

![](<.gitbook/assets/image (2).png>)

onErrorReturn 은 단지 up-stream 구독중 error 가 발생했을때 원하는 특정한 값으로 대체해주는 operator 인 것과 대조적으로 onErrorResume 은 단어 그대로 에러가 나도 이를 '재개'해준다. 그리고 이렇게 구독을 재개해줄때 에러가 난 publisher 를 대체해줄 recovery publisher를 정의할 수 있도록 해준다.

물론 recovery publisher의 element 타입은 본래의 up-stream의 element 타입과 같아야 한다.



```java

    @Test
    void onErrorResumeTest_1() {
        Flux<Integer> numbers = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> target = numbers.concatWith(Mono.error(new RuntimeException())).onErrorResume(exception -> {
            System.out.println(exception.getClass().getName());
            return Flux.fromIterable(List.of(4, 5, 6));
        }).log();

        StepVerifier.create(target).expectNext(1, 2, 3, 4, 5, 6).verifyComplete();
    }
    
    @Test
    void onErrorResumeTest_2() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(10, 11, 12));
        Flux<Integer> target = numbers_1
                .concatWith(Mono.error(new RuntimeException()))
                .concatWith(numbers_2).onErrorResume(exception -> {
                    System.out.println(exception.getClass().getName());
                    return Flux.fromIterable(List.of(4, 5, 6));
                }).log();

        StepVerifier.create(target)
                .expectNext(1, 2, 3, 4, 5, 6)
                .verifyComplete();
    }
```

두 번째 예제코드를 좀 주의해서 기억해둬야겠다. 여기서 나는 처음에 expectNext(1, 2, 3, 4, 5, 6, 10, 11, 12)일 거라 생각 했다. 왜냐하면 up-stream 이 1, 2, 3, error, 10, 11, 12 로 구성되어 있기 때문에 error가 onErrorResume()에 의해서 4, 5, 6으로 대체될 것이라 생각했기 때문이다. 하지만 매우 잘못된 생각이다.

onErrorResume()은 에러가 발생 했을때 '대체'될 publisher를 주는 것이 아니라 '갈아 탈' publisher를 주는 것이다. 즉, onErrorResume 에서 제공하도록 설정하는 recovery publisher 는 error를 대체 시키는 것이 아니라 갈아탈 대상이다.

위 예제에서 보면 "1, 2, 3, error, 10, 11, 12" 순으로 emit 될 예정이었는데 error 가 나오면서 onErrorResume 에 의해서 미리 정의된 recovery publisher인 4, 5, 6 Flux가 down-stream 으로 구성된 것이다.

이것을 생각하면서 아래 코드를 다시 보자.

```java

    @Test
    void onErrorResumeTest_3() {
        Flux<Integer> numbers_1 = Flux.fromIterable(List.of(1, 2, 3));
        Flux<Integer> numbers_2 = Flux.fromIterable(List.of(10, 11, 12));
        Flux<Integer> target = numbers_1
                .concatWith(Mono.error(new RuntimeException()))
                .concatWith(numbers_2)
                .onErrorResume(exception -> {
                    System.out.println(exception.getClass().getName());
                    return Flux.empty();
                }).log();

        StepVerifier.create(target)
                .expectNext(1, 2, 3)
                .verifyComplete();
    }
```

갈아 탈 publisher 로 Flux.empty()를 정의해줬으므로 up-stream은 1, 2, 3 -> emtpy 로 갈아타게 되니까 결국 down-stream 은 1, 2, 3이 될 뿐이다.





### onErrorContinue

![](<.gitbook/assets/image (6) (1).png>)

onErrorContinue 의 핵심은 에러를 발생시킨 element 를 drop 시킴으로써 element의 emit이 지속되도록 유지시켜 준다는 것이다. 그림과 같이 앞단의 operator의 subscribe에 영향을 줄 수 있다(influences upstream).

onErrorContinue는 발생한 exception과 그 exception을 발생시킨 element(the value that triggered the error)를 받는 BiConsumer를 parameter로 받는다.

```java
    @Test
    void onErrorContinueTest() {
        Flux<Integer> numbersWithError = Flux.fromIterable(List.of(1, 2, 3))
                .map(number ->
                {
                    if (number == 2) {
                        throw new RuntimeException();
                    }

                    return number;
                })
                .onErrorContinue((exception, number) -> {
                    System.out.println("exception : " + exception.getClass());
                    System.out.println("triggered element : " + number);
                })
                .log();

        StepVerifier.create(numbersWithError)
                .expectNext(1, 3)
                .verifyComplete();
    }
```

```bash
/Users/jungkwonkim/Library/Java/JavaVirtualMachines/corretto-11.0.15/Contents/Home/bin/java -ea -Didea.test.cyclic.buffer.size=1048576 -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=54049:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-launcher/1.8.2/junit-platform-launcher-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-engine/1.8.2/junit-platform-engine-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-commons/1.8.2/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/apiguardian/apiguardian-api/1.1.2/apiguardian-api-1.1.2.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit5-rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit-rt.jar:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/test:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/main:/Users/jungkwonkim/Lab/App/reactor-study/build/resources/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-test/3.4.0/ad22a711534639a5f34dd696594c7a1ade5f8e83/reactor-test-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-core/3.4.0/683c9c676c438e5945b0f12808575f22160c5e54/reactor-core-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-test/2.6.7/8fe7761bc8609b832d8cc1104ff59a0512ef9aaf/spring-boot-starter-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter/2.6.7/59bd62cd60e6e7bb2a525931947c13548b207288/spring-boot-starter-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.7.0-Beta/836df62ad6bfc8a2ed7d4ba59a49bf2d251a75b5/kotlin-stdlib-jdk8-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter/5.5.1/4e3393c2238d1cf48d29b3c79f0c0928e873bc11/junit-jupiter-5.5.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.reactivestreams/reactive-streams/1.0.3/d9fb7a7926ffa635b3dcaa5049fb2bfa25b3e7d0/reactive-streams-1.0.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test-autoconfigure/2.6.7/b95d7d094604e315cbd0c7746d700acdddeee95d/spring-boot-test-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test/2.6.7/78741985ff50aecc5cd2776c74804a288ffc208d/spring-boot-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-test/5.3.19/84d0f06bfe2433173880240808203b69bc40244/spring-test-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-core/5.3.19/344ff3b291d7fdfdb08e865f26238a6caa86acc5/spring-core-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.jayway.jsonpath/json-path/2.6.0/67f565b424f7903a12d4f5b9361b11462ecacdac/json-path-2.6.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.xml.bind/jakarta.xml.bind-api/2.3.3/48e3b9cfc10752fba3521d6511f4165bea951801/jakarta.xml.bind-api-2.3.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.assertj/assertj-core/3.21.0/27a14d6d22c4e3d58f799fb2a5ca8eaf53e6942a/assertj-core-3.21.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.hamcrest/hamcrest/2.2/1820c0968dba3a11a1b30669bb1f01978a91dedc/hamcrest-2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-junit-jupiter/4.0.0/b76de25bd6e5d8f7924d0536729c0076e37e9396/mockito-junit-jupiter-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-core/4.0.0/f5195e0c4a45716bbd2d1d29173adbd148acce3a/mockito-core-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.skyscreamer/jsonassert/1.5.0/6c9d5fe2f59da598d9aefc1cfc6528ff3cf32df3/jsonassert-1.5.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.xmlunit/xmlunit-core/2.8.4/35be57989ca80eefa03161b211630e319a8f36c6/xmlunit-core-2.8.4.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-autoconfigure/2.6.7/d96e51e7a13f16c07f60654071f157f99a0b99be/spring-boot-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot/2.6.7/aef2fff50d49feb43a0726ab7e4945914186e06/spring-boot-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-logging/2.6.7/b49b159bb93086c76fa42555ca074c7ebe9d5fe6/spring-boot-starter-logging-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.annotation/jakarta.annotation-api/1.3.5/59eb84ee0d616332ff44aba065f3888cf002cd2d/jakarta.annotation-api-1.3.5.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.yaml/snakeyaml/1.29/6d0cdafb2010f1297e574656551d7145240f6e25/snakeyaml-1.29.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.7.0-Beta/ab6a690623c06efecd79fd7774de41a29c792d9d/kotlin-stdlib-jdk7-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.7.0-Beta/fb95185efb95996ea0f51dd7aa3e746f5f697236/kotlin-stdlib-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-params/5.8.2/ddeafe92fc263f895bfb73ffeca7fd56e23c2cce/junit-jupiter-params-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-api/5.8.2/4c21029217adf07e4c0d0c5e192b6bf610c94bdc/junit-jupiter-api-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-jcl/5.3.19/eae3d8b8728133782d9e41f9b3ecbd19a4146114/spring-jcl-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/json-smart/2.4.8/7c62f5f72ab05eb54d40e2abf0360a2fe9ea477f/json-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/slf4j-api/1.7.36/6c62681a2f655b49963a5983b8b0950a6120ae14/slf4j-api-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.activation/jakarta.activation-api/1.2.2/99f53adba383cb1bf7c3862844488574b559621f/jakarta.activation-api-1.2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy/1.11.22/8b4c7fa5562a09da1c2a9ab0873cb51f5034d83f/byte-buddy-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy-agent/1.11.22/2fbcf3210dfc09b42242e3b66a5281cc5b9adb80/byte-buddy-agent-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.vaadin.external.google/android-json/0.0.20131108.vaadin1/fa26d351fe62a6a17f5cda1287c1c6110dec413f/android-json-0.0.20131108.vaadin1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-context/5.3.19/d663259767f8fb66229209db0422d65978235525/spring-context-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-classic/1.2.11/4741689214e9d1e8408b206506cbe76d1c6a7d60/logback-classic-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-to-slf4j/2.17.2/17dd0fae2747d9a28c67bc9534108823d2376b46/log4j-to-slf4j-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/jul-to-slf4j/1.7.36/ed46d81cef9c412a88caef405b58f93a678ff2ca/jul-to-slf4j-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.7.0-Beta/5339108aef3a69a5030701ff3ebe9b811dfc54ac/kotlin-stdlib-common-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apiguardian/apiguardian-api/1.1.2/a231e0d844d2721b0fa1b238006d15c6ded6842a/apiguardian-api-1.1.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-commons/1.8.2/32c8b8617c1342376fd5af2053da6410d8866861/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.opentest4j/opentest4j/1.2.0/28c11eb91f9b6d8e200631d46e20a7f407f2a046/opentest4j-1.2.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/accessors-smart/2.4.8/6e1bee5a530caba91893604d6ab41d0edcecca9a/accessors-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-aop/5.3.19/339b0ad295a7af51ac716bd89d36a4fb0febaf5d/spring-aop-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-beans/5.3.19/4bc68c392ed320c9ab5dc439d7f2deb83f03fe76/spring-beans-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-expression/5.3.19/14346b7b84721f61d2b23d3c8baa60c6655527a6/spring-expression-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-core/1.2.11/a01230df5ca5c34540cdaa3ad5efb012f1f1f792/logback-core-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-api/2.17.2/f42d6afa111b4dec5d2aea0fe2197240749a4ea6/log4j-api-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.ow2.asm/asm/9.1/a99500cf6eea30535eeac6be73899d048f8d12a8/asm-9.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-engine/5.8.2/c598b4328d2f397194d11df3b1648d68d7d990e3/junit-jupiter-engine-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.objenesis/objenesis/3.2/7fadf57620c8b8abdf7519533e5527367cb51f09/objenesis-3.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-engine/1.8.2/b737de09f19864bd136805c84df7999a142fec29/junit-platform-engine-1.8.2.jar com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit5 com.fistkim.reactorstudy.exception.ExceptionTest,onErrorContinueTest
16:48:25.517 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
16:48:25.550 [main] INFO reactor.Flux.ContextWrite.1 - | onSubscribe([Fuseable] FluxContextWrite.ContextWriteSubscriber)
16:48:25.555 [main] INFO reactor.Flux.ContextWrite.1 - | request(unbounded)
16:48:25.555 [main] INFO reactor.Flux.ContextWrite.1 - | onNext(1)
exception : class java.lang.RuntimeException
triggered element : 2
16:48:25.564 [main] INFO reactor.Flux.ContextWrite.1 - | onNext(3)
16:48:25.564 [main] INFO reactor.Flux.ContextWrite.1 - | onComplete()

Process finished with exit code 0

```





### onErrorMap

![](<.gitbook/assets/image (4) (1) (1).png>)

recover 하진 않고 error를 바꿔준다. onErrorMap 의 parameter 자체가 throwable 을 받아서 throwable을 내보내주는 mapper Function이다.

```java
    @Test
    void onErrorMapTest() {
        Flux<Integer> numbersWithError = Flux.fromIterable(List.of(1, 2, 3))
                .map(number ->
                {
                    if (number == 2) {
                        throw new RuntimeException();
                    }

                    return number;
                })
                .onErrorMap(exception -> {
                    throw new IllegalArgumentException();
                })
                .log();

        StepVerifier.create(numbersWithError)
                .expectNext(1)
                .expectError(IllegalArgumentException.class)
                .verify();
    }

```





### doOnError

![](<.gitbook/assets/image (8).png>)

error 가 발생 했을때 부가적으로 처리하고자 하는 것이 있을때 사용한다. 일반적인 doOn\~ 의 순서처럼 target event 보다 먼저 실행됨을 기억해두기.

```java
    @Test
    void doOnErrorTest() {
        Flux<Integer> numbersWithError = Flux.fromIterable(List.of(1, 2, 3))
                .concatWith(Mono.error(new RuntimeException()))
                .concatWith(Mono.just(4))
                .doOnError(ex -> {
                    System.out.println("exception : " + ex.getClass().getName());
                })
                .log();

        StepVerifier.create(numbersWithError)
                .expectNext(1, 2, 3)
                .expectError(RuntimeException.class)
                .verify();
    }

```

```bash
/Users/jungkwonkim/Library/Java/JavaVirtualMachines/corretto-11.0.15/Contents/Home/bin/java -ea -Didea.test.cyclic.buffer.size=1048576 -javaagent:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar=57813:/Applications/IntelliJ IDEA.app/Contents/bin -Dfile.encoding=UTF-8 -classpath /Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-launcher/1.8.2/junit-platform-launcher-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-engine/1.8.2/junit-platform-engine-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/opentest4j/opentest4j/1.2.0/opentest4j-1.2.0.jar:/Users/jungkwonkim/.m2/repository/org/junit/platform/junit-platform-commons/1.8.2/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.m2/repository/org/apiguardian/apiguardian-api/1.1.2/apiguardian-api-1.1.2.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit5-rt.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/junit/lib/junit-rt.jar:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/test:/Users/jungkwonkim/Lab/App/reactor-study/build/classes/java/main:/Users/jungkwonkim/Lab/App/reactor-study/build/resources/main:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-test/3.4.0/ad22a711534639a5f34dd696594c7a1ade5f8e83/reactor-test-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/io.projectreactor/reactor-core/3.4.0/683c9c676c438e5945b0f12808575f22160c5e54/reactor-core-3.4.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-test/2.6.7/8fe7761bc8609b832d8cc1104ff59a0512ef9aaf/spring-boot-starter-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter/2.6.7/59bd62cd60e6e7bb2a525931947c13548b207288/spring-boot-starter-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk8/1.7.0-Beta/836df62ad6bfc8a2ed7d4ba59a49bf2d251a75b5/kotlin-stdlib-jdk8-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter/5.5.1/4e3393c2238d1cf48d29b3c79f0c0928e873bc11/junit-jupiter-5.5.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.reactivestreams/reactive-streams/1.0.3/d9fb7a7926ffa635b3dcaa5049fb2bfa25b3e7d0/reactive-streams-1.0.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test-autoconfigure/2.6.7/b95d7d094604e315cbd0c7746d700acdddeee95d/spring-boot-test-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-test/2.6.7/78741985ff50aecc5cd2776c74804a288ffc208d/spring-boot-test-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-test/5.3.19/84d0f06bfe2433173880240808203b69bc40244/spring-test-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-core/5.3.19/344ff3b291d7fdfdb08e865f26238a6caa86acc5/spring-core-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.jayway.jsonpath/json-path/2.6.0/67f565b424f7903a12d4f5b9361b11462ecacdac/json-path-2.6.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.xml.bind/jakarta.xml.bind-api/2.3.3/48e3b9cfc10752fba3521d6511f4165bea951801/jakarta.xml.bind-api-2.3.3.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.assertj/assertj-core/3.21.0/27a14d6d22c4e3d58f799fb2a5ca8eaf53e6942a/assertj-core-3.21.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.hamcrest/hamcrest/2.2/1820c0968dba3a11a1b30669bb1f01978a91dedc/hamcrest-2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-junit-jupiter/4.0.0/b76de25bd6e5d8f7924d0536729c0076e37e9396/mockito-junit-jupiter-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.mockito/mockito-core/4.0.0/f5195e0c4a45716bbd2d1d29173adbd148acce3a/mockito-core-4.0.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.skyscreamer/jsonassert/1.5.0/6c9d5fe2f59da598d9aefc1cfc6528ff3cf32df3/jsonassert-1.5.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.xmlunit/xmlunit-core/2.8.4/35be57989ca80eefa03161b211630e319a8f36c6/xmlunit-core-2.8.4.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-autoconfigure/2.6.7/d96e51e7a13f16c07f60654071f157f99a0b99be/spring-boot-autoconfigure-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot/2.6.7/aef2fff50d49feb43a0726ab7e4945914186e06/spring-boot-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework.boot/spring-boot-starter-logging/2.6.7/b49b159bb93086c76fa42555ca074c7ebe9d5fe6/spring-boot-starter-logging-2.6.7.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.annotation/jakarta.annotation-api/1.3.5/59eb84ee0d616332ff44aba065f3888cf002cd2d/jakarta.annotation-api-1.3.5.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.yaml/snakeyaml/1.29/6d0cdafb2010f1297e574656551d7145240f6e25/snakeyaml-1.29.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-jdk7/1.7.0-Beta/ab6a690623c06efecd79fd7774de41a29c792d9d/kotlin-stdlib-jdk7-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib/1.7.0-Beta/fb95185efb95996ea0f51dd7aa3e746f5f697236/kotlin-stdlib-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-params/5.8.2/ddeafe92fc263f895bfb73ffeca7fd56e23c2cce/junit-jupiter-params-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-api/5.8.2/4c21029217adf07e4c0d0c5e192b6bf610c94bdc/junit-jupiter-api-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-jcl/5.3.19/eae3d8b8728133782d9e41f9b3ecbd19a4146114/spring-jcl-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/json-smart/2.4.8/7c62f5f72ab05eb54d40e2abf0360a2fe9ea477f/json-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/slf4j-api/1.7.36/6c62681a2f655b49963a5983b8b0950a6120ae14/slf4j-api-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/jakarta.activation/jakarta.activation-api/1.2.2/99f53adba383cb1bf7c3862844488574b559621f/jakarta.activation-api-1.2.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy/1.11.22/8b4c7fa5562a09da1c2a9ab0873cb51f5034d83f/byte-buddy-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.bytebuddy/byte-buddy-agent/1.11.22/2fbcf3210dfc09b42242e3b66a5281cc5b9adb80/byte-buddy-agent-1.11.22.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/com.vaadin.external.google/android-json/0.0.20131108.vaadin1/fa26d351fe62a6a17f5cda1287c1c6110dec413f/android-json-0.0.20131108.vaadin1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-context/5.3.19/d663259767f8fb66229209db0422d65978235525/spring-context-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-classic/1.2.11/4741689214e9d1e8408b206506cbe76d1c6a7d60/logback-classic-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-to-slf4j/2.17.2/17dd0fae2747d9a28c67bc9534108823d2376b46/log4j-to-slf4j-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.slf4j/jul-to-slf4j/1.7.36/ed46d81cef9c412a88caef405b58f93a678ff2ca/jul-to-slf4j-1.7.36.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains.kotlin/kotlin-stdlib-common/1.7.0-Beta/5339108aef3a69a5030701ff3ebe9b811dfc54ac/kotlin-stdlib-common-1.7.0-Beta.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.jetbrains/annotations/13.0/919f0dfe192fb4e063e7dacadee7f8bb9a2672a9/annotations-13.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apiguardian/apiguardian-api/1.1.2/a231e0d844d2721b0fa1b238006d15c6ded6842a/apiguardian-api-1.1.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-commons/1.8.2/32c8b8617c1342376fd5af2053da6410d8866861/junit-platform-commons-1.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.opentest4j/opentest4j/1.2.0/28c11eb91f9b6d8e200631d46e20a7f407f2a046/opentest4j-1.2.0.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/net.minidev/accessors-smart/2.4.8/6e1bee5a530caba91893604d6ab41d0edcecca9a/accessors-smart-2.4.8.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-aop/5.3.19/339b0ad295a7af51ac716bd89d36a4fb0febaf5d/spring-aop-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-beans/5.3.19/4bc68c392ed320c9ab5dc439d7f2deb83f03fe76/spring-beans-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.springframework/spring-expression/5.3.19/14346b7b84721f61d2b23d3c8baa60c6655527a6/spring-expression-5.3.19.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/ch.qos.logback/logback-core/1.2.11/a01230df5ca5c34540cdaa3ad5efb012f1f1f792/logback-core-1.2.11.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.apache.logging.log4j/log4j-api/2.17.2/f42d6afa111b4dec5d2aea0fe2197240749a4ea6/log4j-api-2.17.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.ow2.asm/asm/9.1/a99500cf6eea30535eeac6be73899d048f8d12a8/asm-9.1.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.jupiter/junit-jupiter-engine/5.8.2/c598b4328d2f397194d11df3b1648d68d7d990e3/junit-jupiter-engine-5.8.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.objenesis/objenesis/3.2/7fadf57620c8b8abdf7519533e5527367cb51f09/objenesis-3.2.jar:/Users/jungkwonkim/.gradle/caches/modules-2/files-2.1/org.junit.platform/junit-platform-engine/1.8.2/b737de09f19864bd136805c84df7999a142fec29/junit-platform-engine-1.8.2.jar com.intellij.rt.junit.JUnitStarter -ideVersion5 -junit5 com.fistkim.reactorstudy.exception.ExceptionTest,doOnErrorTest
21:06:44.485 [main] DEBUG reactor.util.Loggers$LoggerFactory - Using Slf4j logging framework
21:06:44.539 [main] INFO reactor.Flux.Peek.1 - onSubscribe(FluxPeek.PeekSubscriber)
21:06:44.544 [main] INFO reactor.Flux.Peek.1 - request(unbounded)
21:06:44.545 [main] INFO reactor.Flux.Peek.1 - onNext(1)
21:06:44.546 [main] INFO reactor.Flux.Peek.1 - onNext(2)
21:06:44.546 [main] INFO reactor.Flux.Peek.1 - onNext(3)
exception : java.lang.RuntimeException
21:06:44.554 [main] ERROR reactor.Flux.Peek.1 - onError(java.lang.RuntimeException)
21:06:44.557 [main] ERROR reactor.Flux.Peek.1 - 
java.lang.RuntimeException: null
	at com.fistkim.reactorstudy.exception.ExceptionTest.doOnErrorTest(ExceptionTest.java:14)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.junit.platform.commons.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:725)
	at org.junit.jupiter.engine.execution.MethodInvocation.proceed(MethodInvocation.java:60)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain$ValidatingInvocation.proceed(InvocationInterceptorChain.java:131)
	at org.junit.jupiter.engine.extension.TimeoutExtension.intercept(TimeoutExtension.java:149)
	at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestableMethod(TimeoutExtension.java:140)
	at org.junit.jupiter.engine.extension.TimeoutExtension.interceptTestMethod(TimeoutExtension.java:84)
	at org.junit.jupiter.engine.execution.ExecutableInvoker$ReflectiveInterceptorCall.lambda$ofVoidMethod$0(ExecutableInvoker.java:115)
	at org.junit.jupiter.engine.execution.ExecutableInvoker.lambda$invoke$0(ExecutableInvoker.java:105)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain$InterceptedInvocation.proceed(InvocationInterceptorChain.java:106)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.proceed(InvocationInterceptorChain.java:64)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.chainAndInvoke(InvocationInterceptorChain.java:45)
	at org.junit.jupiter.engine.execution.InvocationInterceptorChain.invoke(InvocationInterceptorChain.java:37)
	at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:104)
	at org.junit.jupiter.engine.execution.ExecutableInvoker.invoke(ExecutableInvoker.java:98)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.lambda$invokeTestMethod$7(TestMethodTestDescriptor.java:214)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.invokeTestMethod(TestMethodTestDescriptor.java:210)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:135)
	at org.junit.jupiter.engine.descriptor.TestMethodTestDescriptor.execute(TestMethodTestDescriptor.java:66)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:151)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1541)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1541)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.invokeAll(SameThreadHierarchicalTestExecutorService.java:41)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$6(NodeTestTask.java:155)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$8(NodeTestTask.java:141)
	at org.junit.platform.engine.support.hierarchical.Node.around(Node.java:137)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.lambda$executeRecursively$9(NodeTestTask.java:139)
	at org.junit.platform.engine.support.hierarchical.ThrowableCollector.execute(ThrowableCollector.java:73)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.executeRecursively(NodeTestTask.java:138)
	at org.junit.platform.engine.support.hierarchical.NodeTestTask.execute(NodeTestTask.java:95)
	at org.junit.platform.engine.support.hierarchical.SameThreadHierarchicalTestExecutorService.submit(SameThreadHierarchicalTestExecutorService.java:35)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestExecutor.execute(HierarchicalTestExecutor.java:57)
	at org.junit.platform.engine.support.hierarchical.HierarchicalTestEngine.execute(HierarchicalTestEngine.java:54)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:107)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:88)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.lambda$execute$0(EngineExecutionOrchestrator.java:54)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.withInterceptedStreams(EngineExecutionOrchestrator.java:67)
	at org.junit.platform.launcher.core.EngineExecutionOrchestrator.execute(EngineExecutionOrchestrator.java:52)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:114)
	at org.junit.platform.launcher.core.DefaultLauncher.execute(DefaultLauncher.java:86)
	at org.junit.platform.launcher.core.DefaultLauncherSession$DelegatingLauncher.execute(DefaultLauncherSession.java:86)
	at org.junit.platform.launcher.core.SessionPerRequestLauncher.execute(SessionPerRequestLauncher.java:53)
	at com.intellij.junit5.JUnit5IdeaTestRunner.startRunnerWithArgs(JUnit5IdeaTestRunner.java:71)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater$1.execute(IdeaTestRunner.java:38)
	at com.intellij.rt.execution.junit.TestsRepeater.repeat(TestsRepeater.java:11)
	at com.intellij.rt.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:35)
	at com.intellij.rt.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:235)
	at com.intellij.rt.junit.JUnitStarter.main(JUnitStarter.java:54)

Process finished with exit code 0
```

