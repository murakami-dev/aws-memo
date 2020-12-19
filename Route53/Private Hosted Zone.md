# 文献
- 1.[VPC内での名前解決を有効にする Route53 のプライベートホストゾーンを使ってみた](https://dev.classmethod.jp/articles/route53-private-hostzone-beginner/)
  - 超基本
- 2.[【Route53】プライベートホストゾーンを使って初めて理解した3つのこと](https://dev.classmethod.jp/articles/route53-privatehostzone-3tips/)

## 考慮事項
- 文献2より
  - プライベートホストゾーンにNSレコードは登録できない（下図参照）
  - プライベートホストゾーンで名前空間は重複してよい
    - 例えばexample.comというホストゾーンとsub.example.comというホストゾーンを、同一のVPCに関連づけることができます。
  - プライベートホストゾーンでACM証明書発行時のDNS検証はできない
![image](https://user-images.githubusercontent.com/60077121/102012299-f6302c00-3d8c-11eb-95dd-caac4a1cb321.png)

# 個人検証
## PrivateHostedZoneを使ってみる
![image](https://user-images.githubusercontent.com/60077121/102682460-f73ddf00-420c-11eb-829f-51d698fa3519.png)


```
[ec2-user@radius ~]$ dig myad.test +short
ip-10-123-10-31.ap-northeast-1.compute.internal.
10.123.10.31
```
