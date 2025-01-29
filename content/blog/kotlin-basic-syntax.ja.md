---
title: "[Kotlin] 基本的な文法"
date: 2020-02-23T21:18:09+09:00
description: "Kotlinの基本的な文法についてまとめます。"
categories: ["Kotlin"]
aliases:
    - /blog/kotlin-declare-variable/
    - /posts/kotlin-declare-variable/
    - /blog/kotlin-constructor/
    - /posts/kotlin-constructor/
    - /blog/kotlin-basic-types/
    - /posts/kotlin-basic-types/
    - /blog/kotlin-nullability/
    - /posts/kotlin-nullability/
draft: false
---

### Kotlinの基本型

Kotlinでは全てはオブジェクトであり、Javaにあるようなプリミティブはありません。以下に挙げるような基本的な型もオブジェクトとして扱われます。ただし、これらの基本的な型はコンパイル時にプリミティブへと最適化されるため、パフォーマンスが犠牲になることはありません（総称型の型パラメータで使われている場合など、一部例外はある）。

### 真偽値型（Boolean）

Boolean型は真偽値を表し、値として`true`または`false`をとります。`true`は真、`false`は偽を表します。Boolean型の変数は、以下のように宣言します。

```kotlin
val boolean: Boolean = true
```

※ 説明のため明示的に型を記載していますが、実際にコードを書く際は、型推論を用いて適宜型の記載を省略してください。

以下のように、何らかの評価結果を直接代入することも可能です。括弧は省略しても問題ありません。

```kotlin
val boolean: Boolean = (a > 0)
```

真偽値の判定には`!`を用います。`not()`を用いても同様の結果を得ることができます。個人的には、`not()`を使った方が可読性が高くて良いかな、と思います。

```kotlin
!boolean
boolean.not()
```

### 数値型（Number）

Kotlinでは、全ての数値型はNumberを親クラスに持ち、Number型で用意されている関数を使うことができます。また、数値型同士の暗黙的な型変換を行うことはできず、Number型で用意されている`toInt()`、`toDouble()`などを用いて明示的に型変換を行います。

#### 整数型

Kotlinで用意されている整数型は、Byte型、Short型、Int型、Long型があります。

|型|ビット数|最小値|最大値|
|:-:|-:|-:|-:|
|Byte|8|-128|127|
|Short|16|-32,768|32,767|
|Int|32|-2,147,483,648|2,147,483,647|
|Long|64|-9,223,372,036,854,775,808|9,223,372,036,854,775,807|

##### 基本はInt型

これらの数値型の中で、基本となるのはInt型になると思います。もちろん、要件次第で他の型を使うのは問題ありません。

##### 数値に区切りを入れる

数値リテラルには、`_`を用いて区切りを入れることができます。`_`を用いて区切りを入れた場合、実際の数値には影響を与えません。

```kotlin
val int: Int = 1_000_000 // 1000000
```

##### Long型の宣言

Long型は、数値リテラルの後に`L`を付けて表します。

```kotlin
val long: Long = 0L
```

Long型の変数は以下のように数値リテラルに`L`を付けずに宣言することも可能ですが、Long型を使いたい時に誤ってInt型を使用しないよう、明示的に`L`を付けた方が良いでしょう。

```kotlin
val long: Long = 0
```

#### 浮動小数点型

Kotlinで用意されている浮動小数点型は、Float型とDouble型があります。整数を扱う整数型に対して、浮動小数点型は少数を扱います。

|型|ビット数|
|:-:|-:|
|Float|32|
|Double|64|

##### Float型の宣言

Float型は、Long型の宣言と同様に、数値リテラルの後に`f`または`F`を付けて表します。Long型と異なり、Float型は明示的に`f`または`F`を付けなければなりません。

```kotlin
val float: Float = 0.0f
```

### 文字型（Char）

Char型は、単一の文字を表します。文字リテラルをシングルクオート`'`で囲んで表します。

```kotlin
val char: Char = 'a'
```

### 文字列型（String）

String型は、文字列を表します。文字リテラルをダブルクオート`"`で囲んで表します。

```kotlin
val string: String = "string"
```

## Nullablity

