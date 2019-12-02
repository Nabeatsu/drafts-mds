# DependencyInjection と DI コンテナを使ってテストをしやすいクライアントを書く下書き

こんにちは。モバイルアプリサービス部の田辺です。
singleton を採用しない例として代わりに DependencyInjection を使用する場合があります。この記事では Dependency Injection を使用してテストがしやすい状態を作ってみたいと思います。

## Dependency Injection と DI コンテナ

Dependency Injection(依存性の注入)は依存オブジェクトを外部からクライアントに注入することでオブジェクトの生成と使用を分離するデザインパターンの一つです。

オブジェクトを取りに行くのではなく、オブジェクトを与えられることでクライアントが動作します。制御関係が反転していることから IoC(Inversion of Control)(制御の反転)などと表現されることもあります。

Swift では、protocol を使って抽象を表現し、それを満たすオブジェクトを後からクライアントに注入するようにすれば、差し替えも可能になります。

とはいえ、クライアントに注入する依存オブジェクトを外部から注入する時に、依存関係が複雑な場合、オブジェクトを生成するコードも複雑になります。複雑になりそうな Dependency Injection をまとめるために DI コンテナというものが使われることがあります。

複数個の依存オブジェクトを一つのコンテナで管理するもので、Dependency Injection と合わせて説明されることが多いです。

あるクライアントの依存オブジェクトがまた別のオブジェクトに依存していて、このオブジェクトはまた別のオブジェクトに依存性があって…

このような複雑さを解決する一つの手段として DI コンテナにこの辺をお任せするパターンがあります。

## singleton

有名なデザインパターン(GoF の 23 のデザインパターン)の一つで、インスタンスが 1 つしか生成されないことを保証します。`shared`や`sharedInstance`などの命名で定義されているのをよく目にします。

```swift
class SingletonHoge {
    static let shared = SingletonHoge()
    private init() {}
}
```

Swift では上記のように static let と initializer を private にすることで容易に singleton が実現出来ます。

有効な場合ももちろんありますが、単体テストではクラスごとメソッドごとにテストコードを書きます。そのためにはオブジェクト単体で動く必要があります。

ファイルシステムやネットワーク云々はテストの際は邪魔になってしまうため、テストコードではネットワーク部分をモックなどを使ってオブジェクト差し替えを行いテストしやすいコードを書くことが多いです。

例えば、外部 API へのアクセスをテストコードの中で行ってしまうと外部 API に何らかのトラブルが起こった時に何も変更していないのにテストが突然失敗する事象が起きたりします。この時オブジェクト差し替えを容易にする一つのパターンとして DI がよく扱われます。

## singleton なクライアントをテストしやすく変更してみる

GitHub の Zen API にリクエストを投げて文字列を取得する ZenUseCaseAPI というクラスを singleton で作ってみます。
このクラスは singleton でなおかつ文字列を取得してくる get()メソッドが外部のクラスに依存しています。依存しているのは ZenAPIClient というクラスです。

ZenUserCase クラスのインスンタンスは get メソッドを呼ぶと text プロパティと isRequested の値が更新されます。そもそもテストする必要あるのかってぐらい単純なんですがもっと良くてなおかつコードが少なくて済む例が浮かばなかったです…

singleton で変数をプロパティに持っているこのクラスは状態を持ちます。値が更新されうるので前回のテストケースの影響を受けます。

```swift
class ZenUseCase {
    static let shared = ZenUseCase()
    private init() {}
    var text: String?
    var isRequested: Bool = false
    func get() {
        let client = ZenAPIClient()
        client.get() { string in
            self.text = string
            self.isRequested = true
        }
    }
}

class ZenAPIClient {
    struct ZenAPI: Codable {
        let text: String
    }
    func get(_ completionHandler: @escaping (String) -> Void) {
        guard let url = URL(string: "https://api.github.com/zen") else { return }
        let request = URLRequest(url: url)
        let session = URLSession.shared
        session.dataTask(with: request) { ( data, response, error) in
            if let data = data {
                let decoded = try! JSONDecoder().decode(ZenAPI.self, from: data)
                completionHandler(decoded.text)
            }
        }
    }
}
```

例えばこのようなテストは失敗してしまいます

```swift
func testZenAPI1() {
        let expectation = self.expectation(description: "ZenAPIへのリクエスト")
        let zenUseCase = ZenUseCase.shared
        zenUseCase.get() {
            print(zenUseCase.text!)
            XCTAssertEqual(zenUseCase.isRequested, true)
            expectation.fulfill()
        }
        self.waitForExpectations(timeout: 15)
    }

    func testZenAPI2() {
        let zenUseCase = ZenUseCase.shared
        XCTAssertEqual(zenUseCase.isRequested, false) // こちらのテストケースではまだAPIリクエストをしていない。状態を持っっているのでその状態を考慮しないといけない。
    }
```

`testZenAPI1`で変更された singleton のインスタンスのプロパティの値が更新されていて`isRequested`は false ではなくなっています。テストケース`testZenAPI2`の中では API リクエストはしていないので、他のテストケースに影響を受けるのは好ましくありません。

また、そもそも`get()`メソッド自体他のクラスに依存しているので単体テストとしてもよくないです。

## 参考・関連リンク

[Swift におけるシングルトン・static メソッドとの付き合い方 – Swift・iOS コラム – Medium](https://medium.com/swift-column/singleton-398078bcc58d)

djfsdfklsdjlf

