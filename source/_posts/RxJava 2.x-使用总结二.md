---
title: RxJava 2.x 使用总结二
tags: RxJava
abbrlink: 26707
date: 2018-04-10 17:42:21
---
传送门:[RxJava 2.x 使用总结<一>](https://bytemr.github.io/2018/04/10/RxJava%202.x%20%E4%BD%BF%E7%94%A8%E6%80%BB%E7%BB%93%E4%B8%80/)


----------


## interval ##
interval操作符是每隔一段时间就产生一个数字，这些数字从0开始，一次递增1直至无穷大，这个间隔时间可以设置，时间单位也可以设置。  
```java
System.out.println("Thread : "+Thread.currentThread().getName());
final Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS);
observable.subscribe(new Consumer<Long>() {
@Override
public void accept(Long value) throws Exception {
    System.out.println("Consumer.accept :  value = "+value+" , Thread = "+Thread.currentThread().getName());
}
});
```
上面的代码执行结果为:

    Thread : main
    Consumer.accept :  value = 0 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 1 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 2 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 3 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 4 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 5 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 6 , Thread = RxComputationThreadPool-1
    ...此处省略N行打印
    
  通过上面打印我们可以看到使用interval(...)方法，会按照该方法传递的参数值，第一个参数为时间数值，第二个参数为单位，上面的例子为，每个一秒产生一个事件，并传递数值，其数值是从0开始


递增1的，而且是会无穷大的递增。  
  
  然后细心的人可能会发现，上面的打印我是多加了一个对线程名字的打印，我们发现，我们在主线程中建立订阅关系，然后事件的接受消费是在一个名字为RxComputationThreadPool-1中执行的，其实这个线程就是一个子线程，其实很正常啊，我们在项目开发中，执行定时任务或者或者间隔任务，其实都是一种耗时任务，会阻塞的，所以都会放在子线程中去执行的。
## take方法妙用 ##

结合上面的interval方法，我们在添点油加点醋。我们先来看个例子，应该就最能清楚take的具体作用了。
```java
System.out.println("Thread : " + Thread.currentThread().getName());
final Observable<Long> observable = Observable.interval(1,TimeUnit.SECONDS).take(5);
observable.subscribe(new Consumer<Long>() {
	@Override
	public void accept(Long value) throws Exception {
		System.out.println("Consumer.accept :  value = " + value+ " , Thread = " + Thread.currentThread().getName());
	}
});
```


我们可以看到，我们调用了take(5)方法，传递了一个参数5。我们来直接看看运行结果吧。

    Thread : main
    Consumer.accept :  value = 0 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 1 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 2 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 3 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 4 , Thread = RxComputationThreadPool-1
    
我们可以看到，我们运行结果只打印了五个数值，从0-4，也就是我们调用take(5),其实起作用，大家也不言而喻了吧。

## 重复发送的repeat ##
再接上面的例子，我们再继而调用一下repeat方法，看看会有什么结果呢？
```java
System.out.println("Thread : "+Thread.currentThread().getName());
final Observable<Long> observable = Observable.interval(1, TimeUnit.SECONDS).take(4).repeat(2);
observable.subscribe(new Consumer<Long>() {
@Override
public void accept(Long value) throws Exception {
    System.out.println("Consumer.accept :  value = "+value+" , Thread = "+Thread.currentThread().getName());
}
});
```
运行结果会如何呢？  

    Thread : main
    Consumer.accept :  value = 0 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 1 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 2 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 3 , Thread = RxComputationThreadPool-1
    Consumer.accept :  value = 0 , Thread = RxComputationThreadPool-2
    Consumer.accept :  value = 1 , Thread = RxComputationThreadPool-2
    Consumer.accept :  value = 2 , Thread = RxComputationThreadPool-2
    Consumer.accept :  value = 3 , Thread = RxComputationThreadPool-2
由此可见，打印0-3重复了2一遍，那也就是我们调用repeat(2)其到了作用，我们知道不调用这个方法，我们只会打印一遍。

## range方法发送特定的整数序列 ##
> range可以创建发射特定整数序列的 Observable。
>> range( int start , int end ) //start :开始的值 ， end ：结束的值
>>> 要求： end >= start

```java 
Observable.range(1, 5).subscribe(new Consumer<Integer>() {
@Override
public void accept(Integer value) throws Exception {
	System.out.println("Consumer.accept : value = "+value);
}
});
```

看看执行结果：

    Consumer.accept : value = 1
    Consumer.accept : value = 2
    Consumer.accept : value = 3
    Consumer.accept : value = 4
    Consumer.accept : value = 5
    
很明显，打印了从1-5的连续的整数序列。

## flatMap 方法 ##
flatMap是变换操作符，使用一个指定的函数对原始Observable发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的Observable，然后flatMap合并这些Observable发射的数据，最后将合并后的结果当作它自己的数据序列发射。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
@Override
public void subscribe(ObservableEmitter<Integer> emitter)throws Exception {
	emitter.onNext(1);
	emitter.onNext(2);
	emitter.onNext(3);
}
}).flatMap(new Function<Integer, ObservableSource<String>>() {
	@Override
	public ObservableSource<String> apply(Integer value)throws Exception {
	    System.out.println("flatMap.apply.value = "+value);
		if(value ==1){
			 return Observable.just("One");
		}else if(value ==2){
			 return Observable.just("Two");
		}else if(value ==3){
			 return Observable.just("Three");
		}else{
			 return Observable.just("Unknow");
		}
	}
}).subscribe(new Consumer<String>() {
	@Override
	public void accept(String content) throws Exception {
	    System.out.println("Consumer.accept :  content = "+content);
	}
});
```
我们来看看执行结果吧：

    flatMap.apply.value = 1
    Consumer.accept :  content = One
    flatMap.apply.value = 2
    Consumer.accept :  content = Two
    flatMap.apply.value = 3
    Consumer.accept :  content = Three

通过结果，表面上看起来和map没有什么区别，map是可以在apply方法中直接返回转换后的数据，但是在flatMap中，是必须返回一个独立的ObservableSource对象的，我们知道Observable是继承自ObservableSource的。

我们就flatMap方法，我们再来看一个例子。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter)throws Exception {
    	emitter.onNext(1);
    	emitter.onNext(2);
    	emitter.onNext(3);
	}
}).flatMap(new Function<Integer, ObservableSource<String>>() {
	@Override
	public ObservableSource<String> apply(Integer value)throws Exception {
	System.out.println("flatMap.apply.value = "+value);
	if(value ==1){
	 return Observable.just("One").delay(3, TimeUnit.SECONDS);
	}else if(value ==2){
	 return Observable.just("Two").delay(2, TimeUnit.SECONDS);
	}else if(value ==3){
	 return Observable.just("Three").delay(1, TimeUnit.SECONDS);
	}else{
        return Observable.just("Unknow").delay(1, TimeUnit.SECONDS);
	}
	}
}).subscribe(new Consumer<String>() {
	@Override
	public void accept(String content) throws Exception {
		System.out.println("Consumer.accept :  content = "+content+" , Thread : "+Thread.currentThread().getName());
	}
});

```
上面的代码，我们调用了delay，延时，我们先看结果。

    flatMap.apply.value = 1
    flatMap.apply.value = 2
    flatMap.apply.value = 3
    Consumer.accept :  content = Three , Thread : RxNewThreadScheduler-3
    Consumer.accept :  content = Two , Thread : RxNewThreadScheduler-2
    Consumer.accept :  content = One , Thread : RxNewThreadScheduler-1
    
