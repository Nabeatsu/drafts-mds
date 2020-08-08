業務でon/ofの状態を持つControlを作る時にUISwitchを思い浮かべたのですが、UISwitchがどの程度カスタマイズができるのかについてざっくりとしか把握していなかったことに気づきました。

※要改修

UIKitはオン・オフといった2つの状態を表現するControlです。UIControlとUIViewを使えば大抵のUIは実現できてしまうので、ちょっと込み入ったデザインのUIを実現する時にUIViewまたはUIControlのサブクラスにあれこれ詰め込んで要件を満たすUIを作ることもあります。ただ、用途に合わせて提供されているUIをなるべくは使いたいと思っています。広く公開された場所でメンテナンスされているライブラリとは異なって、そのような`Custom`UI パーツのメンテナンスや改修がかなり面倒なことが多いです。それでもUIKitで予め提供されているクラスでは実現が難しい時にはUIViewかUIControlを継承して自作することになりますが、その判断のために対象のクラスへの理解が必要です。

実務では簡単なカスタマイズで仕様を満たせましたが、いざという時に何ができて何ができないのか調べたりせずに自分で把握できるように記事にしてみようと思って今回この文章を書こうと思いました。

この記事では何度も使ったことがあるUISwitchの基本から各プロパティ、カスタマイズがどの程度まで行えるのかコードを書きながら改めて試してみたのでそれをまとめて書きます。

## UISwitch

ドキュメントは以下です。

- [UISwitch | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiswitch)

予めデフォルトの値が提供されているので何もカスタマイズせずにOS標準の挙動として使用できます。

![40f1968f533d0935ebfd263358c9a388](/Users/tanabe.nobuyuki/Downloads/40f1968f533d0935ebfd263358c9a388.gif)

UISwitchのインターフェースは以下です。`@available`属性が付与されていますが、実務でサポートするバージョンは現行の最新バージョンの1~5ぐらいまでが多いと思うので現在のメジャーバージョンが13であることを鑑みても`@aveilabe`を気にする必要はなさそうです。

```swift
@available(iOS 2.0, *)
open class UISwitch : UIControl, NSCoding {

    
    @available(iOS 5.0, *)
    open var onTintColor: UIColor?

    @available(iOS 6.0, *)
    open var thumbTintColor: UIColor?

    
    @available(iOS 6.0, *)
    open var onImage: UIImage?

    @available(iOS 6.0, *)
    open var offImage: UIImage?

    
    open var isOn: Bool

    
    public init(frame: CGRect) // This class enforces a size appropriate for the control, and so the frame size is ignored.

    public init?(coder: NSCoder)

    
    open func setOn(_ on: Bool, animated: Bool) // does not send action
}
```

### UISwitchのpublic propertyの概要

isOnがUISwitchの状態を表していて、スイッチをオンにした時の見た目の色合いに使われている色がonTintColorです。オフにした時のtintColorはoffTintColorです。thumbTintColorはonOffを選択する時のデフォルトでは円形になっているボタンの色を表しています。

onImageはOn状態の時、offImageはOffの時に表示される画像に使われていますが、iOS7以降では値をセットしてもレイアウトに変化はありません。

> In iOS 7 and later, this property has no effect.
> In iOS 6, this image represents the interior contents of the switch. The image you specify is composited with the switch’s rounded bezel and thumb to create the final appearance.
>
> https://developer.apple.com/documentation/uikit/uiswitch/1623683-offimage?language=objc

ドキュメントにも記載があります。

