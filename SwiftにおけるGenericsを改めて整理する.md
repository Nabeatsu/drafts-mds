# Swift における Generics をあらためて整理してみる

Swift では、汎用的でかつ型安全なプログラムを記述するための言語仕様としてジェネリクスというものが提供されています。

## Generics の基本

### ジェネリック関数

```swift
func hoge<T>(_ arg: T) -> T { arg }

let i = hoge(1)
let s = hoge("1")
let d = hoge(2.0)

func hogehoge<T>(arg1: T, arg2: T) {
    print(#function)
}

hogehoge(arg1: 2, arg2: 2)
// hogehoge(arg1: 3, arg2: "") // コンパイルエラー: Cannot convert value of type 'String' to expected argument type 'Int'
```

ジェネリック関数`hoge<T>(_:)`と同じような動きを Generics を使わずに実装したい場合型ごとのオーバーロードで実装できます。

```swift
func hoge(_ arg: String) -> String { arg }
func hoge(_ arg: Int) -> Int { arg }
func hoge(_ arg: Double) -> Double { arg }

let i = hoge(1)
let d = hoge(1.0)
let s = hoge("hoge")
```

ジェネリック関数の内部では型引数として抽象的に型を表現していますが、実際にこの関数を呼び出すときは上記のコードのように具体的な型を指定する必要があります。

Generics 使って汎用的に定義されたジェネリック関数などに具体的な型を与えて型を確定させることを特殊化と言います。

```swift
let i = hoge(1) // 具体的な型Intのオブジェクトを引数として渡して特殊化
let s = hoge("1") // 具体的な型Stringのオブジェクトを引数として渡して特殊化
let d = hoge(2.0) // 具体的な型Doubleのオブジェクトを引数として渡して特殊化
```

ジェネリック関数には引数に具体的な型を渡すことによる特殊化以外に戻り値から型推論を行う特殊化の方法があります。

`hoge<T>(_:)`関数の引数を`Any`にして何でも受け取るようにした後、戻り値の型を型引数にします。

```swift
func hoge<T>(_ arg: Any) -> T? { arg as? T }

let i: Int? = hoge(1)
let s: String? = hoge("hoge")
let d: Double? = hoge("a") // コンパイルエラーには習いがnil
let f = hoge(1) // 型引数Tの型が決定できない　'Int' is not convertible to 'Any'
```

### 汎用的で型安全とは

Generics を使わず、`hoge(_:)`関数に Int、String、 Double を渡せるようにしてみます。幸い Swift にはすべての型が準拠している protocol `Any`があります。それを使って`hoge(_:)`を実装してみます。

```swift
func hoge(_ arg: Any) -> Any { arg }
let i = hoge(1) as? Int
let s = hoge("hoge") as? String
let d = hoge(1.0) as? Double
```

定義はできますが、Any で帰ってくるためダウンキャストしないといけません。型の情報が戻り値の時点で消失してしまっています。Generics では型安全性と汎用化を両立しているのでこのようなダウンキャストは必要ありません。

### ジェネリック型

Generics は関数だけでなく型定義にも使えます。

```swift
struct GenericStruct<T> {
    var property1: T
}
```

#### ジェネリック型の特殊化

```swift
// 型引数を直接指定による特殊化
let genericInt = GenericStruct<Int>(property1: 1)
let genericString = GenericStruct<String>(property1: "hoge")

// 型推論による特殊化
let genericInt2 = GenericStruct(property1: 2)
let genericString2 = GenericStruct(property1: "hoge2")
```

## 型制約

型引数には制約を与えることができます。

```swift
// 型引数に対してprotocol制約
func hoge<T: Comparable>(_ arg: T) -> T { arg }
hoge("1")

struct NotComparableStruct {}

hoge(NotComparableStruct()) // Argument type 'NotComparableStruct' does not conform to expected type 'Comparable' Comparableに準拠していないのでコンパイルエラー

// 型引数に対してスーパークラスの制約
func hoge<T: UIViewController>(_ arg: T) -> T { arg }
hoge(UIViewController())
hoge(1) // Cannot convert value of type 'Int' to expected argument type 'UIViewController' IntはUIViewControllerのサブクラスではないのでコンパイルエラー
```

#### where 節を使って型引数の連想型に型制約

二次元配列の中身の要素が配列のようにカウントできる場合にのみ、配列の数をカウントした配列を返す関数を定義するとします。配列は Collection protocol に準拠しているので以下のように定義できます。

```swift
// 配列の中身が配列だった場合要素ごとに配列のカウントの配列を返す関数
func counts<T: Collection>(_ arg: T) -> [Int] where T.Element: Collection {
    return arg.map { $0.count }
}
let countArray = counts([[1, 2, 3], [1, 2]]) // [3, 2]
let countArray2 = counts(["a", "ab", "abc"])

struct NotCollectionStruct {}
let countArray3 = counts([NotCollectionStruct(), NotCollectionStruct()]) // Global function 'counts' requires that 'NotCollectionStruct' conform to 'Collection' コンパイルエラー
```
