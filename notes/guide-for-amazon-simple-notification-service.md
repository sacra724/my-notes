# Amazon Simple Notification Service (Amazon SNS) の利用

参考：

- [Amazon Simple Notification Service の使用開始 - Amazon Simple Notification Service](http://docs.aws.amazon.com/ja_jp/sns/latest/dg/GettingStarted.html)


### 目的

AmazonSNSとAPNsを用いてトピック/エンドポイントによるプッシュ通知配信システムを構築する

### 目次

0.前提条件

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
- プロジェクトの「TARGETS」->「General」にある「Bundle identifier」をApp IDの作成で決めた Bundle ID にします

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

- [Amazon SNS モバイルプッシュの使用](https://goo.gl/dvcSFE)を参考に、「モバイルデバイスへのメッセージの直接的な送信」まではAmazonSNSコンソールからできるのでそこまでやってみましょう
- Platform Applicationを作成した際に設定されるApplication ARN は AWS SDKの実装内で使用します。

参考:

 - [amazon sns で、iOS,Androidにpush通知する方法 - Qiita](https://goo.gl/NcaGcZ)

### 3. SNSでトピックを設定
「2. Amazon SNS Platform Applicationの作成」でコンソールからメッセージの個別送信はできるようになりました。
次はトピックを利用したプッシュ通知の送信をやってみます。

- トピックを作成
- トピックへのサブスクライブ
- トピックへの発行

下記に沿って行います
[Amazon Simple Notification Service の使用開始](Amazon Simple Notification Service - http://docs.aws.amazon.com/ja_jp/sns/latest/dg/GettingStarted.html)


### 3. アプリ(Xcode)とSNSとAPNsを連携させる
「3. SNSでトピックを設定」で作成したトピックを使って自前のAPI経由でプッシュ通知を配信します。

下記の流れでプッシュ通知を配信します。

1. アプリでプッシュ通知を許可する
2. 端末をトピックに登録する
	1. tokenをアプリからAPIに送り、Endpointを作成
	2. APIでEndpointをApplication Plat Formに登録
	3. APIでEndpointでTopicをサブスクライブ
3. APIでメッセージをTopicに送信する
4. Topicをサブスクライブしている端末にプッシュ通知が届く

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


- ###### Amazon SNSクエリ API（この資料内ではこちらを利用）
サービスに HTTPS リクエストを直接発行できる Amazon SNS のクエリ API を使用して、Amazon SNS と AWS にプログラムからアクセスできます。

##### Amazon SNSクエリ APIを利用しAPIを記述
利用できるメソッドやエラーの詳細については、[AWS SDK for PHP 3.x - Amazon Simple Notification Service](http://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sns-2010-03-31.html)、[Amazon Simple Notification Service API Reference](http://docs.aws.amazon.com/ja_jp/sns/latest/api/API_Operations.html) を参照。

APIは `php`で記述します。フレームワークに `phalcon`を利用しています。

---

* platformApplicationにdeviceTokenを投げてEndpointを得る：`createEndpoint()`

参考：[CreatePlatformEndpoint](http://docs.aws.amazon.com/ja_jp/sns/latest/api/API_CreatePlatformEndpoint.html)

```php

$result = $client->createPlatformEndpoint([
	'PlatformApplicationArn' => $platformApplicationArn,
	'Token' => $deviceToken,
	'CustomUserData' => <'custom data'>,  //be in UTF-8 format and less than 2KB.
	'Attributes' => ['Enabled' => 'true'],
]);

$endpointArn = $result['EndpointArn'];

```

---

* トピックを作成する：`createEndpoint()`

参考：[CreatePlatformEndpoint](http://docs.aws.amazon.com/ja_jp/sns/latest/api/API_CreatePlatformEndpoint.html)

```php

$result = $client->createTopic([
	'Name' => 'test-topic',
]);

$topicArn = $result['TopicArn'];

```

---

* トピックをサブスクライブする：`subscribe()`

参考：[Subscribe](http://docs.aws.amazon.com/ja_jp/sns/latest/api/API_Subscribe.html)

```php

$result = $client->subscribe([
	'TopicArn' => $topicArn',
	'Endpoint' => $endpointArn,
	'Protocol' => 'application',
]);

```

---

* 投稿する：`publish()`

参考：[リモート通知のペイロード](https://developer.apple.com/jp/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/TheNotificationPayload.html)

送信するメッセージの形式は、送信先が単一のプロトコルの場合は `string`でも可。

ただJSON形式のほうが何かと便利なので、JSONで記述する。

```php

$result = $client->publish([
	'TopicArn' => $topicArn,
	'MessageStructure' => 'json',
	'Message' => json_encode([
		//default形式は必須
		'default' => $body,

		//develop用のペイロードキー
		'APNS_SANDBOX' => json_encode([
			'aps' => [
				'alert' => [
					'title' => $title,
					'body'  => $body,
				],
				'badge' => 1,
				'sound' => 'bingbong.aiff',
				'test_id' => 1,
			]
		]),

		//production用のペイロードキー
		'APNS' => json_encode([
			'aps' => [
				'alert' => [
					'title' => '',
					'body'  => '',
				],
				'badge' => 1,
				'sound' => 'bingbong.aiff',
				'test_id' => 1,
			]
		]),
	]),
]);

```

---

まとめ

```php

<?php /**
*
*/
use Phalcon\Di;
use Aws\Sns\SnsClient;
use Aws\Sns\Exception\SnsException;

class SNSWrapper
{
	protected $snsClient   = null;
	protected $deviceToken = null;
	protected $PlatformApplicationArn = '<YOUR PLATFORM ARN>';

	public function __construct() {
		// Instantiate the client.
		$this->snsClient = new SnsClient([
			'version' => 'latest',
			'region'  => 'ap-northeast-1',
			'credentials' => '<YOUR CREDENTIALS>'
		]);
	}


	/**
	* platformApplicationにdeviceTokenを登録する
	*
	* @param str $token, str $userId
	* @throws throw new Exception
	* @return str $endpointArn
	*/
	public function createEndpoint($deviceToken, $userId) {
		try {
			$params = [
				'PlatformApplicationArn' => $this->$platformApplicationArn,
				'Token' => $this->deviceToken,
				'CustomUserData' => $userId,
				'Attributes' => ['Enabled' => 'true'],
			];

			if(!$res = $this->snsClient->createPlatformEndpoint($params)) {
				throw new \Exception($this->snsClient->getMessages()[0], 500);
			}

			$endpointArn = $res['EndpointArn'];
			self::subscribeTopic($endpointArn);

		} catch (SnsException $e) {
			return $e->getMessage() . "\n";
		}
	}


	/**
	* トピックを作成し$endpointArnでサブスクライブする
	*
	* @param str $endpointArn
	* @throws throw new Exception
	* @return str $SubscriptionArn
	*/
	public function subscribeTopic($endpointArn) {
		try {

			//名前を付けてtopicを作成。
			$topic = $this->snsClient->createTopic([
			  'Name' => 'test-all-users',
			]);

			$params = [
				'TopicArn' => $topic['TopicArn'],
				'Protocol' => 'application',
				'Endpoint' => $endpointArn,
			];

			if(!$res = $this->snsClient->subscribe($params)) {
				throw new \Exception($this->snsClient->getMessages()[0], 500);
			}

		} catch (SnsException $e) {
			return $e->getMessage() . "\n";
		}
	}



	/**
	* トピックにメッセージを配信する
	*
	* @param str $topicArn
	* @throws throw new Exception
	* @return str $SubscriptionArn
	*/
	public function publishTopic($topicArn, $message) {
		try {
			$result = $this->snsClient->publish([
				'TopicArn' => $topicArn,
				'MessageStructure' => 'json',
				'Message' => json_encode([
					'APNS_SANDBOX' => json_encode([
						'aps' => [
							'alert' => [
								'title' => $title,
								'body'  => $body,
							],
						]
					]),
				]),
  			]);
		} catch (SnsException $e) {
			return false;
		}
	}
}

```

