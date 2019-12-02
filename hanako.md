# タイトル

作ったことのない形式のプログラムを作る時に他の人の実装を見るのは大変参考になります。

実はコマンドラインツールを Swift で作った経験は一度あるのですが、それっきり触っていなかった上に小さな動くコードを継ぎ接ぎしただけのその時さえ問題がなく動けば良いというプログラムでした。
せっかくなので今回は @lovee さんが GitHub で公開されているコマンドラインツール hanako を読んでみます。そして学んだことを元に簡単なコマンドラインツールを作ってみたいと思います。

hanako というプログラムの概要を説明します。指定した長さのランダム文字列を作るコマンドラインツールです。

## Cocoa フレームワーク

ファイルは最初に`import Cocoa`していますが Cocoa は何をしているのでしょう。試しに import コメントアウトしてみます。

するといくつもの行で`Use of unresolved identifier`というエラー文が表示されています。

`arc4random_uniform`や`exit(EXIT_FAILURE)`、 `NSPasteboard`、`NSPasteboardItem`などでこのエラーが表示されます。

`arc4random_uniform`は乱数作成を行うメソッドです。
exit には関してはそのままで exit したい時に使うメソッドです。

```swift
exit(EXIT_SUCCESS)
exit(EXIT_FAILURE)
```

クリップボードを扱うクラスとして`NSPasteboard`、`NSPasteboardItem`などがあります。

ここまでクラスやメソッドなどを見ていった Cocoa ですが、 macOS アプリケーションを構築するためのフレームワークとして提供されています。

また、Objectice-C をコア言語とするオブジェクト指向フレームワークで Foundation と AppKit の二層構造になっています。

## parseCommand()メソッド

Cocoa フレームワークについて簡単に説明したところでコードリーディングに入っていきます。

それなりに長いコードですが実際に実行されているのは parseCommand()メソッドだけです。

parseCommand()を一つずつ見ていけば問題なさそうです。

```swift
let arguments = Array(CommandLine.arguments.dropFirst())
```

定数`arguments`には何が入るのでしょうか。
命名とこのアプリケーションがコマンドラインツールであることから想像できますが呼び出し時のユーザーが渡した引数一覧が取得出来るのでしょう。

`dropFirst()`メソッドは与えられた数の初期要素を除いて全ての要素を返すメソッドです。`CommandLine.arguments`にはコマンドラインから引数の一覧が入っていますが arguments[0]にはコマンド名が入ります。このプログラムを実行した場合は`hanako`が入っているはずです。

その後では定数`arguments`が`-h`オプションまたは`-v`オプションを含んでいるかを確認して含んでいればブロック内の処理を行うようになっています。

-h オプションは help で-v はバージョンを出力するオプションです。その後は exit で処理を抜けています。

-h オプションの時の処理では`printHelp()`、-v オプションの時は`printVersion()`メソッドが実行された後 exit で抜けます。

両メソッドともに文字列が代入された定数を print するメソッドで、それぞれ内部にはオプションなどの使い方の一覧、バージョン情報が定数に代入されています。

### オプションごとの分岐

arguments に-h オプションと-v オプションが含まれていない場合は switch 文への分岐に入ります。

while を使った optional binding を行って arguments.first が nil になるまでブロック内の処理を繰り返します。ブロック内では変数 arguemnt の中身で分岐する switch 文で、それぞれの処理が終わった後に removeFirst しています（exit(EXIT_FAILURE)で終了するもの以外）。

while の中身の概要はわかった所で内部の処理を一つずつみていきたいところですが、その前に先程の printHelp()メソッドで print される文字列を見ていきましょう。

分岐ごとに何をするのか見ていくよりもざっくりそれぞれの分岐で何が行われるのか把握しておいた方が理解が楽になるかと思います。

```swift
let help =
	"hanako is a random string generator.\n" +
	"\n" +
	"Usage:\n" +
	"\t$ hanako [options]\n" +
	"\n" +
	"Arguments:\n" +
	"\t-h / --help: Help. (This will ignore all other arguments and won't perform an output.)\n" +
	"\t-nu / --no-uppercase: Don't use uppercased string for the output.\n" +
	"\t-nl / --no-lowercase: Don't use lowercased string for the output.\n" +
	"\t-nn / --no-numeral: Don't use numeral string for the output.\n" +
	"\t-nc / --no-copy: Don't copy the result to pasteboard" +
	"\t-l [length] / --length [length]: Output length. Replace [length] with a number. Default length is 10.\n" +
	"\t-hf [frequency] / --hyphen-frequency [frequency]: Insert a hyphen after every certain characters. Replace [frequncy] with a number. e.g.: abc-def-ghi if you set frequency to 3, abcdefghi if you set frequncy to 0. Default frequency is 0, and hyphens are also counted in the length.\n"
```

この文字列からわかることを既知のことを除いて列挙していきます。

