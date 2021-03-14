---
layout:		post
title:		"Loading states of SwiftUI views"
description: "在SwfitUI中如何控制异步状态更新"
date:		2021-03-14
author:		"Yawei"
categories: "SwfitUI"
keywords:
    - SwiftUI
    - iOS
    - Combine
---

# 自加载View

这是一种最为简单的情况，在view中调用异步数据加载并更新到view上去。下面的实例代码中，我们假设有一个异步的loader,通过view的生命周期函数调用loader并更新。

```swift
struct ArticleView: View {
    var articleID: Article.ID
    var loader: ArticleLoader
    
    @State private var result: Result<Article, Error>?

    var body: some View {
        switch result {
        case .success(let article):
            // Rendering our article content within a scroll view:
            ScrollView {
                VStack(spacing: 20) {
                    Text(article.title).font(.title)
                    Text(article.body)
                }
                .padding()
            }
        case .failure(let error):
            // Showing any error that was encountered using a
            // dedicated ErrorView, which runs a given closure
            // when the user tapped an embedded "Retry" button:
            ErrorView(error: error, retryHandler: loadArticle)
        case nil:
            // We display a classic loading spinner while we're
            // waiting for our content to load, and we start our
            // loading operation once that view appears:
            ProgressView().onAppear(perform: loadArticle)
        }
    }

    private func loadArticle() {
        loader.loadArticle(withID: articleID) {
            result = $0
        }
    }
}
```

# ViewModel

但是上面不是一种好的方式，因为当我们有更多的异步请求时，这些代码就会变得非常混乱，甚至出现很多层case嵌套的情况。这时候，ArticleViewModel就可以出场了，它本质是一个ObservableObject。

```swift
class ArticleViewModel: ObservableObject {
    enum State {
        case idle
        case loading
        case failed(Error)
        case loaded(Article)
    }

    @Published private(set) var state = State.idle
    
    private let articleID: Article.ID
    private let loader: ArticleLoader

    init(articleID: Article.ID, loader: ArticleLoader) {
        self.articleID = articleID
        self.loader = loader
    }

    func load() {
        state = .loading

        loader.loadArticle(withID: articleID) { [weak self] result in
            switch result {
            case .success(let article):
                self?.state = .loaded(article)
            case .failure(let error):
                self?.state = .failed(error)
            }
        }
    }
}
```

接下来，我们使用@ObservedObject属性包装器来使用这个ViewModel

```swift
struct ArticleView: View {
    @ObservedObject var viewModel: ArticleViewModel

    var body: some View {
        switch viewModel.state {
        case .idle:
            // Render a clear color and start the loading process
            // when the view first appears, which should make the
            // view model transition into its loading state:
            Color.clear.onAppear(perform: viewModel.load)
        case .loading:
            ProgressView()
        case .failed(let error):
            ErrorView(error: error, retryHandler: viewModel.load)
        case .loaded(let article):
            ScrollView {
                VStack(spacing: 20) {
                    Text(article.title).font(.title)
                    Text(article.body)
                }
                .padding()
            }
        }
    }
}
```

# 通用模型封装
上面我们已经成功地将网络请求逻辑和视图地更新逻辑分离开了，这让我们可以更加专注地投入与对应地逻辑开发中。接下来我们可以更近一步，从中抽象出一套异步请求通用的模式出来。

## 状态封装
首先我们发现State枚举其实不止适用于Article，对于所有的异步请求几乎都适用。

```swift
enum LoadingState<Value> {
    case idle
    case loading
    case failed(Error)
    case loaded(Value)
}
```

## LoadableObject

接下来，我们从ArticleViewModel中抽象出加载数据的ViewModel的通用模型

```swift
protocol LoadableObject: ObservableObject {
    associatedtype Output
    var state: LoadingState<Output> { get }
    func load()
}
```

# AsyncContentView
到这里结束了吗？其实还没有，我们同样可以对适用异步请求的View进行统一抽象。这样我们就可以统一对我们的加载视图进行管理了。

```swift
struct AsyncContentView<Source: LoadableObject, Content: View>: View {
    @ObservedObject var source: Source
    var content: (Source.Output) -> Content

    var body: some View {
        switch source.state {
        case .idle:
            Color.clear.onAppear(perform: source.load)
        case .loading:
            ProgressView()
        case .failed(let error):
            ErrorView(error: error, retryHandler: source.load)
        case .loaded(let output):
            content(output)
        }
    }
}
```

这样我们只需要对每一个content闭包返回一个单视图，就可以很完美的工作了。不过，我们还可以做的更好一点，那就是使用@ViewBuilder属性包装器来让这闭包可以支持完整的SwfitUI's DSL功能了

```swift
struct AsyncContentView<Source: LoadableObject, Content: View>: View {
    @ObservedObject var source: Source
    var content: (Source.Output) -> Content

    init(source: Source,
         @ViewBuilder content: @escaping (Source.Output) -> Content) {
        self.source = source
        self.content = content
    }
    
    ...
}
```

