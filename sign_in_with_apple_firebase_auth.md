11月にfirebase-ios-sdkにSign in with Apple対応がマージされました。


Sign in with Appleは認証にApple IDを利用できる機能です。Firebase Authenticationを使ってこのSign in with Appleを使ってみます。

事前準備

IdentifiersでSign in with Appleを有効にします。
![signin](https://gyazo.com/1575f2e2e817b365c5c6d1b068459e76.png)

![https://gyazo.com/5c16f78b06b76770dd22fe666dc1ff70](https://gyazo.com/5c16f78b06b76770dd22fe666dc1ff70.png)

### 書かないといけないこと

- 登録とログイン
- 既存ユーザーのステータス確認
- ログアウトとログインの繰り返し
