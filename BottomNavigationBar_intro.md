# タイトル

Flutter で画面下部にナビゲーションメニューを配置する時は BottomNavigationBar という Widget を使用できます。基本的な Widget の使い方は覚えて状態管理のスタンダードを軽く学んだので実際にアプリを作り始めたのですがその際に使用した BottomNavigationBar について記事を書きたいと思います。

とは無関係なきっかけなのでシリーズには含めません。

内容は以下になります

- BottomNavigationBar の基本
- BottomNavigationBar のページングでの画面遷移
- BottomNavigationBar を使用した場合の Push 遷移での挙動制御
- BottomNavigationBar の画面遷移のアニメーション
- provider + BottomNavigationBar

## BottomNavigationBar の基本

1 例として 空の Container から一つずつ進めて作ってみます。以下のコードは Container を返しているだけのコードです。

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  static const String _title = 'Flutter Code Sample';

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: _title,
      home: HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

実行してみると真っ白になるはずです。

BottomNavigationBar で画面を切り替えるために最低限必要なものを用意します。bottom navigation bar で表示する Widget の一覧と表示中の Widget を取り出すための index としての int 型の mutable な値を定義します。

```dart
class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // 表示中の Widget を取り出すための index としての int 型の mutable な stored property
  int _selectedIndex = 0;

  // 表示する Widget の一覧
  static List<Widget> _pageList = [
    CustomPage(pannelColor: Colors.cyan, title: 'Home'),
    CustomPage(pannelColor: Colors.green, title: 'Settings'),
    CustomPage(pannelColor: Colors.pink, title: 'Search')
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('サンプル1'),
      ),
      body: _pageList[_selectedIndex],
    );
  }
}

class CustomPage extends StatelessWidget {} // ナビゲーションバーをタップした時に切り替わるWidgetの定義
}
```

次は StatefulWidget の build メソッドを実装します。Scaffold のコンストラクタ引数 bottomNavigationBar に BottomNavigationBar を渡します。

BottomNavigationBar の引数 currentIndex を mutable property にして引数 onTap に渡したメソッドで setState を呼びその mutable property を更新します。これで画面が切り替わるはずです。

また引数 items にはナビゲーションバーに表示する item（BottomNavigationBarItem）のリストを指定します。

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: const Text('サンプル1'),
    ),
    body: _pageList[_selectedIndex],
    bottomNavigationBar: BottomNavigationBar(
      items: const <BottomNavigationBarItem>[
        BottomNavigationBarItem(
          icon: Icon(Icons.home),
          title: Text('Home'),
        ),
        BottomNavigationBarItem(
          icon: Icon(Icons.settings),
          title: Text('Setting'),
        ),
        BottomNavigationBarItem(
          icon: Icon(Icons.search),
          title: Text('Search'),
        ),
      ],
      currentIndex: _selectedIndex,
      onTap: _onItemTapped,
    ),
  );
}

// タップ時の処理
void _onItemTapped(int index) {
  setState(() {
    _selectedIndex = index;
  });
}
```

コード全文です。codepen 埋め込みです。実行結果も見れますがその場合は Chrome での閲覧を推奨します。

<p class="codepen" data-height="700" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="XWmRgoW" style="height: 700px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="bottom_navigation_bar_sample1">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/XWmRgoW">
  bottom_navigation_bar_sample1</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

BottomNavigationBar のドキュメントは以下になります。選択時の見た目の制御を選択できる type プロパティや選択時、非選択時のスタイルを調整できるプロパティもあり、ある程度カスタマイズが効くと思います。

- [BottomNavigationBar class - material library - Dart API](https://api.flutter.dev/flutter/material/BottomNavigationBar-class.html)

## BottomNavigationBar のページングでの画面遷移

このような UI だとページングで画面が切り替わるパターンがあります。それを実現する一つの例としては BottomNavigationBar に加えて PageView、PageController を使用する方法があります。

PageViewController でスワイプを検知してアニメーションさせます。initialPage プロパティを使って開始ページを設定します。PageView のコンストラクタ引数に PageViewController
のインスタンスを渡して遷移を実装します。

\_selectedIndex を setState で更新しているのでページングによる BottomNavigationItem のハイライトも合わせて反映されます。

ここで終わると BottomNavigationBar の onTap の処理を変更しないと画面遷移の動きに差異が出てしまいます。ボタンタップ時は現状ページングされません。

そのために private プロパティとして PageController のインスタンスを保持しています。

onTap のタイミングで PageController の animateToPage メソッドでページングによる画面遷移を行うよう実装します。

コードは以下です。Chrome の人は Result タブでページングが実装できていることが確認できると思います。

<p class="codepen" data-height="700" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="abvwEbO" style="height: 700px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="bottom_navigation_bar_sample2">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/abvwEbO">
  bottom_navigation_bar_sample2</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## BottomNavigationBar を使用した場合の Push 遷移で下タブを残したまま遷移したい時

BottomNavigationBar を使った Push 遷移は BottomNavigationBar が隠れてしまいます。Material Design 的にはその動きで問題ないらしいのですが残したまま Push 遷移したいこともあります。iOS アプリの場合は特にそうだと思います。

その際は公式に提供されている CupertinoTabScaffold を使うのが良いです。

※参考

- [Flutter: Keep BottomNavigationBar When Push to New Screen with Navigator - Stack Overflow](https://stackoverflow.com/questions/49628510/flutter-keep-bottomnavigationbar-when-push-to-new-screen-with-navigator)

- [Keep BottomNavigationBar When Push to New Screen with Navigator · Issue #16181 · flutter/flutter](https://github.com/flutter/flutter/issues/16181)

BottomNavigationBar で同じ動きを実装するワークアラウンドもあります。写経して試しましたが動いた時は感動したもののここまでするなら公式の Widget に乗っかりたいなとも思いました。写経のため自分が書いたような語り口で紹介するのも気が引けるので手順のみ載せて参考記事とリポジトリを紹介します。

- Navigator.of で BottomNavigationBar の祖先の Navigator を context から見つけてしまう
- Navigator.of で BottomNavigationBar の祖先に当たらない Navigator を見つけるように Navigator を内包するカスタムウィジェット内に Push の処理を記述する
- Navigator の識別のために GlobalKey を使う
- 下タブのタップ時の動きは Stack を使って表示する画面一覧を保持して Offstage を使って任意のタブを Offstage（見えないよう）にして実現する

この実装について紹介した記事は以下になります。

- [Flutter Case Study: Multiple Navigators with BottomNavigationBar https://medium.com/coding-with-flutter/flutter-case-study-multiple-navigators-with-bottomnavigationbar-90eb6caa6dbf]

リポジトリは以下。

- [bizz84/nested-navigation-demo-flutter: Nested navigation with BottomNavigationBar](https://github.com/bizz84/nested-navigation-demo-flutter)

## BottomNavigationBar の状態管理

状態管理に推奨されている provider というライブラリを使って画面遷移を管理してみた実装です。外部ライブラリを使っているので codepen 埋め込みではないです。

<script src="https://gist.github.com/Nabeatsu/f4642f723f49648a2692b6ad7cd18c55.js"></script>

int でやりくりしたくない気持ちもあったので BottomNavigationBar をラップしたカスタムウィジェットを使って実装したのが以下です。