可以看到先打印了Three，然后是Two，最后是One，和之前的结果不一样了，而且线程都不同的子线程(如果不调用delay的话，默认都在订阅所在线程)，顺序改变了，可是有时候，我们需要延时，却并不要，改变执行顺序，依然需要One、Two、Three...这样的一个顺序，那么我们该怎么办？接下来我们需要讲讲concatMap操作。

## concatMap ##
我们上一个话题说到使用flatMap操作时候，如果最终转换出来的Observable是调用了delay的话且又都在不同的线程，如何去保证原始顺序呢？
所以我们才需要concatMap这个方法，其实简答的来说，concatMap和flatMap的区别就在于，concatMap保证了其事件派发的顺序。

这里我们其他地方都不改动，直接把flatMap替换成concatMap再来看看结果吧。
```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(ObservableEmitter<Integer> emitter)throws Exception {
    	emitter.onNext(1);
    	emitter.onNext(2);
    	emitter.onNext(3);
    }
}).concatMap(new Function<Integer, ObservableSource<String>>() {
    @Override
    public ObservableSource<String> apply(Integer value)throws Exception {
    	System.out.println("flatMap.apply.value = "+value);
    	if(value ==1){
    		 return Observable.just("One").delay(3, TimeUnit.SECONDS);
    	}else if(value ==2){
    		 return Observable.just("Two").delay(2, TimeUnit.SECONDS);
    	}else if(value ==3){
    		 return Observable.just("Three").delay(1, TimeUnit.SECONDS);
    	}else{
    		 return Observable.just("Unknow").delay(1, TimeUnit.SECONDS);
    	}
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(String content) throws Exception {
    	System.out.println("Consumer.accept :  content = "+content+" , Thread : "+Thread.currentThread().getName());
    }
});

```
那么接下来，我们直接看看运行效果呗。  

    flatMap.apply.value = 1
    Consumer.accept :  content = One , Thread : RxComputationThreadPool-1
    flatMap.apply.value = 2
    Consumer.accept :  content = Two , Thread : RxComputationThreadPool-2
    flatMap.apply.value = 3
    Consumer.accept :  content = Three , Thread : RxComputationThreadPool-3

