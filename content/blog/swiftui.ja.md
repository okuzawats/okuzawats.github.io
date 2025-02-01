---
title: "はじめてのSwiftUI"
date: 2024-08-16T15:03:00+09:00
description: "SwiftUIについて学びます。"
categories: ["Swift"]
draft: false
---

SwiftUIについて学んでいこうと思いますので、SwiftUIのコードを読んでいきます。まずはXcodeから新たにSwiftUIのファイルを作成すると、こんな感じのコードが生成されます。Xcodeのバージョンは `15.4` です。

```swift
import SwiftUI
 
struct MyFirstSwiftUI: View {
  var body: some View {
    Text("Hello, World!")
  }
}

#Preview {
  MyFirstSwiftUI()
}
```

1行目から見ていきます。1行目には `import` が書かれています。

```swift
import SwiftUI
```

Swiftにおける `import` は、モジュールやフレームワークを使用可能にするためのキーワードです。ここでは、 `SwiftUI` を `import` しています。これによって、SwiftUIを使用できる状態になります。

空行を挟んで3行目には以下のコードが書かれています。

```swift
struct MyFirstSwiftUI: View {
```

`struct` は、構造体を宣言するためのキーワードです。ここでは、 `MyFirstSwiftUI` という名前の構造体を宣言しています。また、 `: View` は、 `MyFirstSwiftUI` が `View` プロトコルに準拠していることを宣言しています。 `View` プロトコルに準拠している型は、Viewの表示内容を示すために後述する `body` を宣言する必要があります。

その `body` は、4行目で宣言されています。

```swift
  var body: some View {
```

`some` は、ここでは `body` が `View` プロトコルに準拠する何らかの型である、ということを表すSwiftのキーワードです。これは、**Opaque Return Type**と呼ばれます。金田(2019)では、以下のように説明されています。

> これは「Viewプロトコルを採用した何らかの型」を意味し、someがつけられた型をOpaque Return Typeと呼びます。一言でいうと戻り値の型を抽象的に記述する方法です。
> 
> ジェネリクスが抽象化するのは引数のみですが、Opaque Return Typeは、関数の戻り値や、この例のようにプロパティの型指定を抽象化します。

すなわち、 `body` は `View` プロトコルに準拠する型を返す、computed propertyである、ということです。具体的に何を返すのか？は次の5行目を見ます。

```swift
    Text("Hello, World!")
```

今回のサンプルコードでは、この `Text` が `View` プロトコルに準拠する型となっています。 `View` プロトコルに準拠する具体的な型によってUIにどのように表示されるのかは異なり、 `Text` はテキストを表示するための型です。 `MyFirstSwiftUI` の `body` の具象は `Text` であり、抽象は `View` であると言えます。

さて、 `body` の本体である以下のコードを評価した結果が `Text` となるのは何故でしょうか。

```swift
{ Text("Hello, World!") }
```

これは、Swiftのクロージャに単一の式のみが書かれている場合、 `return` を省略してもその単一式の評価結果を返すためです。すなわち、上記のクロージャ内に唯一存在する式である `Text("Hello, World!")` の評価結果が返されるということです。

このSwiftの構文により、SwiftUIの構文をシンプルに書くことができています。

次に、9行目〜11行目を見てみます。 `#Preview` は、SwiftUIのViewをプレビューするためのマクロです。以下の例では第1引数が省略され、第2引数として `View` プロトコルを返すクロージャを渡しています（trailing closure構文が使われています）。このViewがXcode上にプレビューされます。

```swift
#Preview {
  MyFirstSwiftUI()
}
```

[Appleのドキュメント](https://developer.apple.com/documentation/swiftui/preview(_:body:))には、以下のように第1引数の使い方が示されています。

```swift
#Preview("Input true") {
    ContentView(someInput: true)
}


#Preview("Input false") {
    ContentView(someInput: false)
}
```

## References

1. [View | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/view)（最終アクセス日：2024/08/16）
2. [Opaque and Boxed Protocol Types | Documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/opaquetypes/)（最終アクセス日：2024/08/16）
3. 金田浩明, (2019), SwiftUI 徹底入門, SB Creative
4. [Text | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/text)（最終アクセス日：2024/08/16）
5. [Preview(_:body:) | Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/preview(_:body:))
