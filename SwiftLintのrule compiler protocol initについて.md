今業務で取り組んでいるプロジェクトで段階的に SwiftLint を導入しています。SwiftLint はコードの静的解析を行うツールで、定義された rule や custom rule を元にコードをチェックしてくれます。

新規ではなく、途中参加の案件だったので最初は SwiftLint の rule を全て無効にしてちょっとした作業の切れ目の時間に別ブランチで一つずつ rule を有効にしながらコードを修正しています。

SwiftLint の rule 一覧は[SwiftLint/Rules.md](https://github.com/realm/SwiftLint/blob/master/Rules.md)で確認できますが、デフォルトで有効になっている rule の一つに`Compiler Protocol Init`というものがあります。

この rule に関する issue は以下です。

- [Add compiler_protocol_init rule by marcelofabri · Pull Request #1101 · realm/SwiftLint](https://github.com/realm/SwiftLint/pull/1101)

この rule を無効にしていないと`The initializers declared in compiler protocol shouldn't be called directly.`と警告が表示されます。

```swift
NSNumber(booleanLiteral: false) // 警告
NSNumber(false) // 警告が出ない　
```

なぜこの rule がデフォルトで有効になっているのか、そして compiler protocol とは何なのかわからなかったため少し調べたりコードを書いたりしたのでそれをまとめます。

## SwiftLint での compiler_protocol_init について

SwiftLint/Rules.md には Compiler Protocol Init について言及している部分があります。

```swift
## Compiler Protocol Init

Identifier | Enabled by default | Supports autocorrection | Kind | Analyzer | Minimum Swift Compiler Version
--- | --- | --- | --- | --- | ---
`compiler_protocol_init` | Enabled | No | lint | No | 3.0.0

The initializers declared in compiler protocols such as `ExpressibleByArrayLiteral` shouldn't be called directly.
```

`Enabled by default`とあり、デフォルトでは有効になっているのがわかります。SwiftLint でデフォルトで有効な rule を無効にするには`.swiftlint.yml`で disabled_rules に追加します。

```yml
disabled_rules:
  - compiler_protocol_init
```

ここで例として挙げられている`ExpressibleByArrayLiteral`は protocol で、この protocol に準拠するためには init(arrayLiteral:)の実装が必要です。

この protocol に conform している型の一つが Array で、Array の init(arrayLiteral:)のドキュメントを見てみると

> Do not call this initializer directly. It is used by the compiler when you use an array literal. Instead, create a new array by using an array literal as its value. To do this, enclose a comma-separated list of values in square brackets.
>
> [init(arrayLiteral:) - Array | Apple Developer Documentation](https://developer.apple.com/documentation/swift/array/1540235-init)

直接呼ばないように明示してあります。

Do not call this initializer directly で Swift のリポジトリを検索して見ると他にも多くの型のイニシャライザが同じ理由で直接呼ばないように明記してあることがわかります。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/12/865b0551aec876db91a9f718cbdf892f.png" alt="" width="260" height="868" class="alignnone size-full wp-image-505922" />

ドキュメントに明示してある、このことだけで SwiftLint のデフォルトな rule として有効になっているのは当然だなと思えるのですが`It is used by the compiler when you use an array literal.`この文言が何を意味しているのか自分の言葉で説明できなかったのでいまいち納得できませんでした。

compiler protocol とは何なのかもわからないままです。

そこで SwiftLint でこの rule に関する実装を行っている部分を少し見てみることにしました。コードを抜粋しながら説明します。

SwiftLint のソースコードの内、compiler_protocol_init に関わりのある実装がされているところは[SwiftLint/CompilerProtocolInitRule.swift](https://github.com/realm/SwiftLint/blob/master/Source/SwiftLintFramework/Rules/Lint/CompilerProtocolInitRule.swift)です。

ここの実装を見ると`ExpressibleByCompiler`という構造体が定義されています。

```swift
private struct ExpressibleByCompiler {
    let protocolName: String
    let initCallNames: Set<String>
    private let arguments: [[String]]
    ...
```

この構造体の static 定数 `allProtocols`の中身を見ていくと、この rule によって警告が出る initializer の実装が準拠条件になっているプロトコルの一覧がわかります。

```swift
static let allProtocols = [
    byArrayLiteral,
    byNilLiteral,
    byBooleanLiteral,
    byFloatLiteral,
    byIntegerLiteral,
    byUnicodeScalarLiteral,
    byExtendedGraphemeClusterLiteral,
    byStringLiteral,
    byStringInterpolation,
    byDictionaryLiteral
]
```

allProtocols は ExpressibleByCompiler に準拠する型が要素になっていて全て private な定数として定義されています。一つずつ読んで警告が出る initializer の実装が準拠条件になっているプロトコルをひと通り確認しました。

- ExpressibleByArrayLiteral
- ExpressibleByNilLiteral
- ExpressibleByBooleanLiteral
- ExpressibleByFloatLiteral
- ExpressibleByIntegerLiteral
- ExpressibleByUnicodeScalarLiteral
- ExpressibleByExtendedGraphemeClusterLiteral
- ExpressibleByStringLiteral
- ExpressibleByStringInterpolation
- ExpressibleByDictionaryLiteral

どうやら ExpressibleBy で始まる protocol に準拠するために必要なイニシャライザを直接呼ぶのは推奨されないようです。

## ExpressibleBy で始まる protocol

Swift では特定の型の初期化にはリテラルを直接代入できます。

```swift
let string: String = "string"
let int: Int = 0
let bool: Bool = true
```

型アノテーションを省略してもコンパイルエラーにならず勝手に推論してくれます。

```swift
let string = "string" // String型
let int = 0 // Int型
let bool = true // Bool型
```

ユーザーが定義した型はリテラルの代入で初期化できません。

```swift
struct UserDefinedString {
    var value: String
}

let userDefinedString: UserDefinedString = "user defined" // コンパイルエラー Cannot convert value of type 'String' to specified type 'UserDefinedString'
```

文字列のリテラルの代入による初期化を行うために準拠する必要がある proocol が`ExpressibleByStringLiteral`です。コンパイルエラーを避けつつ UserDefinedString を ExpressibleByStringLiteral に準拠させてみます。

```swift
extension UserDefinedString: ExpressibleByStringLiteral {
    typealias StringLiteralType = String

    // これを実装しないと以下のコンパイルエラー
    // Initializer 'init(value:)' has different argument labels from those required by protocol 'ExpressibleByStringLiteral' ('init(stringLiteral:)')
    init(stringLiteral value: Self.StringLiteralType) {
        self.value = value
    }
}
```

associatedtype を持つ protocol の準拠は typealias で型を明示するだけでなくメソッドの戻り値からも推論させられるので以下のように書けます。

```swift
extension UserDefinedString: ExpressibleByStringLiteral {
    init(stringLiteral value: String) {
        self.value = value
    }
}
```

準拠できたので使ってみます。

```swift
let userDefinedString: UserDefinedString = "user defined" // 文字列リテラルを代入して初期化できる
print(userDefinedString.value) // user defined
```

このように ExpressibleBy で始まる protocol は準拠することでリテラルからの初期化が行えるようになります。初期化時に引数に String 型の値を渡すような型の初期化を extension を使って簡単にできたりするかもしれません。URL などがそうですが forced unwrap しないといけなくなるので URL 型は厳しそうですね…

swift-evolution の proposal では`Literal Syntax Protocols`と書いていたので総称した呼び方はこれかもしれません。元々は`*LiteralConvertible`で `ExpressibleBy*Literal`にリネームされました。

## Compiler protocol

[Swift Style Guide](https://google.github.io/swift/)に Compiler protocol のイニシャライザに関する言及があります。

> The initializers declared by the special ExpressibleBy\*Literal compiler protocols are never called directly.
> Explicitly calling .init(...) is allowed only when the receiver of the call is a metatype variable. In direct calls to the initializer using the literal type name, .init is omitted. (Referring to the initializer directly by using MyType.init syntax to convert it to a closure is permitted.)
>
> https://google.github.io/swift/

compiler protocol のイニシャライザを明示的に呼んで良いのはレシーバが metatype variable である場合のみと記載されています。

compiler protocol が何なのかは未だわからないままですが呼んではいけない理由はわかってきました。

※ここからは Swift のソースコードやコンパイル時に生成される中間言語などから読み取ったことを記載します。このレイヤはまだビギナーなので誤りもあるかと思います。何かお気づきの際はコメントや SNS などでご指摘お願いします。

compiler protocol が何なのか調べるために ExpressibleBy で始まる protocol の定義場所を見てみました。Swift のリポジトリを clone して定義元を探してみると[swift/CompilerProtocols.swift](https://github.com/apple/swift/blob/e94361e642907675c8fce9248c6b5a39d01746b0/stdlib/public/core/CompilerProtocols.swift)が見つかります。

このファイルの冒頭にコメントで

```swift
// Intrinsic protocols shared with the compiler
```

と記載されています。コンパイラと共有される組み込みプロトコルと訳しました。この内コンパイラと共有される、という部分は各イニシャライザのドキュメントに記載されていた

// It is used by the compiler when you use an \*\*\* literal.

のことだと考えています。

このコンパイラと共有される部分を知るために SIL というファイルを生成してみます。

### SIL(Swift Intermediate Language)

Swift では Swift コードからバイナリコードが生成されるまでに SIL という中間言語を経由します。
Swift のコードは最初 AST(抽象構文木)に変換され、意味情報を付与された AST に、そして SIL に、その後 LLVM IR が生成されバイナリコードが出力されます。

SIL は最適化やフローに基づいた解析などに使われますが、Swift コードからは読み取れない暗黙的な挙動も SIL からは窺い知ることができます。

SIL については詳細に紹介してくださっている記事がたくさんあるので詳しく知りたい方は参照されてください。

- [Swift の中間言語 SIL を読む その 1 - SIL に入門するための準備 – ukitaka – iOS 開発とかのメモ](https://blog.waft.me/2018/01/09/swift-sil-1/)
- [Swift 中間言語の、ひとまず入り口手前まで - Qiita](https://qiita.com/es_kumagai/items/b0b123526329909ae2a2)

SIL を読む時は[swift/SIL.rst](https://github.com/apple/swift/blob/master/docs/SIL.rst)を片手に読みます。用語や各命令について詳細に解説がされています。

SIL を生成する swift ファイルの中身は先程自分で定義した型を定数に入れているだけの以下のコードです。

```swift
import Foundation

struct UserDefinedString {
    var value: String
}

extension UserDefinedString: ExpressibleByStringLiteral {
    init(stringLiteral value: String) {
        self.value = value
    }
}

let userDefinedString: UserDefinedString = "user"
```

sil はコマンドラインで生成できます。

```bash
$ swiftc -emit-sil -parse-as-library int.swift -o int.sil
```

生成された SIL の中身が[このファイル](https://gist.github.com/Nabeatsu/cd82a2abc48faf4c345afdcad8b899cd)ですが長すぎるので知りたいこと(自分で定義した型を ExpressibleByStringLiteral に準拠させるために定義したイニシャライザが SIL でも共有されていることを示す部分があるか)について書かれている付近のみ抜粋します。

%9 のところで Swift で定義した UserDefinedString.init(stringLiteral:)が呼ばれていることがわかります。確かにコンパイラと Swift の間で実装が共有されています。

```swift
// globalinit_33_EE3F0113CE9469DE1DA818E2E4164E99_func0
sil private @globalinit_33_EE3F0113CE9469DE1DA818E2E4164E99_func0 : $@convention(c) () -> () {
bb0:
  alloc_global @$s11userdefined17userDefinedStringAA04UsercD0Vvp // id: %0
  %1 = global_addr @$s11userdefined17userDefinedStringAA04UsercD0Vvp : $*UserDefinedString // user: %11
  %2 = string_literal utf8 "user defined"         // user: %7
  %3 = integer_literal $Builtin.Word, 12          // user: %7
  %4 = integer_literal $Builtin.Int1, -1          // user: %7
  %5 = metatype $@thin String.Type                // user: %7
  // function_ref String.init(_builtinStringLiteral:utf8CodeUnitCount:isASCII:)
  %6 = function_ref @$sSS21_builtinStringLiteral17utf8CodeUnitCount7isASCIISSBp_BwBi1_tcfC : $@convention(method) (Builtin.RawPointer, Builtin.Word, Builtin.Int1, @thin String.Type) -> @owned String // user: %7
  %7 = apply %6(%2, %3, %4, %5) : $@convention(method) (Builtin.RawPointer, Builtin.Word, Builtin.Int1, @thin String.Type) -> @owned String // user: %10
  %8 = metatype $@thin UserDefinedString.Type     // user: %10
  // function_ref UserDefinedString.init(stringLiteral:)
  %9 = function_ref @$s11userdefined17UserDefinedStringV13stringLiteralACSS_tcfC : $@convention(method) (@owned String, @thin UserDefinedString.Type) -> @owned UserDefinedString // user: %10
  %10 = apply %9(%7, %8) : $@convention(method) (@owned String, @thin UserDefinedString.Type) -> @owned UserDefinedString // user: %11
  store %10 to %1 : $*UserDefinedString           // id: %11
  %12 = tuple ()                                  // user: %13
  return %12 : $()                                // id: %13
} // end sil function 'globalinit_33_EE3F0113CE9469DE1DA818E2E4164E99_func0'
```

呼ばれている部分

```swift
  // 定義した関数への参照が作られている
  // function_ref UserDefinedString.init(stringLiteral:)
  %9 = function_ref @$s11userdefined17UserDefinedStringV13stringLiteralACSS_tcfC : $@convention(method) (@owned String, @thin UserDefinedString.Type) -> @owned UserDefinedString // user: %10
```

文字列リテラルの変換に代入するためにinit(stringLiteral:)を呼んでいます。Swiftで定義したprotocolのinitializerがコンパイラでも呼ばれていることがわかりました。

## ここまでのまとめと今調べているところ

SwiftLintのデフォルトで有効になっているruleに登場するcompiler protocol initはExpressibleBy~で始まるprotocolにconformする際に実装を要求されるinitializerのことでした。

このprotocolはCompilerProtocol.swiftで定義されていて、ここで定義されているprotocolはコンパイラと共有される組み込みのprotocolと記載がありました。

実際にSwiftコードがバイナリコードとして出力される過程で経由する中間表現を見ると、リテラルを具体的な型に変換する際にprotocolにconformする際に実装を要求されたinitializerが呼ばれていることがわかりました。

この後は実際にどのようにSwiftで定義したinitialzierを呼んでいるのか、TypeCheckerがリテラルにたどり着いた後リテラルに対応するprotocolを探してinitialzerを呼び出す部分のコードを読みながら理解できればと思います。実際に読み始めましたがC++のコードは思った以上に読みづらく記事として書けるぐらい正確に読むにはまだまだ時間がかかりそう



ここまでの説明で認識に誤りがあったり説明が不足しているようなところもあるかと思います。何かお気づきの際はコメントかTwitterなどでご指摘いただけるとありがたいです。
