---
title: "[Dart] 基本的な文法"
date: 2020-05-21T21:47:08+09:00
description: "Dartの基本的な文法についてまとめます。"
categories: ["Dart"]
aliases:
    - /blog/dart-extension-method/
    - /posts/dart-extension-method/
    - /blog/dart-identical/
    - /posts/dart-identical/
    - /blog/dart-inheritance/
    - /posts/dart-inheritance/
    - /blog/dart-lambda-expression/
    - /posts/dart-lambda-expression/
    - /blog/dart-costructor/
    - /posts/dart-costructor/
    - /blog/dart-access-modifier/
    - /posts/dart-access-modifier/
draft: false
---

Dartの基本的な文法についてまとめます。

## コンストラクタ

Dartにおけるコンストラクタの最も原始的な構文は以下のように書けます。コンストラクタの引数としてString型のパラメータ`name`を受け取り、コンストラクタの本体の中でメンバ変数に代入しています。

```dart
class Dog {
  String name;

  Dog(String name) {
    this.name = name;
  }
}
```

以下の構文では、コンストラクタの引数として受け取ったパラメータを直接メンバ変数に代入しています。この構文では、メンバ変数をfinalにすることができます。

```dart
class Dog {
  final String name;

  Dog(this.name);
}
```

上記の構文を用いた場合にも、コンストラクタ本体を定義することができます。

```dart
class Dog {
  final String name;

  Dog(this.name) {
    print(name);
  }
}

Dog dog = Dog('pochi'); // => pochi
```

また、`this.name`の部分を`{}`で囲うと、コンストラクタ呼び出しを名前付き引数で行うことができます。

```dart
class Dog {
  final String name;

  Dog({this.name});
}

Dog dog = Dog(name: 'pochi');
```

Dartでは、コンストラクタのオーバーロードができません。その代わりに、Dartでは名前付きコンストラクタを定義することができます。

```dart
class Dog {
  String name;

  Dog(String name) {
    this.name = name;
  }

  Dog.noName() {
    name = 'no name';
  }
}

Dog noName = Dog.noName();
print(noName.name); // => no name
```

なお、上記の例において、プライマリコンストラクタでは`this`を用いてメンバ変数にアクセスし、名前付きコンストラクタでは`this`を用いずにメンバ変数にアクセスしている理由は、前者では名前の隠蔽が発生し、後者では発生しないためです。

## アクセス修飾子

Dartにはアクセス修飾子がありません。その代わり、メンバの名前で可視性を与えることができます。メンバの名前が`_`から始まるものは、いわゆるprivateな可視性となります。それ以外の名前は、いわゆるpublicな可視性となります。

