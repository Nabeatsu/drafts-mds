# SwiftSyntax

## 概要

静的解析ツールを作るための情報を提供している。

### 静的コード解析とは

- ファイルを実行することなく解析を行うこと。逆にソフトウェアを実行して行う解析を動的プログラム解析と呼ぶ。
- コードを実行せずに解析できるところが特徴。
- Swift のような静的型付き言語でコンパイル時にエラーを検知するように、静的にコードを解析して予期していないコードを検査したり、ルールにのっとっていないコードをフォーマットしたりできます。

### 静的解析ツールのしくみ

ソースコードを構文木や抽象構文木に変換して、それを元に解析しています。

構文木
　 `swift -frontend -emit-syntax syntax.swift`
抽象構文木
　`swiftc -dump-parse syntax.swift`

### SwiftSyntax の概要

`libSyntax` の Swift 用ラッパ。
libSyntax は Swift コンパイラで Swift シンタックスのデータ構造を実装しているライブラリです。
API は Swift と C++で提供されていて、コードの解析やフォーマッタなどを可能にします。

### SourceKit

シンタックスハイライトなど。
IDE に必要な機能を提供するライブラリ。
sourcekitd なら見たことがあるはず。
Swift コードのシンタックスハイライトが効かなくなった時にクラッシュしているのが SourceKit です。
C で提供されている。

## SwiftSyntax の基礎

### Syntax

本章では SwiftSyntax の基礎となる概念をソースコードとともに紹介。

まずはすべてのシンタックスのベースとなる Syntax プロトコルを見てみましょう。

Swift のソースコードは木構造として出力される。
次のプロパティで木構造から必要なシンタックスやトークンを取得できます。

- `isToken`
- `previousToken`
- `nextToken`
- `firstToken`
- `lastToken`
- `children`

次のプロパティはシンタックスの種類を教えてくれます。

- `isToken`
- `isExpr`
- `isCollection`
- `isDecl`
- `isStmt`
- `isType`
- `isPattern`
- `isUnknown`

次のプロパティはスペース数や改行、ソースコメントなどの情報を提供する Trivia 情報を取得できます。

- leadingTrivia
- trailingTrivia

### TokenSyntax

シンタックスのノードを表現する struct。

tokenKind プロパティからトークンの種類を取得できて text プロパティからテキストを取得できる。

字句解析で細分化された各トークンの状態を表します。TokenKind 名を見れば、何を意味しているかだいたい分かるはず。

### DeclSyntax

この protocol は StructDeclSyntax、EnumDeclSyntax、VariableDeclSyntax など宣言に関するシンタックスに使われています。

### ExprSyntax

FunctionCallExpreSyntax、 IdentifierExprSyntax、IntegerLiteralExprSyntax など、式に関するシンタックスに使われています。

### TypeSyntax

型に関する Syntax。

### PatternSyntax

パターンに関するシンタックス。
