# AnyとAnyObject

最近業務でObjective-Cを使ってプロダクションのコードを書くことが増えてきました。Objective-CからSwiftへの移行を進めつつも相互にコードを利用することも多く良い勉強になっていたのですが、AnyとAnyObjectの違いをなんとなくIDEのエラーメッセージなどからなんとなく理解したまま使っているだけできちんと理解していない気がしたのでこの機会に改めて調べてみることにしました。

## AnyObject

あらゆるクラス、またはClass only protocolの具象型として利用でき、型付けされていないオブジェクトの柔軟性が必要な時、ブリッジされたObjective-Cのメソッドとプロパティを使用する場合にAnyObjectを使用します。

型付けされていないオブジェクトとして利用したり

```swift
class ClassCat {
    let name: String
    init(_ name: String) {
        self.name = name
    }
}

let classCat = ClassCat("たま")
let anyObj: AnyObject = classCat
print(anyObj.self) // "__lldb_expr_28.ClassCat\n"
```

Objective-Cクラスにブリッジする型のインスタンスの具象型としても使用したりします。

```swift
let i: AnyObject = 100 as NSNumber
```

キャストはas(as!, as?)やオプショナルバインディングを使って行います。


AnyObjectはクラスの任意のインスタンスを参照し、Objective-Cのidと同等です。 Swiftの構造体または列挙型のいずれも使用できないため、参照型を特に使用する場合に便利です。 AnyObjectは、クラスでのみ使用できるようにプロトコルを制限する場合にも使用されます。

またメソッドの動的ディスパッチをサポートしているので@objcを付与したメソッドを動的に呼び出せます。

```swift
class ClassCat {
    let name: String
    init(_ name: String) {
        self.name = name
    }
    
    @objc func cry() -> String {
        return "にゃー"
    }
}


var anyObject: AnyObject
anyObject = ClassCat("たま")

let s = anyObject.cry?() // Optional("にゃー")
```

AnyObjectはClass only protocolの宣言にも使われます。

```swift
protocol ClassOnly: AnyObject {}

class C: ClassOnly {}
struct S: ClassOnly {} // Non-class type 'S' cannot conform to class protocol 'ClassOnly'
```

### as AnyObjectの挙動

AnyObjectは構造体としては振る舞えません。Swiftで提供されているオブジェクトは構造体で実装されているものが多くSwiftで数値リテラルのdefault typeになっているInt型も構造体で実装されています。そのためAnyObjectの配列の要素としてInt型の値を挿入しようとするとコンパイルエラーになります。

```swift
let i: Int = 1
let anyObjArray: [AnyObject] = [i] // Cannot convert value of type 'Int' to expected element type 'AnyObject'
```

ですが、as AnyObjectを使ってキャストすると

```swift
let i: Int = 1
let anyObjArray: [AnyObject] = [i as AnyObject]
```

コンパイルエラーは回避できます。この動きからSwiftで定義されている値型のオブジェクトがクラスのインスタンスに変換されていることが想像できます。

また、isなどを使って値型に戻せます。

```swift
anyObjArray[0] is Int // true
anyObjArray[0] as? Int
anyObjArray[0] as! Int
```

実際にその変換が行われていることを確認するためにSILを取り出してみます。最適化の過程を見たいわけではないのでraw SILを取り出します。SILを取り出す元になるswiftファイルの実体は以下です。

```swift
let i: Int = 1
let anyObj: AnyObject = i as AnyObject
```

以下のコマンドでraw SILを取り出します。

