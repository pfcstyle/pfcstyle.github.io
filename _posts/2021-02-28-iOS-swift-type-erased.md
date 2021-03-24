---
layout:		post
title:		"Swift Type-Erasure(类型抹除)"
description: ""
date:		2021-02-28
author:		"Yawei"
categories: [iOS, Swift]
keywords:
    - iOS
    - Swift
    - type-erasure
---

# 什么是Type-Erasure

swift中在引用一些通用协议时，往往需要用到`type erasure`。它使我们能够更轻松地与通用协议进行交互，这些通用协议对将要实现它们的各种类型具有特定的要求。

常见的：
```swift
struct ContentView: View {
    var body: AnyView {
        AnyView(Text("Hello, world!"))
    }
}
 
// 当然，5.1之后有更简单的写法(Opaque return types)
struct ContentView: View {
    var body: some View {
        Text("Hello, world!")
    }
}
```

# 实例
以Equatable标准库中的协议为例。由于都是为了使相等类型的两个值能够按相等性进行比较，因此它使用元Self类型作为其唯一方法要求的参数

```Swift
protocol Equatable {
    static func ==(lhs: Self, rhs: Self) -> Bool
}

```
这种方法的优点在于，它不可能意外地比较两个不相关的相等类型（例如User和String），但是，它也使得它不可能被引用Equatable为独立协议（例如创建类似的数组[Equatable]），因为为了能够使用它，编译器需要知道确切的确切类型实际上符合该协议。

当协议包含关联类型时，也是如此。例如，在这里我们定义了一个Request协议，使我们能够在单个统一的实现后隐藏各种形式的数据请求（例如网络调用，数据库查询和缓存提取）：

```swift
protocol Request {
    associatedtype Response
    associatedtype Error: Swift.Error

    typealias Handler = (Result<Response, Error>) -> Void

    func perform(then handler: @escaping Handler)
}
```

上面的方法为我们提供了相同的权衡方法，Equatable它非常强大，因为它使我们能够为任何类型的请求创建通用的抽象，但是这也使得无法直接引用Request协议本身，如下所示：

```swift
class RequestQueue {
    // Error: protocol 'Request' can only be used as a generic
    // constraint because it has Self or associated type requirements
    func add(_ request: Request,
             handler: @escaping Request.Handler) {
        ...
    }
}
```

解决上述问题的一种方法是准确执行错误消息中所说的内容，而不是Request直接引用，而是将其用作一般约束：
```swift
class RequestQueue {
    func add<R: Request>(_ request: R,
                         handler: @escaping R.Handler) {
        ...
    }
}
```

上面的方法起作用了，因为现在编译器能够保证传递handler的确实与Request传递的实现兼容request—因为它们都基于泛型R，而泛型又被限制为与一致Request。

但是，尽管我们解决了方法签名问题，但实际上仍然无法对传递的请求做很多工作，因为我们无法将其存储为Request属性或[Request]数组的一部分，这将使得继续建立RequestQueue。那么，type-erasure该出场了。

## 通用包装器类型

我们将探讨的第一种类型擦除实际上并不涉及擦除任何类型，而是将它们包装在一个我们可以更容易引用的泛型类型中。继续构建RequestQueue，我们将首先创建该包装器类型，该包装器类型将捕获每个请求的perform方法作为闭包，以及在请求完成后应调用的handler方法：

```swift
// This will let us wrap a Request protocol implementation in a
// generic has the same Response and Error types as the protocol.
struct AnyRequest<Response, Error: Swift.Error> {
    typealias Handler = (Result<Response, Error>) -> Void

    let perform: (@escaping Handler) -> Void
    let handler: Handler
}
```

接下来，实现RequestQueue，为其添加Response和Error类型的泛型，这样编译器可以将泛型类型和associatetype对齐，从而允许我们对Request存储和引用。

> 这个实现时线程不安全的，我们只用来说明类型抹除的问题

