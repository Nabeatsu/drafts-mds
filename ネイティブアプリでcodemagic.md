

Flutter専門のCI/CDサービスとして開始されたCodemagicはアップデートによりネイティブアプリでも同等に扱えるようになりました。この記事では`codemagic.yaml`ファイルを使ったネイティブアプリのCI/CI設定方法を説明します。

UIで直接AndroidやiOSのネイティブアプリをビルドすることはできませんが、ネイティブアプリをビルドするために`codemagic.yaml`を使って簡単に設定することができます。

## 導入

1. [Codemagic - CI/CD for Flutter and mobile app projects](https://codemagic.io/start/)にログインします。
2. Applications overviewで任意のプロジェクトを選択（ここではネイティブのiOSアプリのプロジェクトを選択してください）します。そして`Set up build`を選択してください。
3. Select a starter workflowページに遷移するので、Continuous Integration workflowsのiOS Appを選択します。

![21a10c983697a3d2f4750fd6d30b33bf](/Users/tanabe.nobuyuki/Documents/drafts/images/21a10c983697a3d2f4750fd6d30b33bf.png)

4. 初めてのネイティブのアプリでのCodemagicの利用になる場合には手短なガイドモーダルが表示されます。

![9e0f7ac1ed396a2b28b4c8f637e4f6a9](/Users/tanabe.nobuyuki/Documents/drafts/images/9e0f7ac1ed396a2b28b4c8f637e4f6a9.png)

### デフォルトのYAMLファイルについて

ガイド用のモーダルは合計で4ステップあり、それに従い設定していくことでbuildのみを行える環境の構築が可能です。`Download or copy the configuration file example`ではプロジェクトのルートディレクトリに設置する設定ファイルの雛形が提供されます。yamlファイルのコメントにある通り、iOSネイティブアプリ用のyamlファイルについての詳しい情報が知りたい場合は[Building a native iOS app - Codemagic Docs](https://docs.codemagic.io/yaml/building-a-native-ios-app/)を参照します。

また、テンプレートの`{}`部分を適した値に更新するように指示されています。ダウンロードが良いと思いますが、今回は説明も兼ねているのでクリップボードにコピーして新規ファイル作成します。以下のようなコードになります。

```yaml
# Check out https://docs.codemagic.io/yaml/building-a-native-ios-app/ for more information
# Please review and update values in curly braces

workflows:
    ios-app:
        name: iOS App
        environment:
            vars:
                XCODE_WORKSPACE: "{{ ADD WORKSPACE NAME HERE }}"
                XCODE_SCHEME: "{{ ADD SCHEME NAME HERE }}"
            xcode: latest
            cocoapods: default
        scripts:
            - xcodebuild build -workspace "$XCODE_WORKSPACE.xcworkspace" -scheme "$XCODE_SCHEME" CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
        artifacts:
            - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.app
            - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.dSYM
```

デフォルトのワークフローIDとワークフロー名が指定されていることがわかりますが、好きなように変更することができます。ワークフローはXcodeとCocoaPodsの両方の最新バージョンを使用しています。

プロジェクトに合った正しいフォルダ構造とツールのバージョンを選択してください。ファイルには、依存関係、変数、その他の関連データを必ず含めてください。

プロジェクトのニーズに合わせて codemagic.yaml を調整し終わったら、プロジェクトリポジトリのお好みのブランチにプッシュしてください。

値を更新したyamlファイルは以下になります。

```yaml
# Check out https://docs.codemagic.io/yaml/building-a-native-ios-app/ for more information
# Please review and update values in curly braces

workflows:
  ios-app:
    name: iOS App
    environment:
      vars:
        XCODE_WORKSPACE: "CodeMagicTestSample"
        XCODE_SCHEME: "FileterHB"
      xcode: latest
      cocoapods: default
    scripts:
      - xcodebuild build -workspace "$XCODE_WORKSPACE.xcworkspace" -scheme "$XCODE_SCHEME" CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
    artifacts:
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.app
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.dSYM

```



![bd4a8983d32534e322d9966bf67cfbcc](/Users/tanabe.nobuyuki/Documents/drafts/images/bd4a8983d32534e322d9966bf67cfbcc.png)

![258604dbd5c0f2e19e42c2a3b110668d](/Users/tanabe.nobuyuki/Documents/drafts/images/258604dbd5c0f2e19e42c2a3b110668d.png)

変更をブランチにプッシュして問題ないことを確認した後、モーダルの最後のステップ、`Check for uploaded configuration file`で、`Check for configuration file`をクリックします。

![5861194fa2723c45fb62a4163f20d7fa](/Users/tanabe.nobuyuki/Documents/drafts/images/5861194fa2723c45fb62a4163f20d7fa.png)

codemagic.yamlファイルが任意のブランチに存在することが確認できたらダイアログが表示されるので`Start your first build`をクリックします。

![9a69316400159eb2333d375eccf3bf87](/Users/tanabe.nobuyuki/Documents/drafts/images/9a69316400159eb2333d375eccf3bf87.png)

クリックするとbuild configurationの設定画面に遷移します。

![85b30e15d4dd831420909bba67e5792f](/Users/tanabe.nobuyuki/Documents/drafts/images/85b30e15d4dd831420909bba67e5792f.png)

ビルドが始まったら結果が反映されるまで待機します。

![765e9d6358fd236984265dedfabace3f](/Users/tanabe.nobuyuki/Documents/drafts/images/765e9d6358fd236984265dedfabace3f.png)

このプロジェクトでは数分後に成功したことを確認しました。

ここまでで`.app`ファイルを生成するworkflowを通したネイティブアプリのビルドは終了です。

#### コードサイニング

`.ipa`ファイルを生成するにはcode-signingが必要になります。

- Certificate
- Provisioning profile

個人利用のためにipaファイルをpublishする場合、iOS Development Ceritificateを使います。また、App Store ConnectにpublishするためにはiOS Distribution Certificateが必要になります。リリース用のiOS Distribution Certificateを作成したい場合には[こちら](https://blog.codemagic.io/distributing-native-ios-sdk-with-flutter-module-using-codemagic/)を参照してください。作成済みのProvisioning profileとCeritificateのダウンロードの取得方法を紹介します。

Xcodeを開いて設定画面に遷移した後、Accountsタブで任意のアカウントを選択して右カラムの`Download Manual Profile`を選択します。ここでダウンロードしたものは`~/Library/MobileDevice/Provisioning Profiles/`に配置してあります。

![19518e066a5714068bf3e31794f77bdd](/Users/tanabe.nobuyuki/Documents/drafts/images/19518e066a5714068bf3e31794f77bdd.png)

Ceritificateの取得方法も説明します。`Download Manual Profiles`の右側の`Manage Certificates...`をクリックします。

![19518e066a5714068bf3e31794f77bdd](/Users/tanabe.nobuyuki/Documents/drafts/images/19518e066a5714068bf3e31794f77bdd.png)

Certificateの管理画面ダイアログが表示されるのでエクスポートしたいCertficateにマウスオーバーしてコンテキストメニューを開きます。すると`Export Certificate`をクリックします。

![01e3a25b3a02831293401cd4fddc83c9](/Users/tanabe.nobuyuki/Documents/drafts/images/01e3a25b3a02831293401cd4fddc83c9.png)

`.p12`ファイルの名前やパスワードを設定して任意のディレクトリに保存することができます。

![a8d26d3d570f8423a6625a69c20fe9aa](/Users/tanabe.nobuyuki/Documents/drafts/images/a8d26d3d570f8423a6625a69c20fe9aa.png)

以上で設定済みの証明書とprovisioning profileの取得の説明は終わりです。

#### codemagic.yamlファイルへの反映

テンプレートとして予め提供されているcodemagic.yamlファイルは`.app`ファイルを生成するwolkflowを通してビルドを行うだけなので、`.ipaファイル`を生成したい場合はこのファイルにいくつかの環境変数をcode-signing用に追加しなければいけません。

証明書とプロビジョニングプロファイルを暗号化された形で、codemagic.yamlファイルの環境変数セクションにキーと値のペアとして追加する必要があります。

- CM_CERTIFICATE (encrypted version of the certificate)
-  CM_CERTIFICATE_PASSWORD (encrypted version of the certificate password)
- CM_PROVISIONING_PROFILE (encrypted version of the provisioning profile)

プロジェクトの設定画面に移動します。遷移先の`Encrypt environment variables`というリンクをクリックします。

![92ce48c647e6f525e8efd576e5f1ec48](/Users/tanabe.nobuyuki/Documents/drafts/images/92ce48c647e6f525e8efd576e5f1ec48.png)

リンクをクリックすると`Encrypt environment variables for YAML configuration`というモーダルが表示されます。`Choose a file or drag it here`にファイルをドラッグすると暗号化された文字列が表示されるのでコピーしておきましょう。codemagic.yamlファイルに環境変数を設定する時に使用します。もう一つ、証明書のパスワードについても環境変数に設定しておきたいので、ダイアログの`Paste the value of the variable here`というプレースホルダーが書いてあるテキストエリアにパスワードを押して`Encrypt`ボタンをクリックします。ファイルをドラッグした時と同じように暗号化された文字列が出力されるのでコピーしておきましょう。

codemagic.yamlに証明書とprovisioning profile、および.p12ファイルに設定したパスワードを環境変数として設定します。YAMLファイルは以下のようになりました。

```yaml
# Check out https://docs.codemagic.io/yaml/building-a-native-ios-app/ for more information
# Please review and update values in curly braces

workflows:
  ios-app:
    name: iOS App
    environment:
      vars:
        XCODE_WORKSPACE: "CodeMagicTestSample"
        XCODE_SCHEME: "CodeMagicTestSample"
      xcode: latest
      cocoapods: default
      CM_CERTIFICATE: Encrypted(...)　# 証明書ファイルの暗号化文字列
      CM_PROVISIONING_PROFILE: Encrypted(...) # provisioning profileの暗号化文字列
      CM_CERTIFICATE_PASSWORD: Encrypted(...) # .p12ファイルに指定したパスワードを暗号化した文字列
# ... 以下略

```

#### scriptの更新

ipaファイルの生成やテストの実行を行うために環境変数の設定以外に、scriptの変更が必要になります。

まずは keychain を初期化します。

```sh
- keychain initialize
```

Provisioning profileをデコードし、code-signingのプロセス間でアクセスできるようにフォルダに配置します。

```sh
PROFILES_HOME="$HOME/Library/MobileDevice/Provisioning Profiles"
mkdir -p "$PROFILES_HOME"
PROFILE_PATH="$(mktemp "$PROFILES_HOME"/$(uuidgen).mobileprovision)"
echo ${CM_PROVISIONING_PROFILE} | base64 --decode > $PROFILE_PATH
echo "Saved provisioning profile $PROFILE_PATH"
```

Certificateをデコードしてキーチェーンに追加します。

```sh
echo $CM_CERTIFICATE | base64 --decode > /tmp/certificate.p12
keychain add-certificates --certificate /tmp/certificate.p12 --certificate-password $CM_CERTIFICATE_PASSWORD
```

テスト実行用のコマンドも追加します。

```sh
      - |
        # run tests
        xcodebuild \
        -workspace "CodeMagicTestSample.xcodeproj" \
        -scheme "CodeMagicTestSample" \
        -sdk iphonesimulator \
        -destination 'platform=iOS Simulator,name=iPhone 11 Pro Max,OS=13.4' \
        clean build test CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
```

ここで指定している`-destinationの値と、project.pbxprojeの`IPHONEOS_DEPLOYMENT_TARGET`とのバージョンが一致していることを確認してください。

.ipaファイル生成コマンドもこのタイミングで追加します。

```sh
      - |
        # build ipa
        xcode-project use-profiles
        xcode-project build-ipa --workspace "CodeMagicTestSample.xcodeproj" --scheme "CodeMagicTestSample"
```

私の環境に合わせて設定後のcodemagic.yamlファイルを掲載しておきます。

```yaml
# Check out https://docs.codemagic.io/yaml/building-a-native-ios-app/ for more information
# Please review and update values in curly braces

workflows:
  ios-app:
    name: iOS App
    environment:
      vars:
        XCODE_WORKSPACE: "CodeMagicTestSample"
        XCODE_SCHEME: "CodeMagicTestSample"
      xcode: latest
      cocoapods: default
      CM_CERTIFICATE: Encrypted()
      CM_CERTIFICATE_PASSWORD: Encrypted(...)
      CM_PROVISIONING_PROFILE: Encrypted(...)
      CM_CERTIFICATE_PASSWORD: Encrypted(...)
    scripts:
      - xcodebuild build -workspace "$XCODE_WORKSPACE.xcworkspace" -scheme "$XCODE_SCHEME" CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
    artifacts:
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.app
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.dSYM

# You can get more information regarding the YAML file here:
# https://docs.codemagic.io/building/yaml

# Workflow setup for building Native iOS project
workflows:
  # The following workflow is for generating a debug build (.app)
  ios-project-debug: # workflow ID
    name: CodeMagicTestSampleDebug # workflow name
    environment:
      xcode: latest
      cocoapods: default
    scripts:
      - |
        # run tests
        xcodebuild \
        -workspace "CodeMagicTestSample.xcworkspace" \
        -scheme "CodeMagicTestSample" \
        -sdk iphonesimulator \
        -destination 'platform=iOS Simulator,name=iPhone 11 Pro Max,OS=13.4' \
        clean build test CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
      - |
        # build .app
        xcodebuild build -workspace "CodeMagicTestSample.xcworkspace" -scheme "CodeMagicTestSample" CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO
    artifacts:
      - $HOME/Library/Developer/Xcode/DerivedData/**/Build/**/*.app
    publishing:
      email:
        recipients:
          - xxx@gmail.com # enter your email id here

  # The following workflow is for generating a release build (.ipa)
  ios-project-release: # workflow ID
    name: CodeMagicTestSample # workflow name
    environment:
      vars:
        CM_CERTIFICATE: Encrypted(...) # enter the encrypted version of your certificate
        CM_PROVISIONING_PROFILE: Encrypted(...) # enter the encrypted version of your provisioning profile
      xcode: latest
      cocoapods: default
    scripts:
      - keychain initialize
      - |
        # set up provisioning profiles
        PROFILES_HOME="$HOME/Library/MobileDevice/Provisioning Profiles"
        mkdir -p "$PROFILES_HOME"
        PROFILE_PATH="$(mktemp "$PROFILES_HOME"/$(uuidgen).mobileprovision)"
        echo ${CM_PROVISIONING_PROFILE} | base64 --decode > $PROFILE_PATH
        echo "Saved provisioning profile $PROFILE_PATH"
      - |
        # set up signing certificate
        echo $CM_CERTIFICATE | base64 --decode > /tmp/certificate.p12
        keychain add-certificates --certificate /tmp/certificate.p12  --certificate-password $CM_CERTIFICATE_PASSWORD
      - |
        # run tests
        xcodebuild \
        -workspace "CodeMagicTestSample.xcodeproj" \
        -scheme "CodeMagicTestSample" \
        -sdk iphonesimulator \
        -destination 'platform=iOS Simulator,name=iPhone 11 Pro Max,OS=13.4' \
        clean build test CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO
      - |
        # build ipa
        xcode-project use-profiles
        xcode-project build-ipa --workspace "CodeMagicTestSample.xcodeproj" --scheme "CodeMagicTestSample"
    artifacts:
      - build/ios/ipa/*.ipa
      - /tmp/xcodebuild_logs/*.log
    publishing:
      email:
        recipients:
          - xxx@gmail.com # enter your email id here


```

codemagic.yamlの変更を行ったcommitをpushした後、管理画面から`Start new build`をクリックしてビルドを開始します。

![bc1f1c2844efe0d3e70410431b20b885](/Users/tanabe.nobuyuki/Documents/drafts/images/bc1f1c2844efe0d3e70410431b20b885.png)

記載されたテストの実行やPublishingが成功したことを確認しました。

![056ad7313abeecd4c31be0cadbd6d5c1](/Users/tanabe.nobuyuki/Documents/drafts/images/056ad7313abeecd4c31be0cadbd6d5c1.png)

GitHubを見るとリポジトリにもテストの結果が反映されています。

![6421483054ddcbceb598258e258b58a0](/Users/tanabe.nobuyuki/Documents/drafts/images/6421483054ddcbceb598258e258b58a0.png)

まとめ





