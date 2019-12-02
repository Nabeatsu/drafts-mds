# R.Swiftのアップデートによるエラーを解決するついでにRun Scriptについて整理してみた



こんにちは。クラスメソッド福岡オフィス CX事業本部でiOSアプリエンジニアとして働いている田辺です。手元のプロジェクトがのSwiftのバージョンが4.0なのですが、5.0にアップデートしたくて少しずつ進めていました。

プロジェクトの依存ライブラリの一つにR.Swiftがあり、ビルドが通らなかったのでエラーメッセージを見ていると

```swift
error: [R.swift] Output path must specify a file, it should not be a directory.
error: [R.swift] Build phase Intput Files does not contain '$TEMP_DIR/rswift-lastrun'.
error: [R.swift] Build phase Output Files do not contain '$SRCROOT/R.generated.swift'.
```

というエラーありました。

R.Swiftのこのエラーは以下の画像のように修正する必要があります。また、変更の手順は[R.swift/Migration.md at master · mac-cain13/R.swift](https://github.com/mac-cain13/R.swift/blob/master/Documentation/Migration.md)を参照するようエラーで示されています。

https://gyazo.com/e3770ae21c9bb21443c70cc032c6957d

そもそもRun ScriptなどのBuild Phaseについてきちんと理解していないとBuild Phrasesについて触れる時は指示通り従うだけになってしまうので良い機会なので調べてみました。

## Run Script

Xcodeでは、Build Phasesセクションの`Run Script Phase`を使用して、ビルドプロセスの一部としてユーザーが定義したコードを実行することができます。

Build Phasesではターゲットをビルドする時に様々なタスクを実行できるよう設定できます。

Run ScriptはそんなBuild Phasesのうちのひとつです。

- Compile sources
- Copy bundle resources
- Copy files
- Headers
- Link binary with libraries
- Run script
- Target dependencies

### Compile sources

Compile sourcesではターゲットを構築する過程でどのソースファイルをコンパイルすべきかをコンパイラに知らせることができます。

プロジェクトで複数のターゲットのソースを含めたり除外するIdlesの例に加えて、 "Compile Sources"からソースを除外する可能性があるもう1つの現実的なシナリオは、サードパーティのライブラリ（またはクラスのセット）を使用する場合です。 ソースコードを提供しますが、どのソースをプロジェクトでコンパイルするかを制御したいと考えています。サンプルコード以外にも大規模なフレームワークで特定のAppleのフレームワークをサポートしている場合があります。そのサポートが必要ないことが確定している場合などが考えられます。

ファイルを追加した時にtargetに追加するオプションを選択するのを忘れた時に後になってコンパイルするファイルに含める時に使用することができます。正直こちらの用途でお世話になることの方が多そうだなと感じます。

### Copy bundle resources

Build PhasesのCopy bundle resourcesにあるものはリソースとして内部から扱うことができるようにコピーされます。リソースはターゲットに関連付けられ、ビルド後のプロダクトのResourcesサブフォルダにコピーされます。

プロジェクト内のファイルを使用する際に`Bundle.main.path(forResource:ofType:)`というメソッドを使ってファイルまでのパスを取得する時があります。Copy bundle resourceで指定したファイルにはこのメソッドでアクセスできます。

### Copy files

プロジェクトファイルと他のターゲットのプロダクトをターゲットと関連付けます。Copy filesというBuild phaseは、すべてのビルドに対して有効にすることも、インストールビルド中にのみ有効にすることもできます。

### Headers

フレームワークの定義をAppleの[Frame work Programming Guide]を引用すると

> A framework is a hierarchical directory that encapsulates shared resources, such as a dynamic shared library, nib files, image files, localized strings, header files, and reference documentation in a single package.

とあります。フレームワークのリソースの一つと考えて良さそうです。

Wikipediaによると、コンパイラが別のソースファイルの一部として自動的に展開して使用するファイルとあります。

HeadersというBuild PhaseではPublic 、Private、Projectのヘッダーファイルをターゲットに関連付けルことができます。 パブリックヘッダーとプライベートヘッダーは、他のクライアントによる使用を目的としたAPIを定義し、インストール用にプロダクトにコピーされます。

ライブラリ、フレームワークをプロジェクトの導入で触ったことがある人もいるかと思います。

### Link binary with libraries

ライブラリやフレームワークの導入の際に使います。ビルドのタイミングでアプリケーションにリンクされるライブラリやフレームワークの導入です。

Xcodeで外部から導入できるモジュールにはStatic Library、Dynamic Library、Frameworkがあります。Static Libraryは文字通り静的に組み込まれるライブラリのことです。コンパイル時にリンクされます。Dynamic Libraryはランタイム時に実行されます。.dylibという拡張位をみたことがある人もいるかと思います。

.frameworkという拡張子がFrameworkです。画像やxibを含むことができます。XcodeのNewからプロジェクトを新規作成してフレームワークを作るとframeworkが作れます。

アプリケーション内でFrameworkやLibraryのAPIを参照したい時はこちらで設定していきます。

### Target dependencies

現在のターゲットを構築する前に構築する必要があるもう1つのターゲットをここで指定します。 たとえば、アプリターゲットとフレームワークターゲットがある場合、アプリのターゲットはフレームワークに「依存」しています。ライブラリを追加する時などにLink binary with librariesと共に設定する必要があります。

### Run scriptの新規作成

Build Phasesそれぞれについて説明したのでRun scriptに戻ります。

ProjectのBuild Phasesで「+」ボタンをタップして、`New Run Script Phase`を選択する。Run Scriptは上から順番に実行されていきます。文字列を出力するだけのスクリプトを書いてBuildしてみます。

Report Navigatorの直近のBuildをクリックして先ほど入力した文字列を探しみると確かに出力されているのがわかります。	

https://gyazo.com/112bd8fa0c9fb4772e8bcaba70b53d9f

### Scriptの実行を特定の構成に制限したい

あると思います。そういう場合は以下のようにします。

```sh
if [ "${CONFIGURATION}" = "ここに任意の構成" ]; then
    echo "hogehoge"
fi
```

### 環境変数

[ドキュメント](https://developer.apple.com/library/archive/documentation/DeveloperTools/Reference/XcodeBuildSettingRef/0-Introduction/introduction.html)はこちらになります。

*それぞれの環境変数が何を指すか確認できる方法があります。

Add Run Script Build PhaseでRun Scriptを追加して、Shell下の入力欄に
' env > ./env.txt' と入力後ビルドするとenv.txtファイルが生成されるのでその中に書かれている。

https://gyazo.com/80811724e0b2b6ac7996120651c19926

ビルドすると指定するとenv.txtファイルが生成されていて開くと環境変数の中身が全て見れます。非常に長いので画像を貼ります。

https://gyazo.com/ccc94d0d18853b63377d6d4614052d24

### Xcode10以降のRun Script

Xcodeのビルドシステムが変わり、ファイル管理の方法も変わりました。

実は今回の記事のきっかけになったR.Swiftのエラーもそれに関係するものです。

シェルスクリプトによって生成されるファイルがビルド時に別のどこかでInput fileとして使われる場合、その際に生成されるファイルをOutputとして明示的に設定される必要があります。ない場合はビルドエラーが発生します。

※参考

- [Build System Release Notes for Xcode 10 | Apple Developer Documentation](https://developer.apple.com/documentation/xcode_release_notes/xcode_10_release_notes/build_system_release_notes_for_xcode_10)

## まとめ

この辺りはライブラリの導入周りで触れるぐらいで通常の開発で頻繁に目にする設定項目ではありませんでした。XCodeのビルドの仕組みや各種設定項目の内容を理解することで同様のエラーが返ってきた時にきちんと対応できると思います。

これからもトラブルやエラーが発生した時は原因と背景をきちんと理解して解決して、できればブログに書けるようにしたいと思います。

