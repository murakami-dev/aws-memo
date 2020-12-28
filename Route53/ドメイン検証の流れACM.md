# やること
- EC2にウェブサーバ（以下、仮想オンプレサーバとする）
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

# 手順
## EC2にウェブサーバ構築（仮想オンプレサーバ）
割愛
## Route53に仮想オンプレサーバのIPアドレスのAレコードを登録
- パブリックホストゾーンの作成、Aレコードの登録
- レジストラでRoute53のネームサーバを使用するよう、Route53のNSレコードを登録
- [ネームサーバー/DNSについて](https://www.onamae.com/guide/p/67)
  - 24h~とあるがすぐに反映された

```
C:\Users\user>nslookup -type=ns stg-development.work
サーバー:  buffalo.setup
Address:  192.168.11.1

権限のない回答:
stg-development.work    nameserver = ns-1522.awsdns-62.org
stg-development.work    nameserver = ns-1771.awsdns-29.co.uk
stg-development.work    nameserver = ns-414.awsdns-51.com
stg-development.work    nameserver = ns-898.awsdns-48.net
```
![image](https://user-images.githubusercontent.com/60077121/103218060-61692900-495d-11eb-8565-1e238b130c04.png)

## ACMで証明書取得、ドメイン検証
- 下記の操作後、すぐに検証状態が成功になった
![image](https://user-images.githubusercontent.com/60077121/103218481-72666a00-495e-11eb-8da2-916d76f47c75.png)

