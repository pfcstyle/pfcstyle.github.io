---
layout:		post
title:		"What’s the difference between @ObservedObject, @State, @Environment and @EnvironmentObject and @Binding?"
description: ""
date:		2021-02-07
author:		"Yawei"
categories: "iOS, Swift, SwiftUI"
keywords:
    - iOS
    - Swift
    - SwiftUI
    - Property Wrapper
    - "@ObservedObject"
    - "@State"
    - "@Environment"
    - "@EnvironmentObject"
    - "@Binding"
---

## @State

只在view内部发生改变和使用，且是较为简单的数据类型。@State将变量的内存管理交给了SwiftUI。所有的view都是结构体，是标量，这意味着它们无法修改（只能重新赋值）。因此，当我们使用@State时， 我们将变量的控制权交给了SwiftUI，只要view存在，他就会一直在内存中存在。当变量发生改变时，SwiftUI也会自动重新加载。

```Swift
struct ContentView: View {
    @State private var score = 0
    init() {
        score = 1
    }
    // more code
}
```

## @Binding

当需要读写祖先view的@State或@ObservableObject(可能其中某个属性)的属性时

```Swift

struct MyView: View {
    @State var isPresentingAlert = false

    var body: some View {
        Button(action: {
            self.isPresentingAlert = true
        }, label: {
            Text("Present an Alert")
        })
        .customAlert(isPresented: $isPresentingAlert) {
            CustomAlertView(title: Text("Alert!"))
        }
    }}

struct CustomAlertView: View {
    let title: Text
    
    /// it needs read/write access to a State-
    /// wrapped property of an ancestor view
    @Binding var isBeingPresented: Bool
}
```

```Swfit

struct MyView: View {

    /// it needs to provide read/write access of 
    /// one of its properties to a descendant view
    @State var isPresentingAlert = false

    var body: some View {
        Button(action: {
            self.isPresentingAlert = true
        }, label: {
            Text("Present an Alert")
        })
        .alert(isPresented: $isPresentingAlert) {
            Alert(title: Text("Alert!"))
        }
    }
}
```

## @ObservedObject

适用于自定义类型，具备多个属性和方法，或者需要跨view使用的情况。大部分情况和@State相同，只是结构更复杂。需要实现ObservableObject protocol，这意味着其内部的属性可以被SwiftUI binding. 大多数情况，我们直接使用@Published来声明内部属性，当然也可以使用combine框架实现自定义的Publisher

```Swift
# 自己内部初始化
class MovieViewModel: ObservableObject {
    
    @Published var movies: [Movie] = [] // 1
    
    // more code
    
}


struct MoviesView: View {
    
    // 1
    @ObservedObject var viewModel = MovieViewModel()
    
    var body: some View {
        List(viewModel.movies) { movie in // 2
            HStack {
                VStack(alignment: .leading) {
                    Text(movie.title) // 3a
                        .font(.headline)
                    Text(movie.originalTitle) // 3b
                        .font(.subheadline)
                }
            }
        }
    }
}
```

```Swift
# 外部初始化（view init时）

struct MyView: View {

    /// it is dependent on an object that can
    /// easily be passed to its initializer
    @ObservedObject var dessertFetcher: DessertFetcher

    var body: some View {
        List(dessertFetcher.desserts) {
            Text($0.name)
        }.onAppear {
            self.dessertFetcher.fetch()
        }
    }}

extension UIViewController {

    func observedObjectExampleTwo() -> UIViewController {
        let fetcher = DessertFetcher(preferences: .init(toleratesMint: false))
        let view = ObservedObjectExampleTwo(dessertFetcher: fetcher)
        let host = UIHostingController(rootView: view)
        return host
    }
}
```
> Warning: When you use a custom publisher to announce that your object has changed, this must happen on the main thread.

## @EnvironmentObject

ObservedObject当需要跨view共享或者是内嵌很深的view需要但又不适合自己初始化时，需要使用@EnvironmentObject。

> @EnvironmentObject只支持1个实例

```Swift

struct SomeChildView: View {

    /// it would be too cumbersome to pass that 
    /// observed object through all the initializers 
    /// of all your view's ancestors
    @EnvironmentObject var veggieFetcher: VegetableFetcher

    var body: some View {
        List(veggieFetcher.veggies) {
            Text($0.name)
        }.onAppear {
            self.veggieFetcher.fetch()
        }
    }}

struct SomeParentView: View {
    var body: some View {
        SomeChildView()
    }}

struct SomeGrandparentView: View {
    var body: some View {
        SomeParentView()
    }
}
```

## @Environment

当view依赖的不能满足ObservableObject协议时，可能是以下几种情况：

* 依赖的是一个值类型
* 依赖项仅作为协议而不是具体类型公开
* 依赖是一个闭包

> @Environment 支持多实例
> 修改@Published的属性，不会引发view的重绘

```Swift

struct MyView: View {

    /// it is dependent on a type that cannot 
    /// conform to ObservableObject
    @Environment(\.theme) var theme: Theme

    var body: some View {
        Text("Me and my dad make models of clipper ships.")
            .foregroundColor(theme.foregroundColor)
            .background(theme.backgroundColor)
    }}

// MARK: - Dependencies

protocol Theme {
    var foregroundColor: Color { get }
    var backgroundColor: Color { get }
}

struct PinkTheme: Theme {
    var foregroundColor: Color { .white }
    var backgroundColor: Color { .pink }
}

// MARK: - Environment Boilerplate

struct ThemeKey: EnvironmentKey {
    static var defaultValue: Theme {
        return PinkTheme()
    }}

extension EnvironmentValues {
    var theme: Theme {
        get { return self[ThemeKey.self]  }
        set { self[ThemeKey.self] = newValue }
    }
}
```

# Workaround for Multiple Instances of an EnvironmentObject

```Swift

struct MyView: View {

    @DistinctEnvironmentObject(\.posts) var postsService: Microservice
    @DistinctEnvironmentObject(\.users) var usersService: Microservice
    @DistinctEnvironmentObject(\.channels) var channelsService: Microservice

    var body: some View {
        Form {
            Section(header: Text("Posts")) {
                List(postsService.content, id: \.self) {
                    Text($0)
                }
            }
            Section(header: Text("Users")) {
                List(usersService.content, id: \.self) {
                    Text($0)
                }
            }
            Section(header: Text("Channels")) {
                List(channelsService.content, id: \.self) {
                    Text($0)
                }
            }
        }.onAppear(perform: fetchContent)
    }}

// MARK: - Property Wrapper To Make This All Work

@propertyWrapperstruct DistinctEnvironmentObject<Wrapped>: DynamicProperty where Wrapped : ObservableObject {
    var wrappedValue: Wrapped { _wrapped }
    @ObservedObject private var _wrapped: Wrapped

    init(_ keypath: KeyPath<EnvironmentValues, Wrapped>) {
        _wrapped = Environment<Wrapped>(keypath).wrappedValue
    }}

// MARK: - Wherever You Create Your View Hierarchy

MyView()
    .environment(\.posts, Microservice.posts)
    .environment(\.users, Microservice.users)
    .environment(\.channels, Microservice.channels)
```
