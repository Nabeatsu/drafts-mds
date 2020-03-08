Human Interface Guidelinesを読むとAppleはSign in with Apple 用のボタンを提供していて、それを使う必要があると明記されています。また、ボタンのカスタマイズできる点も限られていて、開発者側がコントロール出来る部分は多くありません。この記事ではSign in with Apple用に提供されているボタンでどのようなカスタマイズが出来るのか実際に動かしながら一通り触ってみたので記事にしようと思います。この記事ではSign in with Appleの仕組みといったボタン以外の要素には触れません。Sign in with Appleについてより深く知りたい場合はガイドラインやその他の記事をご参照ください。

- [はじめに - Appleでサインイン - Apple Developer](https://developer.apple.com/jp/sign-in-with-apple/get-started/)
- [Firebase AuthenticationでSign In With Appleを実装する ｜ Developers.IO](https://dev.classmethod.jp/smartphone/firebase-auth-sign-with-apple/)

## 用意されているAPIを使う

ボタンを提供するためのAPIが予め用意されているので、開発者はそれを利用するだけでSign in with Appleボタンを実装できます。

- アップルが実装したスタイルなのでガイドラインに抵触する心配がない
- スタイルの変更を行っても比率が維持される
- 自動的なローカライゼーション
- UIに応じたボタンのCorner radiusの変更サポート
- Voice overサポート

今回はiOSで実装するので`ASAuthorizationAppleIDButton`というクラスを使ってボタンを実装します。

### ボタン表示まで実装

ASAuthorizationAppleIDButtonのinitializerのシグネチャは以下です。引数はButtonTypeというenumとStyleというenumを取ります。

```swift
public convenience init(type: ASAuthorizationAppleIDButton.ButtonType, style: ASAuthorizationAppleIDButton.Style)
public init(authorizationButtonType type: ASAuthorizationAppleIDButton.ButtonType, authorizationButtonStyle style: ASAuthorizationAppleIDButton.Style)
```

authorizationButtonTypeをdefault、authorizationButtonStyleにwhiteOutlineを指定して初期化してみます。

```swift
let appleLoginButton = ASAuthorizationAppleIDButton(
    authorizationButtonType: .default,
    authorizationButtonStyle: .whiteOutline
)
```

実行結果は以下のようになります。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/36f43603fdeb91e96568596c57e7432c.png" alt="" width="408" height="108" class="alignnone size-full wp-image-539776" />

Appleが提供しているログインボタンには白、黒のアウトライン付きの白、黒のスタイルがありASAuthorizationAppleIDButton.Styleではこれを指定します。

### 変更できる部分

ASAuthorizationAppleIDButtonを私用する場合はカスタマイズできるのはごく僅かでcornerRadiusとボタンのサイズのみになります。
```swift
appleLoginButton.cornerRadius = <#T##CGFloat#>
appleLoginButton.bounds = CGRect(x: <#T##CGFloat#>, y: <#T##CGFloat#>, width: <#T##CGFloat#>, height: <#T##CGFloat#>)
```

#### ASAuthorizationAppleIDButton.Styleにwhiteを指定した場合

```swift
let appleLoginButton = ASAuthorizationAppleIDButton(
    authorizationButtonType: .default,
    authorizationButtonStyle: .white
)
```

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/eb9f4336a70459128628d4f28e42e566.png" alt="" width="410" height="142" class="alignnone size-full wp-image-539777" />

#### ASAuthorizationAppleIDButton.Styleにblackを指定した場合

```swift
let appleLoginButton = ASAuthorizationAppleIDButton(
    authorizationButtonType: .default,
    authorizationButtonStyle: .white
)
```

それぞれのStyleに使用すべきシチュエーションが記載されています。コードにもドキュメンテーションで書いて欲しいとちょっと思いましたがガイドラインに記載されています。使用すべき場合と使用してはいけない場合が明確に明記されていて画像も紹介されているのでアプリの画面に合わせて指定するのが良いでしょう。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/5357c1da9581c6654b9f021dd0bbd8f2.png" alt="" width="410" height="107" class="alignnone size-full wp-image-539778" />

### cornerRadiusの指定

数少ない指定可能なプロパティは先述しましたがcornerRadiusです。デフォルトでも少し角丸なっていますが任意の値を代入できます。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/532f2bc1baf03095648f7652b38d06b9.png" alt="" width="406" height="95" class="alignnone size-full wp-image-539779" />

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/a1bab9bbd2f4a8296d8cc9c99144c0f8.png" alt="" width="407" height="97" class="alignnone size-full wp-image-539780" />


### ボタンのテキスト

ボタンに表示されるテキストは英語です。ですがAppleが提供しているボタンを使用するメリットに自動的なローカライゼーションをあげました。これはドキュメントにも記載されています。このローカライゼーションを行うにはXcodeのProject Navigator > Info > Localizationsで言語を追加すれば良いです

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/ca1f8f03ff2ad2dd589a36c186044c36.png" alt="" width="839" height="145" class="alignnone size-full wp-image-539782" />

## ボタンのデザインに関するガイドライン

該当のページは以下です。

- [Buttons - Sign in with Apple - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/sign-in-with-apple/overview/buttons/)


Sign in with Appleボタンを目立つように表示するように規約には記載されています。具体的には他のボタンよりも小さく表示させない、極端な場所に配置したりせずボタンを見るために人々がスクロールせずに済むよう優先的に配置するといった対応をすることになるのでしょう。

## レイアウトに応じたカスタマイズについて

ボタンについてのガイドラインの`Creating a Custom Sign in with Apple Button`という節で画面レイアウトに応じたカスタマイズについて触れられています。

Appleは最近Human Interface GuidelinesをアップデートしResourcesに変更を加えました。そしてSign In with Apple向けに左寄せ用の素材とロゴだけを表示するボタン用の素材が加えられています。

- [Apple's Developer HIG updated with new buttons, icons, Sign In with Apple logo - 9to5Mac](https://9to5mac.com/2020/02/14/apple-developer-hig-update-logo-buttons-icons/)

[Apple Design Resources - Apple Developer](https://developer.apple.com/design/resources/)にダウンロードできるリソースへのリンクがあります。それをダウンロードしてアプリ内で使用できるようにします。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/460242f572cde024ba6b7cf307a4babd.png" alt="" width="914" height="247" class="alignnone size-full wp-image-539783" />

### ロゴの左揃え

今回はstoryboardを使ってボタンを作成してみました。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2020/03/caf4bba43a43fb0624618dfd7f1e1845.png" alt="" width="372" height="102" class="alignnone size-full wp-image-539784" />

カスタムボタンの作成では気にしないといけないことがかなりあります。

- ベクター形式でないPNGファイルを使用する場合はボタンの高さは44pt固定
- ボタンタイトルはシステムフォント
- フォントサイズはボタンの高さに対して43%。高さを44pt固定にしている場合のフォントサイズは19pt
- テキストはSign in with Apple, Sign up with Apple, Continue with Appleのいずれか。大文字、小文字のスタイルも変更不可
- タイトルとロゴをボタン内で垂直に揃える
- タイトルとボタンの間の最小マージンを守る。ボタン幅に対して8%以上
- ボタンの最小サイズとボタンの周囲の余白を維持する

| 最小値 | 最小の高さ | 最小マージン |
| :---- | :---- | :---- |
| 140pt（140px @ 1x、280px @ 2x） | 30pt（30px @ 1x、60px @ 2x） | ボタンの高さの1/10 |

### 画像のみのボタン

画像のみのボタンの制限事項は３つです。

- PNGファイルの場合は44ptx44ptで固定
- 水平方向のパディング追加の禁止。常にアスペクト比は1:1
- 素材の加工(トリミング)は禁止。マスクを使用して表示をカスタマイズする

## まとめ

今までに公私でSign in with Appleの実装を行ってきた中で必ずボタンはどこまでカスタマイズできるか尋ねられました。HIGのリンクを投げて読んどいてで終わり、という関係性の人だと楽ですがそうでない場合も多いのでそんな時に見せるページとして作りたかったというのもこの記事を書こうとした理由の一つです。

提供されいているAPIしか使ったことがなかったのでこの記事ではじめてカスタムボタンを作成して一連の流れを触れたのは良かったと思います。

もし業務でカスタムボタンを使用することになったらstoryboardか規約を満たせなかった時に気づけるようにpreconditionを使いながらで外から引数としてカスタマイズできる値を受け取るメソッドをextensionに生やすかどちらかで実装すると思います。

### 公式のドキュメント以外に参照したリンク一覧

- [如何整合 Sign in with Apple 到自己的 iOS App 上 (iOS & Backend)](https://medium.com/@tuzaiz/%E5%A6%82%E4%BD%95%E6%95%B4%E5%90%88-sign-in-with-apple-%E5%88%B0%E8%87%AA%E5%B7%B1%E7%9A%84-ios-app-%E4%B8%8A-ios-backend-e64d9de15410)
- [How to Use ASAuthorizationAppleIDButton in Storyboard](https://swiftsenpai.com/xcode/asauthorizationappleidbutton-in-storyboard/)
- [titanium-apple-sign-in/TiApplesigninLoginButton.swift at master · appcelerator-modules/titanium-apple-sign-in](https://github.com/appcelerator-modules/titanium-apple-sign-in/blob/master/ios/Classes/TiApplesigninLoginButton.swift)
