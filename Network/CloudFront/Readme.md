# CloudFront設定
- 以下は詳細を勉強したい
- [ディストリビューションを作成または更新する場合に指定する値](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html)
## 各項目
### Origin Domain Name
- パブリックDNS名を入れる
### Origin Path
- >オプション。 CloudFrontがAmazonS3バケット内のディレクトリまたはカスタムオリジンからコンテンツをリクエストするようにする場合は、ここに/で始まるディレクトリ名を入力します。 CloudFrontは、リクエストをオリジンに転送するときに、オリジンドメイン名の値にディレクトリ名を追加します（例：myawsbucket / production）。ディレクトリ名の末尾に/を含めないでください。
### Enable Origin Shield
- [Amazon CloudFrontにオリジンへのリクエストを軽減するキャッシュレイヤー(Origin Shield)が追加されました](https://dev.classmethod.jp/articles/amazon-cloudfront-support-cache-layer-origin-shield/)
- >Origin Shield を利用すると、オリジンとの通信は Origin Shield に集約されます。 オリジンへのリクエストが発生するのは、Origin Shield にキャッシュがない場合だけです。
- >各リージョナルエッジキャッシュが個別にオリジンにリクエストしなくなるため、オリジンの負荷が軽減
- >エッジローケーション 〜 リージョナルエッジキャッシュ 〜 Origin Shield(オリジンのそば) 間は CloudFront のグローバルネットワークを利用するためネットワークパフォーマンスが向上
- ![image](https://user-images.githubusercontent.com/60077121/108588508-f8ba8d80-739c-11eb-9893-8e146cece093.png)

##### リージョナルエッジキャッシュとは
- [CloudFrontのリージョナルエッジキャッシュが発表されました](https://dev.classmethod.jp/articles/cloudfront-regional-edge-cache/)
- クライアント→エッジロケーション→リージョナルエッジキャッシュ→オリジンシールド→オリジン

### Origin ID

### Origin Custom Headers


# キャッシュの仕組み
- [コンテンツがキャッシュに保持される期間 (有効期限) の管理](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/Expiration.html)
  - 一番わかりやすい。
- [CloudFront の Cache Policy と Origin Request Policy を理解する](https://qiita.com/t-kigi/items/6d7cfccdb629690b8d29)

## キャッシュポリシーで決めること
### そもそも論（以下はCFでは制御不能、オリジンが決めること）
- `Cache-Control max-age` ディレクティブや`Expires`ディレクティブはオリジンサーバのレスポンスメッセージ（レスポンスヘッダ）に含まれるもの
  - `Cache-Control max-age`は一般ヘッダ（リクエスト・レスポンスどっちにもある）
  - `Expires`はエンティティヘッダ（同上）
- `Cache-Control s-maxage`はキャッシュサーバにキャッシュを保持する期間
  - `Cache-Control max-age`が3600秒、`Cache-Control s-maxage`が300秒なら、ブラウザキャッシュに3600秒、CFキャッシュには300秒、キャッシュが保持。
- >オブジェクトのキャッシュを制御するには、Cache-Control max-age ヘッダーフィールドではなく、Expires ディレクティブを使用することをお勧めします。Cache-Control max-age と Expires の両方の値を指定した場合、CloudFront は Cache-Control max-age の値のみを使用します。
## Managed-CachingOptimized（デフォルトのキャッシュポリシー）
### TTL Settings
- Minimum TTL(1sec)
  - 最小TTLを0にしても「キャッシュしない」というわけではない。Cache-Control max-ageなどのメッセージヘッダに依存する。詳細はドキュメント
- Maximum TTL(31536000=1年)めっちゃ長い
- Default TTL(86400=24h)

#### よくあるパターン（詳細は`コンテンツがキャッシュに保持される期間 (有効期限) の管理`の表を見ること）
- **注意：CFの最小TTLを0にすると下記は変わってくるのでドキュメント見ること。**
- 1:オリジンが Cache-Control max-age ディレクティブをオブジェクトに追加する
  - だいたい最小 TTL < max-age < 最大 TTLになる　→　CloudFront は、Cache-Control max-age ディレクティブの値に対応する期間、オブジェクトをキャッシュに保持します。
- 2:オリジンが Cache-Control max-age ディレクティブをオブジェクトに追加しない
  - CloudFront は、CloudFront 最小 TTL またはデフォルト TTL の値のうち、大きい方の値に対応する期間、オブジェクトをキャッシュに保持します。**つまり24時間**
- 3:オリジンが Expires ヘッダーをオブジェクトに追加する
  - だいたい最小 TTL < Expires < 最大 TTLになる　→　CloudFront は、Expires ヘッダーにある日時までオブジェクトをキャッシュに保持します。
- 4:オリジンが、Cache-Control: no-cache、no-store、および private ディレクティブ、またはこのいずれかをオブジェクトに追加する
  - CloudFront は、CloudFront の最小 TTL の値に対応する期間、オブジェクトをキャッシュします。 **注意：1秒間はキャッシュしちゃう**
  - **明示的に最小TTLを0にすると、「CloudFront とブラウザはヘッダーを優先させます。」** 

### Cache key contents
- 以下の3種類の値を区別するかどうか。デフォルトでは区別しない（全部同じオブジェクトと見なし、キャッシュがあればそれを返す）。[メルカリではこれが原因でユーザーAにユーザーBのセッションを返してしまった](https://engineering.mercari.com/blog/entry/2017-06-22-204500/)
  - Headers
  - Cookies
  - Query strings

- 区別するなら、それぞれ以下からオプションを選択する。
  - None（デフォルト）
  - All
  - Whitelist：特定のヘッダやクエリ文字列でオブジェクトを区別したい場合
  - All-Except (次を除くすべて)

## Origin Request Policy
### ここは「キャッシュキーには使わないが、オリジンサーバーに送信したい値」を定義する。
> 2 種類のポリシーは別々のものですが、関連性があります。キャッシュキーに含めるすべての URL クエリ文字列、HTTP ヘッダー、および Cookie (キャッシュポリシーを使用) は、オリジンリクエストに自動的に含まれます。オリジンリクエストポリシーを使用して、オリジンリクエストに含めるが、キャッシュキーには含めない情報を指定します。
https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/controlling-origin-requests.html

### Origin Request Policy に Authorization ヘッダを含めることはできない
- [認証ヘッダーをオリジンに転送するように CloudFront を設定するにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/cloudfront-authorization-header/)

### デフォルトではOrigin Request Policyはなし


# オリジンへのアクセスをCloufFrontからのみに絞る
- S3ならOAI
- ELBならカスタムヘッダ
  - https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/restrict-access-to-load-balancer.html

# ログ
## 標準ログ
- 無料。ストレージ料金だけ。
- S3に入る。**選択したS3のプレフィックス配下に`E2LM2M8JS9TNGA.2021-02-28-00.c5259020.gz`**という形式。ログのパーティション分割はされない（2021/2/28みたいな）
- >通常は数分以内に出力されますが、ベストエフォートではあるため公式ガイド上では 1 時間以内 または 最大で 24 時間 遅れることもあると記載されています。
- >ディストリビューション専用のログファイルに書き込みます。
### ログはディストリビューションごとにアクセスごとにS3へ配信
- だから結構細かい単位になる
#### curlでアクセス
```
murakami@HW-1191:~$ curl httpbin.org/ip
{
  "origin": "182.251.254.14"
}
murakami@HW-1191:~$ curl stg-development.work
略
murakami@HW-1191:~$ curl stg-development.work
murakami@HW-1191:~$ curl stg-development.work
murakami@HW-1191:~$ date
Sun Feb 28 16:35:17 JST 2021
```
#### S3には5分後に出力
![image](https://user-images.githubusercontent.com/60077121/109411257-22e6fd80-79e4-11eb-85ce-688b02ad7cd4.png)

#### ログの中身
```
#Version: 1.0
#Fields: date time x-edge-location sc-bytes c-ip cs-method cs(Host) cs-uri-stem sc-status cs(Referer) cs(User-Agent) cs-uri-query cs(Cookie) x-edge-result-type x-edge-request-id x-host-header cs-protocol cs-bytes time-taken x-forwarded-for ssl-protocol ssl-cipher x-edge-response-result-type cs-protocol-version fle-status fle-encrypted-fields c-port time-to-first-byte x-edge-detailed-result-type sc-content-type sc-content-len sc-range-start sc-range-end
2021-02-28	07:35:14	NRT51-C4	586	182.251.254.14	GET	d1pktia78oopcc.cloudfront.net	/	301	-	curl/7.68.0	-	-	Redirect	HjoYpgtwxmNZfbMKBIxqQPmN5VqzFi4PB2uFQqDdjYNJXZXEpWi8Ug==	stg-development.work	http	84	0.000	-	-	-	Redirect	HTTP/1.1	-	-	3779	0.000	Redirect	text/html	183	-	-
2021-02-28	07:35:09	NRT51-C4	586	182.251.254.14	GET	d1pktia78oopcc.cloudfront.net	/	301	-	curl/7.68.0	-	-	Redirect	cRUsOpLOGNkVraUTIX26CmCbefpnZjHQST5_wzRgJSi0tueK8daf3g==	stg-development.work	http	84	0.000	-	-	-	Redirect	HTTP/1.1	-	-	3411	0.000	Redirect	text/html	183	-	-
2021-02-28	07:35:12	NRT51-C4	586	182.251.254.14	GET	d1pktia78oopcc.cloudfront.net	/	301	-	curl/7.68.0	-	-	Redirect	myJmzs1vNjKmvqLr9RkssCTTwJ9U87ALEzJrM3SC7nI3gi1xJ3AE1A==	stg-development.work	http	84	0.000	-	-	-	Redirect	HTTP/1.1	-	-	3605	0.000	Redirect	text/html	183	-	-
```

## リアルタイムログ
- [[アップデート] Amazon CloudFront でリアルタイムなログ出力が可能になりました](https://dev.classmethod.jp/articles/cloudfront-support-real-time-log/)
  - 良い記事
  - >リアルタイムログは Kinesis Data Streams を使用し、S3、Amazon Redshift、Amazon Elasticsearch Service、またはサードパーティのログ処理サービスにログを配信することが出来ます。
  - >リクエストから数秒以内にアクセスログ出力が可能
  - 標準ログよりも詳細なログが見れる。出力するログは選ぶ形式
    - >リアルタイムログではエッジサーバーがビューワーへの応答で送信した HTTP ヘッダーなども確認することができます。 
- 追加料金必要
### ログのパーティション分割・変換が標準ログより自由
- >リアルタイムログは Kinesis Data Streams を介して出力します。コンシューマーとして Kinesis Data Firehose を使うことで YYYY/MM/DD/HH のようなパーティション分割が簡単に出来ます。
- 出力先としてはKinesis Data Streamsだけ。
  - よくあるパターンはCF -> Kinesis Data Streams -> Kiesis Data Firehose(ログのパーティション分割) -> S3

### Kinesis Data Streamsのシャード算出
- CloudFront 使用状況レポート、または CloudFront Requests メトリクスをもとに 1 秒あたりのリクエスト数を確認
- 単一のリアルタイムログレコードサイズを決める
  - 一般的に約 500 バイトですが、すべてのフィールドを含む場合は約 1 Kバイト
- 1. と 2. を掛けてリアルタイムログが Kinesis Data Streams に送る 1 秒あたりデータ量が決まります
- ひとつのシャードが 1 秒あたりに処理できるデータ量は 1 MB、または 1000 レコードです。10 〜 20 % 程度のバッファを持たせてシャード数を決めることが推奨


