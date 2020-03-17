Flutter は動画コンテンツも充実していて週毎にウィジェットを紹介している Flutter Widget of the Week では短い動画でウィジェットの概要を掴むことができる動画を視聴できます。

- [Flutter Widget of the Week - YouTube](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/02/maxresdefault.jpg" alt="" width="1280" height="720" class="alignnone size-full wp-image-532693" />

Flutter Widget of the Week を見るだけでも勉強になるのですが、記憶力が良くないので、視聴によるインプットだけでなく手元でコードを動かしてそれをブログ記事にする所までやって後から思い出しやすいようにしてみるシリーズです。

これまでの記事は以下のリンクから参照できます。

- [Flutter の Widget をコードを動かしながら学ぶ ｜ シリーズ ｜ Developers.IO](https://dev.classmethod.jp/series/flutter%e3%81%aewidget%e3%82%92%e3%82%b3%e3%83%bc%e3%83%89%e3%82%92%e5%8b%95%e3%81%8b%e3%81%97%e3%81%aa%e3%81%8c%e3%82%89%e5%ad%a6%e3%81%b6/)

今回取り上げるのは Wrap です。

## Wrap とは

Flutter Widget of the Week で Wrap をとりあげた動画は以下です。Expanded のドキュメントを見ると、複数の行または列でその子のウィジェットを表示するためのウィジェットと記載されています。

https://www.youtube.com/watch?v=z5iw2SeFx2M

## Expanded を使ってみる

[ドキュメントに記載されているサンプルコード](https://api.flutter.dev/flutter/widgets/Wrap-class.html#widgets.Wrap.1)を実行すると以下のような表示になります。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/ee8f62f23fffbdf5ceb5b8da7e7a4aef.png" alt="" width="862" height="1836" class="alignnone size-full wp-image-543826" />

ラップは row や column のように小ウィジェットを direction に指定された軸で各々を隣接して配置します。spacing に指定された分スペースを空けます。direction に指定された方向に子ウィジェットをレイアウトしていきながら子を配置するための十分なスペースがない場合、新たに行・列を作成してレイアウトします。

コードを弄りつつ説明するためサンプルコードを用意します。margin に 10.0 をもつ Container を複数 children に持つ Row ウィジェットを用意しました。Row は x 軸方向に子ウィジェットを配置し、配置された子ウィジェットは画面内に収まっているとします。

コピペでそのまま動くサンプルは以下です。

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        title: 'Flutter Demo',
        theme: ThemeData(
          primarySwatch: Colors.blue,
        ),
        home: Scaffold(
          appBar: AppBar(
            title: Text('Wrapのサンプルコード'),
          ),
          body: body(),
        ));
  }
}

Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Row(
      children: <Widget>[
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
      ],
    ),
  );
}

Container marginedColorContainer(
    {EdgeInsets margin = const EdgeInsets.all(10.0),
    Color color = Colors.yellow}) {
  return Container(
    margin: margin,
    width: 100.0,
    height: 100.0,
    color: color,
  );
}
```

実行結果は以下のようになります。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/3c117a228b4d58fbc86046977cda50a8.png" alt="" width="848" height="1826" class="alignnone size-full wp-image-543831" />

Rows の children にもう一つ Container を追加してみます。画面幅に収まらない想定をしています。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/569f5a8bb88204ff127287911e7a9c7c.png" alt="" width="864" height="1818" class="alignnone size-full wp-image-543832" />

実行してみると収まらない部分に関する警告が表示されているのが確認できます。

`A RenderFlex overflowed by 88 pixels on the right.`と表示されています。

```
════════ Exception caught by rendering library ═════════════════════════════════════════════════════
The following assertion was thrown during layout:
A RenderFlex overflowed by 88 pixels on the right.

The relevant error-causing widget was:
  Row ../wrap_playgrounds/lib/main.dart:27:12
The overflowing RenderFlex has an orientation of Axis.horizontal.
The edge of the RenderFlex that is overflowing has been marked in the rendering with a yellow and black striped pattern. This is usually caused by the contents being too big for the RenderFlex.

Consider applying a flex factor (e.g. using an Expanded widget) to force the children of the RenderFlex to fit within the available space instead of being sized to their natural size.
This is considered an error condition because it indicates that there is content that cannot be seen. If the content is legitimately bigger than the available space, consider clipping it with a ClipRect widget before putting it in the flex, or using a scrollable container rather than a Flex, like a ListView.

The specific RenderFlex in question is: RenderFlex#9ff89 relayoutBoundary=up3 OVERFLOWING
...  parentData: offset=Offset(20.0, 20.0) (can use size)
...  constraints: BoxConstraints(0.0<=w<=392.0, 0.0<=h<=696.0)
...  size: Size(392.0, 120.0)
...  direction: horizontal
...  mainAxisAlignment: start
...  mainAxisSize: max
...  crossAxisAlignment: center
...  textDirection: ltr
...  verticalDirection: down
```

エラーメッセージを読むと Axis.horizontal な方向にオーバーフローしていて、その原因は RenderFlex に対して大きすぎるコンテンツが原因と記載されています。RenderFlex のエッジは黄色と黒の縞模様でマーキングされている、と記載されていますが先程の実行結果を見るとたしかにエッジが確認できます。これに内部のコンテンツが収まらなかったためにエラーを表示しているということでしょう。

そこで Wrap を使用します。オーバーフローしている方向は Axis.horizontal なので、Wrap のコンストラクタ引数 direction には Axis.horizontal を指定したいところですがソースを読むとデフォルト値が Axis.horizontal なので今回は指定する必要はないです。

```
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      direction: Axis.horizontal,
      children: <Widget>[
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(), // 追加した
      ],
    ),
  );
}

