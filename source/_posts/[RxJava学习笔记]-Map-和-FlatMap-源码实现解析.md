## [RxJava学习笔记]-Map-和-FlatMap-源码实现解析
<!--more-->
## Map方法
[官网介绍](http://reactivex.io/documentation/operators/map.html)
#### 简单用法

```
//伪代码
//把传入的参数转化之后返回另一个对象
Observable.just("images/logo.png") // 输入类型 String 
.map(new Func1<String, Bitmap>() { 
    @override 
    public Bitmap call(String path) { // 参数类型 String 
      return getBitmap(path); // 返回类型 Bitmap
   } 
}) 
.subscribe(new Action1<Bitmap>() {
    @override 
    public void call(Bitmap bitmap) { // 参数类型Bitmap
      showBitmap(bitmap);
   } 
});
```
#### map方法的源码
```
 /**
  * Returns an Observable that applies a specified function to each item emitted by the source Observable and
  * emits the results of these function applications.
  *
  * @param <R> the output type
  * @param func
  *            a function to apply to each item emitted by the Observable
  * @return an Observable that emits the items from the source Observable, transformed by the specified
  *         function
  */
 public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
     return create(new OnSubscribeMap<T, R>(this, func));
 }
```
#### OnSubscribeMap源码
```
/**
 * Applies a function of your choosing to every item emitted by an {@code Observable}, and emits the results of
 * this transformation as a new {@code Observable}.
 *
 * @param <T> the input value type
 * @param <R> the return value type
 */
public final class OnSubscribeMap<T, R> implements OnSubscribe<R> {  //实现了OnSubscribe接口
    final Observable<T> source;
    final Func1<? super T, ? extends R> transformer;
    public OnSubscribeMap(Observable<T> source, Func1<? super T, ? extends R> transformer) {
        this.source = source;
        this.transformer = transformer;
    }

    @Override
    public void call(final Subscriber<? super R> o) {
        MapSubscriber<T, R> parent = new MapSubscriber<T, R>(o, transformer);
        o.add(parent);
        source.unsafeSubscribe(parent);
    }

    static final class MapSubscriber<T, R> extends Subscriber<T> {
        final Subscriber<? super R> actual;
        final Func1<? super T, ? extends R> mapper;

        boolean done;

        public MapSubscriber(Subscriber<? super R> actual, Func1<? super T, ? extends R> mapper) {
            this.actual = actual;
            this.mapper = mapper;
        }

        @Override
        public void onNext(T t) {
            R result;

            try {
                result = mapper.call(t);  //重要
            } catch (Throwable ex) {
                Exceptions.throwIfFatal(ex);
                unsubscribe();
                onError(OnErrorThrowable.addValueAsLastCause(ex, t));
                return;
            }

            actual.onNext(result);  //处理后调用原始的Subscriber
        }

        @Override
        public void onError(Throwable e) {
            if (done) {
                RxJavaHooks.onError(e);
                return;
            }
            done = true;

            actual.onError(e);
        }

        @Override
        public void onCompleted() {
            if (done) {
                return;
            }
            actual.onCompleted();
        }

        @Override
        public void setProducer(Producer p) {
            actual.setProducer(p);
        }
    }

}
```
是不是很眼熟, 这个实现和[filter方法和OnSubscribeFilter类源码解析](http://www.jianshu.com/p/7c34e656fe10) 的流程一致, 有兴趣的可以看一下这篇文章, 这里不再详解了. 
OnSubscribeFilter 和 OnSubscribeMap 的区别在于 OnSubscribeMap 中新建的Subscriber调用OnNext的操作, 最终将func1方法返回的数据传递给Subscriber.

## FlatMap方法
[官网介绍](http://reactivex.io/documentation/operators/flatmap.html)
FlatMap操作符通过一个可以操作每个从原始Observable发射的数据的方法来实现变换Observable, 这个方法可以返回发射其他数据的Observable. FlatMap会合并这些Observable发射的数据并把合并后的数据当做一个序列.

#### 简单用法
```
//伪代码
Student[] students = ...;

Observable.from(students) //创建的Observable会连续调用subscriber的onNext()方法
.flatMap(new Func1<Student, Observable<Course>>() { 
  @override 
  public Observable<Course> call(Student student) { //该方法会被调用students.count次
   //getCourses()方法返回的是列表
   Observable observable = Observable.from(student.getCourses());   // 关键
   return observable;
  } 
}) 
.subscribe(new Subscriber<Course>() { 
  @override
  public void onNext(Course course) {  //该方法会被调用(students.count * 每个学生的课程数)次
    Log.d(tag, course.getName()); 
  }
   //...
});
```

#### FlatMap方法的源码
```
/**
 * Returns an Observable that emits items 
 * based on 
 * applying a function that you supply to each item emitted
 * by the source Observable, where that function returns an Observable, and then merging those resulting
 * Observables and emitting the results of this merger.
 *
 * @param <R> the value type of the inner Observables and the output type
 * @param func
 *            a function that, when applied to an item emitted by the source Observable, returns an
 *            Observable
 * @return an Observable that emits the result of applying the transformation function to each item emitted
 *         by the source Observable and merging the results of the Observables obtained from this
 *         transformation
 */
public final Observable flatMap(Func1 func) {
    //...
    return merge(map(func)); //map(func) 返回一个Observable, 再调用merge方法
}

/**
 * Flattens an Observable that emits Observables into a single Observable that emits the items emitted by
 * those Observables, without any transformation.
 *
 * @param <T> the common element base type
 * @param source
 *            an Observable that emits Observables
 * @return an Observable that emits items that are the result of flattening the Observables emitted by the
 *         {@code source} Observable
 */
public static Observable merge(Observable source) {
    //...
    return source.lift(OperatorMerge.<T>instance(false));  // 这个OperatorMerge类很重要
}
```
[merge方法的官方介绍](http://reactivex.io/documentation/operators/merge.html)
对lift具体实现有兴趣的可以看[变换的基础方法: lift](),

```
// 伪代码
// 原代码位置: RxJava/src/main/java/rx/internal/operators/OperatorMerge.java
public final class OperatorMerge<T> implements Operator<T, Observable<? extends T>> {
  static final class MergeSubscriber<T> extends Subscriber<Observable<? extends T>> {}
  static final class MergeProducer<T> extends AtomicLong implements Producer {}
  static final class InnerSubscriber<T> extends Subscriber<T> {}
}


```
