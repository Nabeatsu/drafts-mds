SwiftUIで使われているProperty Wrappersをそれぞれ使って違いを実感してみる

SwiftUIにはProperty Wrappersというものが登場します。`@State`などをSwiftUI  Tutorialsを試してみた方は見たことがあると思います。

Proposalには以下のようにあります。

> We propose the introduction of **property wrappers**, which allow a property declaration to state which **wrapper** is used to implement it. The wrapper is described via an attribute:

@を使って宣言したプロパティへのアクセス方法を自分で定義することができます。@はアノテーションとも呼ばれます。

このProperty Wrappersによって解決したことは、コードベース内で再利用するために分離した構造体から取り出せるロジックでプロパティをラップすることです。

繰り返し登場するプロパティの実装パターンがあり、固定されたパターンのセットをコンパイラにハードコーディングするのではなく、パターンをライブラリとして定義できるようにするためのgenericな方法を提供することがこのProperty Wrappersが目指すゴールです。

そしてProperty Wrappersは、分離した構造体に抽出できるロジックでプロパティをラップします。

Proposalにもある通りこの機能は`Returned for revision`です。仕様が確定しておらず議論が進められています。

