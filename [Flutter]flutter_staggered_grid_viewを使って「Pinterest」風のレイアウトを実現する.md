# [Flutter]flutter_staggered_grid_viewを使って「Pinterest」風のレイアウトを実現する



## 環境について

pubspec.yamlの設定です。

```yaml
environment:
  sdk: ">=2.7.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter

  cupertino_icons: ^0.1.3
  flutter_staggered_grid_view: ^0.3.1

```



## flutter_staggerred_grid_view

cross axisを等倍の大きさの複数に分割して空いているmain axisのスペースを基準に最適な位置に要素を配置していくグリッドレイアウトを実装することができるWidgetを提供しているパッケージです。

- [flutter_staggered_grid_view | Flutter Package](https://pub.dev/packages/flutter_staggered_grid_view)

### インストール方法

他のパッケージと同じく`pubspec.yaml`ファイルに追記して`flutter pub get`コマンドを実行することで導入できます。アプリケーションコードでimportすることでパッケージ内でexportされているウィジェットを使用できます。

```dart
import 'package:flutter_staggered_grid_view/flutter_staggered_grid_view.dart';
```

アップデートによりインストール方法などが変わるかもしれないので実際に使用される際には[Installing](https://pub.dev/packages/flutter_staggered_grid_view/install)を確認することをすすめます。

### 主要なクラス

#### StaggeredGridView

スクロール可能な、可変サイズのウィジェット群を表示するクラスです。main axisは、スクロールする方向（scrollDirection）です。決まった数のtileをcross axisにレイアウトするためにStaggeredGridView.countがよく使われます。今回使用するのはこちらのコンストラクタです。また、 最大のcross axisへのextentを持つタイルでレイアウトするためにStaggeredGridView.が使われます。metro grid styleな画面を実装するならこちらを使うのだと思います。

#### StaggeredTile

StaggeredGridViewのタイルの寸法を保持するクラスです。

StaggeredTileもユースケースに合わせた3つのコンストラクタを提供しています。StaggeredTile.countコンストラクタは`crossAxisCellCount`列、`mainAxisCellCount`行のタイルを作成することを表します。静的にタイルのレイアウトを決めたい時はこちらを使うのだと思います。

2つ目のコンストラクタがStaggeredTile.extentコンストラクタです。こちらはドキュメントにある通りmain axisへの固定された範囲を保持したレイアウトを保持します。

3つ目のコンストラクタがStaggeredTIle.fitコンストラクタです。 `crossAxisCellCount`を指定する必要があります。mainAxisへの範囲を内容にあわせて表示します。今回のPinterest風のレイアウトを実現するならこのコンストラクタを使うのが良さそうです。

#### SliverStaggeredGrid

StaggeredGridViewをSliver系のウィジェットと組み合わせて使えるようにするために提供されているクラスです。Sliver系ウィジェットは併用する時にCustomScrollView.sliversにListで渡すのでその時に使用できます。

### StaggeredGridView.countの基本的な使い方　

登場する基本的なクラスについて説明したので簡易的な実装をしてみます。

```dart
import 'package:flutter/material.dart';
import 'package:flutter_staggered_grid_view/flutter_staggered_grid_view.dart';

void main() => runApp(MaterialApp(
      home: StaggeredExample(),
      theme: ThemeData(
        primaryColor: Colors.black,
      ),
    ));

class StaggeredExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('StaggeredExample'),
      ),
      body: Padding(
        padding: EdgeInsets.only(top: 12.0),
        child: StaggeredGridView.count(
          crossAxisCount: 4,
          staggeredTiles: const <StaggeredTile>[
            const StaggeredTile.count(3, 2),
            const StaggeredTile.count(1, 1),
            const StaggeredTile.count(1, 1),
            const StaggeredTile.count(2, 2),
            const StaggeredTile.count(2, 1),
            const StaggeredTile.count(1, 2),
            const StaggeredTile.count(1, 1),
            const StaggeredTile.count(2, 2),
            const StaggeredTile.count(1, 2),
            const StaggeredTile.count(1, 1),
            const StaggeredTile.count(3, 1),
            const StaggeredTile.count(1, 1),
            const StaggeredTile.count(4, 1),
          ],
          children: [
            const _SampleTile(Colors.green),
            const _SampleTile(Colors.lightBlue),
            const _SampleTile(Colors.amber),
            const _SampleTile(Colors.brown),
            const _SampleTile(Colors.deepOrange),
            const _SampleTile(Colors.indigo),
            const _SampleTile(Colors.red),
            const _SampleTile(Colors.pink),
            const _SampleTile(Colors.purple),
            const _SampleTile(Colors.blue),
            const _SampleTile(Colors.black),
            const _SampleTile(Colors.red),
            const _SampleTile(Colors.brown),
          ],
        ),
      ),
    );
  }
}

class _SampleTile extends StatelessWidget {
  const _SampleTile(this.backgroundColor);

  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return new Card(
      color: backgroundColor,
      child: new InkWell(
        onTap: () {},
        child: new Center(
          child: new Padding(
            padding: const EdgeInsets.all(4.0),
          ),
        ),
      ),
    );
  }
}
```

![2020-08-03 4.55.21](/Users/tanabe.nobuyuki/Documents/drafts/images/2020-08-03 4.55.21.png)

静的に寸法を指定してContainerを使ってレイアウトを組みました。

crossAxisCountが4にしているので最大4のうちどれだけcross axisに対して専有するのか、main axisに対してどれだけ専有するのかをstaggeredTilesパラメータで指定しています。指定した`List<StaggeredTile>`と同じ数のWidgetのListをchildrenパラメータに渡します。

これで静的にではありますがStaggeredGridViewの実装は済みました。このコードのstaggeredTileseパラメータの値やcrossAxisCountの値を変えることで自由にレイアウトを組み替えることができます。　

## Pinterest風のUIを作る

実現したいのはcross axisには決まった数（cross axisに対してちょうど半分）でmain axisにはコンテンツのサイズ分の範囲を持つTile上のレイアウトです。

そこで今回使うのは

- StaggeredGridView.countコンストラクタ
- StaggeredTile.fitコンストラクタ

の2つです。

### 画像について

UIのみの実装としたいので画像はassetとしてアプリケーションで保持します。Flutterで画像を追加する場合はpubspec.yamlで設定を行います。

今回はassetフォルダを画像を保持するフォルダとして設定したいのでassetsフォルダに画像を移動させた後、pubspec.yamlファイルに以下のように記述します。

```yaml
# To add assets to your application, add an assets section, like this:
  assets:
    - assets/
```

記述後、flutter pub getコマンドを実行します。

※アプリ内で使用させていただいた画像それぞれへのリンク先です。

- <a href="https://pixabay.com/ja/users/May_hokkaido-11960248/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5248955">May_hokkaido</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5248955">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/BarneyElo-2940289/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5312344">Barney Elo</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5312344">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/drphuc-8528636/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5292554">Phuc Nguyen</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5292554">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/thomaschung-17126556/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5323170">토마스 정</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5323170">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/eunseong0331-17315499/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5365324">서 은성</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5365324">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/blambasa-277544/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5419522">Branimir Lambaša</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5419522">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/zhuwei06191973-11952162/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5409203">wei zhu</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5409203">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/Photos_kast-11966122/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5408739">Ioannis Ioannidis</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5408739">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/MichaelGaida-652234/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5429112">https://500px.com/ michael-gaida</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5429112">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/TimHill-5727184/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5428427">Tim Hill</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5428427">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/enriquelopezgarre-3764790/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5413447">enriquelopezgarre</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5413447">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/Heidelbergerin-1425977/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5430404">Heidelbergerin</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5430404">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/analogicus-8164369/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5427299">analogicus</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5427299">Pixabay</a>からの画像
- <a href="https://pixabay.com/ja/users/pixel2013-2364555/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5332396">S. Hermann &amp; F. Richter</a>による<a href="https://pixabay.com/ja/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=5332396">Pixabay</a>からの画像

### 実装

```dart
class Home extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      appBar: AppBar(
        centerTitle: true,
        iconTheme: IconThemeData(
          color: Colors.white,
        ),
        title: Text('Pinterst風UI Sample'),
        actions: <Widget>[
          Container(
            margin: EdgeInsets.only(right: 10),
            child: Icon(Icons.message),
          )
        ],
      ),
      body: StaggeredGridView.count(
        crossAxisCount: 4,
        children: List.generate(15, (index) {
          return _Tile(index);
        }),
        staggeredTiles: List.generate(15, (index) {
          return StaggeredTile.fit(2);
        }),
      ),
    );
  }
}

class _Tile extends StatelessWidget {
  _Tile(this.index);

  final int index;

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: EdgeInsets.all(10),
      child: ClipRRect(
        borderRadius: BorderRadius.all(Radius.circular(5)),
        child: Image.asset('assets/${1 + index}.jpg'),
      ),
    );
  }
```

実行結果は以下になります。

![2020-08-03 20.53.11](/Users/tanabe.nobuyuki/Documents/drafts/images/2020-08-03 20.53.11.png)

![2020-08-03 20.54.33](/Users/tanabe.nobuyuki/Documents/drafts/images/2020-08-03 20.54.33.png)

StaggeredGridView.countコンストラクタのcrossAxisパラメータに4を渡してcross axis方向への全体的な寸法を指定しています。List.generateコンストラクタを使って`_Tile`ウィジェットを指定の数もつListをchildrenパラメータに渡しています。`_Tile`はパラメータにint型の値を受け取っていますがこれをString Interpolationで画像ファイルを指定するのに使っています。

staggeredTilesパラメータにもList.generateコンストラクタを使用しています。ここでは先述の通り要件にあわせてStaggeredTile.fitコンストラクタを、引数にはcross axisに指定した4に対してちょうど半分になるよう2を渡します。fitなのでmain axisへの範囲はコンテンツに依存します。実装の解説は以上になります。

### その他クラスについて

今までに記事で扱っていないクラスがいくつか登場しているので簡単な解説とドキュメントへのリンクを併記します。Widgetの基本的な使い方を解説しているシリーズで扱った時にはリンクを書き換えたいと思っています。

#### ClipRRect

childに指定したウィジェットを矩形に切り取るWidgetです。borderRadiusプロパティで丸みを指定できます。ドキュメントは[こちら](https://api.flutter.dev/flutter/widgets/ClipRect-class.html)です。

#### Padding

childに指定したWidgetにパディングをもたせることができるWidgetです。ドキュメントは[こちら](https://api.flutter.dev/flutter/widgets/Padding-class.html)です。

#### InkWell

マテリアルデザインでアクション可能な部品をタップすると波紋のように広がるアニメーションがありますが、リップルエフェクトと呼びます。そのようなエフェクトを持つコントロールを実装する時に使用するWidgetです。ドキュメントは[こちら](https://api.flutter.dev/flutter/material/InkWell-class.html)です。



## まとめ

初めて使用するクラスを使ってレイアウトを組む時には、試行錯誤を高速に繰り返せるHot Reloadが本当にありがたいです。Flutterは言語仕様がすごいシンプルで扱いやすいので日々の開発で得た経験を活かしてロジックを記述できていて不満を感じていません。一方思い通りのレイアウトを作るにはもう少しインプットが必要だなと感じているので良さそうなレイアウトをFlutterで作ってみることを繰り返しながら練習したいと考えています。