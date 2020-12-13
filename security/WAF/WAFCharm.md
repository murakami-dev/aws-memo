
# 文献
- 1.[AWS WAF + WafCharmトライアルの導入方法](https://oji-cloud.net/2020/08/30/post-5295/)
- 2.[WafCharm の AWS 環境構成図](https://blog.serverworks.co.jp/WafCharm-diagram)
- 3.[ALBのアクセスログをS3に保存して中身を読み解いてみる](https://dev.classmethod.jp/articles/alb-log-to-s3/)
  - 3-1.[Application Load Balancer のアクセスログ](https://docs.aws.amazon.com/ja_jp/elasticloadbalancing/latest/application/load-balancer-access-logs.html#enable-access-logging)
- 4.[WafCharmご利用手順](https://www.wafcharm.com/blog/check-wafcharm-setting-jp/)
- 5.[WafCharmご利用の際に必要となるIAMポリシー](https://www.wafcharm.com/blog/aws-iam-setting-for-wafcharm-jp/)
- 6.[WafCharm レポート機能の構築方法](https://oji-cloud.net/2020/09/03/post-5344/)
- 7.[AWS WAF のログからブロックが発生したAWSリソースを特定する](https://blog.serverworks.co.jp/awswaf-httpsourceId)
- 8.[AWS WAF v2でWafCharmを導入設定する際に知っておきたいこと](https://dev.classmethod.jp/articles/aws-waf-v2-wafcharm/)
  - 9.[new AWS WAF でのルールグループの例外設定](https://www.wafcharm.com/blog/new-aws-waf-managed-rule-rulegourp-exception-jp/)

# 文献8が良記事なので抜粋
- `Web ACL Config`の作成時に`Default WAF Action`を`COUNT`にすることで、適用されるルールグループに`Override rules action`が適用される（要はカウントモード）
- `Web Site Config`について
  - >Web Site Configの作成は、この「S3 Path」ごとに必要となります。
  - >なお、Web Site Configには「FQDN」という設定項目がありますが、ここで指定した値にWafCharmがアクセスすることはないため、任意の値を設定して構いません。ただし、WafCharmを管理していくことを考えると「FQDN」でどのサイトに対する設定なのか識別できる値が望ましいと思います。**ここでお伝えしたいことは、Web Site Configの作成単位は「S3 Path」ごとに作成するのであって、「FQDN」ごとに作成するのではないといった点になります。**
- >WafCharmを利用する上で、FirehoseのPrefix設定はwaflog/のような設定であれば問題になりませんが、Firehoseによりデフォルトで付与される**プレフィックスYYYY/MM/DD/HHをカスタマイズすると、エラーになると思いますのでご注意ください。**
![image](https://user-images.githubusercontent.com/60077121/101999951-4f647500-3d25-11eb-88a0-717fac138fd6.png)


# 導入手順
## ①AWS WAFの設定（ALB等のリソースに割り当てるのは後で）
- ルールなしのWEB ACLを作成する（文献1参照）
- `Request sampling options`とは？（Disableにする）
  - 

## ②ALBのアクセスログ有効化
### なんのために？
- 文献2参照
```
「WafCharm の Web Site Config」 は ALB 1つにつき1つ作成し 「ALB のアクセスログ」 と紐づく設定です。
Web Site Config に登録する IAM User (Credential) が持つ権限は「登録した ALB のアクセスログ」が出力される S3 バケットに対するアクセス権になります。
またそのため、 WafCharm の利用には ALB から S3 へのアクセスログの出力が必要になります。
```
- ![image](https://user-images.githubusercontent.com/60077121/99891459-3419c300-2cad-11eb-98e9-12aa8ca514d6.png)

### 設定方法
- 文献3参照
- 以下のバケットポリシーが付与されたS3ができる
  - パブリックアクセス許可になっている。Elastic Load Balancing アカウント ID`582318560864`が書き込むため。
```
{
    "Version": "2012-10-17",
    "Id": "AWSConsole-AccessLogs-Policy-1606008181437",
    "Statement": [
        {
            "Sid": "AWSConsoleStmt-1606008181437",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::582318560864:root"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::elb-log-20201123/AWSLogs/913926916566/*"
        },
        {
            "Sid": "AWSLogDeliveryWrite",
            "Effect": "Allow",
            "Principal": {
                "Service": "delivery.logs.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::elb-log-20201123/AWSLogs/913926916566/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control"
                }
            }
        },
        {
            "Sid": "AWSLogDeliveryAclCheck",
            "Effect": "Allow",
            "Principal": {
                "Service": "delivery.logs.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::elb-log-20201123"
        }
    ]
}
```

## ③IAMの作成
- IAMユーザとIAMポリシーを作成する
- 仕組みとしてはWAFCharm用のIAMユーザーがプログラムによるアクセスでログインし、WAFを運用してくれる。
- 必要なポリシーは「AWSWAFFullAccess」と「AmazonS3ReadOnlyAccess」の2つ
### 以上でルールが設定される
- ![image](https://user-images.githubusercontent.com/60077121/101999272-b4689c80-3d1e-11eb-893c-69063a3ceea1.png)

# レポート通知機能の導入
## ④WAFログ出力設定
- 以下はレポート設定手順に記載あり。管理画面のHELPより参照できる。
### 概要
>AWS WAF のログを S3 バケットに出力するには、Kinesis Data Firehose を利用する必要があります。
>WafCharm の利用にこの設定は必須ではありませんが、後述する月次レポートの機能で利用します。（文献2）

### Name and Resource
- Kinesis Data Firehoseの配信ストリームを作成
  - **`aws-waf-logs-<任意の名前>`に設定しないとWAFと連携できない**
  - ポイントはWeb ACLに設定したリソースと同じリージョンを選択してData Firehoseのリソースを作成すること(CloudFrontならバージニア北部にリソースを作成します）
- Sourceに`Direct PUT or other sources`
- 暗号化はチェックせず次へ

### Process records
- Data transformation、Record format conversionは、`Disabeld`

### Choose a destination
- S3を選択
- WAFのログ出力用のバケットを選択
- **オプションでS3のプレフィックスを指定できる**
  - 未入力だとUTCの日付名のファイル？
  - `waf-log/`と入力してみる
  - https://docs.aws.amazon.com/ja_jp/firehose/latest/dev/s3-prefixes.html

### Configure settings
- 以下の設定
  - Buffer size: 5 MiB
  - Buffer interval: 60 seconds
  - S3 compression: GZIP
  - S3 encryption: Disabled
- IAMロールは新規作成してもらう（`KinesisFirehoseServiceRole-test-waf-ap-northeast-1-xxxx`ができる）

## ⑤WAFでログの有効化
- 東京リージョンへ移動
- ログのタブから有効化
- `Redacted`はセンシティブ情報を記録しない仕組み。チェックしない

## ⑥月次レポート のための AWS Lambda
![image](https://user-images.githubusercontent.com/60077121/101999217-30aeb000-3d1e-11eb-9499-def4b5a4b291.png)
- Helpページにソースあり。

### 概要
>AWS WAF のログをサイバーセキュリティクラウド (CSC) が管理する S3 バケットに転送することで、WafCharm の月次レポートが作成可能です。このためには S3 バケットに配置されたファイルをそのまま転送するように AWS Lambda を構築します。AWS Lambda に必要な権限(IAM Role)は、AWS WAF のログを出力している S3 バケットへの Read (Get 及び List) 権限と wafcharm.com という S3 バケットへの Put 権限 (PutObject, PutObjectAcl) です。
![image](https://user-images.githubusercontent.com/60077121/99891831-9eccfd80-2cb1-11eb-9a6e-cfe25ec2d668.png)

# 通知機能
- これもHELPのレポート・通知機能のPDFにマニュアルあり

## WafCharm の Web ACL Config 、 Web Site Configとは
