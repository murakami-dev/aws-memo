# やりたいこと
- EFSからS3へバックアップ

# 文献
- [DataSync を使用して Amazon EFS ファイルシステムから Amazon S3 バケットにデータを転送する方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/datasync-transfer-efs-s3/)
  - ちょっと古いかも。エージェントは不要
- [DataSync を使用して、プライベートネットワーク経由で異なるリージョンの Amazon EFS ファイルシステム間でデータを転送するにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/datasync-transfer-efs-cross-region/)
- [AWS DataSync を使用して VPC を離れることなく、オンプレミスから AWS にファイルを転送する](https://aws.amazon.com/jp/blogs/news/transferring-files-from-on-premises-to-aws-and-back-without-leaving-your-vpc-using-aws-datasync/)
- [AWS DataSync を使用して Amazon FSx for Windows File Server に移行する](https://aws.amazon.com/jp/blogs/news/migrate-to-amazon-fsx-for-windows-file-server-using-aws-datasync/)
  - オンプレからの移行
# 手順
## EFS作成

## S3作成
- 普通に作成

## DataSync設定


## ハマった
### DataSyncからのアクセスをEFSで許可する必要あり
- DataSyncにアタッチするSGをソースにしてEFSのSGで許可する必要がある。
  - >Error
  - >Task failed to access location loc-0cec9936205a6243a: x40016: Failed to connect to EFS mount target with IP: 10.123.21.226. Please ensure that mount target's security group allows 2049 ingress from the DataSync security group or hosts within the mount target's subnet. The DataSync security group should also allow all egress to the EFS mount target and its security group.


