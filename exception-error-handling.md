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

![](<.gitbook/assets/스크린샷 2022-05-08 오후 5.03.17.png>)

![](<.gitbook/assets/스크린샷 2022-05-08 오후 5.03.39.png>)

로직 처리중 에러가 발생했을시 에러에 기반하여 응답을 해줘야 하는 경우가 있고, 에러가 났다고 해도 이를 무시하고 처리하던 flow를 이어서 처리를 해야할 경우가 있는데 이 두 경우를 고려하여 적절한 operator 를 사용해야 한다.





### onErrorReturn

![](<.gitbook/assets/image (7).png>)

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

