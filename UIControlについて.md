UIButtonなどのユーザーのインタラクションに応答して視覚的な効果を表したりするパーツはUIControlを基底クラスに持ちます。UIButtonを筆頭に仕様にあわせて柔軟に変更することが叶わないことも多く、その場合はそれらのクラスと同じようにUIControlを継承すると仕様にあわせたコントロールを作成できます。

今回はUIControlの概要と基本的な使い方を紹介できればと思います。この記事の後にもう少し細かいユースケースをUIControlを使って実現してみる記事を書くつもりでいます。　

## UIControl

UIControlのControlについて説明します。

Appleの[Views and Controls](https://developer.apple.com/documentation/uikit/uicontrol)というドキュメントにViewとControlはアプリのUIにおける視覚的な構成要素として説明があります。

[UIControlのドキュメント](https://developer.apple.com/documentation/uikit/uicontrol)にはコントロールはユーザーのインタラクションに応じて特定の動作や意図を伝える視覚的な要素のことを指していて、それを表すのがUIControlであると記載されています。

UIControlはUIView、UIViewはUIResponder、UIResponderはNSObjectを基底クラスに持ちます。

```
NSObject -> UIResponder -> UIView -> UIControl
```

UIControlが公開している様々なプロパティやメソッドにアクセスでき、クラス階層にあるクラスのプロパティ、メソッドも使用できるためカスタマイズ性は高いですが、かなり多いので把握するのは大変です。

Swift自体は値型中心のプログラミング言語ですがUIKitはクラスベースで作られているので参照型を扱わなければいけません。だからといって抽象化の時にサブクラスを重ねてクラス階層を深めるのは辛くなる印象があります。UIControlを継承したXXControlを継承したサブクラス……のような作り方はメンテナンスの時にコードジャンプを繰り返さなければいけないため、あまりしたくないです。

なので実際に使う際はUIControlを継承したサブクラスを作ってそこにレイアウトに関する実装を閉じ込めて他のUIKitが提供しているクラスのようにcompositional layoutっぽく使うことが多いのではないかなと思っています。

UIControlでアプリケーションコードからアクセスできるプロパティやメソッドは主に以下のようになります。

```swift
@available(iOS 2.0, *)
open class UIControl : UIView {
    open var isEnabled: Bool
    open var isSelected: Bool
    open var isHighlighted: Bool
    open var contentVerticalAlignment: UIControl.ContentVerticalAlignment
    open var contentHorizontalAlignment: UIControl.ContentHorizontalAlignment
    open var effectiveContentHorizontalAlignment: UIControl.ContentHorizontalAlignment { get }     
    open var state: UIControl.State { get }
    open var isTracking: Bool { get }
    open var isTouchInside: Bool { get }    
    open func beginTracking(_ touch: UITouch, with event: UIEvent?) -> Bool
    open func continueTracking(_ touch: UITouch, with event: UIEvent?) -> Bool
    open func endTracking(_ touch: UITouch?, with event: UIEvent?)
    open func cancelTracking(with event: UIEvent?)
    open func addTarget(_ target: Any?, action: Selector, for controlEvents: UIControl.Event)
    open func removeTarget(_ target: Any?, action: Selector?, for controlEvents: UIControl.Event)
    open var allTargets: Set<AnyHashable> { get }
    open var allControlEvents: UIControl.Event { get }    
    open func actions(forTarget target: Any?, forControlEvent controlEvent: UIControl.Event) -> [String]?
    open func sendAction(_ action: Selector, to target: Any?, for event: UIEvent?)
    open func sendActions(for controlEvents: UIControl.Event)
}
```











