クラスメソッド福岡オフィス CX事業本部でiOSアプリエンジニアとして働いている田辺です。

WWDC2019のセッション動画が既に公開されています。
チュートリアルと簡単なアプリの実装はやってみたものの、セッションについては記事をいくつか読んだ程度でした。記事に書くことをモチベーションにしてセッション動画を観ていくことにしました。iOSDC開催日までに全部観るのを目標にやっていきます。

内容の要約やセッションで登場した概念を少し掘り下げて調べたり実際にコードを書いて試した内容をまとめたものになります。記事で説明する概念がセッションのどのあたりなのかについてもリンクを張ります。

今回観たのはSwiftUI Essentialというセッションです。動画もスライドもtranscriptもリンク先でダウンロードできます。動画はストリーミングで視聴可能です。後々日本語の翻訳もつくかもしれないですが今はまだありません。

- [SwiftUI Essentials - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/216/)
<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/WWDC-WWDC-2019-WWDC-live-stream-WWDC-Apple-WWDC-Apple-live-Apple-event-live-WWDC-2019-uk-time-WWDC-time-uk-apple-1135622.jpg" alt="" width="590" height="350" class="alignnone size-full wp-image-457539" />

このセッションの冒頭で、SwiftUIは優れたUIへの最も短いpathを提供するために作られたツールと説明されています。アプリの機能にはナビゲーションやコントロール、様々なデバイスで適切に表示できるレイアウトなど、ユーザーなら誰もが期待するような基本的な機能と、そのアプリをユニークなものたらしめるような独自の、特徴的な機能の二つがあります。

SwiftUIは基本的な機能の実装にかける時間をより少なく、開発者が情熱を注ぐことができるアプリの特徴的な機能の実装によりたくさんの時間を注げるようになることをゴールとしています。

SwiftUI EssentialというセッションではそんなSwiftUIを支える仕組みを覗き見ることでSwiftUIについてより理解を深め、具体的なビュー、コントロールを使用して簡単なアプリケーションを実装しつつ、SwiftUIの根幹を支える概念や実装の意図が紹介されています。

## ViewBuilder

SwiftUIの基本的なオブジェクトは`View`です。Viewはstructで定義されていて、SwiftUIで実装したレイアウトで表示されているものは何らかの形でViewまで遡ることができます。そんなViewで階層構造を作りながらレイアウトを作っていくわけですが、SwiftUIがこれまでのフレームワークと異なるのはViewがコードで表現されるということです。

コードはViewをDSLライクな宣言的な構文で表現することができます。

```swift
VStack {
  Text(...)
  Image(..)
  Spacer()
  Text()
}
```

この構文を実現するためVstackのAPIではViewBuilderというattributeを使っています。

```swift
public struct VStack<Content: View> : View {
  public init(
    alignment: HorizontalAlignment = .center,
    spacing: Length? = nil,
    @ViewBuilder content: () -> Content
  )
}
```

Swift Compilerは、このattributeがついたクロージャを、スタック内のすべてのコンテンツを表す単一のビューを返す新しいクロージャに変換します。

このViewBuilderというattributeはFunction Builderという言語機能に支えられています。

### Function Builder

```swift
@_functionBuilder public struct Mirror {
    public static func buildBlock(_ first: String) -> String {
        return """
        入力された文字列
        1: \(first)
        """
    }
    
    public static func buildBlock(_ first: String, _ second: String) -> String {
        return """
        入力された文字列
        1: \(first)
        2: \(second)
        """
    }
    
    public static func buildBlock(_ first: String, _ second: String, _ third: String) -> String {
        return """
        入力された文字列
        1: \(first)
        2: \(second)
        3: \(third)
        """
    }
}

func mirror(@Mirror block: () -> String) -> String {
    return block()
}

let fuga = mirror {
    "a"
    "b"
}
// 入力された文字列
// 1: a
// 2: b


let foo = mirror {
    "a"
    "b"
    "c"
}

// 入力された文字列
// 1: a
// 2: b
// 3: c
```



