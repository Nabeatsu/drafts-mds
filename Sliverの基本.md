# Sliverの基本

SliverとつくWidget群を使うことでスクロールをカスタマイズすることができます。正直に自白しますがSliverについて自発的に調べるまで`Silver`と読んでいたのですが、ドキュメントを読むと参考リンクと動画へのリンクを参照するように案内されているのみです。

- [Slivers - Flutter](https://flutter.dev/docs/development/ui/advanced/slivers)

動画によると基本的にFlutterアプリ開発者がSliverを使用するべき場合はそれほどないらしいですが、実際に調べるときにそれなりに苦労したので整理した上でサンプルのコードを紹介したいと思っています。

その上で別途込み入ったレイアウトを作る記事も書いていきたいと思います。

ドキュメントで参照するよう案内されているのは以下の記事と動画です。

- [Slivers, Demystified. Or, how to do fancy scrolling… | by Emily Fortuna | Flutter | Medium](https://medium.com/flutter/slivers-demystified-6ff68ab0296f)
- https://www.youtube.com/watch?v=Mz3kHQxBjGg

Flutter Widget of the WeekではSliverはスクロール可能領域の一部を表し細長い小片`と訳がついています。低レベルなインタフェースで実行中のスクロール可能領域上きめ細かいコントロールを提供します。

- 標準外の動作（スクロールすると消える、スクロールするとサイズや色が変わるなど）のアプリバーが欲しい。
- アイテムのリストとアイテムのグリッドを一つのユニットとして`効率よく`スクロールする必要がある。
- ヘッダ付きのリストを折りたたむようなことをする

Sliver系のWidgetを使うのはこのような時で基本的なレイアウトを作る時にはSliverを使う必要はないと説明されています。(動画でも`It's not the simplest of our APIs.On the other hand, many people don't really need to know Slivers`、`this is advanced class`と冒頭でいきなり話しています。)

Slivers Explained - Making Dynamic 試聴メモ

Sliverについて知る前にFlutterが基本的にどのようにレイアウトを構築するのか知る必要がある。Render Object、特に広く私達が目にするrender box(Container, SizedBoxなど)は自身が高さや幅を持ち、最大幅・高さ、最小幅・高さを持ちその制約の中で子のWidgetをレイアウトしていく。

基本的にそれらはうまくやってくれるが、スクロールが入るとそれがうまく機能しない場合がある。app barのスクロール時の動きやその他変わったスクロール効果を実現したい時にそれは顕著。

そのような場合を想定してSliver protocolというプロトコルを提供している。

`as in a slice of something`、`a slice of something that scrolls`

スライバーを使用している先進的なものを試してみたいと思ったのですが、それがSliverAppBarです。でも、はっきり言ってしまえば、ListViewでさえもSliverを使っているんですよね。フラッターのスクロールのほとんどは スライバを使っている でも一つだけ例外があります。ビューポートか何だっけ？あれは一つのアイテムをスクロールして移動できるビューポートです。

それだけが例外ではあるが、殆どの場合それを使うことはない。lazyではないから。

  But anytime you have a list, where  you don't know how many things are in the list you use ListView, you use GridView, CustomScrollView, all of those that you think Sliver's under the hood.

でもいつでもリストがあって、リストにどれだけのものがあるかわからないときはListViewを使い、GridViewを使い、CustomScrollViewを使い、スライバーはフードの下にあると思っているものは全部使っています。



### Sliver protocolにおいて