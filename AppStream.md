# AppStreamのメモ

## 参考資料
ブラックベルト
https://d1.awsstatic.com/webinars/jp/pdf/services/20191126_AWS-BlackBelt_AmazonAppStream2.0.pdf

良記事`AWS再入門ブログリレー Amazon AppStream 2.0 編`
https://dev.classmethod.jp/articles/re-introduction-2020-appstream2/

入門ガイド
https://aws.amazon.com/jp/appstream2/resources/?amazon-appstream-2-0.sort-by=item.additionalFields.postDateTime&amazon-appstream-2-0.sort-order=desc

<img width="914" alt="スクリーンショット 2020-09-12 23 21 52" src="https://user-images.githubusercontent.com/60077121/92997566-3940f000-f54f-11ea-963e-e580f9c9c5d0.png">

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
