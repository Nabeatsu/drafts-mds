## はじめに
社内で行われた勉強会のレポートです。参加者のRustの経験は実務で少しずつ導入している人からRustの本を買って積んだままにしている人まで様々です。

クラスメソッドのSlackには様々なチャンネルがあり、その一つRustについて話す所があります。そこで社内勉強会を行うことになったのでそのレポートを記事にします。
<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/08/95db28c457bf3f132af0b94c9911c86c.png" alt="" width="654" height="145" class="alignnone size-full wp-image-460727" />

Developers.IOでは社内勉強会についての記事は過去にテスト駆動開発の読書勉強会などのレポートなどが公開されています。この勉強会のレポートもそれを踏襲します。

- [[社内勉強会レポート] 『テスト駆動開発』読書勉強会 #1 ｜ DevelopersIO](https://dev.classmethod.jp/study_meeting/read/tdd-by-example-reading-1/)

## TRPLについて

今回勉強会のテーマになっている『TRPL』はRustが公式に提供している入門書です。`The Rust Programming Language`の略です。the bookとも呼ばれています。

日本語版もありますが、日本語版は最新のバージョンに追従していない部分があり、英語版を適宜参照しながら進めていく形になりました。

- [プログラミング言語Rust](https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/)
- [The Rust Programming Language - The Rust Programming Language](https://doc.rust-lang.org/stable/book/)

## Rustについて
- [プログラミング言語Rust](https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/)

## 検証環境

```bash
$ rustc --version
rustc 1.36.0 (a53f9df32 2019-07-03)
$ cargo --verseion
cargo 1.36.0 (c4fcfb725 2019-05-15)
```

## 開発環境構築、基本的な使い方
事前に各々自由に開発環境を作っていましたが、私はJetBrainsのIntelliJのcommunity版にRustのプラグインを入れてコーディングの環境を整えました。

※参考

- [IntelliJ Rust | JetBrains ブログ](https://blog.jetbrains.com/jp/category/intellij-rust)
- [IntelliJ Rust](https://intellij-rust.github.io/)
- [Quick start](https://intellij-rust.github.io/docs/quick-start.html)

プラグインの導入はPreferenceから行えます。加えて保存時にフォーマッタが走るように設定しました。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/08/f24af622bbe4e8fd30267b26c461246e.png" alt="" width="1006" height="688" class="alignnone size-full wp-image-460728" />

必要なCLIツールのインストールもコピペで終了です。

```bash
$ curl https://sh.rustup.rs -sSf | sh
```

- [Install Rust - Rust Programming Language](https://www.rust-lang.org/tools/install)

### Hello worldまで

プロジェクト用のディレクトリを作ってrustcコマンドを叩くとコンパイルに成功した場合実行ファイルが生成されます。
```bash
$ touch main.rs
$ vi main.rs
```

main.rsの中身

```
fn main() {
    println!("Hello, world!");
}
```

#### コンパイルと実行

```bash
$ rustc main.rs
```

コンパイラがコンパイルに成功したら実行可能なバイナリを出力します。

実行は以下のようにします

```bash
$ ./main
Hello, world!
```

きちんと出力されました。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/08/2c65d5adabc8b3455536a2ed3d18b9f1.png" alt="" width="337" height="77" class="alignnone size-full wp-image-460735" />

余談ですがSwiftもRustと同じ事前にコンパイルが必要な言語です。swiftcというコマンドがあり、同じような方法で実行ファイルが生成できます。

```bash
$ swiftc -o hoge.out hoge.swift
```

## Cargo

> CargoはRustのビルドシステムであり、パッケージマネージャであり、RustaceanはCargoをRustプロジェクトの管理にも使います

他の言語だと複数のツールを使って実現することをCargo一つでやっている印象を持ちました。

- [cargoのどこがいいのか | κeenのHappy Hacκing Blog](https://keens.github.io/blog/2018/08/26/cargonodokogaiinoka/)
- [Why Cargo Exists - The Cargo Book](https://doc.rust-lang.org/cargo/guide/why-cargo-exists.html)

パッケージの追加や削除といった作業はプロジェクトのルートディレクトリ直下のCargo.tomlを編集して行います。

```bash
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

日本語版ではこのように記載されていますが、英語版では以下のように

```bash
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]

```

editionというのが追加されています。

### Edtion
Rustのstable版は後方互換性を高く維持してきましたが、絶え間なく改良されていくコンパイラの設計や言語仕様は互換性に影響のある変更が含まれることがあります。そのような非互換の変更を扱うための仕組みがeditionです。

Rustの公式ブログでも案内がなされているので興味のある人は読んでみてください。

- [Rust's 2018 roadmap | Rust Blog](https://blog.rust-lang.org/2018/03/12/roadmap.html#compatibility-across-editions)

Cargo buildコマンドを叩くと

- targetディレクトリが出来てその下にビルドバリアント名のディレクトリ、その下に実行ファイルが生成されます。`--release`でリリースビルドが行えます。Swiftのコンパイラもそうですが、ビルドバリアントによって最適化のされ方等が異なります。

実行は`$ cargo run`コマンドです。

Cargoでプロジェクトを作成する時は

```bash
$ cargo new project_name
```

newコマンドを使用します。日本語版には`--bin`オプションが指定されていますが、最新版ではbinオプションを指定しなくても実行可能なアプリケーションを作成するようになっています。`--lib`オプションをするとライブラリ用のプロジェクトが生成されます。

今回はアプリケーションの作成のみできれば十分でしたが、せっかくなので`--lib`オプションをつけてnewコマンドを叩いてみます。違うのはsrcディレクトリです。
```bash
drwxr-xr-x  9 hoge  staff   288B  8  1 23:44 .git
-rw-r--r--  1 hoge  staff    30B  8  1 23:44 .gitignore
-rw-r--r--  1 hoge  staff   247B  8  1 23:44 Cargo.toml
drwxr-xr-x  3 hoge  staff    96B  8  1 23:44 src
```
--libオプションを付けるとsrcディレクトリの中身がlib.rsになります。

中のコードは以下のようになっています。

```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

## 数当てゲームを作る

導入に関する説明のあとは数当てゲームを作ってみる章に入りました。　

[数当てゲームをプログラムする - The Rust Programming Language](https://doc.rust-jp.rs/book/second-edition/ch02-00-guessing-game-tutorial.html#a%E6%95%B0%E5%BD%93%E3%81%A6%E3%82%B2%E3%83%BC%E3%83%A0%E3%82%92%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%A0%E3%81%99%E3%82%8B)

順番に進めるだけでこのまま記事にするとただのリライトになるのでここからは学習を進めつつ残したメモや理解するために別で調べる必要のあった概念などを列挙していきます。

### マクロ

hello worldで出てきた`println!`もこれにあたります。

構文レベルでの抽象化を可能にするメタプログラミング機構です。マクロ呼び出しは展開された構文への短縮表現です。展開はコンパイルの初期段階、全ての静的なチェックが実行される前に行われれますが、マクロの実装は頻繁に行なって良いものではなく`最終手段となる機能`と位置付けています。マクロを使用することで適切な抽象化が可能になる場面があるものの、良いマクロの設計が困難なためとドキュメントでは説明されています。

- [ドキュメント](https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/macros.html)

トレイトによって実装を強制されるメソッドを定義したい時にIDEの機能で実装しないといけないメソッドの雛形を用意してくれるのですがその時にもマクロが使われていました。

```
impl HasArea for Circle {
    fn area(&self) -> f64 {
        unimplemented!()
    }
}
```

`panic!`と呼ばれる回復不能なエラーを発生させ、プログラムにエラーメッセージを表示させて終了させるマクロの一つです。`unimplemented!`は"not yet implemented"というメッセージを表示させます。Swiftで同じようなことをしようと思ったら

```swift
func hoge() {
  fatalError("not implemented")
}
```
とかでしょうか。ちなみに`println!`は標準出力に一行分の文字列を出力させるマクロです。

#### Rust標準マクロのソースコード
Rustの標準マクロは一つのファイルにまとめられています。

- [macros.rs.html -- source](https://doc.rust-lang.org/src/std/macros.rs.html)

### `let mut`

```
use std::io;
fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
    println!("You guessed: {}", guess);
}

```

サンプルコード中に以下のように変数を宣言する行があります。
```
let mut guess = String::new();
```
Rustでは変数は標準でimmutableで変数名の前に`mut`をつけて変数を可変にできます。
`::`という記法で左辺の型の関連関数newを呼び出していることを示しています。
この場合Stringの関連関数newを呼び出すことでmutableな空の文字列オブジェクトを生成しています。

実装していくのは数当てゲームなのでこの後このmutableなString型のオブジェクトがユーザーの入力を受け取って入力された文字列が代入されます。

入力を受け取るメソッドread_lineのシグネチャが`pub fn read_line(&self, buf: &mut String) -> Result<usize>`になっていて第２引数は&mutがついたString型となるのでここに渡すための変数を先程宣言していたのだと理解することができます。

- [std::io::Stdin - Rust](https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line)

ここでの返り値がResultという型になっています。エラーが発生する可能性のある操作に対しよく使用される型です。Swiftでも5.0からResult型が使用できます。同じような用途でEitherという型が用意されている言語もありますね。モダンと言われている言語には類似した命名の型や言語機能があることが多いので比較しながら理解しようとするのも楽しいです。

### アトリビュート
Rustで型が提供しなければいけない機能をRustのコンパイラに伝える言語機能をトレイトといいます。

println!というマクロを使用する時に自分で定義した型を出力しようとするとDisplayというトレイトを満たしていないと出力できない旨をコンパイラが教えてくれます。ここで初めてトレイトという概念を知りました。

stdライブラリの型のように自動で出力できるものもありますが、基本的にはトレイトにより実装を強制されるメソッドの実装が必要です。

しかし、デバッグのときに毎回そのためのメソッドを実装するのは手間です。それを回避する方法も用意されています。deriveアトリビュートというものを使います。

```
// この構造体は`fmt::Display`、`fmt::Debug`のいずれによっても
// プリントすることができません。
struct UnPrintable(i32);

// `derive`アトリビュートは、
// この構造体を`fmt::Debug`でプリントするための実装を自動で提供します。
#[derive(Debug)]
struct DebugPrintable(i32);
```

Swiftのattributeという機能に少し似ていて、こちらはコンパイラに対して宣言や型の補足情報を伝えます。deriveアトリビュートも宣言前に記述しています。

Rustではアトリビュートを使って宣言を修飾することができます。

その他のアトリビュートについては[アトリビュート](https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/attributes.html)を参照してください。

### その他
- ファイル名に一つ以上の単語を使うならアンダースコアを使う(例：`hello_world`)
- Rust 2018 Editionから外部モジュールをインポートする時に`extern crate xxxx`ではなくいきなりモジュール名がかけるようになった`use rand::Rng;`
- インデントはスペース4つ。議論は収束していてフォーマッタはタブをオプションでサポートしている。議論はここ[Line and indent width · Issue #1 · rust-dev-tools/fmt-rfcs](https://github.com/rust-dev-tools/fmt-rfcs/issues/1#issuecomment-288847963)

## まとめ
iOSアプリエンジニアの自分がRustを使える場面はそう多くないです。Rustはコンパイルエラー時の出力がすごく丁寧で、所有権などのRustの優れた言語機能を学びつつ、Rustのコンパイラにより良いコードを書けるよう教育してもらうぐらいのつもりでRustに興味を持ち勉強会に参加しました。

最終的にはCLIツールを作るぐらいを当面の目標にこの勉強会でRustの基礎を身に着けたいなと思っています。

これからも引き続き勉強会のレポートを残していきたいとおもいます。また、この記事はTRPLや公式のドキュメントやコンパイルの出力、実際に書いたコードなどを元に構成しています。まだまだ初心者なので誤りがあればコメントや[Twitter](https://twitter.com/t__nabe)などでご指摘いただけるとありがたいです。