接下来我们看如何使用,重新实现ArticleView

```swift
struct ArticleView: View {
    @ObservedObject var viewModel: ArticleViewModel

    var body: some View {
        AsyncContentView(source: viewModel) { article in
            ScrollView {
                VStack(spacing: 20) {
                    Text(article.title).font(.title)
                    Text(article.body)
                }
                .padding()
            }
        }
    }
}
```

# Combine + SwfitUI

SwiftUI经常会与Combine一起适配使用，因为这两个框架都遵循非常相似的声明式语法和数据驱动设计模式。

所以，与其让我们的视图遵循经典的单次一对一加载渲染模式，不如使用数据驱动的模式，来让数据随着app的状态变化可以持续的反馈到视图去,这就是flux。

接下来，我们使用Combine publisher来实现LoadableObject，然后使用其来加载和更新发布的状态。

```swift
class PublishedObject<Wrapped: Publisher>: LoadableObject {
    @Published private(set) var state = LoadingState<Wrapped.Output>.idle

    private let publisher: Wrapped
    private var cancellable: AnyCancellable?

    init(publisher: Wrapped) {
        self.publisher = publisher
    }

    func load() {
        state = .loading

        cancellable = publisher
            .map(LoadingState.loaded)
            .catch { error in
                Just(LoadingState.failed(error))
            }
            .sink { [weak self] state in
                self?.state = state
            }
    }
}
```

接下来，通过Swift的泛型能力，我们队AsyncContentView进行改造。

```swift
extension AsyncContentView {
    init<P: Publisher>(
        source: P,
        @ViewBuilder content: @escaping (P.Output) -> Content
    ) where Source == PublishedObject<P> {
        self.init(
            source: PublishedObject(publisher: source),
            content: content
        )
    }
}
```

再看下ArticleView如何实现。

```swift
struct ArticleView: View {
    var publisher: AnyPublisher<Article, Error>

    var body: some View {
        AsyncContentView(source: publisher) { article in
            ScrollView {
                VStack(spacing: 20) {
                    Text(article.title).font(.title)
                    Text(article.body)
                }
                .padding()
            }
        }
    }
}


struct ArticleListView: View {
    @ObservedObject var viewModel: ArticleListViewModel
    
    var body: some View {
        List(viewModel.articlePreviews) { preview in
            NavigationLink(preview.title
                destination: ArticleView(
                    publisher: viewModel.publisher(for: preview.id)
                )
            )
        }
    }
}
```
大家可能会疑惑loader去哪里了，什么时候加载数据呢？实际上，在这里publisher取代了loader的位置，这里就要求你的网络请求需要返回一个publisher对象，而这，恰恰是当下比较流行的网络加载库已经做好的事。

# 支持自定义的Loading View

```swift
struct AsyncContentView<Source: LoadableObject,
                        LoadingView: View,
                        Content: View>: View {
    @ObservedObject var source: Source
    var loadingView: LoadingView
    var content: (Source.Output) -> Content

    init(source: Source,
         loadingView: LoadingView,
         @ViewBuilder content: @escaping (Source.Output) -> Content) {
        self.source = source
        self.loadingView = loadingView
        self.content = content
    }

    var body: some View {
        switch source.state {
        case .idle:
            Color.clear.onAppear(perform: source.load)
        case .loading:
            loadingView
        case .failed(let error):
            ErrorView(error: error, retryHandler: source.load)
        case .loaded(let output):
            content(output)
        }
    }
}

// 为了更进一步的方便使用，我们通过extension来增加默认加载视图地能力。
typealias DefaultProgressView = ProgressView<EmptyView, EmptyView>

extension AsyncContentView where LoadingView == DefaultProgressView {
    init(
        source: Source,
        @ViewBuilder content: @escaping (Source.Output) -> Content
    ) {
        self.init(
            source: source,
            loadingView: ProgressView(),
            content: content
        )
    }
}
```

最后，我们再看看ArticleView。我们为其添加了自定义的占位加载视图，在Article加载时，就可以显示我们自定义的占位图。

```swift
extension ArticleView {
    struct ContentView: View {
        var article: Article

        var body: some View {
            VStack(spacing: 20) {
                Text(article.title).font(.title)
                Text(article.body)
            }
            .padding()
        }
    }

    struct Placeholder: View {
        var body: some View {
            ContentView(article: Article(
                title: "Title",
                body: String(repeating: "Body", count: 100)
            )).redacted(reason: .placeholder)
        }
    }
}

struct ArticleView: View {
    var publisher: AnyPublisher<Article, Error>

    var body: some View {
        ScrollView {
            AsyncContentView(
                source: publisher,
                loadingView: Placeholder(),
                content: ContentView.init
            )
        }
    }
}
```

最后的最后，这是MVVM设计模式吗？ 答案是：是的。这就是MVVM，只是我们这里并没有给出Article Model的实现。