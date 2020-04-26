Flutter は動画コンテンツも充実していて週毎にウィジェットを紹介している Flutter Widget of the Week では短い動画でウィジェットの概要を掴むことができる動画を視聴できます。

- [Flutter Widget of the Week - YouTube](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/02/maxresdefault.jpg" alt="" width="1280" height="720" class="alignnone size-full wp-image-532693" />

Flutter Widget of the Week を見るだけでも勉強になるのですが、記憶力が良くないので、視聴によるインプットだけでなく手元でコードを動かしてそれをブログ記事にする所までやって後から思い出しやすいようにしてみるシリーズです。

これまでの記事は以下のリンクから参照できます。

- [Flutter の Widget をコードを動かしながら学ぶ ｜ シリーズ ｜ Developers.IO](https://dev.classmethod.jp/series/flutter%e3%81%aewidget%e3%82%92%e3%82%b3%e3%83%bc%e3%83%89%e3%82%92%e5%8b%95%e3%81%8b%e3%81%97%e3%81%aa%e3%81%8c%e3%82%89%e5%ad%a6%e3%81%b6/)

今回取り上げるのは AnimatedContainer です。

## AnimatedContainer

指定した Duration に対して徐々に値を変更してアニメーションしてくれる Container です。Container の各プロパティを変更するだけでアニメーションを作れるので便利そうです。

ドキュメントは以下です。

- [AnimatedContainer class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/AnimatedContainer-class.html)

ドキュメントには以下のように記載されています。

> Animated version of Container that gradually changes its values over a period of time.

https://www.youtube.com/watch?v=yI-8QHpGIP4

## AnimatedContainer とは

Flutter Widget of the Week で AnimatedContainer をとりあげた動画は以下です。AnimatedContainer のドキュメントを見ると、複数の行または列でその子のウィジェットを表示するためのウィジェットと記載されています。

https://www.youtube.com/watch?v=z5iw2SeFx2M

## AnimatedContainer を使ってみる

提供されているプロパティの変化をアニメーションしてくれるこのウィジェットを使えば色やサイズ、配置など、様々なプロパティを指定するだけでアニメーションを実装できます。最初は Container の width と height を FloatingActionButton のタップの度に変更するよう実装してみます。

<script src="https://gist.github.com/Nabeatsu/009be59d0a0b4ed6261a88e9d3f6f722.js"></script>

実行結果は以下のようになります。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/04/cca29c403cca69cf8373a0d8a5883adf.gif" alt="" width="238" height="510" class="alignnone size-full wp-image-559036" />

今回は StatefulWidget と setState を使って変数の値を書き換えてアニメーションを実現しています。

### ImplicityAnimatedWidget

AnimatedController のソースコードを読むと ImplicitlyAnimatedWidget というウィジェットを継承していることがわかります。

```
class AnimatedContainer extends ImplicitlyAnimatedWidget {
```

ImplicitlyAnimatedWidget はプロパティの変化をアニメーションさせるウィジェットを構築するための抽象クラスとドキュメントには記載があります。ImplicitlyAniamtedWidget を理解することがこれを継承する AnimatedContainer を含めたウィジェットの挙動を理解するのに繋がりそうです。

> An abstract class for building widgets that animate changes to their properties.

- [ImplicitlyAnimatedWidget class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/ImplicitlyAnimatedWidget-class.html)

ImplicitlyAnimatedWidget は初期状態ではアニメーションが行われないことが特徴です。プロパティの値の変化が会った時に指定された`Duration`に応じてアニメーションが行われます。

#### Duration

ImplicitlyAnimatedWidget の duration プロパティはプロパティのパラメータをアニメーションさせる期間を指定するクラスです。ソースコードを読んでコンストラクタと duration プロパティの定義を確認してみると final キーワードが付与されていてコンストラクタで値が渡されることを保証していることがわかります。curve プロパティも同じ扱いですが一旦は duration プロパティだけに目を向けます。

```
const ImplicitlyAnimatedWidget({
    Key key,
    this.curve = Curves.linear,
    @required this.duration,
    this.onEnd,
  }) : assert(curve != null),
       assert(duration != null),
       super(key: key);

  /// The curve to apply when animating the parameters of this container.
  final Curve curve;

  /// The duration over which to animate the parameters of this container.
  final Duration duration;
```

- [Duration class - dart:core library - Dart API](https://api.flutter.dev/flutter/dart-core/Duration-class.html)

Duration は、ある時点から別の時点への差(期間)を表すクラスです。

ドキュメントには宣言方法が記載されてあります。

```
Duration fastestMarathon = new Duration(hours:2, minutes:3, seconds:2);
```

#### Curve

次は、先程一旦脇に置いておいた AnimatedContainer が継承している抽象クラス ImplicitlyAnimatedWidget のコンストラクタでコンストラクタ引数に渡すことが必須の curve プロパティの話をします。

duration プロパティの時と同じ様にプロパティの型について見ていく前にこのプロパティが ImplicitylyAnimatedWidget でどのように扱われているか見てみます。

> The curve to apply when animating the parameters of this container.

定義を見るとわかったようなわからないような、となるので Curve クラスのドキュメントを見てみます。

- [Curve class - animation library - Dart API](https://api.flutter.dev/flutter/animation/Curve-class.html)

Curve クラスはある単位間隔から単位間隔へのマッピングを表すイージングを定義します。

イージングを使うことで直線的な値の変化でなく加速（ease in）または減速(ease out)する自然なアニメーションが表現できます。イージングという言葉については Google が公開している Web Fundamentals というドキュメントのイージングについて扱ったページのドキュメントなどがわかりやすいです。

- [イージングの基本  |  Web  |  Google Developers](https://developers.google.com/web/fundamentals/design-and-ux/animations/the-basics-of-easing?hl=ja)
- [世界一わかりやすい「イージング」と、その応用｜ ritar ｜ note](https://note.com/ritar/n/n5e8ed0e07917)

Flutter では Color という色を表現するクラスの値を指定する時に Color のコンストラクタで Color を指定することもありますが、Colors というクラスに定義された static な定数を渡すことで指定することも多いです。同じ様に Curve クラスの値を指定するために Curves クラスに static 定数が定義されています。

ソースコードだけを見て Curves に定義された定数を選択するのも良いですが Flutter では Curves に定義された各定数の詳細な動きがビジュアライズされたページを提供しています。

- [Curves class - animation library - Dart API](https://api.flutter.dev/flutter/animation/Curves-class.html)

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/04/f3774652fbef01a00677d463e4e00233.gif" alt="" width="502" height="916" class="alignnone size-full wp-image-559608" />

## AnimatedContainer のその他主要なプロパティについて

Container で提供されているプロパティは AnimatedContainer でも提供されています。例えば Container の子ウィジェットの配置を設定できる alignment をアニメーションさせようとすると以下のようなコードになります。ここからはそのまま動くように codepen の埋め込みを使用します。codepen の動作が Safari だと安定しない所がありました（iOS の Safari で確認）。Chrome での閲覧・利用推奨です。

<p class="codepen" data-height="977" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="OJybLEv" style="height: 977px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="alignmentのアニメーション">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/OJybLEv">
  alignmentのアニメーション</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

子ウィジェットとの Padding を調整できる padding プロパティの切り替わりをアニメーションさせるには以下のようなコードになります。

<p class="codepen" data-height="977" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="BaoQBMP" style="height: 977px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="animated_container_sample2">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/BaoQBMP">
  animated_container_sample2</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

transform プロパティの型は Matrix4 クラスです。ドキュメントを読むと 4D Matrix という言葉が出てきます。

- [Matrix4 class - vector_math_64 library - Dart API](https://api.flutter.dev/flutter/vector_math_64/Matrix4-class.html)

> 4D Matrix. Values are stored in column major order.

行列計算でどのように transform(変形)するか自由に設定できますが知識がなくても factory コンストラクタを利用することでいくつかのパターンから選択できます。

要素を移動させる transfo ｒｍが設定できる translationValues を使ってみます。ソースコードは以下です。

```
/// Translation matrix.
  factory Matrix4.translationValues(double x, double y, double z) =>
      new Matrix4.zero()
        ..setIdentity()
        ..setTranslationRaw(x, y, z);
```

codepen は以下になります。

<p class="codepen" data-height="973" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="QWjGWyR" style="height: 973px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="animated_container_sample3">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/QWjGWyR">
  animated_container_sample3</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

要素を回転させる transform を設定できる rotationX、rotationY、rotationZ があります。

factory コンストラクタは以下です。

```
/// Rotation of [radians_] around Z.
  factory Matrix4.rotationZ(double radians) => new Matrix4.zero()
    .._m4storage[15] = 1.0
    ..setRotationZ(radians);
```

<p class="codepen" data-height="1211" data-theme-id="light" data-default-tab="js,result" data-user="nabeatsu" data-slug-hash="QWjGWJj" style="height: 1211px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="animated_container_sample4">
  <span>See the Pen <a href="https://codepen.io/nabeatsu/pen/QWjGWJj">
  animated_container_sample4</a> by Nobuyuki Tanabe (<a href="https://codepen.io/nabeatsu">@nabeatsu</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

## AnimatedContainer などの Implicit なアニメーションウィジェットの位置付け

利用するだけで簡単にアニメーションが適用できる ImplicitlyAnimatedWidget の派生クラスなどの Implicit widgets の位置付けについては[Flutter のお手軽にアニメーションを扱える Animated 系 Widget をすべて紹介 - Flutter 🇯🇵 - Medium](https://medium.com/flutter-jp/implicit-animation-b9d4b7358c28)という記事で紹介されていたセッション動画のスライドの 1 ページがわかりやすいと思います。
<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/04/47596ec9ac0b445e036d0d6b00f86d3d.png" alt="" width="1027" height="518" class="alignnone size-full wp-image-560037" />

動画はこちら

https://youtu.be/PFKg_woOJmI?t=164

動画や記事にあるように AnimationController を使う前に AnimatedContainer などの手軽に使えるウィジェットで解決できないか検討してみるようにしたいと思いました。実は過去に書いた記事で紹介した Udemy のコースでは AnimationController の使い方について詳細に紹介されていました。そこで AnimatedContainer ならもっと簡単に終わるプロパティの変化をアニメーションさせる実装を行ったのを覚えています。同じことをいくつもの方法で実現できることを知ることは引き出しが増えていざという時に役に立ったりするのでコードを書くのと並行してインプットを続けていきたいと思います。

## まとめ

記事の主題とはそれますが codepen の埋め込みが Flutter 対応したとのことだったので使ってみました。プログラミング初心者の頃を思い出すとそのままで動くコードがありがたかったなと思い出したのも全コードを載せる codepen を使ってみた理由です。
Flutter の学習・開発は始めたばかりで説明やその裏にある認識に誤りがあるかもしれません。なにかお気づきの際はコメントなどでお知らせください。
