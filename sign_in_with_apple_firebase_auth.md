11月にfirebase-ios-sdkにSign in with Apple対応がマージされました。


Sign in with Appleは認証にApple IDを利用できる機能です。

業務で関わっているアプリでは外部サービス認証をしているので将来的にSign In With Appleにも対応しないといけないです。

そこでbetaではありますがFirebase AuthenticationがSign in with Appleに対応したので試してみました。

実装の大きな流れです。

- Identifiersで該当のIdentifierの更新
- Provisioning Profileの更新
- XcodeのCapabilityでSign In With Appleの追加
- Firebase consoleからSign In With Appleが行えるよう準備
- Sign In With Appleのリクエスト
- Delegate protocolのメソッドを通したユーザーの操作のハンドリング
- 認証情報からFirebase Authenticationの認証
- 認証情報を設定画面から削除された場合のハンドリング




## 事前準備

[Certificates, Identifiers & Profiles - Apple Developer](https://developer.apple.com/account/resources/certificates/list)のIdntifiersで任意のIdentifierのCapabilitiesからSign in with Appleを有効にします。

![signin](https://gyazo.com/1575f2e2e817b365c5c6d1b068459e76.png)


Identifierを更新した後はProvisioning profileを更新します。

Xcodeでプロジェクトの設定の「Signing & Capabilities」タブを開きCapabilityをクリックして「Sign In with Apple」を有効にしておきます。

![a](https://gyazo.com/c5aca529fcd03e66723c18dbbc46546c.png)

Sign In With Appleが追加されているのが確認できます。

![a](https://gyazo.com/e3daee97399c3cddf247f94065b6dffc.png)


## ボタンの表示まで

アカウントのサインアップとサインインのユーザー体験については[Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/sign-in-with-apple/overview/)のDesigning Account Setup and Sign-Inという項に記載があります。

ボタンのデザインについても上のリンクのSign in with Apple Buttonsに記載があります。

Human Interface Guidelineでも提供されているボタンを使うのが推奨されているのでそれに従うことにします。

提供されているボタンをコード内で使用するにはAuthenticationServices frameworkが必要なので`import AuthenticationServices`する必要があります。

- [AuthenticationServices | Apple Developer Documentation](https://developer.apple.com/documentation/authenticationservices)

importするとASAuthorizationButtonが使えるようになるのでセットアップしていきます。

ASAuthorizationAppleIDButtonのドキュメントにはcorner radiusの変更はできるがボタンのスタイルは変更しないように明記されていたので今回も特にスタイルは変えずに使用します。

- [ASAuthorizationAppleIDButton - AuthenticationServices | Apple Developer Documentation](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidbutton)


またドキュメントに記載されている通りタップ時にプロバイダを通してリクエストを作成し、ASAuthorizationControllerのインスタンスを使用してリクエストを投げるように実装します。

そして今回ログインには他のサービスも対応しているのでクラスやメソッドにavailable attributeを指定するのではなく `if #availabe(iOS 13.0, *)`を使用して表示を切り替えます。

```swift
import AuthenticationServices

overide func viewDidLoad() {
    super.viewDidLoad()


    if #available(iOS 13.0, *) {
        let appleLoginButton = ASAuthorizationAppleIDButton(
                authorizationButtonType: .default,
                authorizationButtonStyle: .whiteOutline
        )
        
        appleLoginButton.addTarget(
            self,
            action: #selector(handleTappedAppleLoginButton(_:)),
            for: .touchUpInside
        )

        // 制約などの定義やaddSubViewなので省略
    }
}

@available(iOS 13.0, *)
    @objc func handleTappedAppleLoginButton(_ sender: ASAuthorizationAppleIDButton) {
        requestSignInWithAppleFlow()
}

```

表示されたボタン

![https://gyazo.com/5c16f78b06b76770dd22fe666dc1ff70](https://gyazo.com/5c16f78b06b76770dd22fe666dc1ff70.png)


最後のrequestSignInWithAppleFlow()が

> プロバイダを通してリクエストを作成し、ASAuthorizationControllerのインスタンスを使用してリクエストを投げるように実装します。

この部分を実装しています。

プロバイダはASAuthorizationAppleIDProviderでcreateRequest()というインスタントメソッドでrequestを生成します。
型はASAuthorizationOpenIDRequestというクラスです。

ASAuthorizationOpenIDRequestにはnonceプロパティが定義されています。

クライアントセッションとIDトークンを関連付けるために使用される文字列でリプレイ攻撃を対策に使用するようです。認証リクエスト中でのみ使用します。

- [ノンス - Wikipedia](https://ja.wikipedia.org/wiki/%E3%83%8E%E3%83%B3%E3%82%B9)
- [反射攻撃 - Wikipedia](https://ja.wikipedia.org/wiki/%E5%8F%8D%E5%B0%84%E6%94%BB%E6%92%83)

Sign in with Appleの認証がどのような流れで行なわれるか解説したドキュメントにもnonceに関する言及があります。

- [Authenticating Users with Sign in with Apple | Apple Developer Documentation](https://developer.apple.com/documentation/signinwithapplerestapi/authenticating_users_with_sign_in_with_apple)


また、Firebase側のドキュメントでもnonceに関する言及があります。こちらではnonce生成のサンプルコードも用意されています。

- [iOS で Apple を使用して認証する  |  Firebase](https://firebase.google.com/docs/auth/ios/apple#sign_in_with_apple_and_authenticate_with_firebase)

```swift
    func requestSignInAppleFlow() {
        let nonce = randomNonceString()
        currentNonce = nonce
        let request = ASAuthorizationAppleIDProvider().createRequest()
        request.requestedScopes = [.fullName, .email]
        request.nonce = sha256(nonce)
        let controller = ASAuthorizationController(authorizationRequests: [request])
        controller.delegate = self
        controller.presentationContextProvider = self
        controller.performRequests()
    }

    private func randomNonceString(length: Int = 32) -> String {
        // 実装は自由
        https://auth0.com/docs/api-auth/tutorials/nonce#generate-a-cryptographically-random-nonce を参照するか
        https://firebase.google.com/docs/auth/ios/apple#sign_in_with_apple_and_authenticate_with_firebase のサンプルコードを流用するか

    }
```

requestedScopesプロパティで取得したい情報を指定します。

```swift
extension ASAuthorization.Scope {
    public static let fullName: ASAuthorization.Scope
    public static let email: ASAuthorization.Scope
}
```

今回はfullnameとemailを要求することにしました。


## 認証

また、上記のコードにもある通り、ASAuthorizationControllerのインスタンスにはdelegateプロパティとpresentationContextProviderプロパティがあります。それぞれASAuthorizationControllerDelegate、ASAuthorizationControllerPresentationContextProvidingに準拠したインスタンスを代入します。 
今回はログイン画面のViewControllerに準拠させてリクエスト後のハンドリングなどを行います。

ASAuthorizationControllerDelegateに準拠することでリクエストに対するレスポンスを受け取れます。ASAuthorizationControllerPresentationContextProvidingへの準拠は認証を行う画面を表示するために必要です。


```swift

@available(iOS 13.0, *)
extension LoginViewController: CredentialWithAppleMakeable & ASAuthorizationControllerDelegate & ASAuthorizationControllerPresentationContextProviding {

    func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        // 認証完了時のハンドリング
        handle(authorization.credential)
    }

    func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
        // エラーハンドリング
    }

    func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
        // ASAuthorizationControllerPresentationContextProvidingのメソッド
        return view.window!
    }

    func handle(_ credential: ASAuthorizationCredential) {
        // エラー時のハンドリングはアプリごとに違うので省略
        guard let appleIDCredential = credential as? ASAuthorizationAppleIDCredential else { 
            // キャスト失敗
            return
        }
        guard let nonce = currentNonce else {
            // ログインリクエスト失敗
            return
        }

        guard let appleIDToken = appleIDCredential.identityToken else {
            print("Unable to fetch identity token")
            // ユーザーに関する情報をアプリに伝えるためのJSON Web Tokenの取得に失敗
            return
        }

        guard let idTokenString = String(data: appleIDToken, encoding: .utf8) else {
            // JWTからトークン(文字列)へのシリアライズに失敗
            return
        }

        // Sign In With Appleの認証情報を元にFirebase Authenticationの認証
        let oAuthCredential = OAuthProvider.credential(
            withProviderID: "apple.com",
            idToken: idTokenString,
            rawNonce: nonce)

        Auth.auth().signIn(with: oAuthCredential) { (authResult, error) in
            if (error != nil) {
                return
            }
            self.completeSigningInWithApple()
            // 今回のアプリでは認証完了
            // FireStore側に初期データを保存する処理を呼んだ後に前画面に戻る
            self.navigationController?.popViewController(animated: true)
        }
    }
}
```

今回リクエストに成功した場合はauthorizationController(controller:didCompleteWithAuthorization:)
でASAuthorizationAppleIDCredentialが取得できるはずです。

そのハンドリングをアプリの仕様に合わせてFirebase Authentication側の認証を行います。

## 認証情報が変更された場合のハンドリング
