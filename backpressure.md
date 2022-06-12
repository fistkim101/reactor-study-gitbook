# BackPressure

### onBackPressureDrop()

* Overrides the subscribers request and requests for unbounded data
* It stores all the data in an internal queue
* Drops the remaining elements that are not needed by the subscriber
* This operator helps to track the items that are not needed by the subscriber



### onBackPressureBuffer()

* Overrides the subscribers request and requests for unbounded data
* It stores all the data in an internal queue
* Buffers the remaining elements that are not needed by the subscriber
* The advantage is that the following requests after the initial request for data from the subscriber does not need to go all the way to the Publisher



### onBackPressureError()

* Request an unbounded demand and push to the returned [`Flux`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html), or emit onError fom [`Exceptions.failWithOverflow()`](https://projectreactor.io/docs/core/release/api/reactor/core/Exceptions.html#failWithOverflow--) if not enough demand is requested downstream.



```java

public class BackPressure {

    @Test
    void onBackPressureDrop() throws InterruptedException {

        AtomicInteger index = new AtomicInteger();

        Flux.range(1, 100)
                .onBackpressureDrop(element -> {
                    System.out.println("dropped element : " + element);
                })
                .log()
                .subscribe(new BaseSubscriber<Integer>() {
                    @Override
                    protected void hookOnSubscribe(Subscription subscription) {
                        request(1);
                    }

                    @Override
                    protected void hookOnNext(Integer value) {
                        if (value < 50) {
                            request(1);
                        }
                    }

                    @Override
                    protected void hookOnComplete() {
                        super.hookOnComplete();
                    }

                    @Override
                    protected void hookOnError(Throwable throwable) {
                        super.hookOnError(throwable);
                    }

                    @Override
                    protected void hookOnCancel() {
                        super.hookOnCancel();
                    }
                });

        Thread.sleep(5000L);
    }


    @Test
    void onBackPressureBuffer() throws InterruptedException {

        AtomicInteger index = new AtomicInteger();

        Flux.range(1, 100)
                .onBackpressureBuffer(1, element -> {
                    System.out.println("last buffered element : " + element);
                })
                .log()
                .subscribe(new BaseSubscriber<Integer>() {
                    @Override
                    protected void hookOnSubscribe(Subscription subscription) {
                        request(1);
                    }

                    @Override
                    protected void hookOnNext(Integer value) {
                        System.out.println("hookOnNext : " + value);
                        if (value < 50) {
                            request(1);
                        }
                    }

                    @Override
                    protected void hookOnComplete() {
                        super.hookOnComplete();
                    }

                    @Override
                    protected void hookOnError(Throwable throwable) {
                        super.hookOnError(throwable);
                    }

                    @Override
                    protected void hookOnCancel() {
                        super.hookOnCancel();
                    }
                });

        Thread.sleep(5000L);
    }

}

```
