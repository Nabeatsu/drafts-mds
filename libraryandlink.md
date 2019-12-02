iOSDC で[kishikawa katsumi（@k_katsumi）さん](https://twitter.com/k_katsumi)による「ライブラリのインポートとリンクの仕組み完全解説」というセッションを聞かせていただきました。

> <script async class="speakerdeck-embed" data-id="db2ba0482e2e4eceb83dff7f52eeea5b" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
>
> インポート・リンクの仕組みがよくわかっていない状態だと、エラーと自分の加えた変更が結びつかないので、よくわからないエラーが無限に起こっていると感じます（同じエラーメッセージを引き起こす原因は複数あるため）。
>
> しかし実際はそうではないので、可能性の高い順番で確認していけば問題を解決できます。
>
> そのための基礎として、インポート・リンクの仕組みを理解が必要になります。
>
> モジュールのインポート・リンクがどのように解決されるのか、リンクとはいったい何をしているのかを学ぶことで、システマチックに問題を切り分け、解決できるようになります。
>
> [iOSDC 2019 で「ライブラリのインポートとリンクの仕組み完全解説」という話をします - 24/7 twenty-four seven](https://blog.kishikawakatsumi.com/entry/2019/09/05/074831)より引用

## 感想

日ごろの開発では、さまざまな理由でライブラリやフレームワークを使用します。

それらの導入の過程でリンクエラーやパッケージマネージャーが出力するエラーメッセージに苦しめられてきました。そういう時はエラーメッセージでググったりしつつそれらしいものを試しながらエラーを解決してきました。

このセッションでは

- ライブラリの形式
- なぜ問題の解決が難しいのか
- モジュールのインポート・リンクのしくみ
- リンク、インポートは何をしているのか

をひとつずつ解説しながら進められていたので曖昧な知識しかない自分にはありがたかったです。

ライブラリの利用者である自分たちがスキルを向上させることは作成者・利用者ともにメリットがあり、さらに Slack などでのコミュニケーションで問題を解決するのが難しいので各々が知識をつけると良いということでした。

## メモ

- CocoaPods や Carthage などのパッケージマネージャ
- Xcode の機能
- マニュアル設定

などでライブラリを利用できる。

### ライブラリの形式と違い

- framework
  - .framework
- dynamic library
  - .dylib
- static library
  - .a

これではすまない。

#### 問題を複雑にしている部分

- フレームワークで dynamic static をアイコン（見た目）で判断できない
- モジュールを含むフレームワークか
- 何の言語で作っているのか
- アーキテクチャの違い
- プラットフォーム間の差異

が掛け算になっているのが問題の把握を難しくしている。

一度に関係するのは縦の関係で横の関係は別の問題ですので、詰まったらまず縦を潰す。

### フレームワークとライブラリの違い

- Bundle をもつ可能性があるのがフレームワーク
- 持たないのがライブラリ

#### Bundle

- 開発を便利にするための、ファイルのように扱えるディレクトリ
  - ダブルクリックでアプリケーションが開いたりプロジェクトが開いたりする
  - 一定のルールを持った構造
  - ライブラリと違ってフレームワークには Resources など決まっている
  - コード以外のものをフレームワークには含めることができる

#### Static Link と Dynamic Link

- Static link はビルド時にターゲットに結合されるが Dynamic Link は参照だけを持ち実行時に解決される

iOS アプリケーション開発においては Bundle を持ているのでライブラリよりフレームワークの方が便利。また、static より dynamic の方がやりやすい。

スタティックリンクを複数のターゲットでリンクするとシンボル衝突だけでなく、アプリケーションサイズも大きくなっていいことがない。

#### dynamic link か static link かを見分ける

dynamic link、static link
file コマンド
ライブラリの本体を与えると
dynamic か static か分かる

## import と link

いつも Xcode が暗黙的にやっていることを理解する。

### import

- 外部ライブラリを自分のコードベースで利用可能にする
- ライブラリが公開しているシンボルを自分のコードで参照できるようにする
- 各ファイルのコンパイル時で解決される

### link

- ソースコードをコンパイルして作られたオブジェクトファイル、外部ライブラリを連結（リンク）してひとつの実行形式ファイルにする
- Linker が行う
- すべてのソースコードをコンパイルした後に行う
- static,dynamic にかかわらずすべてのシンボルが解決される。そうでない場合はリンクエラーに
  - コンパイルで生成されたオブジェクトファイルのリンクは失敗しない
    - 起こり得るのは外部ライブラリとのリンク
- ビルドログを見るとコンパイルした後にリンクしているのが分かる

リンクに対して公開しているものだけがわかればよい。

#### ライブラリを利用する一般的（暗黙的）な方法

Xcode を使って外部ライブラリを利用可能にする。

- Embedded Binaries
  - 入れると Linked Frameworks and Libraries にも入る
  - Xcode11 からは Link 設定が 1 つになって embed するか選択できるようになる。

Xcode が設定するもの

- FRAMEWORK_SEARCH_PATH にフレームワークがあるディレクトリの親ディレクトリの追加
  - 外すとインポートでエラーになる
- Linked Framework and Libraries にフレームワークが追加
  - Automatic Linking を無効にする（-disable-autolink-framework を OTHER_SWIFT_FRAG に入れる）
    - リンクエラーになる
    - storyboard だけで設定するとアプリケーションがクラッシュする。それは適切なものを import（Link）していないから
- Build Phase に Copy Files Phase を追加してフレームワークをアプリケーションバンドルの Frameworks ディレクトリにコピーする
  - ファイルを消すと dynamic ならビルドに成功するが実行時に失敗する
  - `dyld: Library not loaded`

次はインポートとリンクをマニュアル作業

- フレームワークからファイルを消してみる
- インポートで必要なファイルだけ残してほかを消す。インポートに必要なファイルは arm64.swiftmodule と x86...swiftmodule だけ
- コンパイルエラーは起きずにリンクエラー

##### モジュール

- Swift の framework には hoge.swiftmodule というディレクトリがある
- BUILD_LIBRARY_FOR_DISTRIBUTION = YES にすると.swiftInterface ファイルが生成される
- Swift 以外で作った framework には modulemap が作成される
- Swift がライブラリをインポートするのに必要
- Public な API を宣言する
- Swift 製のライブラリではモジュールをコンパイラが自動生成する
- Objective-C/C のライブラリを import するには.h(ヘッダ)ファイルを Modulemap でモジュールに変換する

##### Bridging Header と Module Map の違い

- モジュール map は bridging header の上位互換
- Bridiging Header はアプリケーションターゲットでしか使えない
- Bridging Header はグローバル
  - 必要なところで import できない

Swift5.1 から モジュール Stability があるのでライブラリをリコンパイルする必要はない。

モジュールのインポートは`SWIFT_INCLUDE_PATHS`
ライブラリのリンクの方法

- LIBRARY_SEARCH_PATHS に指定。
- OTHER_LDFLAGS に-l<LIbraryName>

info.plist はいらないが、ないと iOS とかシミュレーターにインストールができない。

リンクエラーの 1 つにアーキテクチャ・プラットフォームが一致しなければいけないというものがある。
全アーキテクチャをまとめる Universal Binary を作るために xcrun libo(ライボ)

# 最後に

発表をしていただいた[kishikawa katsumi（@k_katsumi）さん](https://twitter.com/k_katsumi)、本当にありがとうございました。