Kotlinでは、`null`となり得る変数と`null`となり得ない変数を区別して扱います。これにより、変数が`null`になり得るか、なり得ないかを型のレベルで区別して取り扱うことが可能です。ここでは、`null`となり得る変数をnullableな変数、`null`になり得ない変数をnon-nullな変数と呼びます。

### nullableな変数の宣言

nullableな変数を宣言するためには、型の名称の後ろに`?`を付けて宣言します。以下のコードでは、nullableなString型の変数`nullableString`を宣言しています。

```kotlin
var nullableString: String? = "string"
```

`nullableString`には、String型の値の他に、`null`を代入することが可能です。

```kotlin
nullableString = null
```

### non-nullな変数の宣言

これに対して、non-nullな変数を宣言する場合は、型の名称の後ろに`?`を付けずに変数を宣言します。

```kotlin
var nonNullString: String = "string"
```

また、non-nullな変数を宣言する場合には、型推論を用いることも可能です。

```kotlin
var nonNullString = "string"
```

non-nullな変数に`null`を代入することはできません。例えば、以下のコードはコンパイルエラーとなります。

```kotlin
nonNullString = null // コンパイルエラー
```

Kotlinでは、non-nullな変数にnullを代入することは許されていません。これにより、変数が`null`になり得るか、なり得ないかを型のレベルで識別することが可能です。

### nullableな型とnon-nullな型の関係

Kotlinでは、全てのnullableな型はAny?型を親クラスに持ちます。例えば、String?型はAny?型を親クラスに持ちます。

```kotlin
println(nullableString is Any?) // true
```

一方、Kotlinの全てのnon-nullな型は、Any型を親クラスに持ちます。例えば、String型はAny型を親クラスに持ちます。

```kotlin
println(nonNullString is Any) // true
```

さらに、Any型はAny?型を親クラスに持ちます。すなわち、Any?型の変数にAny型の値を代入することは可能ですが、逆にAny型の変数にAny?型の値を代入することはできません。

これは同時に、nullableな変数にnon-nullな値を代入することは可能ですが、non-nullな値にnullableな値を代入することはできない、ということを意味します。

このようにして、Kotlinではnullableな型とnon-nullな型を区別して取り扱い、null安全を実現しています。

## コンストラクタ

### プライマリコンストラクタ

クラス名の後の括弧内にプライマリコンストラクタに渡す引数を定義することができます。プライマリコンストラクタに引数を渡す必要がない場合は省略可能です。プライマリコンストラクタに渡した引数は、プロパティの宣言と初期化ブロックの中でアクセスすることが可能です。

```kotlin
class Person constructor(name: String) {
    val name: String = name
}
```

以下の例では、初期化ブロックにおいてプライマリコンストラクタで渡された引数を出力した後、プロパティに代入しています。

```kotlin
class Person constructor(name: String) {
    val name: String

    init {
        println("set name as $name")
        this.name = name
    }
}
```

プライマリコンストラクタに渡された引数をそのままプロパティとして宣言することも可能です。この場合、引数名の前にアクセス修飾子と`val`キーワードまたは`var`キーワードを与えます。この表記には、プロパティの宣言と代入を一箇所で完結できるためコードが簡潔になるという利点があります。特別な理由がなければ、この記法を用います。

```kotlin
class Person constructor(
    val name: String
)
```

プライマリコンストラクタの`constructor`キーワードは省略することが可能です。特別な理由がなければ、`constructor`キーワードは省略した方が簡潔なコードとなります。

```kotlin
class Person (
    val name: String
)
```

### セカンダリコンストラクタ

Kotlinでは、プライマリコンストラクタに加えて、クラス本体においてセカンダリコンストラクタを定義することが可能です。

以下の例では、引数なしでPersonクラスをインスタンス化した場合に、デフォルトの値を使ってPersonクラスをインスタンス化しています。セカンダリコンストラクタからプライマリコンストラクタを呼び出しています。

```kotlin
class Person (
    val name: String
) {
    constructor() : this("no name")
}
```

セカンダリコンストラクタは、引数の数や型が異なれば、複数宣言することも可能です。以下の例では、上記の例で宣言したセカンダリコンストラクタに加えて、String型の引数を2つ定義したセカンダリコンストラクタを宣言しています。

```kotlin
class Person (
    val name: String
) {
    constructor() : this("no name")

    constructor(firstName: String, lastName: String) : this("$firstName $lastName")
}
```

