# Understanding ARKit Tracking and Detectionセッションレポート

かねてからARに関心があったのと、ARを学び始めるのに都合の良い状況になったので1からARKitを初めてみることにしました。さて、WWDCにUnderstanding ARKit Tracking and Detectionというセッションがあります。このセッションではARKitの概要やARの基礎的な事項についてTrackingとDetectionを中心に解説されます。今回の記事では約1時間ほどあるこのセッションのレポートをしてみようと思います。

- [Understanding ARKit Tracking and Detection - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/610/)



## レポート

### Trackingとは

現実世界においてカメラが認識すべき位置を提供します。するとカメラ越しに仮想のコンテンツを付与できます。仮想コンテンツは常に正確な配置、サイズ、見え方で表示されます。

多様なTracking技術がカメラに参照用のシステムを提供します。これはカメラが現実世界や情報を捉えることを意味する。

### Basics of ARKit 

ARKitに内蔵された入力システムについて

- ARSessionはオブジェクトでAR技術の構成から稼働まですべてを処理して、AR技術の処理結果も返します。
- ARConfigurationを取得してARSession上のメソッドを呼び出します。
- するとARSessionはAVCaptureSessionの構成を開始、画像の受信を始めます。
  - CMMotionManageも同様に構成されます。
- 処理結果はARFrameに毎秒60フレームで返る
- ARFrameはレンダリングに必要な情報を与えてくれます 
  - 例: カメラが捉えた画像、追跡した情報、検知した平面の情報などの環境情報も含まれる

### Orientation Tracking(位置特定)、World Tracking(方向特定)、Plane Detaction(平面検出)

#### Oriaentation Tracking

回転だけを追跡することを表す(3 DoF)

- 首を動かすだけで仮想コンテンツを追跡する
- この場合同じ位置の仮想コンテンツは取得できても違う位置からのものはできません
- 3つの軸の動きだけを追跡します。3DOFの由来
- 球状の仮想環境で活用できます
  - 360度動画
  - 仮想コンテンツを同じ位置から見られます。
  - 遠くにあるものを拡大するためにも使えます
  - 様々な角度で見たい場合にはorientation Trackingによる拡大は適切ではありません。

#### Orientation Trackingを実行中の内部の処理を説明します

https://gyazo.com/87919c57ac5ba6c6187f1a5a74fedaa1

Core motionによる回転データだけを使用してモーションセンサーのデータに適用します。
モーションセンサの動作データはカメラの画像よりも早く提供されます。そのため画像が有効になってから最新の動作データを取得します。そして双方の結果を返します。
カメラフィードはOrientation Trackingで処理されません。コンピュータビジョンがないのです。Orientation Tracking処理を実行するためにはARSessionを構成する必要があります。(AROrientationTrackingConfiguration)

処理結果はARFrameから提供されたARCameraに返ります。

```swift
open class ARCamera: NSObject, NSCopying {
  open var transform: simd_float4x4 { get }
  open var eularAngles: simd_float3 { get } 
}
```

transformはOrientation Trackingにおいては回転データのみが含まれます。カメラが追跡したデータです。eularAnglesでも代用が可能です。最適な方を使ってください。

#### World Tracking(方向特定)

カメラビューの位置や位置の変更を追跡し現実世界に反映
現実世界に関する事前の情報は不要です。


#### ARKit2

- マップの保存、ロード
- 画像追跡
- 物体検出





