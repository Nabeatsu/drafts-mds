

わからないもののインターフェースを覚えて課題をを解決するのは大切なスキルだと思います。が、自分の興味のある領域ぐらいは腹落ちしているレベルで理解しておきたいのが正直な気持ちです。

SwiftUIを一定期間触り続けていたのですが、SwiftUIとも関係が深いProperty wrappersのことをよくわかってないまま使っているのにそろそろ耐えられなくなってきたので、プロポーザルやドキュメントを調べて他の人の実装を読んで、自分でもProperty wrappersを使って玩具を実装していると少しずつ頭が整理されてきたので記事にしようと思います。


# 腑に落ちるまでProperty wrappers

プロパティを定義する際によく使うパターンがあります。Proposalでは`lazy`や`@NSCopying`のように個別のパターンをハードコーディングするよりも、これらのパターンを様々なライブラリのように定義できるよう提供されているより汎用的なメカニズムとしてProperty wrappersが提供されています。

プロポーザルは以下です。

- [swift-evolution/0258-property-wrappers.md at master · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)

## Property wrappersとwrappedValue

最初の例として提示されているのは`lazy`です。プロパティがオプションとして公開されるのを避けるために使われることが多い遅延初期化はSwiftではプリミティブな言語機能として`lazy`キーワードが提供されています。これがなければ自ら多くのボイラープレートを記述しないといけません。

```swift
struct HavingLazyProperty {
    // lazy var i = 100
    private var _i: Int?
    var i: Int {
        get {
            if let value = _i { return value }
            let initialValue = 0
            _i = initialValue
            return initialValue
        }
    }
}
```

他にもマルチフェーズでの初期化をサポートするために、一度値を割り当てられてからimmutableな`delayed`なプロパティを持つプロパティ実装パターンがあります。

```swift
class Foo {
  let immediatelyInitialized = "foo"
  var _initializedLater: String?

  // We want initializedLater to present like a non-optional 'let' to user code;
  // it can only be assigned once, and can't be accessed before being assigned.
  var initializedLater: String {
    get { return _initializedLater! }
    set {
      assert(_initializedLater == nil)
      _initializedLater = newValue
    }
  }
}

let foo = Foo()
//foo.initializedLater　// 割り当てられるまえにアクセスするとエラー
// error: Execution was interrupted, reason: EXC_BAD_INSTRUCTION (code=EXC_I386_INVOP, subcode=0x0).
foo.initializedLater = "hoge"
foo.initializedLater // 値を割り当てた後なのでエラーにならない
```

プロポーザルに記載されていたクラス定義では、マルチフェーズでの初期化をImplicity unwrapped optionalで実現していますがこれは結果としてnull安全性を損なっています。

個別の実装ケースに対してハードコーディングする手法に対して、Property wrappersを使用することでプロパティに関する処理を他の型に委譲することができます。プロパティの実装をプロパティに代わって行う`wrapper`を`@propertyWrapper`で指定します。

引き続きプロポーザルに記載されているサンプルからコードを借用します。

```swift
@propertyWrapper
enum Lazy<Value> {
  case uninitialized(() -> Value)
  case initialized(Value)

  init(wrappedValue: @autoclosure @escaping () -> Value) {
    self = .uninitialized(wrappedValue)
  }

  var wrappedValue: Value {
    mutating get {
      switch self {
      case .uninitialized(let initializer):
        let value = initializer()
        self = .initialized(value)
        return value
      case .initialized(let value):
        return value
      }
    }
    set {
      self = .initialized(newValue)
    }
  }
}

extension Lazy {
    mutating func reset(_ newValue: @autoclosure @escaping () -> Value) {
        self = .uninitialized(newValue)
    }
}
```

Property Wrappersはwrapperとして使用するプロパティのストレージを提供します。`wrappedValue`プロパティはラッパーの実際の実装を提供ししてoptionの`init（wrappedValue :)`はプロパティのタイプの値からストレージの初期化を有効にします。

また、`_`をプレフィックスに持つストレージのプロパティは自動で生成されますがprivateなので、内部からならアクセスできます。この`_foo`を使ってリセットするメソッドを試しに実装して使ってみます。

