# [社内勉強会レポート]『The Rust Programming Language』勉強会#2

クラスメソッド 福岡オフィスで iOS アプリケーションエンジニアとして働いている田辺です。先週に続き、社内で Rust の勉強会が行われたのでそのレポートを記事にします。

前回の記事。

- [[社内勉強会レポート]『The Rust Programming Language』勉強会#1 ｜ DevelopersIO](https://dev.classmethod.jp/server-side/language/trpl-study-meeting-report/)

## 数当てゲーム

引き続き学習を進めつつ残したメモや理解するために別で調べる必要のあった概念などを列挙していきます。業務では Swift でコーディングしているので Rust の説明をしているのに Swift の話が頻繁に出てきます。ご容赦ください。

## match

<script src="https://gist.github.com/cm-tanabe-nobuyuki/8c2695a726200c0f06d64b70d1ac1513.js"></script>

Swift でいう switch と同じようなことができます。ただ Swift の switch は文なので値を返しません。それに対して match は式です。式なので代入できます。

<script src="https://gist.github.com/cm-tanabe-nobuyuki/7071aca9224d67460177e26dc13f8cd7.js"></script>

数当てゲームの実装では列挙型 Result に対して match 式が使われています。

<script src="https://gist.github.com/cm-tanabe-nobuyuki/50e16575f5aeecaf1d46b663ba81334f.js"></script>

String に対する`trim()`メソッドで、両端の空白を取り除いて parse（)メソッドで文字列を解析して数値型にします。

