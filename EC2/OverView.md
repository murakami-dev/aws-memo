
### 文献（基本は項目ごとに掲載する、ここには未作成の文献をメモ的に載せる）

- [InstanceMetaDataV2を分かりやすく解説してみる](https://blog.serverworks.co.jp/tech/2019/11/27/imdsv2/)
- [Windows EC2インスタンスのハイバネーション(休止)を試してみた](https://dev.classmethod.jp/articles/ec2-windows-support-hibernation/)

# Linuxのドキュメントを基にする
## AMI
- AMIに含まれるもの
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-instances-and-amis.html
>AMI に含まれるものとして、ウェブサーバー、関連する静的コンテンツ、動的ページ用のコードが考えられます。この AMI からインスタンスを起動すると、ウェブサーバーが起動し、アプリケーションはリクエストを受け付け可能な状態になります。

- ストレージ情報も含まれる
>AMI の説明に、ルートデバイスのタイプ (ebs または instance store) が明記されています。

- AMIタイプ
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ComponentsAMIs.html#storage-for-the-root-device
  - パブリックか特定のアカウントに起動許可を与えるか
  - Amazon EC2 instance store-backed インスタンスは停止できない（他の違いは上記参照）
  - 課金は秒で計算される

## シャットダウン操作
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingInstanceInitiatedShutdownBehavior


これまとめたい
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-launch-templates.html
