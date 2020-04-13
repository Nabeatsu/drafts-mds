# FlutterのThemeについて

CX事業本部の田辺です。少し前にFlutterのThemeクラスを理解したいと思って周辺のドキュメントを流し読みしていました。今回はもう一度ドキュメントを読み込み手元で動かして挙動を確認した内容を記事にします。


## Theme とは

子以下のWidgetにテーマを適用するためのクラスです。ここでいうテーマは`A theme describes the colors and typographic choices of an application`です。アプリケーションの色やテキストの表現方法に統一性をもたせて管理できます。

### 基本的な使い方

手元に最低限の機能を持ったToDoアプリがあります。このアプリのコードはThemeを使っておらず、個別にスタイルを指定しています。

![4-1](images/4-1.png)

![4-2](images/4-2.png)

統一感を持たせるかは個別のスタイルの指定次第です。

このアプリにThemeを使ってみます。

基本的にThemeはMaterialAppのコンストラクタ引数themeにThemeDataを指定することでアプリ全体にテーマが適用されます。

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

import 'models/task_data.dart';
import 'screens/tasks_screen.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  final String data = "Top Secret Data";
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => TaskData(),
      child: MaterialApp(
        home: TasksScreen(),
      ),
    );
  }
}
```

を以下のようにします。

```dart
class MyApp extends StatelessWidget {
  final String data = "Top Secret Data";
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (context) => TaskData(),
      child: MaterialApp(
        home: TasksScreen(),
        theme: ThemeData(primaryColor: Colors.purple), // コンストラクタ引数themeにThemeDataを渡す
      ),
    );
  }
}

```

#### ThemeDataについて

ThemeDataを指定しますがここまでThemeDataに関する説明を行ってません。そこでまずはThemeDataとは何か、どのような役割を持ったクラスなのかを紹介します。

ドキュメントは以下になります。

- [ThemeData class - material library - Dart API](https://api.flutter.dev/flutter/material/ThemeData-class.html)

マテリアルデザインのテーマの色と文字表現のデザイン処理(タイポグラフィと表現されてます)を保持します。現在のテーマを取得するには、Theme.ofを使用します。

ThemeDataの各プロパティに値をコンストラクタ経由でセットすることでスタイルの微調整ができます。

ThemeDataのソースコードを読むとfactory Constructorが定義されています。コンストラクタ引数には`@required`がついていないので省略可能です。省略された場合はFlutter側で値をセットしてくれます。使用者は適宜変えたい所だけ変更すれば良いだけです。少しだけコードを貼ります。長いので省略してかつコメントを挿入しています。

```dart
factory ThemeData({
    Brightness brightness,
    MaterialColor primarySwatch,
    Color primaryColor,
    Brightness primaryColorBrightness,
    Color primaryColorLight,
    Color primaryColorDark,
    Color accentColor,
    Brightness accentColorBrightness,
    Color canvasColor,
    // 以下大量のコンストラクタ引数の列挙   
  }) {
    // コンストラクタの中身がこちら。
    // このコンストラクタの引数に値が渡されなくてもFlutter側でよしなにThemeDataを作ってくれていることがわかる
    brightness ??= Brightness.light;
    final bool isDark = brightness == Brightness.dark;
    primarySwatch ??= Colors.blue;
    primaryColor ??= isDark ? Colors.grey[900] : primarySwatch;
    primaryColorBrightness ??= estimateBrightnessForColor(primaryColor);
    primaryColorLight ??= isDark ? Colors.grey[500] : primarySwatch[100];
    primaryColorDark ??= isDark ? Colors.black : primarySwatch[700];
    final bool primaryIsDark = primaryColorBrightness == Brightness.dark;
    toggleableActiveColor ??= isDark ? Colors.tealAccent[200] : (accentColor ?? primarySwatch[600]);
    accentColor ??= isDark ? Colors.tealAccent[200] : primarySwatch[500];
    accentColorBrightness ??= estimateBrightnessForColor(accentColor);
    final bool accentIsDark = accentColorBrightness == Brightness.dark;
    canvasColor ??= isDark ? Colors.grey[850] : Colors.grey[50];
    scaffoldBackgroundColor ??= canvasColor;
    bottomAppBarColor ??= isDark ? Colors.grey[800] : Colors.white;
    cardColor ??= isDark ? Colors.grey[800] : Colors.white;
    dividerColor ??= isDark ? const Color(0x1FFFFFFF) : const Color(0x1F000000);

    // Create a ColorScheme that is backwards compatible as possible
    // with the existing default ThemeData color values.
    colorScheme ??= ColorScheme.fromSwatch(
      primarySwatch: primarySwatch,
      primaryColorDark: primaryColorDark,
      accentColor: accentColor,
      cardColor: cardColor,
      backgroundColor: backgroundColor,
      errorColor: errorColor,
      brightness: brightness,
    );
```

ThemeDataのプロパティはかなりたくさんあるのでデフォルト設定を利用しつつ個別に必要な所のみ変更することになるのかなと思っています。

#### ThemeDataを子Widgetで利用する

MaterialAppのTHemeに指定しても直接色を指定している部分はThemeが反映されません。

```dart
@override
  Widget build(BuildContext context) {
    return ListTile(
      onLongPress: longPressCallback,
      title: Text(
        taskTitle,
        style: TextStyle(
          decoration: isChecked ? TextDecoration.lineThrough : null,
        ),
      ),
      trailing: Checkbox(
        activeColor: Colors.lightBlueAccent, // 直接指定しているのでThemeが反映されない
        value: isChecked,
        onChanged: checkboxCallback,
      ),
    );
  }
```


ListTileはタスク一覧画面で使用しています。チェックボックスに直接色を指定しているのでこれを削除します。

削除前
![4-3](images/4-3.png)

削除後
![4-4](images/4-4.png)

自動で各Widgetで設定される色やスタイルでなく設定したThemeData由来ではあるものの少しカスタマイズしたい場合があります。そのような時は冒頭に記載したようにTHeme.of(context)で共通のThemeDataを取得してプロパティを利用します。

今回はTheme.ofで取り出したつつこれまでどおりチェックが入った時は取り消し線が入るようにdecorationプロパティをカスタマイズしたりフォントサイズを変更したりするため`copyWith()`を使います。

```dart
 @override
  Widget build(BuildContext context) {
    return ListTile(
      onLongPress: longPressCallback,
      title: Text(
        taskTitle,
        style: Theme.of(context).textTheme.body2.copyWith(
              decoration: isChecked ? TextDecoration.lineThrough : null,
            ),
      ),
      trailing: Checkbox(
        value: isChecked,
        onChanged: checkboxCallback,
      ),
    );
  }
}

```


変更前
![4-5](images/4-5.png)

変更後
![4-6](images/4-6.png)


## ダークモード対応
