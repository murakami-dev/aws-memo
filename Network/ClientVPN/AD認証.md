
## EC2にAD入れてAD Connector用のユーザ作成
- サーバマネージャからADをインストール
- 再起動
- しばらく待ってから再度RDP
- ユーザ作成（Domain AdminsとAD Connector用のユーザ）

- 以下の「権限をサービスアカウントに委任する」を参照)
  - https://docs.aws.amazon.com/ja_jp/directoryservice/latest/admin-guide/prereq_connector.html
![image](https://user-images.githubusercontent.com/60077121/98249118-3b588580-1fb9-11eb-8c5b-30419ef01cfd.png)

## AD Connector作成
- ADでインバウンドのポート空けること
  - TCP/UDP 53 - DNS
  - TCP/UDP 88 - Kerberos 認証
  - TCP/UDP 389 - LDAP

## 証明書をACMへアップロード
- 別で作成しzip化しておいた`easy-rsa.zip`を使うものとする。詳細は[easy-rsaの検証](https://github.com/murakami-dev/study-memo/blob/master/Network/ClientVPN/easy-rsa%E3%81%AE%E6%A4%9C%E8%A8%BC.md)

- ローカルでzip展開してACMへインポート
  - サーバ証明書とCA
    - pki/issuedにあるcrtをコピペ（証明書本文）
    - pki/privateにあるkeyをコピペ(プライベートキー)
    - ca.crtも（キーチェン、CA局）
  - クライアント証明書（相互認証する場合はクライアント証明書もACMにアップする必要あり）
  - `tld`とあるのはクライアント。

### 以下は手間かかるやり方（EC2へアップしてACMへCLIからアップ・割愛）
- 適当にLinuxのEC2たてて、unzip
- `yum update`しないとzipファイルの転送がうまくいかない
![image](https://user-images.githubusercontent.com/60077121/98253282-2b8f7000-1fbe-11eb-8775-06f4acd6bf14.png)
![image](https://user-images.githubusercontent.com/60077121/98253978-10713000-1fbf-11eb-9dab-dab88775cca1.png)


```
[ec2-user@ip-10-123-10-212 rsa-test]$ ls
easy-rsa.zip
[ec2-user@ip-10-123-10-212 rsa-test]$ unzip easy-rsa
Archive:  easy-rsa.zip
  inflating: easy-rsa/op_test.orig
  inflating: easy-rsa/ChangeLog
 ・・・中略
  inflating: easy-rsa/.travis.yml
[ec2-user@ip-10-123-10-212 rsa-test]$ ls
easy-rsa  easy-rsa.zip
```

## clientVPNの作成

## 設定ファイルの配布（セルフサービスポータルを使う）
https://dev.classmethod.jp/articles/control-bgp-route-on-site-to-site-vpn/

- ADの認証情報でログインする形式（相互認証のみのパターンだと使えないらしい）
- ログイン後、以下から設定ファイルをダウンロード
![image](https://user-images.githubusercontent.com/60077121/98258119-d3f40300-1fc3-11eb-8111-d650a899106c.png)

- クライアントソフトの場合、ファイル⇒プロファイルを追加から設定ファイルをアップする
![image](https://user-images.githubusercontent.com/60077121/98258340-19183500-1fc4-11eb-84f6-7841cadc7e88.png)