```sh
$swiftc -emit-silgen hoge.swift -o hoge.sil
```
```sil
// main
sil @main : $@convention(c) (Int32, UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>) -> Int32 {
bb0(%0 : @trivial $Int32, %1 : @trivial $UnsafeMutablePointer<Optional<UnsafeMutablePointer<Int8>>>):
  alloc_global @$s4hoge1iSivp                     // id: %2
  %3 = global_addr @$s4hoge1iSivp : $*Int         // users: %11, %8
  %4 = metatype $@thin Int.Type                   // user: %7
  %5 = integer_literal $Builtin.IntLiteral, 1     // user: %7
  // function_ref Int.init(_builtinIntegerLiteral:)
  %6 = function_ref @$sSi22_builtinIntegerLiteralSiBI_tcfC : $@convention(method) (Builtin.IntLiteral, @thin Int.Type) -> Int // user: %7
  %7 = apply %6(%5, %4) : $@convention(method) (Builtin.IntLiteral, @thin Int.Type) -> Int // user: %8
  store %7 to [trivial] %3 : $*Int                // id: %8
  alloc_global @$s4hoge6anyObjyXlvp               // id: %9
  %10 = global_addr @$s4hoge6anyObjyXlvp : $*AnyObject // user: %16
  %11 = load [trivial] %3 : $*Int                 // user: %13
  %12 = alloc_stack $Int                          // users: %17, %15, %13
  store %11 to [trivial] %12 : $*Int              // id: %13
  // function_ref _bridgeAnythingToObjectiveC<A>(_:)
  %14 = function_ref @$ss27_bridgeAnythingToObjectiveCyyXlxlF : $@convention(thin) <τ_0_0> (@in_guaranteed τ_0_0) -> @owned AnyObject // user: %15
  %15 = apply %14<Int>(%12) : $@convention(thin) <τ_0_0> (@in_guaranteed τ_0_0) -> @owned AnyObject // user: %16
  store %15 to [init] %10 : $*AnyObject           // id: %16
  dealloc_stack %12 : $*Int                       // id: %17
  %18 = integer_literal $Builtin.Int32, 0         // user: %19
  %19 = struct $Int32 (%18 : $Builtin.Int32)      // user: %20
  return %19 : $Int32                             // id: %20
} // end sil function 'main'
```

%3から%7で数値リテラル`1`から数値型のインスタンスを生成してます。%14で関数`ss27_bridgeAnythingToObjectiveCyyXlxlF`への参照を作成しています。命名からわかりますがこれが値型をObjective-Cでも利用できる型の値にブリッジしている関数です。マングリングされているのでちょっと読みにくいです。

実際にas AnyObjectでObjective-Cでも利用できるようなオブジェクト型への変換が行われていることはわかりました。どのような操作が実際に行われているのかですが、Swift Evolutionの[swift-evolution/0116-id-as-any.md at master · apple/swift-evolution](https://github.com/apple/swift-evolution/blob/master/proposals/0116-id-as-any.md)によればimmutableはclassでboxingしているとあります。また、`_ObjectiveCBridgeable`にconformしている型はこのprotocolの実装を流用してas AnyObjectが行われます。

この辺りに興味がある人はSwift Evolutionの内容を元に具体的なコードからas AnyObject を説明している以下の記事をご覧ください。これ以上の説明ができる気がしないのでこちらに譲ります。

- [`as AnyObject` で何が起こるのか - Qiita](https://qiita.com/takasek/items/1977878c966dcaf6cb5b)


## Any

Anyは、クラス、構造体、または列挙型のすべてのインスタンスを指します。これには関数型も含まれます。AnyObjectよりも広く利用できます。

```swift
let values: [Any] = [1, 2, "Fish"]
```

Anyについて調べるとすぐにAppleのSwift BlogのObjective-C id as Swift Anyという記事が見つかります。

[Objective-C id as Swift Any - Swift Blog - Apple Developer](https://developer.apple.com/swift/blog/?id=39)

Swift2まではObjective-Cのid型がSwiftのAnyObjectにマッピングされていました。この記事の冒頭からずっとAnyObjectの話をしていましたがAnyObjectはクラス(オブジェクト型)としてしか振る舞えません。また、NSString、NSArray、またはNSString、NSArrayを使うCocoa APIでネイティブのSwift型を簡単に使用できるように、String、Array、Dictionary、Set、およびいくつかの数値などのブリッジ値型のAnyObjectへの暗黙的な変換も提供されていました。

Swift3ではObjective-Cのid型がSwiftのAnyにマッピングされました。これによりSwiftで定義された値型をObjective-Cの APIに渡してSwift型として取り出せるので、この変更によりSwiftでObjective-C APIがより柔軟にしようできて手動の「ボックス」型が不要になりました。