## @Stateと $ prefix

```swift
VStack {
    Text("Avocado Toast").font(.title)
        Toggle(isOn: $order.includeSalt) {
        Text("include Salt")
    }
    Toggle(isOne: $order.includeRedPepperFlakes) {
        Text("Include Red. Pepper Flakes")
    }
    Stepper(value: $order.quantity, in: 1...10) {
        Text("Quantity: \(order.quantity)")
    }
  
    Button(action: submitOrder) {
        Text("Order")
    }
}
```

先頭のドル記号は、通常の値ではなく、値をバインディングしているしていることを示しています。
State attributeを使用してプロパティを宣言します。SwiftUIがこの属性でマークされたプロパティを見ると、その背後にある永続的な状態を自動的に作成および管理してから、このプロパティを通じてその状態の値を公開します。

`＄`prefixは、読み取り専用の値を渡すのではなく、Stateでそのquantityプロパティへのバインディングを渡す必要があることを示します。

バインディングは、あるビューが別のビューの状態を編集できるようにする一種の`a kind of managed reference`です。a kind of managed referenceのしっくりくる訳が思い浮かばなかったのでそのままにしています。

Stateとバインディング、アプリで使用する他のすべての種類のデータ、依存関係の管理方法を学びたい場合は以下のセッションをチェックすることを推奨されていました。この記事の次に以下のセッションを扱いたいと思います。

- [Data Flow Through SwiftUI - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/226/)
<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/e1eb9be34b2319c70ad5bcdf45c016cc.png" alt="" width="891" height="495" class="alignnone size-full wp-image-457542" />

このセッションを聴く上では、SwiftUIが背後で管理しているある種のデータ依存関係を表すstateのようなプロパティ属性があるということで十分です。

@Stateは宣言する度にSwiftUIのメモリ上に新しくアロケートされるため、初期値の設定が必要です。そのためプロパティの定義時にprivateを付与できるようなローカルで使用するデータのアクセスに適しています。

```swift
struct CustomForm: View {
  @State private var userData: UserData
  var body: some View {
    Stepper(value: $order.age, in: 1...100) {
      Text("Age: \(userData.age)")
    }
  }
}
```

## modifier

```swift
VStack {
  Text("hoge").font(.title)
}
```

上のようなコードにおける`font(_:)`のようなメソッドをSwiftUIでは`modifier`と呼びます。

modifierは既存のViewから新しいViewを生成するためのメソッドです。この例で言うと テキストに`font(_:)`modifierが追加されると、既存のテキストをラップする新しいビューが挿入されます。そして新しいビューは、そのテキストを新しいフォントでレンダリングするようにSwiftUIに指示します。また、modifierはメソッドチェインで追加できます。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/11d1bd00298b79deeff0499f7bcb1601.png" alt="" width="930" height="523" class="alignnone size-full wp-image-457543" />

メソッドチェインで追加されるとラップしたViewに新しいViewが挿入されます

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/fcbf17e030826f4f597564ef3993e32c.png" alt="" width="360" height="197" class="alignnone size-full wp-image-457544" />

UIkitでのUI構築では可能な限りビューの階層を小さくすることが良しとされてきました。アプリのパフォーマンスを最適化する方法の一つでした。

SwiftUIでは上記のスライドのようにテキストを複数のラッパービューでラッピングする必要があったとしても、SwiftUIがそれを背後で効率的なデータ構造に折りたたみ、そのデータ構造がレンダリングシステムで使用されます。

そのためビューの階層を深く、大きくしても問題ないと説明しています。

Viewの階層構造を深くすることのデメリットがSwiftUIでは解消されていることによって、開発者各々にとって最も理にかなった方法でビューのコードを編成することと、アプリケーションから最高のパフォーマンスを引き出すこととの間で妥協する必要がなくなりました。

### modifier chainのメリット

また、modifierを使ったこのような宣言的な構文のメリットは決定論的順序付けを視覚的に強制できることです。

ButtonのLabelクロージャを使って具体的な例を示します。

