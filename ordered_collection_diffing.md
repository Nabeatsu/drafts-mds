Swift5.1から導入された機能の一つに`Ordered Collection Diffing`というものがあります。

WWDC2019では[Advances in Foundation](https://developer.apple.com/vidaeos/play/wwdc2019/723/)というセッションで紹介されています。

## Ordered Collection Diffingについて

Ordered Collection Diffingを使用すると2つのコレクションの差分を計算して何がどう変わっているのかを知ることができます。
この機能の主役になるのは`CollectionDifference<ChangeElement>`です。

実装はSwiftで行なわれているのでSwiftを書いたことがあればアルゴリズムの実装の部分を知らなくてもそれほど複雑に感じないAPIです。
CollectionDifferenceは差分を保持していて、差分はコレクションのどの要素が削除されてどの要素が追加されているのかを保持しています。

```swift
let stringArray = ["d", "b", "c", "a"]
let newArray = ["a", "b", "c", "d"]

let diff = newArray.difference(from: stringArray)

print(diff)

CollectionDifference<String>(
//    insertions: [
//        Swift.CollectionDifference<Swift.String>.Change.insert(offset: 0, element: "a", associatedWith: nil),
//        Swift.CollectionDifference<Swift.String>.Change.insert(offset: 3, element: "d", associatedWith: nil)
//    ],
//    removals: [
//        Swift.CollectionDifference<Swift.String>.Change.remove(offset: 0, element: "d", associatedWith: nil),
//        Swift.CollectionDifference<Swift.String>.Change.remove(offset: 3, element: "a", associatedWith: nil)
//    ]
//)
```

差分の内、追加の方はinsertionsプロパティ、削除の方はremovalsプロパティで保持しています。`CollectionDifference.Change`はinsertとremoveという2つのケースで表現されているenumです。

caseはassociated valuesとしてoffset、element、associatedWithを持っていますが、命名から想像ができる2つは説明を省いてassociatedWithについてのみ補足します。

associatedWithには要素を移動した時、insertの場合は移動後、removeの場合は移動前のオフセットが入ります。上記のサンプルコードではすべてnilになっていますが、これはこの値を取得しようと思うと計算コストが高くなってしまうのでデフォルトではnilです。nilではなくassociatedWithを取得したい場合は`inferringMoves()`メソッドを呼び出します。

```swift
print(diff.inferringMoves())

CollectionDifference<String>(
//    insertions: [
//        Swift.CollectionDifference<Swift.String>.Change.insert(offset: 0, element: "a", associatedWith: Optional(3)),
//        Swift.CollectionDifference<Swift.String>.Change.insert(offset: 3, element: "d", associatedWith: Optional(0))
//    ],
//    removals: [
//        Swift.CollectionDifference<Swift.String>.Change.remove(offset: 0, element: "d", associatedWith: Optional(3)),
//        Swift.CollectionDifference<Swift.String>.Change.remove(offset: 3, element: "a", associatedWith: Optional(0))
//    ]
//)
```

CollectionDifferenceは`Collection`protocolにconfomしているので、for-inでイテレートできます。並び順は順番に取得した要素を変化前のコレクションに適用すると変化後のコレクションになるように変化を再現できます。

```swift
for change in diff {
    switch change {
    case let .insert(offset, element, _):
        print("\(offset)番目に要素\(element)を追加")
    case let .remove(offset, element, _):
        print("\(offset)番目の要素\(element)を削除")
    }
}
```

Ordered Collection Diffingについて詳細に知りたい場合はソースコードを追いながら紹介している以下の記事が参考になると思います。

- [【Swift】Swift5.1の新機能 Ordered Collection Diffing入門 - Qiita](https://qiita.com/shiz/items/0e363219a0151d790d03)



## SwiftUIで使ってみる

宣言的な構文で記述できるUIフレームワーク SwiftUIは状態の変遷による最描画をフレームワークにまかせてレンダリングし続けることができるのが大きなメリットです。差分計算のユースケースはいくつか思い浮かびますが、今回はSwiftUIでOrdered Collection Diffingを使用してみたいと思います。


```swift
struct ContentView: View {
    
    @State private var models: [SampleModel] = []

    func update(with newModels: [SampleModel]) {
        let diff = newModels.difference(from: models)
        for change in diff {
            switch change {
            case .insert(let offset, let model, _):
                // 追加時に行いたい処理
            case .remove(let offset, let model, _):
                // 削除時に行いたい処理
            }
        }
        models = newModels
    }
    
    var body: some View {...} // modelsプロパティを元にリスト表示
    }
}    
```

サンプルコードはシンプルでユニークな文字列を保持しているだけのオブジェクトの配列をリスト表示するだけのものです。Ordered Collection Diffingはデータの追加、削除、Undoで使用しています。

実装も変更後と変更前のデータを元に差分を取り出し追加時と削除時の処理をイテレーションで行っているだけです。サンプルでは追加と削除のみ対応ですが移動にも対応させたい場合はそれぞれのケースでassociatedWithを取り出してnilじゃない時をハンドリングすれば良いです。


<img src="https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2019/12/e844832681f93f7d14611a775d7a1bae-3.gif" alt="" width="736" height="610" class="alignnone size-full wp-image-510066" />


```swift

struct ContentView: View {
    
    @State private var models: [SampleModel] = []
    @State private var backUpModels: [SampleModel] = []
    
    var apiClient: APIClient = SampleAPIClient()
    
    func getList() {
        apiClient.get(completion: { models in
            self.models = models
        })
    }
    
    func delete(at offsets: IndexSet) {
        var newModels = models
        newModels.remove(atOffsets: offsets)
        update(with: newModels)
    }
    
    func update(with newModels: [SampleModel]) {
        let diff = newModels.difference(from: models)
        for change in diff {
            switch change {
            case .insert(let offset, let model, _):
                apiClient.create(model, at: offset)
            case .remove(let offset, let model, _):
                apiClient.delete(model, at: offset)
            }
        }
        backUpModels = models
        models = newModels
    }
    
    func undo(with backUpModels: [SampleModel]) {
        update(with: backUpModels)
        self.backUpModels = []
    }
    
    var body: some View {
        return VStack {
            List {
                ForEach(models, id: \.self) { model in
                    SampleRow(model: model)
                }
                .onDelete(perform: delete(at:))
            }
            HStack {
                Button(action: {
                    self.getList()
                }) {
                    Text("デフォルトデータ作成")
                }.disabled(!models.isEmpty)
                
                Button(action: {
                    var newModels = self.models
                    newModels.append(SampleModel())
                    self.update(with: newModels)
                }) {
                    Text("追加")
                }
                Button(action: {
                    self.update(with: [])
                }) {
                    Text("全削除")
                }.disabled(models.isEmpty)
                
                Button(action: {
                    self.undo(with: self.backUpModels)
                }) {
                    Text("Undo")
                }.disabled(backUpModels.isEmpty)
            }
            Spacer()
        }
    }
}

struct SampleRow: View {
    var model: SampleModel
    var body: some View {
        Text(model.id)
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct SampleModel: Identifiable, Hashable {
    var id = UUID().uuidString
}

protocol APIClient {
    func create(_ data: SampleModel, at: Int)
    func delete(_ data: SampleModel, at: Int)
    func get(completion: @escaping ([SampleModel]) -> Void)
}

struct SampleAPIClient: APIClient {
    func create(_ data: SampleModel, at: Int) {
        print("\(at)番目に\(data.id)を追加しました")
    }
    
    func delete(_ data: SampleModel, at: Int) {
        print("\(at)番目の\(data.id)を削除しました")
    }
    
    func get(completion: @escaping ([SampleModel]) -> Void) {
        completion(
            Array.init(repeating: SampleModel(), count: 10)
        )
    }
}

```

堅牢な差分アルゴリズムを書くのは簡単なことではないです。その部分を標準ライブラリに丸投げして実装するのは変更のタイミングで行いたい処理を記述するだけです。SwiftUIは状態の変更に合わせてViewを再描画するので記述は更に少なくなります。

### まとめ

実際には非同期なAPIリクエストを行ってその結果を待ち受けてハンドリングしないといけない方が多いと思うのでこんな単純な実装にはならないですが、今までだと自前でアルゴリズムを実装するかライブラリを利用して実装していた部分で標準ライブラリの堅牢で効率の良い実装を利用できるようになるのはありがたいと感じました。

ソースコードを見るとCorrectionDifferenceの初期化はMyerのdiffアルゴリズムを元に実装されています。差分アルゴリズムについては以下の記事が面白かったです。

- [diffの動作原理を知る～どのようにして差分を導き出すのか：一般記事｜gihyo.jp … 技術評論社](https://gihyo.jp/dev/column/01/prog/2011/diff_sd200906)
- [差分検出アルゴリズム三種盛り - Object.create(null)](https://susisu.hatenablog.com/entry/2017/10/09/134032)

Swift5.1ではOrdered Collection Diffingに加えてDiffable DataSourcesなど、これからのアプリ開発が変わりそうな重要な機能がいくつも導入されています。

またそれらの機能を扱うタイミングで記事にできればと思います。