Rust の数値型は多数ありますが、型アノテーションに u32 を指定しているのでそれに基づいて u32 に変換されます。この動きから parse(）メソッドがジェネリックな関数だとわかります。

そういう関数は Rust ではどういう風に定義するのか気になるのでシグネチャを見てみると`parse<F: FromStr>(&self) -> Result<F, F::Err>`とありました。

FromStr はトレイトなのでジェネリックな関数を理解するにはトレイトについて知ることが必要だとわかります。まだ勉強会ではトレイトの詳細な仕様に入っていないのでいったんトレイトには触れずにおきますが、型パラメーターに FromStr を指定しています。`<>`で型パラメーターを指定する方法は、Swift でのジェネリックな関数の定義方法と同じです。

余談ですが Swift ではジェネリックな関数を定義するために Generics という言語機能が提供されています。それを使って定義した関数の型や戻り値に class や struct、protocol の型を割り当てたり、protocol の associatedtype で汎用的な型で関数を定義したりできます。

parse の戻り値は`Result<F, F::Err>`で F は`FromStr`なので match によるパターンマッチが可能で、数字型へ変換できたらそのまま guess に値を代入します。変換に失敗した場合は、後述する loop によって繰り返し入力させられるように continue としています。

### Rust における式と文について

Rust は式がベースになっています。式は値を返します。`;`で終わると文になり、値を返しません。関数の戻り値の行に`;`をつけないのは式として扱い、値を返したいからです。たとえば、値を返さない let は、`let a = 1 let b = 2`のように書き方ができません。1 行で書く場合、正しい書き方は`let a = 1; let b =2;`。

### Cargo に関する追記事項

前回、記事に書き忘れていたことが２つあります。１つ目は Cargo のバージョン管理についてです。

Cargo は Cargo.toml の`[dependencies]`セクションヘッダの一番下に導入したいクレートを追記していきます。そして cargo build コマンドを実行するとレジストリから最新バージョンを取得して、取得していないクレートをダウンロードします。Rust のコンパイラは依存ファイルをコンパイルして利用可能な状態でプロジェクトをコンパイルします。

このとき、Cargo.lock というファイルが生成されます。Swift のプロジェクトで依存ライブラリの導入・管理に使われる CocoaPods というツールも`pod install`というコマンドをたたくと Podfile.lock というものが生成されます。これらの`.lock`という拡張子がついたファイルはそれぞれのツールで同じ役割を担っています。

Cargo ではビルドの際に依存のバージョンを計算して Cargo.lock に記述します。その後は cargo build を叩いてもその時点での利用可能な最新のバージョンを計算する作業は行わず、Cargo.lock ファイルを参照して再現可能なビルドを構成します。

もしクレートを更新したければ cargo update というコマンドを実行します。そのときに Cargo.lock ファイルは更新されます。それ以降は再度 cargo build すると…ここからは同じ説明になるので省略します。

2 つ目は便利な cargo コマンドについてです。ローカルで依存しているクレートのドキュメントをビルドしてブラウザで閲覧できる機能が cargo に用意されています。`cargo doc —open`です。実行すると次のようにブラウザからドキュメントを閲覧できます。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/08/5a5865785aa0d1d211505d60fa49d1b1.png" alt="" width="946" height="932" class="alignnone size-full wp-image-462744" />

## loop

loop を使用することで何らかの終了状態に到達するまでブロック内の処理を e の戻り値は Res ループし続けます。これに各処理を入れて数当てゲームを何度でも繰り返せるようにします。

<script src="https://gist.github.com/cm-tanabe-nobuyuki/cea510d4bc02d9243e82a9a31a86a6a2.js"></script>

これまで書いたユーザーの入力を受け取って出力するだけの処理をブロックの中に書くと入力してその文字列を出力して、という操作を繰り返せます。

## Rust の型推論

数取りゲームの最終的なコードは次のようになりました。

<script src="https://gist.github.com/cm-tanabe-nobuyuki/9d4358b758e4765fcf800d320f7d8c2e.js"></script>

このうち 20 行目の num を実数にしてみると次のようなエラーが返ってきてビルドが失敗します。

```shell
Compiling guessing_game v0.1.0
...
error[E0284]: type annotations required: cannot resolve `<_ as std::str::FromStr>::Err == _`
  --> src/main.rs:19:45
   |
19 |         let guess: u32 = match guess.trim().parse() {
   |                                             ^^^^^

error: aborting due to previous error

error: Could not compile `guessing_game`.

To learn more, run the command again with --verbose.
HL00486:guessing_game tanabe.nobuyuki$
```

`type annotations needed`というメッセージがコンパイラにより出力されています。guess の型、つまり parse()の戻り値の型を推論できなくなっています。

<script src="https://gist.github.com/cm-tanabe-nobuyuki/bef2eb825faac3a801f71a5570929893.js"></script>

実数を num に戻す以外だと parse の戻り値を直接指定する方法でビルドさせることができます。
`parse()`を`parse::<u32>()`にします。

<script src="https://gist.github.com/cm-tanabe-nobuyuki/6a25091eacb5afc9908ff822ab12e8aa.js"></script>

`parse())`に明示的に型パラメーターを与えることによりコンパイルが成功します。メソッドの戻り値の型をこんな風に指定できるのは Swift では馴染みがないので少し驚きました。`parse()`の定義元にジャンプしてみると、parse()メソッドの戻り値の型を指定する方法は`turbofish`と呼ばれていることがわかります。

このシンタックスは TRPL では[Generics - The Rust Programming Language](https://doc.rust-lang.org/1.30.0/book/first-edition/generics.html)
に言及があります。

- [Rust の turbofish と GHC 8 の Type Application ― または我々は如何にして多相な関数を単相化するか - ryota-ka's blog](https://ryota-ka.hatenablog.com/entry/2016/10/15/171851)

## まとめ

今回で数取りゲームの実装が終わりました。
チュートリアルをただ写経して読んで終わるのと違って、疑問を皆で解決したり、経験者の人が補足情報を加えてくれることで少し深く理解できるなと感じました。個人的にはこのような記事を書くのはやったことを整理したり、理解があいまいなところをドキュメントやソースコードを調べたりできるよい機会だと思っています。

最後に社内向けの情報です。この社内勉強会は途中参加の人が困らないように作業ログ用のリポジトリを用意していたり、次回から前回の振り返りの時間を少し設ける予定ですので、Rust に興味がある社員の方は参加してみてください。