`Prefer smaller, single-purpose views`

```swift
struct ContentView: View {
    var body: some View {
        Button(action: hoge) {
            Text("hoge")
                .foregroundColor(Color.white)
                .background(Color.black, cornerRadius: 0)
                .padding(30)
        }
    }
    
    func hoge() {
        print("tapped")
    }
}
```

上記のコードは以下の画像のようになっています。

https://gyazo.com/663247c6b75abea27a37ed81690e39fd

青枠の線が`padding(_:)`のmodifier viewです。宣言通りの順番のViewの改装になっているので、 Textを`foregroundColor(_:)`のmodifier viewがラップして、それを`background(_:cornerRadius:)`のmodifier viewがラップしてそれを`padding(_:)`のmodifier viewがラップして、それをButtonがラップしているので、ボタンの領域が広がった感じになってます。

ボタンとしては不自然なので、タッチ可能な領域を黒背景にしてかつ文字と端がギリギリにならないようにpaddingを効かせたいとします。以下のように。

https://gyazo.com/9cd01a37cf4d58e04fd047771a389ade

Textのmodifierの順番を変更します。`padding(_:)`を一番上にすると`padding(_:)`のmodifier viewをその後のmodifier viewがラップする構造になります。すると画像のような見た目になります。

このようにSwiftUIはコードからViewの構造が分かるようになっています。

## View protocol

UIKitやAppKitに慣れている人だと、ビューは、protocolにconformtした構造体としてではなく、共通のスーパークラスから継承するサブクラスとして定義されることを知っているはずです。UIKitのカスタムビューはUIViewをスーパークラスとして継承しています。

UIViewは、alphaやbackgroundColorのような共通のビュープロパティのための領域を定義しています。UIKitでカスタムビューを作るとき、UIViewのプロパティを継承するだけでなく、独自のUI、動作のためにプロパティを新たに追加することが多いです。

SwiftUIでは、modifierで説明したコードと同じように、同じ種類の一般的なビュープロパティを別々のmodifierとして表しています。

これらのmodifierはそれぞれ独自のビューを生成します。つまりmodifierのための領域が、個々のビューごとに継承されるのではなく、これらの各modifier view内の階層全体に分散されます。

この仕組みによって必要な領域のみを確保して、Viewを軽量化し、目的に合わせて領域を最適化できます。また、全てのViewに共通の領域を提供する必要が無くなります。Viewがただのprotocolになった理由でもあります。

View protocolの役割はUIの一部を定義し、小さなViewを使って大きなViewを構築する仕組みを提供することです。

このprotocolにconformすることでViewの階層の一部を定義して、名前をつけてアプリ全体で再利用できるようにします。View protocolではbodyというcomuted propertyの実装が必要とされています。このbodyが別の種類のビューを返すように定義されています。

SwiftUIでは宣言的にViewを定義しますが、Viewは時間の経過とともに更新される永続的なオブジェクトではありません。入力の関数として宣言的に定義されているそれは、入力の一つが変更されるたびに再度bodyプロパティを呼び出して更新されたViewを取得しなおします。