OK，结果非常明显，不管delay如何变化，都能保证了顺序。

## fromIterable ##
有这样一个使用场景，假设我们发射一个list格式的数据，Observer需要逐一去接收list中的数据并作出相应的操作，像这种场景我们该如何实现呢？

不管三七二十一，我们先来看一段代码。
```java
Observable.create(new ObservableOnSubscribe<List<String>>() {
	@Override
	public void subscribe(ObservableEmitter<List<String>> emitter)throws Exception {
		List<String> list = new ArrayList<String>();
		list.add("A");
		list.add("B");
		list.add("C");
		list.add("D");
		emitter.onNext(list);
	}
}).flatMap(new Function<List<String>, ObservableSource<String>>() {

	@Override
	public ObservableSource<String> apply(List<String> list)
			throws Exception {
		return Observable.fromIterable(list);
	}
}).subscribe(new Consumer<String>() {

	@Override
	public void accept(String content) throws Exception {
		System.out.println("Consumer.accept : content = "+content);
	}
});
Observable.create(new ObservableOnSubscribe<List<String>>() {
	@Override
	public void subscribe(ObservableEmitter<List<String>> emitter)throws Exception {
		List<String> list = new ArrayList<String>();
		list.add("A");
		list.add("B");
		list.add("C");
		list.add("D");
		emitter.onNext(list);
	}
}).flatMap(new Function<List<String>, ObservableSource<String>>() {

	@Override
	public ObservableSource<String> apply(List<String> list)
			throws Exception {
		return Observable.fromIterable(list);
	}
}).subscribe(new Consumer<String>() {

	@Override
	public void accept(String content) throws Exception {
		System.out.println("Consumer.accept : content = "+content);
	}
});
```
上面的代码，我们可以看到，我们发射一个list集合，然后我们使用拿到集合之后，使用flatMap重新进行了转换，我们发现我们直接调用了Observable的fromIterable方法，传入list集合，最后，我们在接收的时候，就跟遍历list集合一样，逐一的打印了list集合中的数据，但是当然你如果有其他的附件操作也是可以的。

与fromIterable异曲同工之妙的**fromArray** 其实效果是一样的，只不过，Observalbe原始发射的是一个数组，这里就不在讲述其使用方法了。

既然讲到数组集合了，那么我们接下来也应该说一说**toList** 了
## toList  ##
这个就比较简单了，从字面上来说，无非就是把一组数据，转成一个集合？
```java

Observable.just("item1", "item2", "item3", "item4", "item5")
.toList()
.subscribe(new Consumer<List<String>>() {
@Override
public void accept(List<String> list) throws Exception {
		System.out.println("Consumer.accept :  list = "+list);
}
});

```
就直接看结果吧。

    Consumer.accept :  list = [item1, item2, item3, item4, item5]
很显然，直接将一组数据转成了一个list发射。

