# The Complete 2020 Flutter Development Bootcamp with Dartを最後までやったので感想

合計で30時間程のFlutterの基礎的な所かProviderを使ったState管理の基本まで抑えたUdemyのオンライン講座です。Flutterの学習にあたり参考にしていた[Flutterの効率良い学び方 - Flutter 🇯🇵 - Medium](https://medium.com/flutter-jp/flutter-learning-c5640c5f05b9)で知って購入し最後まで終えたので記事にしようと思います。書評記事が良いならこれも良いかなと。

Flutterでの開発における基本的な概念を全てカバーしていると謳っています、また、GoogleのFlutter teamと共同で制作したコースなので品質はあまり心配していませんでしたが、Dartの言語機能は必要に応じて本題のサンプルアプリケーションの講義とは別に解説されます。この部分はプログラミングの経験がある人は飛ばしてしまっても構わないと講座中でも言われていたので迷いましたがこのような記事を書くと最初から決めていたので飛ばさずに全部やりました。

## 概要

この講座は全て英語になりますが、字幕もあります。また、GoogleやAppleなどが関わるドキュメントや動画は非ネイティブの人でもわかりやすいように平易な表現が使われていることが多いですが、この講座も同じ様に難易度の高い用語はめったに出てこず、出てきたとしても事後の説明があるか前後のコンテキストから類推できるレベルで問題なかったです。それでもどうしてもわからなかった時が何度かあったのでその際は止めて単語を調べたりしました。

### Introduction to Cross-Platform Development with Flutter and Dart

Flutterを一度も触ったことがない人もこの講座の対象ユーザーに含まれているのでこのセクションでFlutterとDartの特徴について図を用いながら説明があります。

#### このセクションで学ぶこと

- コース自体の説明
- Flutterとは何か
- ネイティブアプリケーションを開発する言語がプラットフォームごとに用意されている現状でなぜFlutterを学ぶ価値があるのか
- このコースの進め方

### SetUp and Installation

#### このセクションで学ぶこと

- Flutterの開発環境構築
- Android StudioでのFlutter開発を便利にすすめるための解説

### I Am Rich - How to Create Flutter Apps From Scratch

このセクションから毎回サンプルのアプリがあります。最初にStub Projectをcheckoutしてそこから講座を進めながら手元でも手を動かした書いてアプリを触ってみることを推奨されています。

#### このセクションで学ぶこと

- Hot Reloadの体験
- Scaffold Widget
- 画像を利用する手順(ディレクトリを作成してpubspec.yamlにAssetsを管理するフォルダとして設定してImage Widgetを使って表示する所まで)

### Running Your App on a Physical Device

手元のデバイスでアプリをビルドして触って見る所までを試します。Flutterのドキュメントに実機ビルドの手順が書いていて既に済ませていたのでこの講座は見ているだけでした。

### MiCard - How to Build Beautiful UIs with Flutter Widgets

ペライチのアプリをScratchで作っていきます。主にFlutterのWidgetを使ったレイアウト構築をどのように行うのか体験してもらうのが目的だと思います。複雑なレイアウトも登場しません。StatefulWidgetは登場せずStatelessWidgetのみを使用します。

#### このセクションで学ぶこと

- Container Widgetの使い方
- Column Widgetの使い方
- Row Widgetの使い方
- 公開されているサードパーティのフォントをアプリ内で使うための手順

### Dicee - Building Apps with State

画面上にサイコロをタップするとサイコロがふられて、目が変わるアプリを作りながらStatefulWidgetを体験します。このセクションから本題とは別にDartの言語仕様について解説することも増えてきます。

#### このセクションで学ぶこと

- StatefulWidgetの基本
- Dartでの関数、データ型について
- GestureDetector Widgetを使ったユーザーインタラクティブなUI実装

### Xylophone - Using Flutter and Dart Packages to Speed Up Development

音声ファイルを扱うためのライブラリを利用しながらFlutter Packages、Flutterでの外部ライブラリの利用方法を扱います。

#### このセクションで学ぶこと

- https://pub.dev/ について
- ライブラリの利用法 pubspec.yamlやpubコマンド
- [audioplayers | Flutter Package](https://pub.dev/packages/audioplayers)を使った音声ファイルの扱い方

### Quizzler -Modularising & Organising Flutter Code

この辺りから少しだけコード量が増えます。そのためファイルやモジュールに分割する必要が出てくるのでその辺りについて解説されます。最後の章までは通して状態の更新はsetState、クロージャの引き回しで行われます。

#### このセクションで学ぶこと

- Flutter(というよりDart)での抽象化、カプセル化、継承、ポリモーフィズムの利用方法
- Dartにおけるコンストラクタ


### BMI Calculator - Building Flutter UI for Intermediates

機能自体はシンプルですがこのセクション以後レイアウトに関するコード量が増えます。Compositional layoutでWidgetを作って切り出して再利用してUIを作ります。これに関しては`Composition vs. Inheritance - Building Flutter Widgets From Scratch`というチャプターで30分程かけて解説されます。Flutterで大切になる概念についてはチャプターを区切って多めの時間を割いて解説されます。


#### このセクションで学ぶこと

- Compositional layout
- Theme
- Navigationによる画面遷移

### Clima - Powering Your Flutter App with Live Web Data

公開されているAPIを介した外部のデータを利用してアプリを作るセクションです。その際必要になる知識(Future, Async, Awaitなど)についてもチャプターを最低説明されます。

#### このセクションで学ぶこと

- Async/Await, Future
- Dartでのエラーハンドリング、Nullの扱い

### Flash Chat - Flutter x Firebase Cloud Firestore

このセクションではFirebaseを使ってチャットアプリを実装します。Firebaseを扱う部分の実装ではStreamなどについても学びます。このセクションではそれよりもアニメーションについての解説が面白かったです。

基本的なアニメーションは他のWidgetと同じ様にアニメーションさせるためのWidgetが提供されていてwrapすることで簡単に利用できます。それとは別にAnimation Controllerを使ったカスタムアニメーションについても学びます。

#### このセクションで学ぶこと

- Staticな関数、変数及び定数
- FlutterでのRouting
- アニメーション
- Mixin
- Stream
- Firebase SDKを使ったFlutterアプリケーション開発の基本

#### 注意点

Firebase の環境構築が講座が古く、ドキュメントと異なりました。通常はStub Projectから講座に合わせて進めていくのですが、この章以後は新規プロジェクトを作って古い所をドキュメントを参照しながら自分で設定してその後Stubプロジェクトと同じ状態までコードを書いて講座を進めるといった形にしました。

### Flutter State Management

このセクションを見るために買った所があるので楽しみにしていました。StatelessWidgetとStatefulWidgetを使ってレイアウトを組んでStateの管理をsetStateとクロージャの引き回しで行いながら実装していきます。

一旦そこまで進めた後、`Listing State Up`と講座では説明されていますが上位のウィジェットでStateを管理するようにリファクタリングしていきます。

その後Providerの概要説明、Providerの基本的な使い方を説明しながらState管理をPrividerを使った書き方に変更していきます。

題材になっているアプリケーションはTODOアプリです。最後に削除などいくつかTODOリストの必須ではあるものの未実装だった機能をProviderを使って実装して終了です。

#### このセクションで学ぶこと

- Providerを使った状態管理
- ChangeNotifier


合計30時間の講座ですが、紹介したセクションの合間にユーザーが自ら学んだことを使ってアプリを作るセクションがあります。なのでそのセクションようにコードを書いた時間を含めるともっと時間がかかったように思います。基礎的な所から解説するので説明が理解できずに自分でググったりするようなことはなかったです。

## まとめ

公式のチュートリアルはやっていたので最低限の書き方はわかっていたのですが、アプリ開発を進めるにあたってiOSではこういう風にやっているけどFlutterではどうするんだろう、というような疑問がこの講座でほいくつか解決されました。紹介した講座は受けている時間には及ばないものの自分で考えてコードを書かないといけないように時間が随所に設けられていたのでそこが一番体験が良かったです。