```swift
struct OrderHistroy: View {
  let previosOrders: [CompletedOrder]
  
   var body: some View {
        List(previousOrders) { order in
            VStack(alignment: .leading) {
                Text(order.summary)
                Text(order.purchaseDate)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

スライドにあったコードを例にすると、previousOrdersコレクションが変更された場合、SwiftUIはリストの新旧のバージョンを比較し、変更内容に基づいて画面上にレンダリング結果を更新します。

例えばpureviousOrdersプロパティの値がクラウド上のデータがリアルタイムに反映されるものである場合、そして他のデバイスのアプリケーションからこのデータが追加されたり削除されたりする場合、プロパティの値が変わった時にレンダリングの結果もリアルタイムに変わっていきます。SwiftUIがコレクション内の変更を自動的に差分化し、挿入と削除を合成してから、適切なデフォルトのアニメーションを伴いながらレンダリングします。

## Composing controls & Navigating your app

後半のもう一人の方によるセッションではSwiftUIの具体的なオブジェクトを紹介しながら、それらのオブジェクトがどのように設計されていて、開発者が使うことによりどのようなメリットがあるか説明されています。

ToggleやStepperなどのコントロールは、見た目ではなく、目的や役割を表します。目的や役割を忠実に表すSwiftUIのこれらの部品は他のプラットフォームでも使用可能です。

コントロールの数は少なく抑えられていて、同時にOpaque Result Typeなどの仕組みによって必要な役割に応じて開発者がカスタマイズできるようにデザインされています。

セッションでは`Adaptability`と呼ばれていたこの柔軟性でアクセシビリティやダークモード、ローカライズなどがサポートされていることを具体的な例で紹介しています。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/00d3486306e02d337084120e7685de5a.png" alt="" width="854" height="499" class="alignnone size-full wp-image-457561" />

SwiftUIのコントロールは一度概念を学ぶと他のどこでも適用できます。同じコードをそのまま他のプラットフォームで実行できるわけではないですが、コアになる概念を学ぶと様々なコンテキスト、プラットフォームで使用できるようになります。

コントロールは目的、それらが果たす役割、アプリケーションのモデルとの関係に基づいて定義され、さまざまな文脈にわたって本質的に再利用可能であり、適切な外観がその文脈、プラットフォームまたは他の情報に基づいて決定されます。また同時に、ビューやラベルやオプションとしての使用や、システムで定義されたスタイルから任意のスタイルにカスタマイズすることも可能です。

後半では様々なビューやコントロールが登場しますが、SwiftUIチュートリアルに登場しなかったもののみサンプルのコードとともに紹介します。

### Form

```swift
var body: some View {
  VStack {
    Text(...)
    Toggle(...)
    Stepper(...)
  }
}
```

Formを使用したい場合はVStackの部分をFormに変更するだけで、プラットフォームに関係なく標準的なフォームの外観にしてくれます。

FormはVStackのようなコンテナですが、他のコントロールやセクションを構築するために特別に構築されたものです。コントロールやセクションを使ってFormを作ってみたコードが以下になります。値を変更できればよかったのでProfileというstructやプロパティの命名は気にしないでください。

```swift
struct ContentView: View {
    @State var profile: Profile
    var body: some View {
        Form {
            Section(header: Text("フォーム").font(.title)) {
                Text("Hello World").font(.title)
                Toggle(isOn: $profile.toggle) {
                    Text("value: \(String(profile.toggle))")
                }
                Stepper("stepper: \(profile.stepper)", value: $profile.stepper, in: 1...100)
            }
            Section {
                Button(action: submit) { Text("Submit") }
            }
        }
        
    }
    
    func submit() -> Void {
        print(profile)
    }
}
struct Profile {
    var name: String
    var stepper: Int
    var toggle: Bool
}
```

実行結果は以下のようになります。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/76c017c170a663c71cb7571d407553e0.png" alt="" width="437" height="931" class="alignnone size-full wp-image-457572" />

FormのContentクロージャ内に入れることで、全体的な背景や、各コントロールを分離する線、ボタンのスタイル設定などフォーム用のスタイルを勝手に整えてくれます。

### Picker

リストから選択するコントロールです。動きは以下のgif画像参照。

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/0d6994946002f0d64d00bc726a90642b.gif" alt="" width="388" height="538" class="alignnone size-full wp-image-457574" />

```swift
var body: some View {
    NavigationView {
        Form {
            Section(header: Text("フォーム").font(.title)) {
                 Picker(selection: $profile.country, label: Text("出身国")) {
                    Text(Country.unselected.rawValue).tag(Country.unselected)
                    Text(Country.usa.rawValue).tag(Country.usa)
                    Text(Country.korea.rawValue).tag(Country.korea)
                    Text(Country.china.rawValue).tag(Country.china)
                    Text(Country.japan.rawValue).tag(Country.japan)
                }
            }           
        }
    }
}

struct Profile {
    var stepper: Int
    var toggle: Bool
    var country: Country
}

