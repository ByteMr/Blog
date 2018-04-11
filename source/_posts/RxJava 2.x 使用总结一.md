---
title: RxJava 2.x 使用总结<一>
tags: RxJava
abbrlink: 15279
date: 2018-04-10 09:58:17
---

本文主要记录一下RxJava2.x的使用，其实网络上已经有很多关于RxJava的使用教程，以及原理剖析类的文章，其实我的理解大概也和那些文章中描述的也差不多，所以这里就不在过多去说类似的东西了。

在这里推荐一篇文章给大家，虽然是基于RxJava 1.X来写的，但是我觉得这是我目前见过写的比较清晰了然的一片文章，很适合初学者去对RxJava进行一番学习和使用。  

[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083#toc_1)  


# 基本使用 #
## 添加依赖 ##
```java
implementation 'io.reactivex.rxjava2:rxjava:2.1.12'
implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
implementation 'org.reactivestreams:reactive-streams:1.0.2'
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
implementation 'com.squareup.retrofit2:retrofit:2.4.0'
implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
implementation 'com.squareup.okhttp3:logging-interceptor:3.10.0'
implementation 'com.squareup.retrofit2:converter-scalars:2.4.0'
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.4.0'
```
当然，上面添加的依赖有点多，但是其实你如果只需要使用RxJava只需要添加如下两个依赖就行 。
```java
implementation 'io.reactivex.rxjava2:rxjava:2.1.12'
implementation 'org.reactivestreams:reactive-streams:1.0.2'
```
但是你如果想要在android上面使用的话，就必须再多加上如下这个依赖。
```java
	implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
```

我们知道现在主流用发就是，RxJava+OkHttp+Retrofit，这三者结合使用是让我们的代码更加简单有层次，提升性能有提升开发速度，所以你如果又要在你的项目中需要请求网络，需要传输json数据的话，不妨再加上如下几个依赖。
```java
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
implementation 'com.squareup.retrofit2:retrofit:2.4.0'
implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
implementation 'com.squareup.okhttp3:logging-interceptor:3.10.0'
implementation 'com.squareup.retrofit2:converter-scalars:2.4.0'
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.4.0'
```
## 使用create(...)创建Observable ##
create(...)方法是Observable对象的一个静态方法，主要用于创建产生一个Observable被观察者对象。
```
final Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> emitter)throws Exception {
    	emitter.onNext("Hello!");
    	emitter.onNext("RxJava");
    	emitter.onNext("Hi!");
    	emitter.onNext("Android");
    }
});
```

上面的代码我们使用create(...)方法创建了一个被观察者Observable，那么我们知道观察者Observer和被观察者Observable之间事件的订阅关系是通过subscribe(...)方法来实现的，一般来说，订阅事件是发生在观察者身上的，因为观察者需要关注订阅发生在被观察者身上的一系列事件，但是在RxJava中，为了方便链式的调用以及RxJava的架构，这个subscribe(...)方法声明在了Observable对象上，现在我们先来创建一个观察者Observer。
```
final Observer<String> observer = new Observer<String>() {
	@Override
	public void onSubscribe(Disposable disposable) {
		System.out.println("Observer.onSubscribe :  isDisposable = "  +disposable.isDisposed());
	}
	@Override
	public void onNext(String content) {
		System.out.println("Observer.onNext :  content = "  +content);
	}
	@Override
	public void onError(Throwable throwable) {
		System.out.println("Observer.onError ");
	}
	@Override
	public void onComplete() {
		System.out.println("Observer.onComplete ");
	}
};

```
Observer我们创建好了，我们发现需要Override四个方法。
```
public void onSubscribe(Disposable disposable)；
public void onNext(String content) ；
public void onError(Throwable throwable)；
public void onComplete()；
```

但是细心的就会发现，我们Observable持有的一个发射器ObservableEmitter，这个对象也有三个比较关键的方法。  

不然发现，是onNext()、onError()、onComplete()三个方法，看起来是和Observer中其中的三个需要Override的方法是对应的。

这样我就把观察者Observer和被观察者Observable创建好了，接下来要做的就是使用subscribe(...)方法把他们关联起来。
```
observable.subscribe(observer);
```

OK，这就把两者的关系给绑定起来了，另外要注意的是，这个方法一旦调用，也就是说，一旦注册订阅，被观察着的事件发射器就开发发射事件，接着观察者的方法就会被被调用。
代码的运行结果吧！

        Observer.onSubscribe :  isDisposable = false
		Observer.onNext :  content = Hello!
		Observer.onNext :  content = RxJava
		Observer.onNext :  content = Hi!
		Observer.onNext :  content = Android

从上面的打印来看，当订阅成功后，会立马先调用Observer的onSubscribe方法，然后会依次按顺序打印了Hello!->RxJava->Hi!->Android  

从上面的代码中，我们知道，和1.x想比，2.X多了一个**Disposable**，我们可以看到在onSubscribe()方法中会传递过来一个Disposable对象，那么这个Disposable其实可以看做是连接Observer和Observable的一个开关，拿到它之后，可以直接调用切断，来解除Observer对Observable的关注和订阅，但是这也意味着，Observer不在能收到订阅事件了，所以Disposable的dispose()方法就可以切断两者的事件驱动，当Disposable的isDisposed()方法返回false的时候，表明正常，可以发送接收事件，但是为true的时候，表明两者事件驱动被切断。  

## 调用onComplete()会如何？##
我们发现上面的实例代码中，仅仅只是调用了onNext(...)方法触发了四个事件，然后Observer接收处理了相应的事件，我们稍微改动一下代码，然后在我们的代码中调用一下onComplete()看看会有什么效果呢？
```
final Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
@Override
public void subscribe(ObservableEmitter<String> emitter)throws Exception {
	emitter.onNext("Hello!");
	emitter.onNext("RxJava");
	emitter.onComplete();
	emitter.onNext("Hi!");
	emitter.onNext("Android");
}
});
```
我们看到，在subscribe(...)方法中，我在四个onNext()调用顺序之间加了一句代码，即调用了oComplete方法，然后我们再来执行一下代码，看看会有什么结果。

    Observer.onSubscribe :  isDisposable = false
	Observer.onNext :  content = Hello!
	Observer.onNext :  content = RxJava
	Observer.onComplete 

结果，我们发现，我们只收到了调用onComplete()方法之前的两个onNext事件，然后就直接执行了Observer的onComplete()方法，由此可知，当调用发射器的onComplete()方法之后，后面的事件是无法再收到了，但是事件的发送还在继续。  

## 简化的Consumer ##
有时候我们觉得使用Observer比较复杂，我们知道Observer是一个接口，如果要实现Observer接口，那么必须覆盖其四个抽象方法，比如有些时候我们只关注订阅的事件，只对订阅的事件的发生做出
相应的操作，那么一般我们只需要Override  onNext(...){}即可。  

如果说RxJava中两个重要的角色关系是观察者和被观察的关系的话，被观察者产生事件，观察者响应事件，那么是不是也可以理解为这是一种生产者和消费者的关系，很显然这么理解是可以的。  

所以RxJava中有一个用起来比简单的Consumer，实现Consumer 接口只需要实现一个方法，即accept(...)，当然Observable和Consumer之间也是通过Observable的subscribe来建立订阅关系的，OK我们之间写一段代码来运行试试看。
```
final Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
@Override
public void subscribe(ObservableEmitter<String> emitter)throws Exception {
	emitter.onNext("Hello!");
	emitter.onNext("RxJava");
	emitter.onComplete();
	emitter.onNext("Hi!");
	emitter.onNext("Android");
}
});
final Consumer<String> consumer = new Consumer<String>() {		
@Override
public void accept(String content) throws Exception {
		System.out.println("Consumer.accept : content = "+content);
}
};

final Disposable disposable = observable.subscribe(consumer);
```
我们的Observable没有变，还有前面创建的对象，我们来试试运行结果。

        Consumer.accept : content = Hello!
		Consumer.accept : content = RxJava

我们发现Consumer的accept()方法只调用了两次，只打印了“Hello！” 和 “RxJava”，和使用Observer的不同之处在于，Observer会关注Observable的任何一个操作，比如，onError()、onComplete()方法，而Consumer就比较简单了，只需要关注和响应onNext(...)事件就行了，有时候我们可能就只需这样简单的场景，当然这是RxJava 2.x出现的特性。

但是我们还发现，使用订阅Consumer的时候，会直接返回一个Disposable对象。
## 事件的转换包装器map ##

先不说map的功能，我们先直接看一段代码，再来谈谈map的功能。
```
Observable.create(new ObservableOnSubscribe<Integer>() {
	@Override
	public void subscribe(ObservableEmitter<Integer> emitter)throws Exception {
		emitter.onNext(100);
		emitter.onNext(200);
		emitter.onNext(300);
		emitter.onNext(400);
	}
}).map(new Function<Integer, String>() {

	@Override
	public String apply(Integer num) throws Exception {
		System.out.println("Function.apply : num = "+num);
		return "我得了"+num+"分";
	}
}).subscribe(new Consumer<String>() {
	@Override
	public void accept(String content) throws Exception {
		System.out.println("Consumer.accept : content = "+content);
	}
});
```
运行结果: 

    	Function.apply : num = 100
		Consumer.accept : content = 我得了100分
		Function.apply : num = 200
		Consumer.accept : content = 我得了200分
		Function.apply : num = 300
		Consumer.accept : content = 我得了300分
		Function.apply : num = 400
		Consumer.accept : content = 我得了400分

可以看到，是先执行了map节点中Function对象的apply方法，这个方法接收一个Integer类型的数据，然后返回一个String类型的数据，继apply(...)方法之后是Consumer的accept(...)方法，而accept收到的参数是一个String类型，其值就是通过Function对象的apply方法返回来的。  

那么不难发现，我们可以把map节点中的Function看做一个工厂，把Integer泛型限定的Observable(Observable(Integer))转换成了String类型的Observable(Observable<String>)。

其实我们把每个节点分开来写会更加清晰一点。
```
final Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
	@Override
	public void subscribe(ObservableEmitter<Integer> emitter)throws Exception {
		emitter.onNext(100);
		emitter.onNext(200);
		emitter.onNext(300);
		emitter.onNext(400);
	}
});
```

不难发现，首先初始是一个Observable<Integer>类型的，当我们调用起map方法之后就变成了Observable<String>类型了。
```
final Observable<String> observable2 = observable.map(new Function<Integer, String>() {
	@Override
	public String apply(Integer num) throws Exception {
		System.out.println("Function.apply : num = "+num);
		return "我得了"+num+"分";
	}
});
```


最后再调用了subscribe方法进行关联订阅。
```
observable2.subscribe(new Consumer<String>() {
	@Override
	public void accept(String content) throws Exception {
		System.out.println("Consumer.accept : content = "+content);
	}
});
```

所以通过上面的一个小例子，我们不难发现，map基本作用就是将一个 Observable 通过某种函数关系，转换为另一种 Observable，上面例子中就是把我们的 Integer 数据变成了 String 类型。

## 事件组合器zip ##
```
final Observable<Integer> integerObservable = Observable.create(new ObservableOnSubscribe<Integer>() {
	@Override
	public void subscribe(ObservableEmitter<Integer> emitter)throws Exception {
		emitter.onNext(100);
		emitter.onNext(80);
		emitter.onNext(60);
	}
});
			
final Observable<String> stringObservable = Observable.create(new ObservableOnSubscribe<String>() {
	@Override
	public void subscribe(ObservableEmitter<String> emitter)throws Exception {
		emitter.onNext("优");
		emitter.onNext("良");
		emitter.onNext("及格");
	}
});
			
final Observable<String> resultObservable = Observable.zip(integerObservable,stringObservable, new BiFunction<Integer, String, String>() {
	@Override
	public String apply(Integer score, String desc)throws Exception {
		System.out.println("<zip>.BiFunction.apply : score = "+score+" , desc = "+desc);
		return score +" 分为 "+ desc;
	}
});

resultObservable.subscribe(new Consumer<String>() {
	@Override
	public void accept(String content) throws Exception {
		System.out.println("Consumer.accept : content = "+content);
	}
});
```

那么直接看结果。

        <zip>.BiFunction.apply : score = 100 , desc = 优
		Consumer.accept : content = 100 分为 优
		<zip>.BiFunction.apply : score = 80 , desc = 良
		Consumer.accept : content = 80 分为 良
		<zip>.BiFunction.apply : score = 60 , desc = 及格
		Consumer.accept : content = 60 分为 及格

我们发现，通过调用Observable的zip(...)方法，方法最后的以参数为一个BiFunction对象，很巧妙的将两个事件进行组合，组合成了一个新的Observable<String>。

zip 组合事件的过程就是分别从integerObservable和stringObservable各取出一个事件来组合，并且一个事件只能被使用一次，组合的顺序是严格按照事件发送的顺序来进行的，所以上面的实例可以看到，100 永远和 “优”结合，80永远和“良”结合等。  

那么会有一个问题，如果我在integerObservable中发送三个事件，在stringObservable发送一个或者两个，或者三个，甚至一个都不发呢?这种情况是如何执行的呢？不妨来来试试看吧，我们改动一下stringObservable里面的逻辑吧。
```
final Observable<String> stringObservable = Observable.create(new ObservableOnSubscribe<String>() {
	@Override
	public void subscribe(ObservableEmitter<String> emitter)throws Exception {
		emitter.onNext("优");
		emitter.onNext("良");
	}
});
```
执行结果如下。
```
<zip>.BiFunction.apply : score = 100 , desc = 优
Consumer.accept : content = 100 分为 优
<zip>.BiFunction.apply : score = 80 , desc = 良
Consumer.accept : content = 80 分为 良
```
我们发现少了一个60和"及格"的组合。  
然后我们全部去掉，什么都不发送呢？
```
final Observable<String> stringObservable = Observable.create(new ObservableOnSubscribe<String>() {
	@Override
	public void subscribe(ObservableEmitter<String> emitter)throws Exception {
	}
});
```

再执行，我们发现，什么打印都没有。所以我们可以总结为最终接收到的事件数量是和发送事件最少的那个Observable发送器的发送事件数目相同，如果其中一个什么都没发送，那么什么事件都接收不到。  

