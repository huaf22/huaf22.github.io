## [RxJava学习笔记]-Observable与Subscriber的关系
<!--more-->

### 最简单的观察者模式
```
//被观察者
public class Observable {
	public interface Listener {
		void call();
    }

    Observable.Listener mListener = null;

    public static Observable create(Listener listener) {
    	mListener = listener;
    }

    public void someMethod() {
		mListener.call();
    }
}

//创建被观察者对象, 传入Observable.Listener对象
Observable observable = Observable.create(new Observable.Listener() {

	@override
	void call() {
	  //do something
   }

```

 ### 抽象出来一个观察者接口(Subscriber), 在调用被观察者的方法时被作为参数回调

```
public class Observable {

	public interface onSubscribe {
		void call(Subscriber subscriber);
    }

	Observable.onSubscribe mOnSubscribe = null;

    public static Observable create(onSubscribe pOnSubscribe) {
    	mOnSubscribe = pOnSubscribe;
    }

    public void subscribe(Subscriber subscribe) {
    	mOnSubscribe.call(subscribe);  //把subscribe作为call方法的参数回调
    }
}

//观察者
public interface Subscriber {
	void onNext(String string);
}

Observable observable = Observable.create(new Observable.onSubscribe() {

	@override
	void call(Subscriber subscriber) {
		//do something
		subscriber.next("1");
		subscriber.next("2");
		subscriber.next("3");
	}
});
```
这时如果新建一个Subscriber对象, 并将它传参到observable对象的subscribe方法中:
```
Subscriber subscriber = new Subscriber() {
    @override
    void onNext(String string) {
      // 打印string
   }
};
observable.subscribe(subscriber);

//subscriber对象打印出
"1"
"2"
"3"
```
将代码连贯起来
```
Observable.create(new Observable.onSubscribe() {
	@override
	void call(Subscriber subscriber) {
		//do something
		subscriber.next("1");
		subscriber.next("2");
		subscriber.next("3");
	}
}).subscribe(new Subscriber() {
    @override
    void onNext(String string) {
      // 打印string
   }
});
```
### RxJava提供了一些Observable的创建方法, 不需要实现Observable.onSubscribe接口
```
Observable.just(1, 2, 3, 4) 
.subscribe(new Action1<Integer>() {
   @Override public void call(Integer number) {
       Log.d(tag, "number:" + number);
   }
 });
```
其中, 在RxJava中Subscriber是Observer的子类
