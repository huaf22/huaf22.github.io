## [RxJava学习笔记]filter方法和OnSubscribeFilter类源码解析
<!--more-->
RxJava提供了很多有用的操作符, 今天就解析一下最简单的filter方法的实现
#### Observable类中filter方法的实现
```
//伪代码
//源代码来源: RxJava/src/main/java/rx/Observable.java
public class Observable<T> {
  //...
  public final Observable<T> filter(Func1<? super T, Boolean> predicate) {
    return create(new OnSubscribeFilter<T>(this, predicate));
   }
  //...
  public static <T> Observable<T> create(OnSubscribe<T> f) {
    return new Observable<T>(RxJavaHooks.onCreate(f));
    }
}
```
filter方法构造了一个新的Observable的对象, 并将当前Observable对象和Func1对象作为参数传入OnSubscribeFilter类, 所以OnSubscribeFilter类应该实现了Observable.OnSubscribe接口, 当新Observable的对象被调用subscribe方法时, OnSubscribeFilter类的call方法就会被回调,我们接着看一下OnSubscribeFilter类的源码.

#### OnSubscribeFilter类的实现
```
//伪代码
//源代码来源: RxJava/src/main/java/rx/internal/operators/OnSubscribeFilter.java
package rx.internal.operators;

import rx.*;
import rx.Observable.OnSubscribe;
import rx.exceptions.*;
import rx.functions.Func1;
import rx.plugins.RxJavaHooks;

/**
 * Filters an Observable by discarding any items it emits that do not meet some test.
 * @param <T> the value type
 */
public final class OnSubscribeFilter<T> implements OnSubscribe<T> {

    final Observable<T> source;

    final Func1<? super T, Boolean> func1;

    public OnSubscribeFilter(Observable<T> source, Func1<? super T, Boolean> func1) {
        this.source = source;
        this.func1 = func1;
    }

    @Override
    public void call(final Subscriber<? super T> child) {
        //...
        FilterSubscriber<T> parent = new FilterSubscriber<T>(child, func1);

        child.add(parent);

        //会调用parent的onStart, onNext, onCompleted/onError方法
        source.unsafeSubscribe(parent);  //关键
    }

    static final class FilterSubscriber<T> extends Subscriber<T> {

        final Subscriber<? super T> actual;
        final Func1<? super T, Boolean> func1;

        boolean done;

        public FilterSubscriber(Subscriber<? super T> actual, Func1<? super T, Boolean> func1) {
            this.actual = actual;
            this.func1 = func1;
            //...
        }

        @Override
        public void onNext(T t) {
            boolean result;
            try {
                result = func1.call(t);
            } catch (Throwable ex) {
               //...
            }
            if (result) {
                actual.onNext(t); //关键
            } 
            //...
        }

        @Override
        public void onError(Throwable e) {
            //...
            actual.onError(e);
        }

        @Override
        public void onCompleted() {
            //...
            actual.onCompleted();
        }

        @Override
        public void setProducer(Producer p) {
            super.setProducer(p);
            actual.setProducer(p);
        }
    }
}
```
构造OnSubscribeFilter对象时会传入参数Func1,这就是下面代码中在调用filter方法时传入的Func1对象 (Func1是具体的过滤算法实现)
```
Observable.just(1, 2, 3, 4, 5)
.filter(new Func1<Integer, Boolean>() { //关键, 过滤算法实现
    @Override
    public Boolean call(Integer item) {
      return( item < 4 );
    }
})
.subscribe(new Subscriber<Integer>() { //原始Subscriber对象
    @Override
    public void onNext(Integer item) {
      System.out.println("Next: " + item);
    }

    @Override
    public void onError(Throwable error) {
      System.err.println("Error: " + error.getMessage());
    }

    @Override
    public void onCompleted() {
      System.out.println("Sequence complete.");
    }
});
```
根据Observable.OnSubscribe接口的用法可以知道OnSubscribeFilter的call方法的Subscriber<? super T> child参数是上面代码中调用subscribe方法时生成的Subscriber对象.
当OnSubscribeFilter的call方法被回调时会创建了一个FilterSubscriber对象,FilterSubscriber类继承了Subscriber类, 构造FilterSubscriber对象时需要传入Subscriber对象和Func1对象.
可以看出FilterSubscriber类只是构造时传参来的Subscriber的代理,并在调用onNext时做了特殊处理,在onNext方法中调用传参来的func1对象的call方法, 当返回的result 为true才会调用被代理的Subscriber对象的onNext方法, 所以才实现了过滤的功能.
#### 总结
filter方法实现过程中会生成一个新的Observable对象,Observable.OnSubscribe对象和Subscriber对象, 并且新的Subscriber对象是原始Subscriber对象的代理,具体的过滤算法由调用者实现Func1接口来提供.

RxJava其他复杂的操作符也按照这样的流程扩展和实现.