```

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/3fc5e5733d867497d93a89cec51c947b.png" alt="" width="878" height="1814" class="alignnone size-full wp-image-543834" />

収まりきらない分は次の行・列に折り返してもう一つ行・列を作ってコンテンツを配置します。これにより先程の警告は表示されなくなりました。

## Wrap のその他主要なプロパティ

ソースを見ずとも Android Studio で Quick Documentation を使うとコンストラクタとデフォルト値が確認できると知りました。

```
(new) Wrap Wrap({Key key, Axis direction = Axis.horizontal, WrapAlignment alignment = WrapAlignment.start, double spacing = 0.0, WrapAlignment runAlignment = WrapAlignment.start, double runSpacing = 0.0, WrapCrossAlignment crossAxisAlignment = WrapCrossAlignment.start, TextDirection textDirection, VerticalDirection verticalDirection = VerticalDirection.down, List<Widget> children = const <Widget>[]})
```

## direction

direction の型は Axis です。Axis は列挙型として宣言されていて上下を表す vertical、左右を表す horizontal が用意されています。最初のサンプルではデフォルト値の Axis.horizontal になるので横方向ですが Axis.vertical を指定してみます。

```swift
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      direction: Axis.vertical, // 追加した。directionを指定
      children: <Widget>[
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
      ],
    ),
  );
}
```

実行結果は以下です。要素が縦方向に配置されています。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/a63eb1b85d9b5885ede1d14a84c584a6.png" alt="" width="856" height="1838" class="alignnone size-full wp-image-543836" />

### space

space の型は double 型ですが、現状のコードだと最初からマージンが設定されているのでコードを修正します。marginedColorContainer メソッドは Optional Default Parameters Function なのでデフォルト値を持つ引数によりコンストラクタ引数に値をセットする必要がありません。

今回は明示的に EdgeInsets.all(0)をセットしてマージンをリセットします。

```
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      direction: Axis.vertical,
      children: <Widget>[
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
      ],
    ),
  );
}
```

表示は以下のようになります。マージンゼロなのでつながっているように見えます。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/c39bfa7cd1b52f3ed28bbdd81e0c5bec.png" alt="" width="880" height="1832" class="alignnone size-full wp-image-543837" />

上記のコードの Wrap に spacing を指定してみます。すると要素とその次の要素の間にスペースができるのが確認できます。

```
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      direction: Axis.vertical,
      spacing: 10.0, // 追加した
      children: <Widget>[
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
        marginedColorContainer(margin: const EdgeInsets.all(0)),
      ],
    ),
  );
}
```

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/3ba1775ee5bf751281726312145ed605.png" alt="" width="830" height="1830" class="alignnone size-full wp-image-543838" />

### runSpace

runSpace は Wrap の行・列ごとの間のスペースを指定するためのプロパティです。

runSpace を指定しない例として以下のコードを記述します。

```
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      children: <Widget>[
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
      ],
    ),
  );
}
```

実行結果は以下です

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/5e78d486daf4ccd56b7cb3819516975f.png" alt="" width="858" height="1854" class="alignnone size-full wp-image-543840" />

次は runSpace を指定してみます。

```
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      runSpacing: 30.0, // 追加した
      children: <Widget>[
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
      ],
    ),
  );
}
```

実行結果は以下になります。行と行の間に先程より広いスペースが与えられていることが確認できます。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/d7b92f763bfe3d25f7d254fe2ee8e6a8.png" alt="" width="870" height="1832" class="alignnone size-full wp-image-543841" />

### alignment

メイン軸の要素をどのように配置するか指定します。型は WrapAlignment でデフォルト値は start です。

```
Widget body() {
  return Container(
    color: Colors.lightBlue,
    padding: EdgeInsets.all(20.0),
    child: Wrap(
      alignment: WrapAlignment.start, // ここを変更していく
      children: <Widget>[
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(),
        marginedColorContainer(), // 追加した
      ],
    ),
  );
}
```

#### start

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/e4476dc4aee173660b6374ce043564ce.png" alt="" width="716" height="456" class="alignnone size-full wp-image-543842" />

#### end

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/cbd1c7260419c891d9f3b99336ba2dd0.png" alt="" width="702" height="438" class="alignnone size-full wp-image-543843" />

#### center

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/c7b2bcf1164ef5f7392977ff460b1631.png" alt="" width="696" height="440" class="alignnone size-full wp-image-543844" />

#### spaceAround

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/567d9fd1a7f573fb88b6db85d37dea4f.png" alt="" width="702" height="440" class="alignnone size-full wp-image-543846" />

#### spaceBetween

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/17a7d3dc4267f1cd50ab392175c50ebd.png" alt="" width="698" height="434" class="alignnone size-full wp-image-543848" />

#### spaceEvenly

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/9978f4e31daceaac713bc0149dcbf6d8.png" alt="" width="684" height="428" class="alignnone size-full wp-image-543850" />

## まとめ

Wrap 頻繁に使うので改めて動画、ドキュメントをみつつコードを手元で動かしながら記事がかけてよかったです。実は Wrap の挙動でまだ腑に落ちてないプロパティがあって、それは長くなりそうなので別途記事にしようと思います。記事内で何か誤りや説明不足な点があればご指摘いただけるとありがたいです。
