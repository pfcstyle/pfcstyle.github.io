---
layout:		post
title:		"Combine详解（1）"
description: ""
date:		2021-02-24
author:		"Yawei"
categories: [iOS, Swift, Combine]
keywords:
    - iOS
    - Swift
    - Combine
---

# Combine基础

* Publisher是发布者，作为数据生产者，会定义一个Output和Failure类型。
* Subscriber是订阅者，作为数据消费者，会定义一个Input合Failure类型。
* Operator是操作者，可以更改Publisher Output类型，产生新的Publisher。
* Publisher的Output类型和Subsciber的Input类型需要相同
  
> 订阅操作是通过`Publisher.subscribe(Subscriber)`来完成的
> 获取到Subscribtion(Publisher.subscibe()返回)实例，可以调用`.cancel()`取消

```
sub?.cancel()
```

# 内置Subscriber

* **sink(receiveCompletion:receiveValue:)**: 常用于publisher和sink一起执行的时候，比如request请求，返回的response会在sink中处理
* **assign(to:on:)**: 通过key path将收到的值赋值到model

```Swift
// publisher之后直接sink
let sub = NotificationCenter.default
    .publisher(for: NSControl.textDidChangeNotification, object: filterField)
    .sink(receiveCompletion: { print ($0) },
          receiveValue: { print ($0) })

```

```Swift
let sub = NotificationCenter.default
    .publisher(for: NSControl.textDidChangeNotification, object: filterField)
    .map( { ($0.object as! NSTextField).stringValue } )
    .assign(to: \MyViewModel.filterString, on: myViewModel)
```

# 通过operator定制publisher
```
let sub = NotificationCenter.default
    .publisher(for: NSControl.textDidChangeNotification, object: filterField)
    .map( { ($0.object as! NSTextField).stringValue } )
    // 过滤非字母和数字字符
    .filter( { $0.unicodeScalars.allSatisfy({CharacterSet.alphanumerics.contains($0)}) } )
    // 防抖
    .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
    // receive方法可以让combine在指定线程里调用subscriber
    .receive(on: RunLoop.main)
    .assign(to:\MyViewModel.filterString, on: myViewModel)
```

# share() operator

大多publishers都是struct, 是值类型，每一次订阅，实际上都是订阅了一个副本。share operator则可以赋予它们引用语义。

```swift
// PassthroughSubject是class类型，但是经过scan operator之后生成的是struct类型
let subject = PassthroughSubject<String, Never>()
// 这里每次emit value, 第一个参数都会累加
let publisher = subject.scan((0, ""), { ($0.0 + 1, $1) })
// subscribe一个
let c1 = publisher.sink { print("c1:", $0) }
subject.send("a")
subject.send("b")

// output
c1: (1, "a")
c1: (2, "b")
// 再次subscribe
let c2 = publisher.sink { print("c2:", $0) }
subject.send("c")

// output  可以发现，对两个subscriber来说，是两个struct, 上述的第一个参数会从0重新开始累加
c1: (3, "c")
c2: (1, "c")

// 如果加上share
publisher = subject.scan((0, ""), { ($0.0 + 1, $1) }).share()

// output
c1: (3, "c")
c2: (3, "c")
```

# Connectable Publishers

有时会遇到如下图的问题，多个subscriber订阅同一个publisher，新的subscriber在订阅时，publisher可能已经emit一部分数据了。为了保证所有的subsciber都订阅以后publisher再开始执行，这就需要用到Connectable Publishers.

![](/img/post/2021-02-24/connectable-publisher.png)

想要生成Connectable Publishers，只需要让publisher执行`makeConnectable()`operator即可，如下代码所示。当所有的subscriber都注册好之后，执行`.connect`函数即可让publisher开始执行

```
let url = URL(string: "https://example.com/")!
let connectable = URLSession.shared
    .dataTaskPublisher(for: url)
    .map() { $0.data }
    .catch() { _ in Just(Data() )}
    .share()
    .makeConnectable()

cancellable1 = connectable
    .sink(receiveCompletion: { print("Received completion 1: \($0).") },
          receiveValue: { print("Received data 1: \($0.count) bytes.") })

DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
    self.cancellable2 = connectable
        .sink(receiveCompletion: { print("Received completion 2: \($0).") },
              receiveValue: { print("Received data 2: \($0.count) bytes.") })
}

DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    self.connection = connectable.connect()
}
```

另外有一些内置的Connectable Publisher, 如`Publishers.Multicast`和`Timer.TimerPublisher`, 这些必须手动调用`.connect()`才可以执行，有时也会造成问题。这时，可以调用`.autoconnect()`, 那么subscriber可以立刻执行。

```
let cancellable = Timer.publish(every: 1, on: .main, in: .default)
    .autoconnect()
    .sink() { date in
        print ("Date now: \(date)")
     }
```

> 可以调用`.cancel()`取消publisher

# Subscriber控制接收Publisher elements的数量
有两种方法
* request(_:)
* 当publisher调用subscriber的receive(:)方法时返回一个新的demand

常用的`sink`和`assign`默认是声明了`unlimited`demand， 一旦声明了`unlimited`就无法再更改了。

```Swift
// Publisher: Uses a timer to emit the date once per second.
let timerPub = Timer.publish(every: 1, on: .main, in: .default)
    .autoconnect()

// Subscriber: Waits 5 seconds after subscription, then requests a
// maximum of 3 values.
class MySubscriber: Subscriber {
    typealias Input = Date
    typealias Failure = Never
    var subscription: Subscription?
    
    func receive(subscription: Subscription) {
        print("published                             received")
        self.subscription = subscription
        DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
            // request请求新的demand, 可以累加
            subscription.request(.max(3))
        }
    }
    
    func receive(_ input: Date) -> Subscribers.Demand {
        print("\(input)             \(Date())")
        // 直接返回新的Demand
        return Subscribers.Demand.none
    }
    
    func receive(completion: Subscribers.Completion<Never>) {
        print ("--done--")
    }
}

// Subscribe to timerPub.
let mySub = MySubscriber()
print ("Subscribing at \(Date())")
timerPub.subscribe(mySub)

// Output

Subscribing at 2019-12-09 18:57:06 +0000
published                             received
2019-12-09 18:57:11 +0000             2019-12-09 18:57:11 +0000
2019-12-09 18:57:12 +0000             2019-12-09 18:57:12 +0000
2019-12-09 18:57:13 +0000             2019-12-09 18:57:13 +0000
```

## 通过Back-Pressure(反压) Operator控制Unlimited Demand
不自定义Subscriber也是可以控制需求的（作用到publisher上）

* buffer(size:prefetch:whenFull:)： 控制到一个固定数量，当数量满了之后，可以选择报错，丢弃等操作

* debounce(for:scheduler:options:)： 防抖，当publisher停止publish达到某个时间后才会收到数据

* throttle(for:scheduler:latest:)： 减流，会有一个最大产生数据的速率，如果收到太多，会只发送最新或最老的数据

* collect(_:)和collect(_:options:)： 将指定数量或时间的所有元素打包成一个数组一次性发送给订阅者