- `-nu`オプションは出力に uppercase を含めないオプション
- `nl`オプションは出力に lowercase を含めないオプション
- `-nn`オプションは出力に数字文字列を含めないオプション
- `nc`オプションは出力をクリップボードにコピーしないオプション
- `-l`オプションは出力文字数を指定できるオプション。デフォルトは 10
- `-hf`オプションは`--hyphen-frequency`の略で、出力文字列にどれだけの感覚で`-`を挿入するか指定できるオプション。デフォルトは 0.例: -hf に数字 3 を渡すと`abc-def-ghi`などと出力される。

help コマンドを見たおかげで各分岐で何が行われるのか予想がつくようになりました。それでは switch 文に戻ります。

help の内容と switch 文の中身を見る限り、この処理は argument の値、つまり仕様時の実行コマンドの引数によって出力される文字列の形式やクリップボードにコピーするかどうかを確定させるためのもののようです。

#### "-nu" || "--no-uppercase"

いくつかの分岐、特に出力文字列の形式を変更するオプションでは types.remove()が実行されています。

`types`は`Set<CharacterType>`型で、`CharacterType`はこのファイルで定義されている enum です。

```swift
private enum CharacterType {
	case uppercasedAlphabet
	case lowercasedAlphabet
	case numeral
}

private var types: Set<CharacterType> = [.uppercasedAlphabet, .lowercasedAlphabet, .numeral]

```

デフォルトでは uppacase な alphabet、lowercase な alphabet、numeral を含んだランダム文字列が返却され、--no-`xxx`で`xxx`をこの types プロパティから remove しているようです。`-nu`オプションは`--no-uppercase`の略と定義されているので、`xxx`には uppercase が入り uppercase な文字列は返却される文字列に含まれないようになりました。
types の中身が`[.uppercasedAlphabet, .lowercasedAlphabet, .numeral]`だった場合この分岐を抜けた後は`[.lowercasedAlphabet, .numeral]`となるはずです。

これで`-nu`オプションをについての説明は終わりで、同時に`-nl`オプション、`-nn`オプションの説明も不要になりました。それでは次の分岐を読みましょう。

#### "-nc" || "--no-copy"

この分岐のとき、内部では`shouldCopyToPasteboard`という変数に false が代入されます。 デフォルト値は true なので、デフォルトではクリップボードに返却される文字列はクリップボードにコピーされるようです。
その後は`removeFirest()`だけなので説明を省きます。
ところでこの変数の値はどのタイミングでどのように使われるのでしょう。
確認してみると`shouldCopyPasteboard`が true の時に copyStringToPastebouard メソッドに定数 result を渡して実行しています。result の中身は後ほど見ていきますが出力される文字列なことはある程度予測できます。print ではクリップボードにコピーした旨を print でユーザーに知らせています。

```swift
if shouldCopyToPasteboard {
  copyStringToPasteboard(result)
  print("Result has been copied to your clipboard, you can use cmd + v to paste it.")
}
```

その後 copyStringPasteboard(\_:)が使われている形跡はないため、せっかくなので今からこのメソッドの中身を見てしまいましょう。おそらくコマンドラインツールを作る上でクリップボードに任意の文字列をコピーする方法がわかるかと思います。

```swift
func copyStringToPasteboard(_ string: String) {
  let board = NSPasteboard.general
  board.clearContents()
  let item = NSPasteboardItem()
  item.setString(string, forType: .string)
  board.writeObjects([item])
}
```

定数やメソッド名からクリップボードを管理するクラスのインスタンスを作り、現在のクリップボードの中身を clear して、クリップボードに書き込むためのオブジェクトを初期化して引数に渡された文字列をセットして書き込んでいることが理解出来ると思います。

[レファレンス](https://developer.apple.com/documentation/appkit/nspasteboard)を見ると NSPasteboard クラスは pasteboard server とデータを送受信するためのオブジェクトです。`general()`でインスタンスを作成した後、`clearContents()`で中身をクリア、`NSPasteboardItem`インスタンスを作成しています。

`NSPasteboardItem`インスタンスはクリップボードへの書き込み、データの編集、データの読み取りに使われます。今回は書き込みが目的なので、`setString(_:forType:)`メソッドを使って item プロパティに書き込みたい情報をセットして`writeObjects(_:)`メソッドでクリップボードに書き込みます。

#### "-l" || "--length"と"-hf" || "--hyphen-frequency"、ついでに default

両方オプションの後に値が続き、Int に変換できなければエラーを返すようになっています。

```swift
guard arguments.count > 1, let lengthParameter = Int(arguments[1]) else { ... }
```

失敗したら両方エラー文字列を出力して`exit(EXIT_FAILURE)`で抜けます。
問題なく Int に変換出来たら両方プロパティが用意されているので代入します。
その後オプションと引数で arguments を 2 つ使用するので removeFirst の引数にも 2 を渡して 2 つ arguments から取り除きます。

default は`printError()`と`exit(EXIT_FAILURE)`なので説明は不要かと思います。

これで swiftch 文の中身は読み終わりました。

### createRandomString()

いよいよこのメソッドを読み終わったら全て終了です。

デフォルトの types プロパティは`[.uppercasedAlphabet, .lowercasedAlphabet, .numeral]`ですが`--no-`系のオプションを全て有効にするとランダム出力する文字列に指定できる文字列がなくなってしまうので`exit(EXIT_FAILURE)`を返します。