例えば、以下のクラスのメンバ変数`_name`はprivateなメンバ変数となり、同一ライブラリ内からのみアクセス可能です（[Dart \- Using Access Modifiers \(Private & Public\) \- Woolha](https://www.woolha.com/tutorials/dart-using-access-modifiers-private-public)）。

```dart
// cat.dart
class Cat {
  String _name;

  Cat(String name) {
    _name = name;
  }
}
```

同一ライブラリ内からのみアクセス可能と言うのは、例えば以下のように同一ファイル内にクラスを定義した場合は、他クラスのprivateなメンバにアクセスすることができます。

```dart
// cat.dart
class Cat {
  String _name;
  String _keeper;

  Cat(String name) {
    _name = name;

    Human human = Human();
    _keeper = human._name;
  }
}

class Human {
  String _name = 'human';
}
```

可視性の範囲外からは、privateなメンバにはアクセスできません。

```dart
// main.dart
void main(List<String> arguments) {
  Cat cat = Cat('tama');

  //cat._name; // 不可
}
```

privateなメソッドも、メソッド名を`_`から始めて定義します。

```dart
class Cat {
  // 略
  void _toilet() {
    // do something here
  }
}
```

setterは隠蔽し、getterだけを公開したい時は、以下のように別途getterを定義します。

```dart
class Cat {
  String _name;
  String get name => _name;
  // 略
}
```

拡張メソッド（後述）もprivateにすることが可能です。privateな拡張メソッドは、拡張メソッドを使いたい場所を限定したい時に使います。

```dart
extension CatExtension on Cat {
  Dog _toDog() {
    return Poodle(name: this.name);
  }
}
```

## 継承

Dartでは、`extends`キーワードを用いてクラスの継承を行うことができます。単一継承のみ可能で、多重継承はできません。

以下の例では、抽象クラスDogを継承して、Poodleクラスを実装しています。抽象クラスを定義するためのキーワードは`abstract`です。

```dart
abstract class Dog {
  final String name;

  Dog({this.name});
}

class Poodle extends Dog {
  Poodle({name}) : super(name: name);
}

Dog poodle = Poodle(name: 'pochi');
```

抽象クラスでは、抽象メソッドを定義することができます。抽象メソッドは、メソッドのシグネチャのみ定義し、メソッド本体を持たないメソッドです。具象クラスの中では定義できません。また、抽象メソッドは、具象クラスの中で必ずオーバーライドしなければなりません。メソッドをオーバーライドするには、対象となるメソッドに`@override`アノテーションを付与します。

```dart
abstract class Dog {
  // 略
  void bark(); // 抽象メソッド
}

class Poodle extends Dog {
  // 略
  @override
  void bark() {
    print('bow wow!');
  }
}
```

また抽象クラスにおいては、メソッドの実装を持つことも可能です。この場合、具象クラスにおいてメソッドをオーバーライドする必要は必ずしもありませんが、オーバーライドすることも可能です。この時、必要に応じて`super.メソッド名()`で継承元のメソッドを呼び出すことが可能です。

```dart
abstract class Dog {
  // 略
  void run() {
    print('running...');
  }
}

class Poodle extends Dog {
  // 略
  @override
  void run() {
    super.run();
    print('it is a poodle...');
  }
}
```

メソッドだけでなく、メンバ変数についてもオーバーライドすることが可能です。なお、メンバ変数を抽象的に定義する方法はわかりませんでした。Dart `2.8.2`では実装されていないものと思われます。

```dart
abstract class Dog {
  // 略
  final String size = 'medium';
}

class Poodle extends Dog {
  // 略
  @override
  final String size = 'large';
}
```

クラスやメソッドのオーバーライドを禁止する機能についても、おそらくDart `2.8.2`では実装されていないです。

## 拡張メソッド

Dart 2.7以降、Dartでは型に拡張メソッドを定義して、機能を追加することができます。構文は以下です（[Extension methods \| Dart](https://dart.dev/guides/language/extension-methods)）。

```dart
extension <extension name> on <type> {
  (<member definition>)*
}
```

以下の例では`int`型に拡張メソッドとして`toTenTimes()`を定義しています。

```dart
extension IntExtension on int {
  int toTenTimes() {
    return this * 10;
  }
}
```

`int`型に対して、あたかも`int`型に元から`toTenTimes()`というメソッドが定義されていたかのように`toTenTimes()`を呼び出すことが可能になります。ただし、インポートは必要です。拡張メソッドの中では、`this`で対象のクラスのインスタンスにアクセスすることができます。

```dart
int answer = 42.toTenTimes();
print(answer); // => 420
```

拡張について名前を定義することができます。上の例では、IntExtensionという名前で拡張をしています。拡張に名前を定義している理由は、「_APIがコンフリクトした時に備えて拡張は名前を持っている（extensions have names, which can be helpful if an API conflict arises）_（[Extension methods \| Dart](https://dart.dev/guides/language/extension-methods)）」とのことです。

## ラムダ

Dartでは以下の形式でラムダを書くことができます。

```dart
(int i) {
  return i + 1;
};
```

変数succに上記のラムダをバインドします。

```dart
int Function(int) succ = (int i) {
  return i + 1;
};
```

バインドした変数を経由してラムダを実行します。

```dart
print(succ(0)); // => 1
print(succ(100)); // => 101
```

以下の形式は上記の形式の糖衣構文です。

```dart
int Function(int) succ2 = (int i) => i + 1;

print(succ2(0)); // => 1
print(succ2(100)); // => 101
```

ラムダを渡して実行してみます。

```dart
int calculate(int Function(int) func, int input) {
  return func.call(input);
}

print(calculate(succ, 42)); // => 43
```

当然、名前をつけないラムダを渡すこともできます。

```dart
print(calculate((int i) => i + 1 , 42)); // => 43
```

## identicalを用いた参照の等価性の評価

`identical`を用いて、参照の等価性を評価することができます。`identical`はdart:coreに含まれています。

```dart
bool identical(
  Object a,
  Object b
)
```

以下の例では、2つの変数の参照が等しいため、`identical`はtrueを返します。

```dart
Dog dog = Dog(name: 'pochi');

bool result = identical(dog, dog);
print(result); // => true
```

以下の例では、2つの変数の参照が異なるため、`identical`はfalseを返します。

```dart
Dog dog = Dog(name: 'pochi');
Cat cat = Cat(name: 'tama');

bool result = identical(dog, cat);
print(result); // => false
```

変数が異なっても、参照が等価であれば`identical`はtrueを返します。

```dart
Dog dog1 = Dog(name: 'pochi');
Dog dog2 = dog1;

bool result = identical(dog1, dog2);
print(result); // => true
```

`identical`が返すbool値は、`==`の結果とは異なります（以下の例では`==`オペレーターをoverrideして`name`が等しければ`==`がtrueを返すようにしています）。

```dart
Dog dog1 = Dog(name: 'pochi');
Dog dog2 = Dog(name: 'pochi');

bool result = identical(dog1, dog2);
print(result); // => false
print(dog1 == dog2); // => true
```
