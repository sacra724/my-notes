# Amazon Simple Notification Service (Amazon SNS) の利用

参考：

- [Amazon Simple Notification Service の使用開始 - Amazon Simple Notification Service](http://docs.aws.amazon.com/ja_jp/sns/latest/dg/GettingStarted.html)


### 目的

AmazonSNSとAPNsを用いてトピック/エンドポイントによるプッシュ通知配信システムを構築する

### 目次

0. 前提条件
1. デバイス トークンを取得する
2. Amazon SNS Platform Applicationの作成
3. SNSでトピックを設定
3. アプリ(Xcode)とSNSとAPNsを連携させる


（2.まででとりあえずSNSのコンソールから一台の端末にプッシュ通知を送れるようになる）

### 0. 前提条件

「APNs の SSL 証明書のプロビジョニング」は終了していること


### 1. デバイス トークンを取得する

プッシュ通知メッセージを受信するためにアプリケーションを APNS に登録すると、デバイストークン (64 バイトの 16 進値) が生成されます。

- Xcodeでプロジェクトを開く
- プロジェクトの「TARGETS」->「General」にある「Bundle identifier」を 3. App IDの作成で決めた Bundle ID にします

	>Projectで作ったプロジェクト名とアプリ名とが合わない場合、App ID で作成した Bundle ID とプロジェクトの名前が合わないなどは「TARGETS」「Info」の Bundle identifier をマクロを消すことで直接書き込みます。
	>例えばApp IDで com.hoge.hage と設定されていても、プロジェクトでは com.hoge.Hage になってしまっているケースがあります。

- プロジェクトの「TARGETS」->「Capabilities」にある「Push Notifications 」をONにします。


- トークンを取得するためのコードを記述する

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

- 実機に繋いだ状態でRunします（simulator不可）
- 最初の起動時に通知の許可を求めるダイアログが出てきます
- 許可をすると XcodeのOutputにデバイストークンが出力されますこのトークンは後で使いますので、コピーしておきましょう
- これで、APNSの利用が完了し、AMAZON SNSの利用準備が整いました



### 2. Amazon SNS Platform Applicationの作成

- まずは「モバイルデバイスへのメッセージの直接的な送信」まではAmazonSNSコンソールからできるのでそこまでやる
- Platform Applicationを作成した際に設定されるApplication ARN は AWS SDKの実装内で使用します。

参考:

 - [Amazon SNS モバイルプッシュの使用](https://goo.gl/dvcSFE)
 - [amazon sns で、iOS,Androidにpush通知する方法 - Qiita](https://goo.gl/NcaGcZ)

### 3. SNSでトピックを設定

- トピックを作成
- トピックへのサブスクライブ
- トピックへの発行

下記に沿って行う
[Amazon Simple Notification Service の使用開始](Amazon Simple Notification Service - http://docs.aws.amazon.com/ja_jp/sns/latest/dg/GettingStarted.html)

### 4. 


### 3. アプリ(Xcode)とSNSとAPNsを連携させる
#### 3-1. アプリ(Xcode)の記述

- CocoaPodsでプロジェクトに必要なAmazon SDKをimportする
- 恐らく下記あたりが必要（AWS serviceへのアクセスはAWS Cognitoを使うのがオススメってAWSが言ってる）


```console
target :'myAppName' do
	pod 'AWSCore'
	pod 'AWSCognito'
	pod 'AWSCognitoIdentityProvider'
	pod 'AWSSNS'
end
```

> We recommend using Amazon Cognito as your credential provider to access AWS services from your mobile app. Amazon Cognito provides a secure mechanism to access AWS services without having to embed credentials in your app. To learn more, see Amazon Cognito for iOS.  
> 参考： [Set Up the SDK for iOS — iOS Developer Guide](https://goo.gl/oxtTNz)

下記参考：[AWS SNSを使ってiOSへpush通知](https://goo.gl/pkCyVM)

>- まずはDeviceTokenをNSUserDefaults等を使って保存しておきます。ただ送られてきたDeviceTokenはそのままでは余計なものが含まれているので、適切な形に成形して保存します。次に送られてきたDeviceTokenをAWS SNSへ登録します。IAMで取得したAccessKeyとSecretKeyをそれぞれ設定して、AWSに接続できるようにします。
>- AWS SNSにDeviceTokenを含めた端末情報を送信します。今回端末情報には言語設定の情報も一緒に送信しています。この情報を送ることで、サーバー側からローカライズされたデータ(文字)を送ることができるようになります。詳しくは後述します。
>- またinput.platformApplicationArnにはAWS SNSで作成したapplicationのArnを設定します。そして登録が成功すると、クロージャのBFTaskにはAWS SNSから割り振られたEndpointArnが入っています。こちらもDeviceTokenと同様にNSUserDefaultsなどで保存しておきましょう。
>- EndpointArnを保持することで、AWS SNSのCustomUserDataの更新が可能になります。例えば端末の言語情報が変更された場合、以下のようにしてAWS SNS上のCustomUserDataを更新することができます。

#### 3-2. SNSとAPI側の設定、記述

##### そもそもAmazon Simple Notification Service とは
Amazon SNS を使用するときは、所有者としてトピックを作成し、そのトピックと通信できる発行者とサブスクライバーを決定するポリシーを定義することで、アクセスを制御します。発行者は、自分が作成したトピック、または発行を許可されたトピックにメッセージを送信します。各メッセージに固有の宛先アドレスを含める代わりに、発行者はトピックにメッセージを送信します。Amazon SNS では、トピックをそのトピックにサブスクライブしているサブスクライバーリストと照合し、メッセージを各サブスクライバーに送信します。各トピックには、発行者がメッセージを投稿したり、サブスクライバーが通知に登録する Amazon SNS エンドポイントを識別するための一意の名前があります。サブスクライバーは、サブスクライブしているトピックに対して発行されたすべてのメッセージを受信します。また、同じトピックのサブスクライバーはすべて同じメッセージを受信します。

##### Amazon SNS へのアクセス方法

- ###### AWS SDK
AWS には、さまざまなプログラミング言語およびプラットフォーム (Java、Python、Ruby、.NET、iOS、Android など) のライブラリとサンプルコードで構成された SDK (ソフトウェア開発キット) が用意されています。SDK は、Amazon SNS および AWS へのプログラムによるアクセスを作成するのに便利です。例えば、SDK は要求への暗号を使用した署名、エラーの管理、要求の自動的な再試行などのタスクを処理します。AWS SDK のダウンロードやインストールなどの詳細については、「Tools for Amazon Web Services」ページを参照してください。


- ###### Amazon SNSクエリ API
サービスに HTTPS リクエストを直接発行できる Amazon SNS のクエリ API を使用して、Amazon SNS と AWS にプログラムからアクセスできます。詳細については、[Amazon Simple Notification Service API Reference](http://docs.aws.amazon.com/ja_jp/sns/latest/api/API_Operations.html) を参照してください。





---

- tokenの有効、無効をチェック
- 配信数やリクエストのサイズ、配信失敗数

- SNSで「Create new topic」
- 「Topic name」に「login-notification」など適当な名前を入力
- TopicのARNが得られる
- `arn:aws:sns:region:account_ID:topic_name`






##### Q: Amazon SNS が送信する構造化通知メッセージのフォーマットは何ですか?

HTTP、HTTPS、Email-JSON、および SQS トランスポートプロトコル経由の配信のために Amazon SNS が送信する通知メッセージは、シンプルな JSON オブジェクトで、以下の情報が含まれます。

- MessageId: 共通のユニークな識別子。発行される各通知で一意です。
- Timestamp: 通知が発行された時刻（GMT）。
- TopicArn: このメッセージが発行されたトピック。
- Type: 通知配信用「通知」に設定された、配信メッセージの種類。
- UnsubscribeURL: このトピックからエンドポイントの登録を解除するためのリンク。これ以上通知を受け取らないようにします。
- メッセージ: 発行者が受信するメッセージのペイロード（本体）。
- Subject: 件名欄 – オプションのパラメータとして公開APIに含まれる場合、メッセージと共に呼び出します。
- Signature: メッセージ、メッセージID、件名（存在する場合）、タイプ、タイムスタンプ、トピック値のBase64-エンコード 「SHA1withRSA」署名。
- SignatureVersion: 使用される Amazon SNS 署名のバージョン。
- 「Eメール」トランスポート経由で送信される通知メッセージには、発行者に受領されるペイロード（本体）のみが含まれます。


デバイストークンを SNS に登録すると、SNS がそのトークンに対応するエンドポイントを作成します。トピックへ発行するのと同じように、トークンエンドポイントに発行できます。


