# Sliver系の主なWidgetをまとめてみる



SliverとつくWidget群を使うことでスクロールをカスタマイズすることができます。正直に自白しますがSliverについて自発的に調べるまで`Silver`と読んでいたのですが、ドキュメントを読むと参考リンクと動画へのリンクを参照するように案内されているのみです。

動画によると基本的にFlutterアプリ開発者がSliverを使用するべき場合はそれほどないらしいですが、実際に調べるときにそれなりに苦労したので整理した上でサンプルのコードを紹介したいと思っています。

参照するように案内されているのは以下の記事です。

- [Slivers, Demystified. Or, how to do fancy scrolling… | by Emily Fortuna | Flutter | Medium](https://medium.com/flutter/slivers-demystified-6ff68ab0296f)
- https://www.youtube.com/watch?v=Mz3kHQxBjGg

Flutter Widget of the WeekではSliverはスクロール可能領域の一部を表し細長い小片`と訳がついています。低レベルなインタフェースで実行中のスクロール可能領域上きめ細かいコントロールを提供します。

- 標準外の動作（スクロールすると消える、スクロールするとサイズや色が変わるなど）のアプリバーが欲しい。
- アイテムのリストとアイテムのグリッドを一つのユニットとして`効率よく`スクロールする必要がある。
- ヘッダ付きのリストを折りたたむようなことをする

Sliver系のWidget

### SliverList

ListViewに対応するSliverList は、リスト内のアイテムをビューにスクロールする際に提供するデリゲート・パラメータを取ります。SliverChildListDelegate を使用して実際の子リストを指定するか、または SliverChildBuilderDelegate を使用してそれらを`lazy`に構築することができます。

ドキュメントは以下です。

- [SliverList class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/SliverList-class.html)

Flutter Widget of the WeekではSliverGridとあわせて紹介されています。

https://www.youtube.com/watch?v=ORiTTaVY6mM

ドキュメントの表現を拙訳すると複数のボックスの子をmain axisに沿って線形に配置するSliverです。

スライバーの可視部分の外側にある子はマテリアライズされないため、SliverList はそのスクロール オフセットを "デッド レコニング" で決定します。その代わり、新たにマテリアライズされた子は既存の子に隣接して配置されます。

リストをレイアウトしている間、子要素、state、およびRenderObjectは、既存のウィジェット（SliverChildListDelegate の場合など）または遅延的に提供されたウィジェット（SliverChildBuilderDelegate の場合など）に基づいて遅延的に作成されます。

SliverListのchildが表示領域の外にスクロールされると、関連する要素のサブツリー、state、およびRenderObjectが破棄されます。Sliver内の同じ位置にある新しいchildは、スクロールして戻ると、新しい要素、状態、およびレンダー オブジェクトと共に、lazyに再作成されます。

SliverListはコンストラクタ引数delegateを受け取ります。リストのレイアウトをlazyに作成するためdelegateプロパティにSliverChildListDelegateを使って実際のchildrenのリストを提供するか、SliverChildBuilderDelegateを使って子リストを返す関数を定義するかのいずれかをdelegateに渡します。

この部分は動画では`SliverChildListDelegate takes an explicit list`、`SliverChildBuilderDelegate builds children on-demand`とそれぞれ表現しています。

SliverChildListDelegateを使ったサンプルが以下です。

<p class="codepen" data-height="579" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="bGExNVX" style="height: 579px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="SliverListUseSliverChildListDelegateSample">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/bGExNVX">
  SliverListUseSliverChildListDelegateSample</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

SliverChildBuilderDelegateを使ったサンプルが以下です。

<p class="codepen" data-height="527" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="XWXPJWp" style="height: 527px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="SliverListUseSlvierChildBuilderDelegateSample">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/XWXPJWp">
  SliverListUseSlvierChildBuilderDelegateSample</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### SliverFixedExtentList

SliverListのドキュメントにmain axisへの固定長のリストを表示したい場合にはこちらを使うように説明がされています。SliverFixedExtentList は、その子に対してレイアウトを実行して主軸内のエクステントを取得する必要がない分SliverListに比べて効率的です。

SliverFixedExtentListは、その子をオフセットゼロから始まり、ギャップなしで主軸に沿った線形配列に配置しますchildは、main axisに itemExtent、cross axisに SliverConstraints.crossAxisExtent を強制的に持ちます。

以下サンプルコードです。

<p class="codepen" data-height="525" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="VweGwqQ" style="height: 525px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="SliverFixedExtentList">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/VweGwqQ">
  SliverFixedExtentList</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### SliverPrototypeExtentList

SilverPrototypeExtentListはどのようなシチュエーションで使うのか、ドキュメントからはぼんやりとしか読み取れませんでしたがソースコードやテストコード、加えて下部に列挙しているリンク先などから把握することができました。

SliverPrototypeExtentListは、その子を隙間なくmain axisに沿って一直線に配置します。子のwidget群は、main axisに沿ったprototypeItemと同じ高さ、幅に拘束され、またSliverConstraints.crossAxisExtentと同じ範囲に拘束されます。

SliverPrototypeExtentListは、main axisに沿ったextentを取得するために子をレイアウトする必要がないので、SliverListよりも効率的です。そして適切なアイテムのエクステントをピクセル単位で決定する必要がないのでSliverFixedExtentListよりもほんの少し柔軟性があります。

- [SliverPrototypeExtentList | Flutter | 老孟](http://laomengit.com/flutter/widgets/SliverPrototypeExtentList.html)
- [flutter/flutter - Gitter](https://gitter.im/flutter/flutter/archives/2018/01/16?at=5a5dc3afba39a53f1a1f0a45)

コンストラクタ引数prototypeItem は、リスト項目の寸法を表すダミーのアイテムです。SliverPrototypeExtentListでは、このダミーアイテムの幅と高さをすべてのリスト項目の幅と高さとして使用します。SliverFixedExtentListと比べて、寸法やパディングなどを計算したりする必要がないのがメリットです。SliverListとSliverFixedExtentListの中間の自由度を持つWidgetと個人的には理解しました。

ソースコードやドキュメント、そして手元で書いたコードからそうだろうと判断したに過ぎないのでおかしな点に気づかれたらご指摘頂けるとありがたいです。

### SliverGrid

基本的にSliverListと使い方は同じ、ただSiverGridは水平方向の追加フォーマットが提供されています。SliverでGridViewのレイアウトを使用したい時に使用します。

- cross axisに何個、この場合は何個のアイテムがあるかをカウントするcountコンストラクタ、
- extent コンストラクタを使用して、アイテムの最大幅を指定してグリッドに収まる数を決定します。これは、グリッド内のアイテムのサイズが異なる場合に最も便利です。(`SliverGrid.extent(children: scrollItems, maxCrossAxisExtent: 90.0) // 90論理ピクセル`)
- 明示的な gridDelegate パラメータを渡すデフォルトのコンストラクタ。

複数のbox childrenを二次元に配置するSliverと紹介されています。SliverGridの



## SliverAppBar

```dart
CustomScrollView(
    slivers: <Widget>[
      SliverAppBar(
        title: Text('SliverAppBar'),
        backgroundColor: Colors.green,
        expandedHeight: 200.0,
        flexibleSpace: FlexibleSpaceBar(
          background: Image.asset('assets/forest.jpg', fit: BoxFit.cover),
        ),
      ),
      SliverFixedExtentList(
        itemExtent: 150.0,
        delegate: SliverChildListDelegate(
          [
            Container(color: Colors.red),
            Container(color: Colors.purple),
            Container(color: Colors.green),
            Container(color: Colors.orange),
            Container(color: Colors.yellow),
            Container(color: Colors.pink),
          ],
        ),
      ),
    ],
);
```

SliverAppBarに追加できるカスタマイズがいくつかあります。フローティングパラメータをtrueに設定すると、下にスクロールしたときに、リストのトップに到達していなくてもアプリバーが再表示されるようになります。スナップパラメータとフローティングパラメータの両方を追加すると、下にスクロールしたときにアプリバーを完全にスナップバックさせて表示させることができます。





l