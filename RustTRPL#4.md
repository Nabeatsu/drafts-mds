クラスメソッド 福岡オフィスで iOS アプリケーションエンジニアとして働いている田辺です。先週に続き、社内で Rust の勉強会が行われたのでそのレポートを記事にします。

前回までの記事。

- [[社内勉強会レポート]『The Rust Programming Language』勉強会#1 ｜ DevelopersIO](https://dev.classmethod.jp/server-side/language/trpl-study-meeting-report/)
- [[社内勉強会レポート]『The Rust Programming Language』勉強会#2 ｜ DevelopersIO](https://dev.classmethod.jp/server-side/trpl-study-meeting-report-part2/)

今回から Rust の言語仕様について解説する章に入るので勉強会で話題になったところや個人的に気になったところのみを扱います。

## 変数と可変性

Rust の変数は標準で不変です。可変な変数が必要な場合に mut キーワードを使って明示的に可変にします。変数の宣言には let を使います。

```
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

定数は const キーワードで、大文字のアンダースコアで単語区切りして宣言します。Rust での変数と定数の違いは以下です。

- 値の型を注釈する必要がある
- グローバルスコープを含めてどんなスコープでも定義できる
- 定数は定数式にしかセットできない。

### シャドーイング

```
fn main() {
    let value = 5;
    let value = value + value;
    let value = value * value;
}

```

アプリケーションエンジニア

Rust ではシャドーイングが可能ですので、上記のように前に定義した変数と同じ名前の変数を新しく宣言できます。

変数 value を 5 で束縛し、二度目の`let x =`で x を覆い隠し value に value を足した値を宣言するので 10 になります。三度目の`let x =`も x を覆い隠し元の値に元の値をかけた数になるので value の値は 100 になります

シャドーイングのメリットとして挙げられていた異なる変数の名前を考える必要がなくなるというのはたしかによいなと思いました。値の型を変更する程度なら同じ名前でシャドーイングした方が楽です。

Swift では optional binding を行う時にシャドーイングが行われていますが、Rust のようなシャドーイングはできません。

```swift
var string: String? = ""
if let string = string { // Optional(String) から String型になっている
    print(string)
}
```

## 関数

```
fn main() {}

fn print_number(x: i32) {
    println!("x is {}", x);
}

```

fn キーワードで関数の宣言を行い、型を明示的に宣言する必要があります。
このように設計されている理由は TRPL にある次の一文で納得できました。

> 関数の型を明示することは強制しつつ、関数本体では型を推論するようにすることが、すべての箇所で型推論をするのとまったく型推論をしないことの間のすばらしいスイートスポットである

> <https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/functions.html>

### 式と文

[[社内勉強会レポート]『The Rust Programming Language』勉強会#2 ｜ DevelopersIO](https://dev.classmethod.jp/server-side/trpl-study-meeting-report-part2/)で触れました。

すでに束縛されている変数への割り当てられている値は空のタプル`()`です。割り当てられる値には`単一の所有者`しかいないからです。

```
fn square(x: i32) -> i32 {
    x * x
}
```

`x * x`に`;`がないですが、`;`がついてしまうと`()`を返します。
cargo run するとコンパイラが`;`を削除するよう提案してきます。

```shell
 --> src/main.rs:5:22
  |
5 | fn square(x: i32) -> i32 {
  |    ------            ^^^ expected i32, found ()
  |    |
  |    this function's body doesn't return
6 |     x * x;
  |          - help: consider removing this semicolon
  |
  = note: expected type `i32`
             found type `()`

error: aborting due to previous error
```

### 発散する関数

Rust には値を返さない関数、発散する関数のための構文がいくつか用意されています。

```
fn hoge() -> ! {
    panic!("hoge")
}
```

`panic!`は実行中の現在のスレッドを与えられたメッセージとともにクラッシュさせるマクロです。

- [std::panic - Rust](https://doc.rust-lang.org/std/macro.panic.html)

この関数は値を返さないため、`!`型を持ちます。読み方は diverges です。

## プリミティブ型

Rust では言語にもともとある型をプリミティブ型といいます。

### 配列のスライス

他のデータ構造への参照で、コピーを行うことなく配列の要素への安全で効率的なアクセスをするのに便利です。

```
fn main() {
    let int_array = [0, 1, 2, 3, 4];
    let complete = &int_array[..];
    let middle = &int_array[1..4];
    println!("complete の要素数は{}", complete.len()); // complete の要素数は5
    println!("middle の要素数は{}", middle.len()); //  middle の要素数は3
}
```

## if

### if 条件のところに書けるもの

if の後の式を評価し true ならブロックが実行されます。動的型付き言語の if に近いと TRPL にはあります。

```
let x = 0;

if x == 0 {
    println!("x は 5です!")
}
```

Boolean 以外は NG で、それを守らないとコンパイル時に警告が出ます。

```
let x = 0;

if x {
    println!("x があります。")
}
```

```shell
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if x {
  |        ^ expected bool, found integer
  |
  = note: expected type `bool`
             found type `{integer}`

error: aborting due to previous error
```

また、if は式なので次のように書くことができます。

```
let y: i32 = if true { 10 } else { 15 };
```

当然ですが、この場合各ブロックの最後の式で型を一致させる必要があります。一致していない場合コンパイラが警告を出します。

```
let y = if true {
    10
} else {
    "10" // NG
};
```

```shell
error[E0308]: if and else have incompatible types
 --> src/main.rs:5:9
  |
2 |       let y = if true {
  |  _____________-
3 | |         10
  | |         -- expected because of this
4 | |     } else {
5 | |         "10"
  | |         ^^^^ expected integer, found &str
6 | |     };
  | |_____- if and else have incompatible types
  |
  = note: expected type `{integer}`
             found type `&str`

error: aborting due to previous error
```

`;`で終わる文の結果は`()`になるので、`10;`では`i32`の`10`ではなく`()`になります。このことは実際に 10 の後に`;`をつけてみると IDE の型推論が i32 から()になっていることからもわかります。

Rust のセミコロンは難しいです。

- [Rust の文でセミコロンを省略してよい条件 - 簡潔な Q](https://qnighy.hatenablog.com/entry/2017/04/22/070000)

## loop

無限ループには loop 、条件付きのループには while 、回数付きのループには for を使います。

### Range オブジェクト

for に渡せるオブジェクトは`IntoIterator`というトレイトを実装している必要があります。

- [std::iter::IntoIterator - Rust](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html)

Range オブジェクトは[満たしているので](https://doc.rust-lang.org/std/ops/struct.Range.html) for に渡すことができます。

```
for x in 0..10 { // 0..10はRange<i32>オブジェクト
    println!("{}", x); // x: i32
}
```

### ループラベル

break や continue は最内のループに適用されるのがデフォルトですが、外側のループに対して break や continue を使いたい状況ではラベルを使用できます。

```
'outer: for x in 0..10 {
    'inner: for y in 0..10 {
        if x % 2 == 0 { continue 'outer; } // x のループを継続
        if y % 2 == 0 { continue 'inner; } // y のループを継続
        println!("x: {}, y: {}", x, y);
    }
}
```

### ループの早期終了

ループの早期終了には break や continue が用意されています。

```
let mut x = 5;
loop {
    x += x -3
    println!("{}", x);
    if x % 5 == 0 { break; }
}
```

また、`return`を使用することでループを中断して最内の関数から抜けることができます。

```
fn main() {
    for _ in 0..3 {
        for x in 0..10 {
            if x % 2 != 0 { break; }
            println!("{}", x);
        }
    }
}
```

上記のコードで実行すると次のように出力されます。

```shell
0
0
0
```

break を return に置き換えると、関数から抜けてしまうので 0 とのみ出力されます。

```shell
0
```

と出力されます。

### break

break は loop でのみ書けます。

```
fn main() {
    let mut x = 5;

    let _y = while true {
        x += x - 3;

        println!("{}", x);

        if x % 5 == 0 { break "a"; }
    };
}
```

loop を while true に置き換えるとコンパイラが警告をだします。

```
fn main() {
    let mut x = 5;

    let _y = while true {
        x += x - 3;

        println!("{}", x);

        if x % 5 == 0 { break "a"; }
    };
}
```

```shell
error[E0571]: `break` with value from a `while` loop
 --> src/main.rs:9:25
  |
9 |         if x % 5 == 0 { break "a"; }
  |                         ^^^^^^^^^ can only break with a value inside `loop` or breakable block
help: instead, use `break` on its own without a value inside this `while` loop
  |
9 |         if x % 5 == 0 { break; }
  |                         ^^^^^

error: aborting due to previous error

For more information about this error, try `rustc --explain E0571`.
```

## 最後に

値や式を変えてみるとどうなるのか試してみたり、Rust に詳しい人がより丁寧に教えてくれるおかげで、ただ読んだり写経するだけより楽しく TRPL を読めるなと感じました。

次からは所有権になります。Rust の重要な概念のひとつなのでしっかり理解したいです。

また社内向けですが、この社内勉強会ではプライベートリポジトリで進捗やメモの管理をしているので途中から参加できます。興味のある社内の方は参加してみてください。
