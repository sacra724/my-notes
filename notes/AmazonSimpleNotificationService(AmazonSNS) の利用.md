# Amazon Simple Notification Service (Amazon SNS) の利用
参照：
* [Apple Push Notification Service の使用開始 - Amazon Simple Notification Service](https://goo.gl/wXg1Xq)
* [プッシュ通知に必要な証明書の作り方2017 - Qiita](https://goo.gl/OYyJ2E)

## ざっくり2STEP

Ⅰ. Apple Push Notificationサービス(APNs)によるプッシュ通知を有効化する（= APNs の SSL 証明書のプロビジョニング）

 Ⅱ. AmazonSNSにAPNsを登録する（= AmazonSNSの Platform appの作成）



## Ⅰ. APNs の SSL 証明書のプロビジョニング

### 目次
1. CSRファイルの作成 on your Mac
2. 開発用証明書(.cer)の作成 on Apple Devとダウンロード
3. AppIDの作成
4. 開発用の実機端末をAppleDevに登録
5. プロビショニングプロファイルの作成とダウンロード
6. APNS SSL 証明書を取得する
7. アプリケーションプライベートキーを取得する



### 1. CSRファイルの作成 on your Mac
* 「キーチェーンアクセス」を開いて、メニューバーの「キーチェーンアクセス」＞「証明書アシスタント」＞をクリック
* アドレスを入力
* 通称は **ローマ字** で
* CAメアド無記入
* ディスクに保存にチェック
* 鍵ペア情報を指定にチェック
* 続ける>完了


### 2. 開発用証明書(.cer)の作成 on Apple Devとダウンロード
* Apple Developer Programのメンバーセンターにログインします
* 「Certificates」＞「All」をクリックして、「iOS App Development」にチェックをいれ、下の方の「Continue」をクリックします
* Choose Fileで 1.のCSRファイルを指定
* Continue
*  **Download** しておく。名称は **myAppleDevCertification** とかで
* Done



### 3. AppIDの作成
* Apple Developer Programのメンバーセンターにログインして、「Certificates, Identifiers & Profiles」をクリックします
* 「Identifiers」の「AppIDs」をクリックして、右上の「＋」をクリックします
* 「App ID Description」にアプリの名前を入力。「app-push-notification-test」とか
* 画面をスクロールするとBundleIDの入力欄が出てきます
* 「Explicit App ID」をチェック。WildCardだとAPNSが使えない

	> AppIDは、 **「App ID Prefix = Team ID」と「デベロッパーアカウント名義企業の逆ドメイン」と「Xcodeプロジェクト名」**で成り立っています。（例：5UC9907AUX.com.leadi.myNewApp）チームで開発している場合、「App ID Prefix」には「Team ID」が自動で自動で入力されます。App ID Prefixを除いたAppIDを「BundleID」と呼びます。「Explicit App ID」を設定するということは、「Xcodeプロジェクト名」を入力するということです。


* 「Bundle ID」はXcodeで作成したアプリと同じものを使用します（いま開発中のｍのが通知対応していないなら、新しいAppID（とBundleID、つまり新しいプロジェクト名）を設定する）
* アプリの作成をこのあと行う場合は、ここで作成したBundleIDを控えてXcodeで作成するアプリに入力する必要があります
* スクロールして「Push Notifications」にチェック
* Continue
* 「Push Notifications」がConfigurableになっていることを確認
* Done



### 4. 開発用の実機端末をAppleDevに登録
* Apple Developer Programのメンバーセンターにログインします
* 「Devices」＞「All」をクリックして、右上の「＋」をクリックします
* 「Register Device」にチェック
* UUIDはDeviceつなげた状態でXcodeの「Window > Device」の「Identifer」
* Continue
* Register
* Done



### 5. プロビショニングプロファイルの作成とダウンロード
* Apple Developer Programのメンバーセンターにログインします
* 「Provisioning Profiles」＞「All」をクリックして、右上の「＋」をクリックします
* 「iOS App Development」
* 3.のAppIDを選択
* 2.の開発用証明書を選択
* 4.の登録した端末を選択
* ファイル名を入力「xxx-dev-ApnsProvisioningProfiles」とか
* Continue
* **Download**しておく
* Done

* ダウンロードした xxx-dev-ApnsProvisioningProfiles をダブルクリックしてXcode に反映させる（Xcodeは終了した状態で行う）



### 6. APNS SSL 証明書を取得する
> Amazon SNS には、Amazon SNS API の使用時にアプリの .pem 形式の APNS SSL 証明書が必要です。.p12 形式の証明書は Amazon SNS コンソールでアップロードできます。Amazon SNS によってその証明書は .pem に変換され、コンソールに表示されます。

#### 6-1. APNS SSL 証明書(.cer)の作成とダウンロード

* Apple Developer Programのメンバーセンターにログインします
* 「Certificates」＞「All」をクリックして、「Apple Push Notification service SSL(Sandcox)」にチェックをいれ、下の方の「Continue」をクリックします
* 3.のAppIDを選択
* 1.のCSRファイルを選択
* Continue
* Download。名称は「myApnsAppCert」など
* Done
* これで3.で作成したAppIDsの「Push Notifications」の項目が「Enabled」になります


#### 6-2. APNS SSL 証明書を .cer 形式から .pem 形式に変換 on your Mac
* コマンドプロンプトで、次のコマンドを入力します。myapnsappcert.cer を6.でダウンロードした証明書の名前に置き換えます。

```console
openssl x509 -in myApnsAppCert.cer -inform DER -out myApnsAppCert.pem
```

新しく作成した .pem ファイルは、モバイルプッシュ通知メッセージを送信するように Amazon SNS を設定するために使用します。


### 7. アプリケーションプライベートキーを取得する
> Amazon SNS には、.pem 形式のアプリケーションプライベートキーが必要です。Mac コンピュータ上の Keychain Access アプリケーションを使用して、アプリケーションプライベートキーをエクスポートします。

#### 7-1. アプリケーションプライベートキー(.p12)の作成とダウンロード on your Mac
* 6-1.で作成した「APNs用証明書(.cer)」をダブルクリックで開きます
* 「Apple Development iOS Push Service xxx〜」の存在を確認
* 鍵ではなく証明書上で右クリック
* 書き出す。名称は「myApnsAppPrivateKey」など
* パスワードは適当に簡単なもの
* 許可

#### 7-2. アプリケーションプライベートキーを .p12 形式から .pem 形式に変換
* コマンドプロンプトで、次のコマンドを入力します。myapnsappprivatekey.p12 を Keychain Access からエクスポートしたプライベートキーの名前に置き換えます。
* パスワードが出てきたら7-1.で設定したパスワードを打ち込む

```comandline
openssl pkcs12 -in myApnsAppPrivateKey.p12 -out myApnsAppPrivateKey.pem -nodes -clcerts
```

### 8. 証明書とアプリケーションプライベートキーを検証する
ここまでで取得した.pem 証明書とプライベートキーファイルは、APNS に接続するために使用することで検証できます。

* コマンドプロンプトで、次のコマンドを入力します。myapnsappcert.pem と myapnsappprivatekey.pem をそれぞれ証明書とプライベートキーの名前に置き換えます。

```comandline
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert myApnsAppCert.pem -key myApnsAppPrivateKey.pem
```

* パスワードが出てきたら7-1.で設定したパスワードを打ち込む
*

***

## Ⅱ. AmazonSNSの Platform appの作成

### 目次
1. デバイス トークンを取得する
2. Amazon SNS Platform Applicationの作成
3. プロジェクトでAMAZON SNSを利用する（swift, API記述）

2.まででとりあえずSNSのコンソールから一台の端末にプッシュ通知を送れるようになります！

### 1. デバイス トークンを取得する
> プッシュ通知メッセージを受信するためにアプリケーションを APNS に登録すると、デバイストークン (64 バイトの 16 進値) が生成されます。

* Xcodeでプロジェクトを開く
* プロジェクトの「TARGETS」->「General」にある「Bundle identifier」を 3. App IDの作成で決めた Bundle ID にします

> Projectで作ったプロジェクト名とアプリ名とが合わない場合、App ID で作成した Bundle ID とプロジェクトの名前が合わないなどは「TARGETS」「Info」の Bundle identifier をマクロを消すことで直接書き込みます。
例えばApp IDで com.hoge.hage と設定されていても、プロジェクトでは com.hoge.Hage になってしまっているケースがあります。

* プロジェクトの「TARGETS」->「Capabilities」にある「Push Notifications 」をONにします。


* トークンを取得するためのコードを記述する

AppDelegate.swift

```swift
import UIKit
import UserNotifications


@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

var window: UIWindow?


func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

	// APNs
	UNUserNotificationCenter.current().requestAuthorization(
		options: [.badge, .alert, .sound]) {(accepted, error) in
		if accepted {
			print("Notification access accepted !")
			// デバイストークンを登録
			UIApplication.shared.registerForRemoteNotifications()
		}
		else{
			print("Notification access denied.")
		}
	}

	return true
}

// Remote Notification のエラーを受け取る
func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
	print(error)
}

// Remote Notification の device token を表示
func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
	var deviceToken = String(format: "%@", deviceToken as CVarArg) as String
	print("deviceToken = \(deviceToken)")

}
...
}
```

* 実機に繋いだ状態でRunします（simulator不可）
* 最初の起動時に通知の許可を求めるダイアログが出てきます
* 許可をすると XcodeのOutputにデバイストークンが出力されますこのトークンは後で使いますので、コピーしておきましょう
* これで、APNSの利用が完了し、AMAZON SNSの利用準備が整いました



### 2. Amazon SNS Platform Applicationの作成
* まずは「モバイルデバイスへのメッセージの直接的な送信」まではAmazonSNSコンソールからできるのでそこまでやる
* 参考: [Amazon SNS モバイルプッシュの使用](https://goo.gl/dvcSFE)
* 参考: [amazon sns で、iOS,Androidにpush通知する方法 - Qiita](https://goo.gl/NcaGcZ)
* Platform Applicationを作成した際に設定されるApplication ARN は AWS SDKの実装内で使用します。



### 3. プロジェクトでAMAZON SNSを利用する（swift, API記述）

#### 3-1. Xcode側の記述

* CocoaPodsでプロジェクトに必要なAmazon SDKをimportする
* 参考： [Set Up the SDK for iOS — iOS Developer Guide](https://goo.gl/oxtTNz)
* 恐らく下記あたりが必要（AWS serviceへのアクセスはAWS Cognitoを使うのがオススメってAWSが言ってる）

> We recommend using Amazon Cognito as your credential provider to access AWS services from your mobile app. Amazon Cognito provides a secure mechanism to access AWS services without having to embed credentials in your app. To learn more, see Amazon Cognito for iOS.

```console
target :'myAppName' do
	pod 'AWSCore'
	pod 'AWSCognito'
	pod 'AWSCognitoIdentityProvider'
	pod 'AWSSNS'
end
```

下記参考：[AWS SNSを使ってiOSへpush通知](https://goo.gl/pkCyVM)

>* まずはDeviceTokenをNSUserDefaults等を使って保存しておきます。ただ送られてきたDeviceTokenはそのままでは余計なものが含まれているので、適切な形に成形して保存します。次に送られてきたDeviceTokenをAWS SNSへ登録します。IAMで取得したAccessKeyとSecretKeyをそれぞれ設定して、AWSに接続できるようにします。
>* AWS SNSにDeviceTokenを含めた端末情報を送信します。今回端末情報には言語設定の情報も一緒に送信しています。この情報を送ることで、サーバー側からローカライズされたデータ(文字)を送ることができるようになります。詳しくは後述します。
>* またinput.platformApplicationArnにはAWS SNSで作成したapplicationのArnを設定します。そして登録が成功すると、クロージャのBFTaskにはAWS SNSから割り振られたEndpointArnが入っています。こちらもDeviceTokenと同様にNSUserDefaultsなどで保存しておきましょう。
>* EndpointArnを保持することで、AWS SNSのCustomUserDataの更新が可能になります。例えば端末の言語情報が変更された場合、以下のようにしてAWS SNS上のCustomUserDataを更新することができます。

#### 3-2. API側の記述

[TODO]











##### Q: Amazon SNS が送信する構造化通知メッセージのフォーマットは何ですか?

HTTP、HTTPS、Email-JSON、および SQS トランスポートプロトコル経由の配信のために Amazon SNS が送信する通知メッセージは、シンプルな JSON オブジェクトで、以下の情報が含まれます。

* MessageId: 共通のユニークな識別子。発行される各通知で一意です。
* Timestamp: 通知が発行された時刻（GMT）。
* TopicArn: このメッセージが発行されたトピック。
* Type: 通知配信用「通知」に設定された、配信メッセージの種類。
* UnsubscribeURL: このトピックからエンドポイントの登録を解除するためのリンク。これ以上通知を受け取らないようにします。
* メッセージ: 発行者が受信するメッセージのペイロード（本体）。
* Subject: 件名欄 – オプションのパラメータとして公開APIに含まれる場合、メッセージと共に呼び出します。
* Signature: メッセージ、メッセージID、件名（存在する場合）、タイプ、タイムスタンプ、トピック値のBase64-エンコード 「SHA1withRSA」署名。
* SignatureVersion: 使用される Amazon SNS 署名のバージョン。
* 「Eメール」トランスポート経由で送信される通知メッセージには、発行者に受領されるペイロード（本体）のみが含まれます。


デバイストークンを SNS に登録すると、SNS がそのトークンに対応するエンドポイントを作成します。トピックへ発行するのと同じように、トークンエンドポイントに発行できます。








