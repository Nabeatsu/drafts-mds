# CocoaPodsをWorkspaceに自動統合せずに使う

今年のiOSDCに行った時に[ライブラリのインポートとリンクの仕組み完全解説 by Kishikawa Katsumi | トーク | iOSDC Japan 2019 #iosdc - fortee.jp](https://fortee.jp/iosdc-japan-2019/proposal/28d1013f-a57b-4d42-b486-a3372c459459)というセッションを聴かせていただきました。

このセッションを通じてSwiftではライブラリの形式からインポートのリンクの違い、モジュールなどについて詳しく解説していただいたのですが、具体的に解決したい問題があって再度このセッションのスライドや動画を見直したりしていました。

セッションの後半で応用編としてCocoaPodsをワークスペースに統合せずに使用するというのが紹介されていたのでそれを実際にやってみました。

- https://speakerdeck.com/kishikawakatsumi/swiftniokeruinpototorinkufalseshi-zu-miwotan-ru?slide=134

メリットの一つにワークスペースにCocoaPodsが管理するプロジェクトとして導入されるためにクリーンビルドに時間がかかってしまう問題が解決できるというのがあります。クリーンビルドじにライブラリの再ビルドを行なわなくて済むからです。

冒頭の具体的に解決したい問題というのがあるプロジェクトの非常に長いビルド時間で、いつもはビルド時間短縮の方法の一つとしてCarthageへの移行などを行っていたのですが、ワークスペースに統合せずに使用したことがなかったので手元でやってみたかったというのが今回の記事を書こうと思った理由です。

といってもこの発表をされた[kishikawa katsumi（@k_katsumi）](https://twitter.com/k_katsumi/likes)さんがすでに以下の記事で具体的な方法を書かれているのと、セッションや紹介されていたリンクを見てうっすら理解して（いるつもりになって）いたので基本的なやり方にこまることはありませんでした。

- [CocoaPodsをWorkspaceに自動統合せずに利用する - 24/7 twenty-four seven](https://blog.kishikawakatsumi.com/entry/2019/06/17/090724)

## 環境

- ruby 2.6.3
- Bundler version 2.0.2
- CocoaPods 1.8.4
- Version 11.2.1

## 準備

CocoaPodsを使える状態にします。
```sh
$ rbenv local 2.6.3
$ rbenv local # .ruby-versionの生成
$ bundle init # なければgem install
$ bundle install --path vendor/bundle # gem 'cocoapods'
$ bundle exec pod init
```

いくつかPodfileにライブラリを追加して通常通り`pod install`します。その後生成される.xcworkspaceを開いてビルドするとimportするだけでライブラリがプロジェクト内で使用できるようになります。コミットは以下です。.xcodeproje以下のdiffを見るとCocoaPodsが自動でやってくれていることが少し垣間見えて面白いです。

[Add libraries · Nabeatsu/use-cocoapods-without-autointegration@e0fc3b7](https://github.com/Nabeatsu/use-cocoapods-without-autointegration/commit/e0fc3b7f2f0539bd101e65d0a4bfa8c605196d1d)

### Podfileの編集
integrate_targetsをfalseにしてpod installするとGenerating Pods projectの後に`Skipping User Project Integration`と表示されました。

```sh
Generating Pods project
Skipping User Project Integration
Pod installation complete! There are 18 dependencies from the Podfile and 37 total pods installed.
```

![https://gyazo.com/453e8b8bb45e9f0b93958c0cc4bec273](https://gyazo.com/453e8b8bb45e9f0b93958c0cc4bec273.png)

### 自動で生成されていたCocoaPods周りのbuild Phaseの削除
- [CP] Copy Pods Resources
- [CP] Embed Pods Frameworks

### 変数定義
PODS_ROOT = $(SRCROOT)/Pods
PODS_CONFIGURATION_BUILD_DIR = $(PODS_ROOT)/Build/Release$(EFFECTIVE_PLATFORM_NAME)

ここまででBuild Succeed

ここからは手作業リンク



