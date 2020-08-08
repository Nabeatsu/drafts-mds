URLComponentsはその構成要素からURLを構造化されたURLにパースする構造体です。

この構造体はRFC 3986に従ってURLを解析して構築します。この構造体の動作は、古いRFCに準拠したURL構造体とは微妙に異なります。しかし、URLComponentsの値の内容に基づいてURLの値を簡単に取得することができます。

％エンコードなどの責務を開発者が負うことなくシステムに委ねる

URLComponentsはURLをComposeする時だけではなくDecomposeする時にも役立つ

URLComponentsはStringかURLから生成できます。どちらもfailable initializerなので戻り値は`Optional<URLComponents>`です。

URLからURLCoponentsを生成する場合の注意点。init?(url:resolvingAgainstBaseURL:)のドキュメンテーションコメントからの引用です。

> If resolvingAgainstBaseURL is true and url is a relative URL, the components of url.absoluteURL are used. If the url string from the URL is malformed, nil is returned.

```swift
let urlString = "https://showsknownfrom.tv/search?q=Sopranos&order=desc"

guard var urlComponents = URLComponents(string: urlString) else {
    return nil
}
```

URLCoponentsの公開されているプロパティの一つにqueryItemsがあります。これはURLQueryItemオブジェクトの配列を返してくれます。URLQueryItemは基本的に個々のクエリ項目のkey/value オブジェクトです。URLQueryItemのドキュメントは[こちら](https://developer.apple.com/documentation/foundation/urlcomponents/1779966-queryitems)です。

実際にURLCoponentsからURLを生成してみます。

```swift
func searchURL(with q: String, optionalParameters: [String: String] = [:]) -> URL? {
    let urlString = "https://showknownfrom.tv/search"
    guard var urlComponents = URLComponents(string: urlString) else { return nil }
    
    var queryItems: [URLQueryItem] = [URLQueryItem(name: "q", value: q)]
    let optionalURLQueryItems = optionalParameters.map { URLQueryItem(name: $0, value: $1) }
    queryItems.append(contentsOf: optionalURLQueryItems)
    urlComponents.queryItems = queryItems
    return urlComponents.url
}
```

最初にurlStringを使用して新しいURLComponentsオブジェクトを作成します。(後にそれに書き込む必要があるので、urlComponentsをvarとして作成することに注意してください)。

次に URLQueryItem オブジェクトの配列を作成し、唯一必要なパラメータである q パラメータを追加します。

次に optionalParameters 辞書をループし、要素を URLQueryItem オブジェクトにマッピングし、既存の配列に追加します。

最後に、URLComponents に URL を返すように依頼します。

シンプルで美しい、すべてのハードワークはURLComponentsに委ねられており、私たちはそれ以上のことを気にする必要はありません。

Decomposeの例

```swift
func receiveURL(_ url: URL, contains parameter: String) -> String? {
    guard let urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true),
        let queryItems = urlComponents.queryItems else { return nil }
    return  queryItems.filter { $0.name == parameter }.first?.value
}
```

```swift
//test
if let url = URL(string: "https://showsknownfrom.tv/shows/12345678?featured=true"),
   let parameter = receivedUrl(url, contains: "featured") {
    print("found \(parameter)")
} else {
    print("not found")
}
```

extensionを定義するともっと簡単になる

```swift
extension URL {
  func append(queryParameters: [String: String]) -> URL? {
      guard var urlComponents = URLComponents(url: self, resolvingAgainstBaseURL: true) else {
          return nil
      }

      let urlQueryItems = queryParameters.map {
          return URLQueryItem(name: $0, value: $1)
      }
      urlComponents.queryItems = urlQueryItems
      return urlComponents.url
  }

  func value(forParameter name: String) -> String? {
    guard let urlComponents = URLComponents(url: self, resolvingAgainstBaseURL: true),
        let queryItems = urlComponents.queryItems else {
            return nil
    }
    let items = queryItems.filter { $0.name == name }
    return items.first?.value
  }
}
```

使用方法

```swift
if let url = URL(string: "https://showsknownfrom.tv"),
   let appendedURL = url.append(queryParameters: ["q": "Halt and Catch Fire", "order" : "desc", "number": "100"]) {
        print(appendedURL) //gives us: https://showsknownfrom.tv?q=Halt%20and%20Catch%20Fire&number=100&order=desc
}

if let url = URL(string: "https://showsknownfrom.tv/shows/12345678?featured=true&test=1"),
   let value = url.value(forParameter: "featured") {
     print("found \(value)") //gives us: found true
}
```



## URLComponentsを使わない場合の実装

## URLComponents

### URLComponetsを使った実装

### URLComponentsの注意点

URLと基準にしているRFCが違う

- [SwiftでURLComponentsを使用して '+'をエンコードする](http://www.366service.com/jp/qa/b63ff6fa3d0f39a79172c9e721abed52)

## エンドポイントの抽象化

## まとめ

