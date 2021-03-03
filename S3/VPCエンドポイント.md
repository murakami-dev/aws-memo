# 目的
- Gateway型とインターフェース型の違い、経路など

# ゲートウェイ型
- 1.[2つのVPCエンドポイントの違いを知る](https://dev.classmethod.jp/articles/vpc-endpoint-gateway-type/#toc-3)
- 2.[VPC Endpointを使ってS3にアクセスしてみる](https://blog.serverworks.co.jp/tech/2015/08/31/vpc-endpoint/)
- 3.[S3 バケットと EC2 インスタンス間でデータをコピーするときの転送速度を向上させるにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/s3-transfer-data-bucket-instance/)


## ポイント
### VPCE->S3はパブリックIPで疎通するけどAWSクラウド内だよ
- EC2 -> VPCEへアクセスする際、名前解決後のIPアドレスはグルーバルIPになる。
  - ![image](https://user-images.githubusercontent.com/60077121/109763279-35a44100-7c35-11eb-8d10-2330c9bb99f7.png)
  - >S3のエンドポイントがグローバルIPのままであるということは、ゲートウェイ型のVPCエンドポイントの１つの特徴です。接続元のEC2インスタンスはプライベートIPのみにも関わらず、ゲートウェイ型のVPCエンドポイントが何らか上手いことやってくれて、そのままS3にアクセスできるようにしてくれます。ただし、エンドポイントのIPはグローバルIPとなるので、ネットワークACLでローカルのIPのみという制限を加えると、エンドポイントを利用しても通信が出来なくなります。
- でもIGWからインターネット（AWSクラウド外）へ出るのではない
  - >文献2では`curl`で8.8.8.8はいけないけどS3にはいけることを確認している

### ↑を追記
#### プライベートな接続できるし速度も上昇
- 文献3より。最終更新日が2021/1/5（インターフェイス型非対応の時点）となっているのでゲートウェイ型でも**プライベートに接続できる（AWSネットワーク上）は言っても良さそう**
- >EC2 インスタンスが S3 バケットと同じリージョンにある場合は、Amazon S3 用 VPC エンドポイントの使用を検討してください。VPC エンドポイントは、全体的なパフォーマンスを向上させ、ネットワークアドレス変換 (NAT) の負荷を軽減するのに役立ちます。
- >VPC エンドポイントを使用するもう 1 つの利点は、インターネットゲートウェイ、NAT デバイス、または VPN 接続を使用せずに VPC にプライベートに接続できることです。VPC 内のインスタンスは、Amazon S3 バケットのようなリソースと通信するのにパブリック IP アドレスを必要としません。VPC エンドポイントを使用すると、VPC と Amazon S3 の間のデータトラフィックは AWS ネットワーク上でルーティングされます。

# インターフェイス型
- 1.[Amazon S3 の PrivateLink が他のサービスと異なるたった1つの点](https://blog.serverworks.co.jp/only-thing-that-makes-Amazon-S3-PrivateLink-different-from-other-services)
  - >`bucketname.vpce-xxxx.s3.ap-northeast-1.vpce.amazonaws.com`というURLが与えられ、これを名前解決するとプライベートIPが返される。
  - つまりRDSやinternal ALBのDNSと一緒。グルーバルで名前解決可能