enum Country: String {
    case unselected = "None"
    case usa = "USA"
    case korea = "Korea"
    case china = "China"
    case japan = "Japan"
}
```

ベタ書きで上記のように実装できますが、ForEachで書いた方が楽です。

enumをForEachに必要なprotocolにconformさせて、CaseIterableにconfomすると使用できるallCasesをForEachに渡します。

```swift
Picker(selection: $profile.country, label: Text("出身国")) {
    ForEach(Country.allCases) { country in
    Text(country.rawValue).tag(country)
        }
}

enum Country: String, CaseIterable, Hashable, Identifiable {
    var id: String {
        return self.rawValue
    }
    case unselected = "None"
    case usa = "USA"
    case korea = "Korea"
    case china = "China"
    case japan = "Japan"
}
```

### TabbedView

[UITabBarController ](https://developer.apple.com/documentation/uikit/uitabbarcontroller)のようなUIを提供してくれます

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/36e10d8b639498f5f52f3310959a529c.gif" alt="" width="424" height="898" class="alignnone size-full wp-image-457666" />

```swift
TabbedView {
    Form {
    }
    .tabItem {
        Text("1つ目の画面")
    }
    VStack {
        Text("2つ目の画面")
    }
    .tabItem {
        Text("2つ目の画面")
    }
}
```

### Environment

<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/b1a92821b57ff991fcd147a755672c40.png" alt="" width="922" height="490" class="alignnone size-full wp-image-457668" />

スライドのdisabled(_:) modifierはFormとそのForm内の全てのビュー、コントロールに有効です。状態を継承するようなこの振る舞いはSwiftUIの部品が値型のオブジェクトであることを踏まえると不思議な挙動です。

このような導入からEnvironmentの説明が始まります。

今までだとglobalなStateや値を取り出すために継承元の親オブジェクトを辿っていかなければいけなかったこのような状態の継承をEnvironmentという仕組みを使って実現しています。 

EnvironmentはViewヒエラルキーの親から継承されるようになっています。これによりViewの階層の中でコンテキストを共有できます。



```swift
struct ContentView : View {
    @Environment(\.isEnabled) var enabled: Bool
    var body: some View {
        Text("\(String(enabled))")
    }
}
```

@Environment attributeの付与されたプロパティの値を変更しようとすると`Cannot use mutating member on immutable value: 'enabled' is a get-only property`と警告が出ます。

セッションで使われていたスライドのこのイラストがEnvironmentの特徴をよく表していると思いました。
<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/07/c31ffac615888385907f2cf7f149aa7c.png" alt="" width="887" height="495" class="alignnone size-full wp-image-457670" />

このセッションでは簡単な説明とサンプルコードの紹介とセッションを通して作っているアプリにそれを使うのみでした。Data Flow Through SwiftUIのセッションに詳細は譲った形になっています。この記事でも、セッションで紹介された@Environment attrubuteのみの記述に留めます。

オススメのセッション

- Data Flow Through SwiftUI
- Building Custom Views with SwiftUI
- Integrating SwiftUI
- Accessibility in SwiftUI

## まとめ
記事で紹介した内容以外でもForEachはListのようにデータのコレクションと各データ項目をそれ自身のビューにマッピングするViewBuilderを取り、自身に視覚効果を追加せずに、コンテナにコンテンツを追加するだけと説明されていたり、SwiftUIチュートリアルを一通りこなすだけではわからないことがいくつも話題に出てきますので気になる方はぜひご自分で視聴されてください。

SwiftUI EssentialではさらにSwiftUIを理解するためにはどのようなセッションを観ていくのが良いかをセッション中やセッションの最後に紹介されています。次の記事ではそのすすめ通りにセッションを観たときのメモを記事として残していきたいと思います。

この記事はセッションの動画やドキュメントなどを参照し、実際にコードでSwiftUIを触りながら書きました。誤訳や誤読などによる認識のずれがあるかと思いますので、なにかお気づきの際は記事のコメントやTwitterなどからお知らせください。