```swift
class RequestQueue<Response, Error: Swift.Error> {
    private typealias TypeErasedRequest = AnyRequest<Response, Error>

    private var queue = [TypeErasedRequest]()
    private var ongoing: TypeErasedRequest?

    // We modify our 'add' method to include a 'where' clause that
    // gives us a guarantee that the passed request's associated
    // types match our queue's generic types.
    func add<R: Request>(
        _ request: R,
        handler: @escaping R.Handler
    ) where R.Response == Response, R.Error == Error {
        // To perform our type erasure, we simply create an instance
        // of 'AnyRequest' and pass it the underlying request's
        // 'perform' method as a closure, along with the handler.
        let typeErased = AnyRequest(
            perform: request.perform,
            handler: handler
        )

        // Since we're implementing a queue, we don't want to perform
        // two requests at once, but rather save the request for
        // later in case there's already an ongoing one.
        guard ongoing == nil else {
            queue.append(typeErased)
            return
        }

        perform(typeErased)
    }

    private func perform(_ request: TypeErasedRequest) {
        ongoing = request

        request.perform { [weak self] result in
            request.handler(result)
            self?.ongoing = nil

            // Perform the next request if the queue isn't empty
            ...
        }
    }
}
```

## 闭包实现

使用闭包擦除类型时，其思想是捕获在闭包内部执行操作所需的所有类型信息，并使闭包仅接受非通用（甚至Void）输入。这样一来，我们就可以引用，存储和传递该功能，而无需实际知道其内部发生了什么—从而为我们提供了更大的灵活性。当然，这样实现会让代码变得难以调试，但优点是方便，而且可以完全封装类型信息。

```swift
class RequestQueue {
    private var queue = [() -> Void]()
    private var isPerformingRequest = false

    func add<R: Request>(_ request: R,
                         handler: @escaping R.Handler) {
        // This closure will capture both the request and its
        // handler, without exposing any of that type information
        // outside of it, providing full type erasure.
        let typeErased = {
            request.perform { [weak self] result in
                handler(result)
                self?.isPerformingRequest = false
                self?.performNextIfNeeded()
            }
        }

        queue.append(typeErased)
        performNextIfNeeded()
    }

    private func performNextIfNeeded() {
        guard !isPerformingRequest && !queue.isEmpty else {
            return
        }

        isPerformingRequest = true
        let closure = queue.removeFirst()
        closure()
    }
}
```

## 外部类型抹除
目前，我们都是在内部实现类型抹除，外部调用者不会关心到这个问题。但是，有时在将协议实现传递给API之前进行一些轻量级转换，可以简化更多工作，而且可以更整齐的封装抹除代码。

对我们这个例子来说，可以要求每一个request在添加到队列之前进行专门化管理，这就衍变出了`RequestOperation`

```swift
struct RequestOperation {
    fileprivate let closure: (@escaping () -> Void) -> Void

    func perform(then handler: @escaping () -> Void) {
        closure(handler)
    }
}
```

类似于闭包实现，我们将其放在extension中
```swift
extension Request {
    func makeOperation(with handler: @escaping Handler) -> RequestOperation {
        return RequestOperation { finisher in
            // We actually want to capture 'self' here, since otherwise
            // we risk not retaining the underlying request anywhere.
            self.perform { result in
                handler(result)
                finisher()
            }
        }
    }
}
```

那么，我们的RequestQueue就可以更加专注于队列的实现，不用关心类型擦除了：
```swift
class RequestQueue {
    private var queue = [RequestOperation]()
    private var ongoing: RequestOperation?

    // Since the type erasure now happens before a request is
    // passed to the queue, it can simply accept a concrete
    // instance of 'RequestOperation'.
    func add(_ operation: RequestOperation) {
        guard ongoing == nil else {
            queue.append(operation)
            return
        }

        perform(operation)
    }

    private func perform(_ operation: RequestOperation) {
        ongoing = operation

        operation.perform { [weak self] in
            self?.ongoing = nil

            // Perform the next request if the queue isn't empty
            ...
        }
    }
}
```

当然，在调用RequestQueue时，就要求我们手动将Request转换为RequestOption了。