## 背压 BackPressure ##
**背压产生的原因：**  
> 被观察者发送消息太快以至于它的操作符或者订阅者不能及时处理相关的消息。在 Rxjava 1.x 版本很容易就会报错，使程序发生崩溃。

    ...
    Caused by: rx.exceptions.MissingBackpressureException
    ...
    ...

我们知道在RxJava 1.x中是不支持背压的，但是在2.X之后支持背压了，可是在Observable和Observer依然还是不支持，不过2.X增加了Flowable（被观察者），Flowable （被观察者）和Subscriber （观察者）配合可以完美实现背压处理。

先来看一段不使用背压处理时候的代码。

```java
Observable.interval(1, TimeUnit.MILLISECONDS)
  .subscribeOn(Schedulers.io())
  .observeOn(Schedulers.newThread())
  .subscribe(new Consumer<Long>() {
        @Override
        public void accept(Long value) throws Exception {
            Thread.sleep(1000);
            System.out.println("value : "+value);
        }
});
```
这段代码中，发射器发射数据的频率为1毫秒，而消费者以1秒处理一个事件的能力在处理，很显然，这会导致RxJava对已经发射出去且未被处理的数据进行了缓存，都存在于内存中，这样一段代码在执行的时候并不会报错，因为这样的组合并不支持背压，并不会抛出MissingBackpressureException背压异常，但是这样的代码其实有很大的问题的。
1. 存在延迟缓存的话，会导致无法即使处理关键、敏感、重要信息。
2. 如果数据量不确定，可能比较大的情况下，都缓存在内存中的话，会占用大量的内存，是不明智的做法。

然后我们如何通过背压处理来解决呢？
```java
Flowable.interval(1, TimeUnit.MILLISECONDS)
.subscribeOn(Schedulers.io())
.observeOn(Schedulers.newThread())
.subscribe(new Consumer<Long>() {
      @Override
       public void accept(Long value) throws Exception {
            Thread.sleep(1000);
            System.out.println("Consumer.accept: value = "+value);
        }
 });
```
我们知道RxJava 2.X中Flowable支持了背压，可是上面的代码然后就解决问题了吗?运行结果会如何呢？

    io.reactivex.exceptions.OnErrorNotImplementedException: Can't deliver value 128 due to lack of requests
    ...
    ...
    Caused by: io.reactivex.exceptions.MissingBackpressureException:     Can't deliver value 128 due to lack of requests
    
很明显发生了 MissingBackpressureException 异常 , 128 代表是 Flowable 最多缓存 128 个数据，缓存次超过 128 个数据，就会报错。可喜的是，Rxjava 已经给我们提供了解决背压的策略。
###背压策略onBackpressureDrop###
> onBackpressureDrop() ：当缓冲区数据满 128             个时候，再新来的数据就会被丢弃，如果此时有数据被消费了，那么就会把当前最新产生的数据，放到缓冲区。简单来说 Drop 就是直接把存不下的事件丢弃。


```java
Flowable.interval(1, TimeUnit.MILLISECONDS)
.onBackpressureDrop()
.subscribeOn(Schedulers.io())
.observeOn(Schedulers.newThread())
.subscribe(new Consumer<Long>() {
      @Override
       public void accept(Long value) throws Exception {
            Thread.sleep(1000);
            System.out.println("Consumer.accept: value = "+value);
        }
 });
```

上面的代码，我们调用了onBackpressureDrop()方法，然后来看看结果。

    ...
    Consumer.accept: value = 124
    Consumer.accept: value = 125
    Consumer.accept: value = 126
    Consumer.accept: value = 127
    Consumer.accept: value = 96093
    Consumer.accept: value = 96094
    Consumer.accept: value = 96095
    Consumer.accept: value = 96096
    Consumer.accept: value = 96097
    ...
我们发现发射器发射的 0 ~ 127 总共 128 个数据是连续的，下一个数据就是 96129 ， 128 ~ 96128 的数据被丢弃了。
onBackpressureDrop的调用 一定要放在 interval 后面否则不会生效。
###onBackpressureBuffer###
> onBackpressureBuffer：默认情况下缓存所有的数据，不会丢弃数据，这个方法可以解决背压问题，但是它有像 Observable 一样的缺点，缓存数据太多，占用太多内存。  
> onBackpressureBuffer(int capacity) ：设置缓存队列大小，但是如果缓冲数据超过了设置的值，就会报错，发生崩溃。

