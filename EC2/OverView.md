
### 文献（基本は項目ごとに掲載する、ここには未作成の文献をメモ的に載せる）

- [InstanceMetaDataV2を分かりやすく解説してみる](https://blog.serverworks.co.jp/tech/2019/11/27/imdsv2/)
- [Windows EC2インスタンスのハイバネーション(休止)を試してみた](https://dev.classmethod.jp/articles/ec2-windows-support-hibernation/)

# Linuxのドキュメントを基にする
## AMI
- AMIに含まれるもの
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-instances-and-amis.html
>AMI に含まれるものとして、ウェブサーバー、関連する静的コンテンツ、動的ページ用のコードが考えられます。この AMI からインスタンスを起動すると、ウェブサーバーが起動し、アプリケーションはリクエストを受け付け可能な状態になります。

- ストレージ情報も含まれる（AMIにはEBSスナップショットが紐づいている）
>AMI の説明に、ルートデバイスのタイプ (ebs または instance store) が明記されています。

- AMIタイプ
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ComponentsAMIs.html#storage-for-the-root-device
  - パブリックか特定のアカウントに起動許可を与えるか
  - Amazon EC2 instance store-backed インスタンスは停止できない（他の違いは上記参照）
  - 課金は秒で計算される
  
### Linux AMI 仮想化タイプ
- HVMが推奨
>2 つの仮想化タイプ (準仮想化 (PV) およびハードウェア仮想マシン (HVM)) のどちらかを使用します。PV AMI と HVM AMI の主な違いは、起動の方法と、パフォーマンス向上のための特別なハードウェア拡張機能 (CPU、ネットワーク、ストレージ) を利用できるかどうかという点です。

### AMI検索方法
- https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/finding-an-ami.html

### 暗号化されたEBS-backed AMIの挙動
- https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/AMIEncryption.html
- もともとのAMIのEBSが暗号化されていれば、起動後も暗号化はされる。
- 下記いろんなパターンがドキュメントにある。

| 元々のAMIに紐づいたスナップショットが暗号化されている | 自己所有のAMIである |起動時に`Encrypted`を指定する | 起動時に`KmsKeyId`を指定する | 結果 |
----|---- |---- |---- |---- 
| 〇 | 〇 | 〇（×でも一緒） | 〇 | `KmsKeyId`で指定したCMKで暗号化される |
| × | 〇 | × | - | 暗号化されない（オーソドックスなパターン）|

## ルートデバイスボリューム
- instance store-backed AMI または Amazon EBS-backed AMI
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/RootDeviceStorage.html
### 永続的ルートボリュームへの変更
- `DeleteOnTermination`（終了に合わせて削除）をfalseにする。
>デフォルトでは、Amazon EBS-backed AMI のルートボリュームは、インスタンスを終了すると削除されます。インスタンスの終了後もボリュームが永続化するように、デフォルトの動作を変更できます。デフォルトの動作を変更するには、ブロックデバイスマッピングを使用して、DeleteOnTermination 属性を false に設定します。


## シャットダウン操作
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingInstanceInitiatedShutdownBehavior


これまとめたい
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-launch-templates.html
