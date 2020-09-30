# AppStreamのメモ

## 参考資料
ブラックベルト
https://d1.awsstatic.com/webinars/jp/pdf/services/20191126_AWS-BlackBelt_AmazonAppStream2.0.pdf

良記事`AWS再入門ブログリレー Amazon AppStream 2.0 編`

https://dev.classmethod.jp/articles/re-introduction-2020-appstream2/

入門ガイド
https://aws.amazon.com/jp/appstream2/resources/?amazon-appstream-2-0.sort-by=item.additionalFields.postDateTime&amazon-appstream-2-0.sort-order=desc

<img width="914" alt="スクリーンショット 2020-09-12 23 21 52" src="https://user-images.githubusercontent.com/60077121/92997566-3940f000-f54f-11ea-963e-e580f9c9c5d0.png">


## クラメソブログに沿ってやってみる
※再掲
https://dev.classmethod.jp/articles/re-introduction-2020-appstream2/

### ユーザ作成

![image](https://user-images.githubusercontent.com/60077121/93658154-f95c9a00-fa73-11ea-9920-16df61c48b1b.png)

### イメージビルダー作成
![image](https://user-images.githubusercontent.com/60077121/93658203-7ab42c80-fa74-11ea-97be-c28bac6a60d1.png)

イメージの選択
![image](https://user-images.githubusercontent.com/60077121/93658311-615fb000-fa75-11ea-93ae-9cb8379365bc.png)

インスタンスタイプの選択  
通常のWindows Serverなので汎用インスタンス、コンピューティング最適化インスタンス、メモリ最適化インスタンスしかない
- IAMロール：AWS 認証情報を手動で管理することなく、Image Builder インスタンスから AWS API リクエストを行うことができます。
![image](https://user-images.githubusercontent.com/60077121/93658384-15613b00-fa76-11ea-8286-08a395e3e4e2.png)

料金について
https://aws.amazon.com/jp/appstream2/pricing/

ネットワーク設定
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/internet-access.html
![image](https://user-images.githubusercontent.com/60077121/93658663-a0dbcb80-fa78-11ea-829a-692b68920b23.png)

接続
![image](https://user-images.githubusercontent.com/60077121/93658971-5b6ccd80-fa7b-11ea-9876-6bdd675382ec.png)

Administratorでログイン

![image](https://user-images.githubusercontent.com/60077121/93658980-70496100-fa7b-11ea-8b60-6042de7fb2de.png)

インターネット経由でアプリの実行ファイルをダウンロード（ここではTeratermをインストールした）

![image](https://user-images.githubusercontent.com/60077121/93666678-110b4100-fabb-11ea-91a8-e2c011d4ae8f.png)

※インターネット経由できないものはS3を経由する
AppStream2.0 インターネット経由でインストールできないアプリをS3経由でインストールする
https://dev.classmethod.jp/articles/appstream2-install-from-s3/


イメージビルダーインスタンスで起動した時点で文字化けした

* `Set-TimeZone -Id "Tokyo Standard Time"`でタイムゾーン設定
* 設定の画面から日本語をインストール
https://blog.trainocate.co.jp/blog/azure_setting_024
![image](https://user-images.githubusercontent.com/60077121/93668276-57b26880-fac6-11ea-8eba-39676b8f4e0a.png)
![image](https://user-images.githubusercontent.com/60077121/93668951-4881e980-facb-11ea-9ca9-3ce13edfb5fb.png)
![image](https://user-images.githubusercontent.com/60077121/93668991-93036600-facb-11ea-8873-37de7f69f8e2.png)

### 最後にマネコンからイメージビルダーインスタンスをstop->runningにすることで再起動され日本語になった



Add Fileからアプリの実行ファイルを選択
![image](https://user-images.githubusercontent.com/60077121/93666731-7eb76d00-fabb-11ea-9017-ffae2104a65c.png)
![image](https://user-images.githubusercontent.com/60077121/93666756-b1f9fc00-fabb-11ea-8845-f42df6fc359d.png)

Switch User

![image](https://user-images.githubusercontent.com/60077121/93667363-58e09700-fac0-11ea-9bd4-fd31decbd18e.png)

テストユーザーに切り替えて動作確認を行う

![image](https://user-images.githubusercontent.com/60077121/93669067-30f73080-facc-11ea-8723-ac13414c05ab.png)

問題なければローンチする
名前など決める
![image](https://user-images.githubusercontent.com/60077121/93669103-85021500-facc-11ea-8fc3-0af1779ea954.png)

うまくいけばセッションが終了する

イメージビルダーのステータスは以下のようになる
`イメージが作成されると、Image Builder のステータスが [Snapshotting] と表示されます。このプロセスが終了するまで Image Builder に接続することはできません。`
2-30分かかった
Snapshottingが終了するとイメージビルダーインスタンスは`stopped`になってくれる。


### フリートの作成
![image](https://user-images.githubusercontent.com/60077121/93669223-94359280-facd-11ea-8bd9-b30a2c353686.png)

イメージの選択
![image](https://user-images.githubusercontent.com/60077121/93693744-b1567980-fb3e-11ea-8a7a-f29e1dd16a79.png)

フリートの設定
![image](https://user-images.githubusercontent.com/60077121/93693805-989a9380-fb3f-11ea-92c6-dc41254a2062.png)

ネットワークの設定
イメージビルダーインスタンスはパブリックサブネット、フリートインスタンスはプライベートとかもできる
![image](https://user-images.githubusercontent.com/60077121/93693828-efa06880-fb3f-11ea-8fea-3b433edda165.png)

### スタックの作成
https://docs.aws.amazon.com/appstream2/latest/developerguide/set-up-stacks-fleets.html#set-up-stacks-fleets-install
アプリケーション設定の永続性のしくみ
https://docs.aws.amazon.com/appstream2/latest/developerguide/how-it-works-app-settings-persistence.html

スタックの設定
![image](https://user-images.githubusercontent.com/60077121/93694026-ffb94780-fb41-11ea-91d1-dae2803cd41c.png)

ストレージの設定
![image](https://user-images.githubusercontent.com/60077121/93694045-27a8ab00-fb42-11ea-8011-42f8d51c26bb.png)

### ユーザーとスタックの紐づけ
ユーザーをスタックにアサイン
![image](https://user-images.githubusercontent.com/60077121/93694122-01cfd600-fb43-11ea-9dfe-75252828c01c.png)

ユーザーにメールが届く
![image](https://user-images.githubusercontent.com/60077121/93694131-2a57d000-fb43-11ea-857b-112ca8a9a0ba.png)

ユーザー登録時の初期パスワードでログインするとアプリが使用できる
![image](https://user-images.githubusercontent.com/60077121/93694256-80794300-fb44-11ea-9461-16a8a5b10eb6.png)

※クライアントソフトからアクセスする場合は以下
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/client-application.html

※オンデマンドの場合アプリの起動には2分ほどかかる

#### ローカル->フリートへファイルのコピー（スタック設定に依る）
左上のメニューからフリートのホームフォルダへアクセスできる
![image](https://user-images.githubusercontent.com/60077121/93694316-26c54880-fb45-11ea-98be-58be1cbf5b10.png)


## チュートリアル
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/getting-started.html

## ステップ1 サンプルスタック作成、イメージ選択、フリート作成
スタックの作成
<img width="1359" alt="スクリーンショット 2020-09-12 23 33 30" src="https://user-images.githubusercontent.com/60077121/92997750-6e017700-f550-11ea-9420-8b3bbe0b45d5.png">

イメージの選択
<img width="1420" alt="スクリーンショット 2020-09-13 6 55 23" src="https://user-images.githubusercontent.com/60077121/93005670-2b5e8f80-f58e-11ea-9178-b2fe9514f36d.png">

フリートの設定
![screencapture-ap-northeast-1-console-aws-amazon-appstream2-home-2020-09-13-07_00_27](https://user-images.githubusercontent.com/60077121/93005743-e6872880-f58e-11ea-8b3e-9dfaa19e1460.png)

ネットワークの設定
インターネットアクセスの場合、パブリックサブネットしか選べない。インターネットアクセスのチェックを外すとプライベートサブネット選択できる。これはフリートを一時停止すれば後から変更可能
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/managing-network-manual-enable-internet-access.html
<img width="1142" alt="スクリーンショット 2020-09-13 7 13 17" src="https://user-images.githubusercontent.com/60077121/93005882-acb72180-f590-11ea-800e-b71e18175b5b.png">

ストレージ
![screencapture-ap-northeast-1-console-aws-amazon-appstream2-home-2020-09-13-07_24_31](https://user-images.githubusercontent.com/60077121/93006047-77abce80-f592-11ea-8bd4-e3166b166ff0.png)

UserSetting
<img width="1412" alt="スクリーンショット 2020-09-13 7 40 23" src="https://user-images.githubusercontent.com/60077121/93006215-6d8acf80-f594-11ea-954a-6c774238a36c.png">

## ステップ2
```
AppStream 2.0 ユーザープールを介してアクセスを提供する場合、ストリーミングセッションにウェブブラウザを使用する必要があります。
SAML 2.0 [シングルサインオン (SSO)] または AppStream 2.0 API を使用してユーザーにアクセスを提供する予定の場合は、AppStream 2.0 クライアントを使用できるようにすることができます。
```
<img width="1420" alt="スクリーンショット 2020-09-13 9 05 09" src="https://user-images.githubusercontent.com/60077121/93007153-56ea7580-f5a0-11ea-8f3b-2ba5c4a50fcb.png">

アプリ一覧からアプリの起動には2分ほど要する
<img width="926" alt="スクリーンショット 2020-09-13 9 28 15" src="https://user-images.githubusercontent.com/60077121/93007387-91a1dd00-f5a3-11ea-8170-a1a36c59910d.png">

# ドキュメントのメモ
## VPCセットアップの推奨事項
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/vpc-setup-recommendations.html
```
ストリーミングインスタンス（フリートインスタンスまたはイメージビルダー）にインターネットへのアクセスを提供することを計画している場合、ストリーミングインスタンス用の2つのプライベートサブネットとパブリックサブネット内のNATゲートウェイでVPCを構成することをお勧めします。
```
また、関連づけるサブネットを2つ指定することで、可用性と容量不足を防ぐ

### ENIのCIDRおよびポートについて
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/appstream2-port-requirements-appstream2.html

### AppStream 2.0ユーザーデバイスのIPアドレスとポートの要件
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/client-application-ports.html

### インターフェイスVPCエンドポイントからの作成とストリーミング
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/creating-streaming-from-interface-vpc-endpoints.html
