# Sign in with Apple 実装でハマったこと

業務で 2 度、業務以外で 1 度 Sign in with Apple を実装しました。Apple での認証後連携する先が Firebase だったりサーバーサイドだったりする中で少しずつ曖昧な所が整理されてきました。実装の中で気をつけた所、ハマった所について整理したいと思います。

## 用語の整理

Sign in with Apple には OAuth Client に紐づく App ID、Service ID、Primary App ID、Key などがあります。iOS だけで Sign in with Apple を実装する際は気にする必要はないものの、Android や Web での認証も考慮に入れる場合に必要になるものもこの中には含まれているのでそれぞれの用語をきちんと理解した上で実装の前準備を進める必要があります。

## サーバーサイドでの検証方法について

ID Token を使った検証方法と Authorization Code を使った検証方法があります。アプリ側から認証情報をセットしてアカウント登録・ログインのリクエストを投げる先の仕様によって渡すべきものも変わってきます。それぞれのフローについてドキュメントを読んでおく必要があります。

また、サーバーサイドの実装についても簡単に知っておくと担当者同士でのコミュニケーションもやりやすく確認しておく事項も明確になると思います。

## Enterprise Developer Program で Sign in with Apple がサポートされていない

Capabilities にて Enterprise 用の TARGET では Sign in with Apple を有効にしないように設定

## 認証後最初だけしか取れない情報がある

Apple が提供している Sign in with Apple のサンプルコードに KeyChain で内部に保存している処理があるのですがそれはこの仕様に対応するために行っています。

## ボタンをカスタムする場合ののロゴについて

画像の加工は NG と明記されているのでマスクの処理を自分でかかないといけません。
