Combine

時間を超えて値を処理するための統一された宣言型のAPI

- Generic
- Type safeコンパイル時にエラーが検出できる
- composition first
- Request-driven
- 

### Composition first

コアとなる概念は単純で理解しやすいということですが、それらをまとめると、その部分の合計以上のものを作ることができます。

## CoreConcept

重要な概念は３つ

- Publishers
- Subscribers
- Operators

### Publisher

Combine frameworkの宣言部分。

Publisherは値とエラーがどのように生成されるかを記述します。実際にその値やエラーを生み出すものではないです。

そしてSubscriberが登録できます。時間を超越して値を受け取ることができます。