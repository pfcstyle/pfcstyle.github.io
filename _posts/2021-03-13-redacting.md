---
layout:		post
title:		"redacted modifier"
description: "为SwiftUI视图生成自动占位符"
date:		2021-03-13
author:		"Yawei"
categories: "SwfitUI"
keywords:
    - redacted
    - iOS
---

```swift
struct ArticleView: View {
    var iconName: String
    var title: String
    var authorName: String
    var description: String

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Image(systemName: iconName)
                    .foregroundColor(.white)
                    .padding()
                    .background(Circle().fill(Color.secondary))
                VStack(alignment: .leading) {
                    Text(title).font(.title)
                    Text("By " + authorName)
                        .font(.subheadline)
                        .foregroundColor(.gray)
                }
            }
            Text(description).padding(.top)
        }
        .padding()
    }
}

let placeholder = ArticleView(
    iconName: "doc",
    title: "Placeholder",
    authorName: "Placeholder author",
    description: String(repeating: "Placeholder ", count: 5)
)
.redacted(reason: .placeholder)

struct RedactingView<Input: View, Output: View>: View {
    var content: Input
    var modifier: (Input) -> Output

    @Environment(\.redactionReasons) private var reasons

    var body: some View {
        if reasons.isEmpty {
            content
        } else {
            modifier(content)
        }
    }
}

extension View {
    func whenRedacted<T: View>(
        apply modifier: @escaping (Self) -> T
    ) -> some View {
        RedactingView(content: self, modifier: modifier)
    }
}

struct ArticleView: View {
    ...

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Image(systemName: iconName)
                    .foregroundColor(.white)
                    .whenRedacted { $0.hidden() }
                    .padding()
                    .background(Circle().fill(Color.secondary))
                ...
            }
            Text(description).padding(.top)
        }
        .padding()
    }
}

struct ArticleView: View {
    ...

    var body: some View {
        VStack(alignment: .leading) {
            HStack {
                Image(systemName: iconName)
                    ...
                VStack(alignment: .leading) {
                    Text(title).font(.title).unredacted()
                    ...
                }
            }
            Text(description).padding(.top)
        }
        .padding()
    }
}
```