プライマリコンストラクタと異なり、セカンダリコンストラクタは、コンストラクタ本体を持つことも可能です。コンストラクタ本体は、それが定義されるコンストラクタが呼び出された時に処理されます。

```kotlin
class Person (
    val name: String
) {
    constructor() : this("no name") {
        println("constructor with no name")
    }

    constructor(firstName: String, lastName: String) : this("$firstName $lastName") {
        println("constructor with first name and last name")
    }
}
```

Kotlinではプライマリコンストラクタにデフォルト引数を定義することが可能であるため、セカンダリコンストラクタが必要とされる機会は限定的です。

### 初期化ブロック

クラス本体において`init`キーワードを用いることで、初期化ブロックを宣言することが可能です。初期化ブロック内には、インスタンスの初期化処理を定義します。

```kotlin
class Person (
    val name: String
) {
    init {
        doSomethingHere()
    }
}
```

初期化ブロックは複数定義することが可能です。初期化ブロックを複数定義した場合は、上から宣言した順に呼び出されます。

```kotlin
class Person (
    val name: String
) {
    init {
        doSomethingHere()
    }

    init {
        doAnotherThingHere()
    }
}
```

セカンダリコンストラクタのコンストラクタ本体と初期化ブロックが同時に宣言されている場合には、初期化ブロック、コンストラクタ本体の順に呼び出されます。

## 変数の宣言

### `var`による変数の宣言

`var`キーワードを用いて変数を宣言することができます。`var`キーワードを用いた変数の宣言は、以下の構文で行います。

```kotlin
var string: String
```

ここで`String`は、変数の型を表します。変数名の後に`:`を置き、半角スペースを開けて型を指定します。`:`の後の半角スペースは無くても動作に問題ありませんが、ここに半角スペースを置くのが普通です。

### 変数の代入

宣言した変数への値の代入は、以下の構文で行います。

```kotlin
string = "string"
```

ここで重要なのは、変数stringはString型で宣言されているため、String型以外の値を代入しようとするとコンパイルエラーとなることです。例えば、以下のコードではString型で宣言された変数stringにInt型の値を代入しようとしているため、コンパイルエラーとなります。

```kotlin
string = 0
```

### 変数の宣言時に代入

変数の宣言と同時に代入することも可能です。

```kotlin
var string: String = "string"
```

むしろ変数の宣言と同時に値を代入することのが普通です。

### 型推論

上記の構文を用いて変数の宣言時に値を代入する場合、値の型から変数の型を推論することが可能です（**型推論**）。この場合、変数の宣言から型の記述を省略することが可能です。

```kotlin
var string = "string"
```

型推論を用いない構文は型推論を用いた構文と比べて冗長であるため、右辺から容易に型を識別できる場合は積極的に型の記述を省略します。右辺から容易に型を識別できない場合は、一見して型を識別できるよう、冗長になっても型の記述を行う場合があります。型の記述の有無については、ケースバイケースで判断します。

### 再代入

`var`キーワードで宣言した変数は再代入可能です。すなわち、一度値を代入した後に、再度別の値を代入することが可能ということです。以下のコードでは、`trueOrFalse()`が返す結果に応じて、Int型の変数intに10または20を再代入しています。

```kotlin
var int: Int
if (trueOrFalse()) {
    int = 10
} else {
    int = 20
}
```

変数が`var`で宣言されている場合、コード内のどこで変数が変更されているかをプログラマーが追わなければならず、思わぬバグを生み出してしまう場合があります。そのため、次に示す`val`を用いて、再代入のできない変数を宣言する方が安全なコードとなります。

### `val`による変数の宣言

再代入を許す`var`キーワードによる変数の宣言に対して、`val`キーワードによる変数の宣言では再代入を許しません。そのため、`val`キーワードを用いて変数を宣言した方が安全なコードとなります。`val`キーワードによる変数の宣言は、以下の構文で行います。`var`キーワードによる変数の宣言と同様です。

```kotlin
val string = "string"
```

再代入の項で示したコードは、`val`を用いて以下のように改善することができます。Kotlinの`if`は値を返す式であるため、このように書くことができます。

```kotlin
val int = if (trueOrFalse()) { 10 } else { 20 }
```

`if`式や`when`式などを用いて、変数を`var`ではなく`val`で宣言することで、より安全なコードとなります。
