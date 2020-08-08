Flutter は動画コンテンツも充実していて週毎にウィジェットを紹介している Flutter Widget of the Week では短い動画でウィジェットの概要を掴むことができる動画を視聴できます。

- [Flutter Widget of the Week - YouTube](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/02/maxresdefault.jpg" alt="" width="1280" height="720" class="alignnone size-full wp-image-532693" />

Flutter Widget of the Week を見るだけでも勉強になるのですが、記憶力が良くないので、視聴によるインプットだけでなく手元でコードを動かしてそれをブログ記事にする所までやって後から思い出しやすいようにしてみるシリーズです。

これまでの記事は以下のリンクから参照できます。

- [Flutter の Widget をコードを動かしながら学ぶ ｜ シリーズ ｜ Developers.IO](https://dev.classmethod.jp/series/flutter%e3%81%aewidget%e3%82%92%e3%82%b3%e3%83%bc%e3%83%89%e3%82%92%e5%8b%95%e3%81%8b%e3%81%97%e3%81%aa%e3%81%8c%e3%82%89%e5%ad%a6%e3%81%b6/)

今回取り上げるのは SliverAppBar です。

## Sliverとは

SliverとつくWidget群を使うことでスクロールをカスタマイズすることができます。Sliverについて自発的に調べるまで`Silver`と読んでいました……。ドキュメントを読むと参考リンクと動画へのリンクを参照するように案内されているのみで最初は扱いづらそうに感じていました。

- [Slivers - Flutter](https://flutter.dev/docs/development/ui/advanced/slivers)

ドキュメントで参照するよう案内されているのは以下の記事と動画です。

- [Slivers, Demystified. Or, how to do fancy scrolling… | by Emily Fortuna | Flutter | Medium](https://medium.com/flutter/slivers-demystified-6ff68ab0296f)
- https://www.youtube.com/watch?v=Mz3kHQxBjGg

Flutter Widget of the WeekではSliverはスクロール可能領域の一部を表す細長い小片`と訳がついています。低レベルなインタフェースで実行中のスクロール可能領域上きめ細かいコントロールを提供します。

- 標準外の動作（スクロールすると消える、スクロールするとサイズや色が変わるなど）のアプリバーが欲しい。
- アイテムのリストとアイテムのグリッドを一つのユニットとして`効率よく`スクロールする必要がある。
- ヘッダ付きのリストを折りたたむようなことをする

Sliver系のWidgetを使うのはこのような時で基本的なレイアウトを作る時にはSliverを使う必要はないと説明されています。(動画でも`It's not the simplest of our APIs.On the other hand, many people don't really need to know Slivers`、`this is advanced class`と冒頭でいきなり話しています。)

### ## SliverAppBarについて

SliverAppBarのドキュメントは[こちら](https://api.flutter.dev/flutter/material/SliverAppBar-class.html)です。

スクロールすると変化したり消えたりするアニメーションをAppBarで行いたい場合にSliverAppBarを使うと便利です。SliverAppBarは通常ScaffoldのappBarパラメータには使用されず、CustomScrollViewとともに使われます。 

<p class="codepen" data-height="568" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="WNrqQOB" style="height: 568px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="SliverAppBarSample">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/WNrqQOB">
  SliverAppBarSample</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

codepenのサンプルコードです。SliverAppBarのスクロール効果を説明するためにSliverFixedExtentListというウィジェットを使っています。ListViewのSliver版であるSliverListを少し効率的にしたウィジェットです。main axisのextentを取得するために子をレイアウトをする必要がない分コストが少なく済みます。

### floating、pinned、snap

floatingがユーザーがアプリバーに向かってスクロールするとすぐにアプリバーが表示されるようにするかどうかを表す真偽値です。pinnedがスクロールビューの開始時にアプリバーを表示したままにするかどうかを表す真偽値です。snapはfloatingがtrueである場合に有効で、trueの場合ヘッダがスクロール時に一気に最大表示されます。

この3つの値の組み合わせでスクロール時のアニメーションの効果が変わりますが、コードより、ドキュメントの動画比較がわかりやすいです。[SliverAppBarのAnimated Examples](https://api.flutter.dev/flutter/material/SliverAppBar-class.html)これら3つのパラメータの組み合わせごとの動画が列強されています。

### leading、title、actions

leading、titleにWidget、actionsに`List<Widget>`、bottomにPreferredSizeのサブクラスを指定することでそれぞれの領域にWidgetを設置できます。このサンプルコードは以下です。

<p class="codepen" data-height="625" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="LYGKZKm" style="height: 625px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="SliverAppBarSample2">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/LYGKZKm">
  SliverAppBarSample2</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### bottomパラメータについて

bottomパラメータの型はPreferredSizeです。PreferredSizeのドキュメントは[こちら](https://api.flutter.dev/flutter/widgets/PreferredSizeWidget-class.html)です。AppBarの下に表示されるウィジェットで、ドキュメントによるとTabBarを想定されているものです。そのため、アプリバーの下部には動的にサイズが変更できないPreferredSizeWidget を実装したウィジェットのみが使用できます。

PreferredSizeウィジェットのサイズに関する説明はStack OverflowのProviderの作者の方の[回答](https://stackoverflow.com/a/49268103)が参考になりました。

### flexibleSpace、expandedHeight

flexibleSpaceパラメータに設定されたウィジェットはツールバーとタブバーの後ろにスタックされます。スクロールで最大限広がった時の高さはexpandedHeightで指定します。渡されたウィジェットの高さが全体の高さと等しくなります。動いているものを見る方がわかりやすいのでcodepen を以下に載せました。

<p class="codepen" data-height="616" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="dyGBjbN" style="height: 616px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="SliverAppBarSample4">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/dyGBjbN">
  SliverAppBarSample4</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## まとめ

SliverAppBarについて知る時にSliver自体に関してと、Sliverとつくその他のWidgetも一通り触ってから記事を書き始めたのでテーマの割に時間がかかってしまいました。何か誤りや認識のずれに気づかれた場合はコメントや[Twitter](https://twitter.com/t__nabe1478)にてご指摘頂けると幸いです。

