# TRPL 社内勉強会

## 所有権を理解する

所有権完全に初見です。誤りや理解にずれがあればご指摘ください。

- ある値の所有権を持っている変数を所有者と呼ぶ
- 変数はひとつの値しか持ていないため、変数が所有権を持っている値はただひとつ
- 値の所有権はひとつしかないので、常に所有者となる変数もひとつ
- 所有者がスコープから外れたら値が破棄される

```rust
fn main() {
    let a = "hello"; // 有効
}                    // スコープが終わっ    た時点でaが有効でなくなる
```

### データの保存領域

- スタティック領域
- スタック
- ヒープ

Swift では変数を定義するとヒープ領域に必要な領域が確保され、定数はスタックに領域が確保されます。また、Swift のコンパイル時の最適化（AllocBoxToStack）でメモリプロモーションが行われ一部のオブジェクトがヒープからスタックへプロモートされたりします。

Rust はスタックの高速性を活かすため、基本的にはローカル変数などの値をスタックにおきます。長く保持する必要がある場合は`Box<T>`型を使ってヒープ領域を確保、そしてそのアドレスを持ったポインタをスタックに確保します。

ヒープに確保することでスコープを気にせず値を保持できますが、スタックへ同時に確保しているポインタが開放された時にヒープからも開放する必要があります。

- [スタックとヒープ](https://doc.rust-jp.rs/the-rust-programming-language-ja/1.6/book/the-stack-and-the-heap.html)

### String 型を通して所有権を理解する

- プログラムにハードコードされる文字列リテラルは不変
- コードを書く際にすべての文字列値が判明するわけではない
  - String 型を使用する

```rust
let string = String::from("hello"); // from関数で引数に文字列リテラルを渡してString型をの値を生成
```

- String 型の値は可変にできる

```rust
fn main() {
    let mut string = String::from("Hello");
    string.push_str(" World!"); // 値が変更される
    println!("{}", string); // Hello World!
}
```

### メモリと確保

- 文字列リテラルの場合、中身はコンパイル時に判明しているので、テキストは最終的なバイナリファイルに直接ハードコードされる
- コンパイル時にサイズが不明だったり、実行時にサイズが可変なテキスト片用にメモリをバイナリに確保しておくことは不可能なので String 型を使う
- String 型はコンパイル時には不明な量のメモリをヒープに確保して内容を保持する
  - 仕様後はメモリを返還する必要がある
- Rust ではメモリを所有している変数がスコープを抜けるとメモリを自動的に返還する
- 変数がスコープを抜ける際に呼ばれる関数が`drop`

```rust
let i1 = 1; // 1をi1に束縛
let i2 = i1; // i1の値をコピーしてi2に束縛

// String型は文字列の中身を保持するメモリへのポインタ、長さ、許容量の3つから構成され、スタックで保持される
let s1 = String::from("hello");  // 定義
let s2 = s1; // スタックにあるポインタ、長さ、許容量の3つをコピーする
```

上記でスコープを抜けると`drop`が呼ばれて変数が使っていたメモリを自動的に返還すると書きましたが、そうなると s1 と s2 が変数を抜けると同じメモリを開放しようとしてしまいます。メモリ安全性を保証するために、 Rust は確保したメモリをコピーする代わりに s1 を有効でないとします。

無効になった参照は使用できなくなります。これが`ムーブ`です。

上記のコードの s1 を println!に渡すと次のようなエラーを返します。

```sh
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `std::string::String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
```

### 所有権の移動

```rust
fn main() {
    let string = String::from("string");
    move_ownership(string); // 所有権が関数に移動する
}

fn move_ownership(string: String) {
    println!("{}", string);
}

```

所有権が移動（ムーブ）しているので`move_ownership`関数に string を渡した後に、println!へ渡すとエラーになります。

```rust
fn main() {
    let string = String::from("string");
    move_ownership(string);
    println!("{}", string); // NG
}

fn move_ownership(string: String) {
    println!("{}", string);
}

```

i32 は Copy というトレイトを実装しているので代入後も変数が使用できます。

- [std::marker::Copy - Rust](https://doc.rust-lang.org/std/marker/trait.Copy.html)

```rust
fn main() {
    let i = 5;
    copy_value(i);
    println!("{}", i); // OK
}

fn copy_value(integer: i32) {
    println!("{}", integer);
}


```

関数の引数で所有権をムーブできるのと同様戻り値でも所有権を移動できます。この要領で所有権をもらっては返してを毎回実装するのは面倒です。関数に値だけを使わせて所有権は渡したくない時に参照と借用というしくみを使います。

## 参照と借用

String 型の値を加工してかつ加工前の値を使用したい場合、参照と借用を用いないなら関数に使用する際移動した所有権を戻さないといけません。

参照と借用のしくみを使い、オブジェクトへの参照のみを関数に渡したい時、`&`を使います。

```rust
let len = calculate_length(&s1); // 渡すときにアンパサンドをつける。関数の引数に参照を取る = 借用
fn calculate_length(s: &String) -> usize { s.len() } // // シグネチャでもアンパサンドを使用して参照であることを示す
```

### 可変の参照

参照を借用しているに過ぎないので参照はデフォルトでは不変です。しかし変数が必要に応じて可変にできるのと同様に参照も可変にできます。

```rust
fn main() {
    let mut s = String::from("元の文字列");
    change_ref(&mut s); // &mutをつける
    println!("{}", s); // 元の文字列, 追加された文字列
}

fn change_ref(string: &mut String) { // シグネチャにも&mutをつける
    string.push_str(", 追加された文字列")
}
```

この可変な参照には特定のスコープで特定のデータに対してひとつしか可変な参照を持ていないという制約があります。

この制約に反するコードを書いてみるとエラーになります。

###
