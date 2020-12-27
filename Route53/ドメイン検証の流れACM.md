# やること
- EC2にウェブサーバ（以下、オンプレEC2とする）
- Route53に取得ドメイン→オンプレEC2のIPアドレスのAレコード
- ACMで証明書取得
- ドメイン検証（CNAMEだけ登録でいけるか
- EC2にウェブサーバ（以下、新EC2とする）
- ELB構築、新EC2をぶら下げる
- 切り替えをどうするか

# 文献
- [Route 53 を使用中のドメインの DNS サービスにする](https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html)
  - TTLを下げるなど、作法が載っている
- [稼働中のDNSをRoute 53へ移行するその前にRoute 53の設定値を既存レコードと比較する](https://dev.classmethod.jp/articles/compare-route53-records-with-existing-dns-records/)
  - 上記をかみ砕いたもの
