# APNs の SSL 証明書のプロビジョニング
参考：

- [Apple Push Notification Service の使用開始 - Amazon Simple Notification Service](https://goo.gl/wXg1Xq)
- [プッシュ通知に必要な証明書の作り方2017 - Qiita](https://goo.gl/OYyJ2E)

###　目的
Apple Push Notificationサービス(APNs)によるiOSプッシュ通知を有効化する（= APNs の SSL 証明書のプロビジョニング）

### 目次

1. CSRファイルの作成 on your Mac
2. 開発用証明書(.cer)の作成 on Apple Devとダウンロード
3. AppIDの作成
4. 開発用の実機端末をAppleDevに登録
5. プロビショニングプロファイルの作成とダウンロード
6. APNS SSL 証明書を取得する
7. アプリケーションプライベートキーを取得する

### 1. CSRファイルの作成 on your Mac
- 「キーチェーンアクセス」を開いて、メニューバーの「キーチェーンアクセス」＞「証明書アシスタント」＞をクリック
- アドレスを入力
- 通称は **ローマ字**で
- CAメアド無記入
- ディスクに保存にチェック
- 鍵ペア情報を指定にチェック
- 続ける>完了


### 2. 開発用証明書(.cer)の作成 on Apple Devとダウンロード
- Apple Developer Programのメンバーセンターにログインします
- 「Certificates」＞「All」をクリックして、「iOS App Development」にチェックをいれ、下の方の「Continue」をクリックします
- Choose Fileで 1.のCSRファイルを指定
- Continue
-  **Download**しておく。名称は **myAppleDevCertification**とかで
- Done



### 3. AppIDの作成
- Apple Developer Programのメンバーセンターにログインして、「Certificates, Identifiers & Profiles」をクリックします
- 「Identifiers」の「AppIDs」をクリックして、右上の「＋」をクリックします
- 「App ID Description」にアプリの名前を入力。「app-push-notification-test」とか
- 画面をスクロールするとBundleIDの入力欄が出てきます
- 「Explicit App ID」をチェック。WildCardだとAPNSが使えない

	> AppIDは、 **「App ID Prefix = Team ID」と「デベロッパーアカウント名義企業の逆ドメイン」と「Xcodeプロジェクト名」**で成り立っています。（例：5UC9907AUX.com.leadi.myNewApp）チームで開発している場合、「App ID Prefix」には「Team ID」が自動で自動で入力されます。App ID Prefixを除いたAppIDを「BundleID」と呼びます。「Explicit App ID」を設定するということは、「Xcodeプロジェクト名」を入力するということです。


- 「Bundle ID」はXcodeで作成したアプリと同じものを使用します（いま開発中のｍのが通知対応していないなら、新しいAppID（とBundleID、つまり新しいプロジェクト名）を設定する）
- アプリの作成をこのあと行う場合は、ここで作成したBundleIDを控えてXcodeで作成するアプリに入力する必要があります
- スクロールして「Push Notifications」にチェック
- Continue
- 「Push Notifications」がConfigurableになっていることを確認
- Done



### 4. 開発用の実機端末をAppleDevに登録
- Apple Developer Programのメンバーセンターにログインします
- 「Devices」＞「All」をクリックして、右上の「＋」をクリックします
- 「Register Device」にチェック
- UUIDはDeviceつなげた状態でXcodeの「Window > Device」の「Identifer」
- Continue
- Register
- Done



### 5. プロビショニングプロファイルの作成とダウンロード
- Apple Developer Programのメンバーセンターにログインします
- 「Provisioning Profiles」＞「All」をクリックして、右上の「＋」をクリックします
- 「iOS App Development」
- 3.のAppIDを選択
- 2.の開発用証明書を選択
- 4.の登録した端末を選択
- ファイル名を入力「xxx-dev-ApnsProvisioningProfiles」とか
- Continue
- **Download**しておく
- Done

- ダウンロードした xxx-dev-ApnsProvisioningProfiles をダブルクリックしてXcode に反映させる（Xcodeは終了した状態で行う）



### 6. APNS SSL 証明書を取得する
> Amazon SNS には、Amazon SNS API の使用時にアプリの .pem 形式の APNS SSL 証明書が必要です。.p12 形式の証明書は Amazon SNS コンソールでアップロードできます。Amazon SNS によってその証明書は .pem に変換され、コンソールに表示されます。

#### 6-1. APNS SSL 証明書(.cer)の作成とダウンロード

- Apple Developer Programのメンバーセンターにログインします
- 「Certificates」＞「All」をクリックして、「Apple Push Notification service SSL(Sandcox)」にチェックをいれ、下の方の「Continue」をクリックします
- 3.のAppIDを選択
- 1.のCSRファイルを選択
- Continue
- Download。名称は「myApnsAppCert」など
- Done
- これで3.で作成したAppIDsの「Push Notifications」の項目が「Enabled」になります


#### 6-2. APNS SSL 証明書を .cer 形式から .pem 形式に変換 on your Mac
- コマンドプロンプトで、次のコマンドを入力します。myapnsappcert.cer を6.でダウンロードした証明書の名前に置き換えます。

```console
openssl x509 -in myApnsAppCert.cer -inform DER -out myApnsAppCert.pem
```

新しく作成した .pem ファイルは、モバイルプッシュ通知メッセージを送信するように Amazon SNS を設定するために使用します。


### 7. アプリケーションプライベートキーを取得する
> Amazon SNS には、.pem 形式のアプリケーションプライベートキーが必要です。Mac コンピュータ上の Keychain Access アプリケーションを使用して、アプリケーションプライベートキーをエクスポートします。

#### 7-1. アプリケーションプライベートキー(.p12)の作成とダウンロード on your Mac
- 6-1.で作成した「APNs用証明書(.cer)」をダブルクリックで開きます
- 「Apple Development iOS Push Service xxx〜」の存在を確認
- 鍵ではなく証明書上で右クリック
- 書き出す。名称は「myApnsAppPrivateKey」など
- パスワードは適当に簡単なもの
- 許可

#### 7-2. アプリケーションプライベートキーを .p12 形式から .pem 形式に変換
- コマンドプロンプトで、次のコマンドを入力します。myapnsappprivatekey.p12 を Keychain Access からエクスポートしたプライベートキーの名前に置き換えます。
- パスワードが出てきたら7-1.で設定したパスワードを打ち込む

```comandline
openssl pkcs12 -in myApnsAppPrivateKey.p12 -out myApnsAppPrivateKey.pem -nodes -clcerts
```

### 8. 証明書とアプリケーションプライベートキーを検証する
ここまでで取得した.pem 証明書とプライベートキーファイルは、APNS に接続するために使用することで検証できます。

- コマンドプロンプトで、次のコマンドを入力します。myapnsappcert.pem と myapnsappprivatekey.pem をそれぞれ証明書とプライベートキーの名前に置き換えます。

```comandline
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert myApnsAppCert.pem -key myApnsAppPrivateKey.pem
```

- パスワードが出てきたら7-1.で設定したパスワードを打ち込む