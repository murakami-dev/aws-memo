# 文献
- 1.[新機能Route53 リゾルバー登場! オンプレミス-VPCの相互名前解決が簡単に実現できるようになりました!](https://dev.classmethod.jp/articles/route53-resolver/)
  - 現手法。手順はブラックベルト末尾の方が分かりやすいかも。
- 2.[AWS Directory Service(Simple AD)を利用したオンプレミスからのPrivate Hosted Zone名前解決](https://dev.classmethod.jp/articles/private-dns-forwarder-by-simple-ad/)
  - 以前の手法。
- 3.[Simple ADのDNSレコードを編集する](https://dev.classmethod.jp/articles/edit-simple-ad-dns-record/)
  - 以前の手法。とてもわかりやすい
- 4.[AWSハイブリッド構成のDNS設計レシピ](https://dev.classmethod.jp/articles/aws-hybrid-cloud-dns-designs/)
  - 「Route53forHybridCloudsが使えないからSimpleAD」は安直すぎ。設計の概念を知る  

# 概要
- 3つのコンポーネント
  - インバウンドエンドポイント
  - アウトバウンドエンドポイント
  - リゾルバールール
![image](https://user-images.githubusercontent.com/60077121/102027269-51dad380-3de6-11eb-88aa-36873a289b32.png)

## ユースケース
![image](https://user-images.githubusercontent.com/60077121/102027524-14774580-3de8-11eb-8d6e-32796981df08.png)


# 個人的な検証
## VPC内・VPC間の名前解決について

- VPC内のインスタンスのプライベートDNSを引いてみる
  - 引けた
```
sh-4.2$ dig ip-10-123-10-33.ap-northeast-1.compute.internal +short
10.123.10.33
```

- VPC間のインスタンスのプライベートDNSを引いてみる（ピアリングで試した）
  - 引けない
```
[ec2-user@ip-10-0-10-129 ~]$ dig ip-10-123-11-122.ap-northeast-1.compute.internal +short
[ec2-user@ip-10-0-10-129 ~]$
```