```swift
class C {
    @Lazy var foo = 2000
    func reset() {
        _foo.reset(0)
    }
}

let c = C()
c.foo // 1738
c.foo = 3000
c.foo // 3000
c.reset()
print(c.foo) // 0
```

また、Property wrapperインスタンスは初期化引数を指定して値を渡すことで代入ではなく直接初期化できます。

```swift
class C {
    @Lazy(wrappedValue: 1738) var foo: Int
}
```


## Property wrappersとprojectedValue

Property wrappersでは`wrappedValue`に加えて、`projectedValue`を定義することで追加の機能をAPIとして公開できます。

projectedValueは`$`がプレフィックスになる点を除いて命名はwrappedValueと同様です。コード上で$で始まるプロパティを定義できないのでprojectedValueが定義したプロパティに干渉することはありません。


```swift
@propertyWrapper struct ForcedUpperCase {
    private var string: String = ""
    var projectedValue = false
    var wrappedValue: String {
        get { return string }
        set {
            let upperCased = newValue.uppercased()
            projectedValue = string != upperCased // 変換されたことを監視するため代入された値と変換後の値が異なる場合はtrueを代入する。
            string = newValue.uppercased()
        }
    }
}

struct User {
    @ForcedUpperCase var name: String
}

var user = User()
user.name = "yamada taro"
user.name // YAMADA TARO
user.$name // true
```

## Property wrappersの制限事項

プロポーザルのRestrictions on the use of property wrappersに列挙されているので原文を読みたい人はそちらをどうぞ。実際にやってみてどのようなエラーをIDEが表示するのか見てみました。

- protocol内で宣言できない

```swift
protocol P {
    @ForcedUpperCase var name: String { get set } // Property 'name' declared inside a protocol cannot have a wrapper
}
```

- `An instance property with a wrapper `をextensionで使用できない。

```swift
extension User {
    @ForcedUpperCase var hoge: String { "hoge" } // Non-static property 'hoge' declared inside an extension cannot have a wrapper
}
```

- `An instance property with a wrapper `をenumで使用できない

```swift
enum Hoge {
    @ForcedUpperCase var hoge: String { return "hoge" }// Non-static property 'hoge' declared inside an enum cannot have a wrapper
    case foo
}
```

- クラスで宣言された`A property with a wrapper`はoverrideできない

```swift
class ParentClass {
    @ForcedUpperCase var name: String
}

class SubClass: ParentClass {
    @ForcedUpperCase override var name: String // Property 'name' with attached wrapper cannot override another property
}
```

- `A property with a wrapper`には@NSCopying、@NSManaged、weak、unownedキーワードなどをつけれない。

- `A property with a wrapper`はgetterとsetterを持てない

持てたらProperty wrappersとは、ということになるのでこれはすんなり納得できるかもしれないです。

```swift
struct User {
    @ForcedUpperCase var name: String { // Property wrapper cannot be applied to a computed property
        get {
            return "hoge"
        }
    }
}
```

- Property Wrapper型とwrappedValueは同じアクセスコントロールを持つ必要がある。init(wrappedValue)を宣言した場合も同様


```swift
@propertyWrapper struct ForcedUpperCase {
    private var string: String = ""
    var projectedValue = false
    private var wrappedValue: String { // Private property 'wrappedValue' cannot have more restrictive access than its enclosing property wrapper type 'ForcedUpperCase' (which is internal)
        get { return string }
        set {
            let upperCased = newValue.uppercased()
            projectedValue = string != upperCased
            string = newValue.uppercased()
        }
    }
}

```

projectedValueを宣言した場合、または`init()`を宣言した場合も同じアクセスコントロールである必要があります。

## ライブラリの実装から見る具体的なユースケース

### プロポーザルで想定されていたユースケースやProperty Wrappersを使った具体的な実装リンク集


## 参考

- [Property wrappers in Swift | Swift by Sundell](https://www.swiftbysundell.com/articles/property-wrappers-in-swift/)
- [Properties — The Swift Programming Language (Swift 5.1)](https://docs.swift.org/swift-book/LanguageGuide/Properties.html)
