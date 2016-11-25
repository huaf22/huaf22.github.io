## [RxJava学习笔记]-变换的基础方法--lift
<!--more-->
## lift方法
lift方法涉及到 Operator 接口, 先看一下 Operator 接口的定义
```
public interface Operator<R, T> extends Func1<Subscriber<? super R>, Subscriber<? super T>> {
   // cover for generics insanity
}
```
```
package rx.functions;

/**
 * Represents a function with one argument.
 * @param <T> the first argument type
 * @param <R> the result type
 */
public interface Func1<T, R> extends Function {
    R call(T t);
}
```
所以Operator接口可以看成
```
//伪代码
public interface Operator<R, T> {
    public Subscriber<R> call(Subscriber<T> subscriber);
}
```
#### Operator的简单实现
```
//伪代码
//将事件序列中的 Integer 对象转换为 String 对象
Operator operator = new Observable.Operator() {
    @override
    public Subscriber call(final Subscriber subscriber) {
        
        return new Subscriber<Integer>() {
            @override
            public void onNext(Integer integer) {
            	String string = "" + integer;  //转换操作
                subscriber.onNext(string);
            }

            @override
            public void onCompleted() {
                subscriber.onCompleted();
            }

            @override
            public void onError(Throwable e) {
                subscriber.onError(e);
            }
        };
    }
});
```
上面的代码里Operator接口实现中新建一个Subscriber对象并在调用传参来的subscriber的onNext方法前做了一个特定操作, 所以只是对Subscriber对象做了一层代理.

#### lift方法源码
```
//伪代码
//Observable中lift方法的实现
public class Observable<T> {
  //...
  Observable.OnSubscribe onSubscribe;

  public Observable lift(Operator operator) {
    return Observable.create(new Observable.OnSubscribe() {
          
      @override
      public void call(Subscriber subscriber) {

        Subscriber newSubscriber = operator.call(subscriber);
        newSubscriber.onStart();

        onSubscribe.call(newSubscriber); //关键调用
      }
    });
  }
  //...
}
```

lift方法中新建了一个Observable对象并在Observable.OnSubscribe的实现中调用传参的Operator对象的call方法新建了一个Subscriber对象, 并调用了onSubscribe.call(newSubscriber), 注意call方法传参的是newSubscriber对象而不是原始的subscriber对象.

### lift方法的用法
```
//伪代码
// lift方法将Int类型转换成String类型
1 Observable.create(new Observable.OnSubscribe() {
2    @override
3     public void call(Subscriber subscriber) {
4         subscriber.onNext(1); //Int类型
5         subscriber.onCompleted();
6     }
7 })
8
9  .lift(operator)  // Int --> String
10
11 .subscribe(new Subscriber() {
12   @override
13    public void onNext(String s) { //String类型
14        Log.d(tag, "Item: " + s);
15    }
16
17    @override
18    public void onCompleted() {
19        Log.d(tag, "Completed!");
20    }
21
22    @override
23    public void onError(Throwable e) {
24        Log.d(tag, "Error!");
25    }
26 });
```
结合fit方法源码和Operator的实现可以整理出程序运行流程:
1. 因为fit返回了一个新Observable对象,所以第11行是调用这个新Observable对象的subscribe方法, 所以会执行fit方法中构造新Observable对象的Observable.OnSubscribe接口的call方法
 ```
 return Observable.create(new Observable.OnSubscribe() {
     @override
     public void call(Subscriber subscriber) {

        Subscriber newSubscriber = operator.call(subscriber);
        newSubscriber.onStart();

        onSubscribe.call(newSubscriber); //关键调用
      }
   });
 ```

2. 执行onSubscribe.call(newSubscriber)方法时, 会调用构造原始Observable对象中的Observable.OnSubscribe接口的call方法, 所以call(Subscriber subscriber)方法中的参数subscriber是operator.call(subscriber)创建newSubscriber对象
```
3   public void call(Subscriber subscriber) {
4         subscriber.onNext(1); //Int类型
5         subscriber.onCompleted();
6     }
```
3. 接着调用第4行subscriber.onNext(1)方法就会执行了newSubscriber对象的onNext方法:
```
 public void onNext(Integer integer) { 
      String string = "" + integer; //转换操作 
      subscriber.onNext(string); 
}
```
该方法中的subscriber是原始subscriber对象,这时根据Int生成了String参数

4. 调用subscriber.onNext(string) 方法时就会执行第13行
```
13    public void onNext(String s) { //String类型
14        Log.d(tag, "Item: " + s);
15    }
```
这样流程就执行完成了

小结: 这种连续调用类似于装饰者模式(Decorator Pattern), 即
```
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
```

在 RxJava 的内部flatMap就是基于基础的lift方法, RxJava中提供的各种变换虽然功能各有不同，但实质上都是**针对事件序列的处理和再发送**.

## 参考资料
* [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
* https://github.com/ReactiveX/RxJava
* http://blog.csdn.net/lzyzsd/article/details/44094895
