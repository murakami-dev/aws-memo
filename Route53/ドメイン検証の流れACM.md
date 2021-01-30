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
### Aレコードが変更されたことを確認
```
C:\Users\user>nslookup stg-development.work
サーバー:  UnKnown
Address:  172.20.10.1

権限のない回答:
名前:    stg-development.work
Address:  54.199.234.1
```
![image](https://user-images.githubusercontent.com/60077121/106347883-3d568a00-6305-11eb-98ac-cd96adb75cf6.png)


## ACMで証明書取得、ドメイン検証
- 下記の操作後、すぐに検証状態が成功になった
- ドメイン単体とワイルドカード付ドメインで取得
  - `stg.development.work`と`*.stg.development.work`
![image](https://user-images.githubusercontent.com/60077121/103218481-72666a00-495e-11eb-8da2-916d76f47c75.png)
![image](https://user-images.githubusercontent.com/60077121/103222409-a5612b80-4967-11eb-82b2-b274270b1e08.png)

## EC2にウェブサーバ（以下、新EC2とする）
割愛

## ELB構築、新EC2をぶら下げる
- `https://ELBのドメイン`で検索
![image](https://user-images.githubusercontent.com/60077121/103221193-6631db00-4965-11eb-846e-3385d6b6851b.png)

## Route53にレコード登録
### 同じレコードは登録できない
- レコードの編集でIPアドレスへのAレコードからALBへのエイリアスへ変更する
- NSレコードのTTLを短くする
  - [ステップ 4: TTL 設定を下げる](https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html#migrate-dns-lower-ttl)
![image](https://user-images.githubusercontent.com/60077121/103224260-c4ad8800-496a-11eb-933f-02b0ecd16e93.png)

### ブラウザのキャッシュをクリアしないと反映されない
Ghromeの場合、その他のツール→閲覧履歴の消去→消去

### ローカルのフルサービスリゾルバのキャッシュもクリアする
```
C:\Users\user>ipconfig /flushdns

Windows IP 構成

DNS リゾルバー キャッシュは正常にフラッシュされました。

C:\Users\user>ipconfig /displaydns
ドメインのキャッシュがないことを確認
```

### これで反映される
![image](https://user-images.githubusercontent.com/60077121/103223559-db071400-4969-11eb-9983-4834cbad6ccf.png)