示例:
```java
Flowable.interval(1, TimeUnit.MILLISECONDS)
.onBackpressureBuffer(20)
.subscribeOn(Schedulers.io())
.observeOn(Schedulers.newThread())
.subscribe(new Consumer<Long>() {
  @Override
   public void accept(Long value) throws Exception {
        Thread.sleep(1000);
        System.out.println("Consumer.accept: value = "+value);
    }
});
```
执行结果:

io.reactivex.exceptions.OnErrorNotImplementedException: Buffer is full
···
Caused by: io.reactivex.exceptions.MissingBackpressureException: Buffer is full
通过日志可以看出，缓冲区已经满了，抛出了异常。

## Subscriber + Flowable处理背压和Subscription.request(long count)方法
先来看一段代码。
```java
Flowable.create(new FlowableOnSubscribe<Integer>() {
    @Override
    public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
    	System.out.println("subscribe");
        for (int i = 0; i<201 ; i++) {
            emitter.onNext(i);
        }
    }
}, BackpressureStrategy.DROP).subscribe(new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
    	System.out.println("onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
    	System.out.println("onNext:integer = "+integer);
    }

    @Override
    public void onError(Throwable t) {
    	System.out.println("onError ");
    }

    @Override
    public void onComplete() {
    	System.out.println("onComplete ");
    }
});
```
上面的代码，有些人是不是觉得，会直接打印0-127的打印？可是万万没想到的是，只是打印了如下两个日志。

    onSubscribe
    subscribe
那么如何才能接收到发射的事件，然后打印出来呢？那么就需要用到Subscription的request(...)方法了。那么Subscription从哪里来呢？
细心的人可能发现了，在onSubscribe(Subscription s)回调方法中，传递了一个Subscription s对象，那么我们直接在这个回调方法中调用s.request(50);方法试试看有什么效果吧。
```java
Flowable.create(new FlowableOnSubscribe<Integer>() {
    @Override
    public void subscribe(FlowableEmitter<Integer> emitter) throws Exception {
    	System.out.println("subscribe");
        for (int i = 0; i<201 ; i++) {
            emitter.onNext(i);
        }
    }
}, BackpressureStrategy.DROP).subscribe(new Subscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
    	System.out.println("onSubscribe");
    	s.request(10);
    }

    @Override
    public void onNext(Integer integer) {
    	System.out.println("onNext:integer = "+integer);
    }

    @Override
    public void onError(Throwable t) {
    	System.out.println("onError ");
    }

    @Override
    public void onComplete() {
    	System.out.println("onComplete ");
    }
});
```

可以看到，我们在onSubscribe回调方法中调用了s.request(10);来看看其结果吧！

    onSubscribe
    subscribe
    onNext:integer = 0
    onNext:integer = 1
    onNext:integer = 2
    onNext:integer = 3
    onNext:integer = 4
    onNext:integer = 5
    onNext:integer = 6
    onNext:integer = 7
    onNext:integer = 8
    onNext:integer = 9

结果我们发现，你请求多少个数据事件，就会给你发射多少个数据事件，这样我们才能接收到数据并消费数据。

那么我们拿到这个Subscription对象之后，在外部再调用一次request(10)呢？

    onSubscribe
    subscribe
    onNext:integer = 0
    onNext:integer = 1
    onNext:integer = 2
    onNext:integer = 3
    onNext:integer = 4
    onNext:integer = 5
    onNext:integer = 6
    onNext:integer = 7
    onNext:integer = 8
    onNext:integer = 9
    onNext:integer = 10
    onNext:integer = 11
    onNext:integer = 12
    onNext:integer = 13
    onNext:integer = 14
    onNext:integer = 15
    onNext:integer = 16
    onNext:integer = 17
    onNext:integer = 18
    onNext:integer = 19
我们发现我们是从第一次调用request(10)之后，又接收消费了从9-19的数据事件，那么我们从中得到的规律是，每次调用request(..)方法，都会从上一次发射的最后一个事件的下一个事件开发发送请求数量的事件。

