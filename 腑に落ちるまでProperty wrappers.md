わからないもののインターフェースを覚えて課題をを解決するのは大切なスキルだと思います。が、自分の興味のある領域ぐらいは腹落ちしているレベルで理解しておきたいのが正直な気持ちです。

SwiftUI を一定期間触り続けていたのですが、SwiftUI とも関係が深い Property wrappers のことをよくわかってないまま使っているのにそろそろ耐えられなくなってきたので、プロポーザルやドキュメントを調べて他の人の実装を読んで、自分でも Property wrappers を使って玩具を実装していると少しずつ頭が整理されてきたので記事にしようと思います。

# 腑に落ちるまで Property wrappers

プロパティを定義する際によく使うパターンがあります。Proposal では`lazy`や`@NSCopying`のように個別のパターンをハードコーディングするよりも、これらのパターンを様々なライブラリのように定義できるよう提供されているより汎用的なメカニズムとして Property wrappers が提供されています。

プロポーザルは以下です。

- [swift-evolution/0258-property-wrappers.md at master · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/master/proposals/0258-property-wrappers.md)

## Property wrappers と wrappedValue

最初の例として提示されているのは`lazy`です。プロパティがオプションとして公開されるのを避けるために使われることが多い遅延初期化は Swift ではプリミティブな言語機能として`lazy`キーワードが提供されています。これがなければ自ら多くのボイラープレートを記述しないといけません。

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

他にもマルチフェーズでの初期化をサポートするために、一度値を割り当てられてから immutable な`delayed`なプロパティを持つプロパティ実装パターンがあります。

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

プロポーザルに記載されていたクラス定義では、マルチフェーズでの初期化を Implicity unwrapped optional で実現していますがこれは結果として null 安全性を損なっています。

個別の実装ケースに対してハードコーディングする手法に対して、Property wrappers を使用することでプロパティに関する処理を他の型に委譲することができます。プロパティの実装をプロパティに代わって行う`wrapper`を`@propertyWrapper`で指定します。

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

Property Wrappers は wrapper として使用するプロパティのストレージを提供します。`wrappedValue`プロパティはラッパーの実際の実装を提供しして option の`init（wrappedValue :)`はプロパティのタイプの値からストレージの初期化を有効にします。

また、`_`をプレフィックスに持つストレージのプロパティは自動で生成されますが private なので、内部からならアクセスできます。この`_foo`を使ってリセットするメソッドを試しに実装して使ってみます。

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

また、Property wrapper インスタンスは初期化引数を指定して値を渡すことで代入ではなく直接初期化できます。

```swift
class C {
    @Lazy(wrappedValue: 1738) var foo: Int
}
```

## Property wrappers と projectedValue

Property wrappers では`wrappedValue`に加えて、`projectedValue`を定義することで追加の機能を API として公開できます。

projectedValue は`$`がプレフィックスになる点を除いて命名は wrappedValue と同様です。コード上で\$で始まるプロパティを定義できないので projectedValue が定義したプロパティに干渉することはありません。

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

## Property wrappers の制限事項

プロポーザルの Restrictions on the use of property wrappers に列挙されているので原文を読みたい人はそちらをどうぞ。実際にやってみてどのようなエラーを IDE が表示するのか見てみました。

- protocol 内で宣言できない

```swift
protocol P {
    @ForcedUpperCase var name: String { get set } // Property 'name' declared inside a protocol cannot have a wrapper
}
```

- `An instance property with a wrapper`を extension で使用できない。

```swift
extension User {
    @ForcedUpperCase var hoge: String { "hoge" } // Non-static property 'hoge' declared inside an extension cannot have a wrapper
}
```

- `An instance property with a wrapper`を enum で使用できない

```swift
enum Hoge {
    @ForcedUpperCase var hoge: String { return "hoge" }// Non-static property 'hoge' declared inside an enum cannot have a wrapper
    case foo
}
```

- クラスで宣言された`A property with a wrapper`は override できない

```swift
class ParentClass {
    @ForcedUpperCase var name: String
}

class SubClass: ParentClass {
    @ForcedUpperCase override var name: String // Property 'name' with attached wrapper cannot override another property
}
```

- `A property with a wrapper`には@NSCopying、@NSManaged、weak、unowned キーワードなどをつけれない。

- `A property with a wrapper`は getter と setter を持てない

持てたら Property wrappers とは、ということになるのでこれはすんなり納得できるかもしれないです。

```swift
struct User {
    @ForcedUpperCase var name: String { // Property wrapper cannot be applied to a computed property
        get {
            return "hoge"
        }
    }
}
```

- Property Wrapper 型と wrappedValue は同じアクセスコントロールを持つ必要がある。init(wrappedValue)を宣言した場合も同様

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

projectedValue を宣言した場合、または`init()`を宣言した場合も同じアクセスコントロールである必要があります。

## 使い所

[SvenTiigi/ValidatedPropertyKit](https://github.com/SvenTiigi/ValidatedPropertyKit)というライブラリは`@ Validated` PropertyWrapper の初期化中に指定された`Validation`に基づいてバリデーションされて条件を満たさない場合に値は nil にしたり直前の値がバリデーションに成功したかどうかを取り出す`isValid`メソッドなどを提供したりしています。

```swift
 @Validated(.nonEmpty)
 var username: String?

 username = "Mr.Robot"
 print($username.isValid) // true

 username = ""
 print($username.isValid) // false
```

ソースコードの実装を追うと`@propertyWrapper`つけた`Validated<Value: Optionalable>`のイニシャライザで`Validation<Value.Wrapped>`型の定数 validation を初期化してバリデーションを行っています。複数のバリデーションを行えるように`&&`や`||`の実装も行われています。where 句をつけて extension を生やせば特定の型に対してカスタマイズしたバリデーションを行うこともできるようです。

以下のコードは`Value`が Comparable に conform している時に比較して比較対象未満の時に失敗とする`Validation`を返す static メソッドを実装している部分です。ユーザー定義の型にたいするバリデーションも extension で実装できて使い所があれば便利そうです。

```swift
public extension Validation where Value: Comparable {

    /// Validation with less `<` than comparable value
    ///
    /// - Parameter comparableValue: The Comparable value
    /// - Returns: The Validation
    static func less(_ comparableValue: Value) -> Validation {
        return .init { value in
            if value < comparableValue {
                return .success
            } else {
                return .failure("\(value) is not less than \(comparableValue)")
            }
        }
    }
}
```

その他にもプロポーザルでは遅延初期化、不可分操作、UserDefaults への CRUD、CoW などの実装が Property wrappers の具体的なユースケースとして紹介されていました。

## まとめ

Property wrappers についてぼんやりとした理解だったのが、プロポーザルを読むことから始めてライブラリのコードを読んだりしているとだんだん頭が整理されてきた気がします。

UserDefaults の利用を Property wrappers を使って手軽にするのはやってみたいと思ったので記事を書いた後にこんなもの([Nabeatsu/UserDefaultsWrapper](https://github.com/Nabeatsu/UserDefaultsWrapper))を作り始めました。

まだ記事として利用するには機能も汎用性も足りないので、飽きずに形にできたら改めて記事にしようと思います。

## 参考

- [Property wrappers in Swift | Swift by Sundell](https://www.swiftbysundell.com/articles/property-wrappers-in-swift/)
- [Properties — The Swift Programming Language (Swift 5.1)](https://docs.swift.org/swift-book/LanguageGuide/Properties.html)
