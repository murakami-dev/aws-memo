# 目的
- Gateway型とインターフェース型の違い、経路など

# ゲートウェイ型
- 1.[2つのVPCエンドポイントの違いを知る](https://dev.classmethod.jp/articles/vpc-endpoint-gateway-type/#toc-3)
- 2.[VPC Endpointを使ってS3にアクセスしてみる](https://blog.serverworks.co.jp/tech/2015/08/31/vpc-endpoint/)


## ポイント
### VPCE->S3はパブリックIPで疎通するけどAWSクラウド内だよ
- EC2 -> VPCEへアクセスする際、名前解決後のIPアドレスはグルーバルIPになる。
  - ![image](https://user-images.githubusercontent.com/60077121/109763279-35a44100-7c35-11eb-8d10-2330c9bb99f7.png)
  - >S3のエンドポイントがグローバルIPのままであるということは、ゲートウェイ型のVPCエンドポイントの１つの特徴です。接続元のEC2インスタンスはプライベートIPのみにも関わらず、ゲートウェイ型のVPCエンドポイントが何らか上手いことやってくれて、そのままS3にアクセスできるようにしてくれます。ただし、エンドポイントのIPはグローバルIPとなるので、ネットワークACLでローカルのIPのみという制限を加えると、エンドポイントを利用しても通信が出来なくなります。
- でもIGWからインターネット（AWSクラウド外）へ出るのではない
  - >文献2では`curl`で8.8.8.8はいけないけどS3にはいけることを確認している


# インターフェイス型
- 1.[Amazon S3 の PrivateLink が他のサービスと異なるたった1つの点](https://blog.serverworks.co.jp/only-thing-that-makes-Amazon-S3-PrivateLink-different-from-other-services)
  - >`bucketname.vpce-xxxx.s3.ap-northeast-1.vpce.amazonaws.com`というURLが与えられ、これを名前解決するとプライベートIPが返される。
  - つまりRDSやinternal ALBのDNSと一緒。グルーバルで名前解決可能
