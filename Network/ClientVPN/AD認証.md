
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

- 証明書本文・鍵へのパス、ドメインの前にランダム文字列を入れる必要あり
```
client
dev tun
proto udp
remote asdfa.cvpn-endpoint-0b32df75b779c0cf7.prod.clientvpn.ap-northeast-1.amazonaws.com 443
remote-random-hostname
resolv-retry infinite
nobind
remote-cert-tls server
cipher AES-256-GCM
verb 3
<ca>
-----BEGIN CERTIFICATE-----
MIIDPjCCAiagAwIBAgIJAIOA3Fa9S6fjMA0GCSqGSIb3DQEBCwUAMBkxFzAVBgNV
BAMMDktPS1VZTyBDby4sTHRkMB4XDTIwMDIyODEwMTUwM1oXDTMwMDIyNTEwMTUw
M1owGTEXMBUGA1UEAwwOS09LVVlPIENvLixMdGQwggEiMA0GCSqGSIb3DQEBAQUA
A4IBDwAwggEKAoIBAQCkDOUZJWPkCviBPcjuLLdTykkrmf6o52oCdmRgmbWaxlBJ
ips+EKzr1UJ3dYfwoiz16UyFkGYefSMZUEKseXGFoabZtITxT8C84c+R2SxNsEjV
e+wsUpL4VQ5u2m8Rpw6Unslamk1UsvbENaGgW/q1W2SmE0TXKlj8HbyRZ/jMxgu9
jbot6vLFAnSyCtvqgOL0rie/m29iqj4bG3qd7dfvcKSXLr8+HM8R8pGTuqvbA6uw
38/VNdQ9TfXieE1W6s2Z706D/jHwc7NJW4dRnBZSG/b6Kdr3aHUx3TVTklNaqB/y
JohHBM3LgWufNLN3DG5V35yfyjC4RdzWKtH2vYSBAgMBAAGjgYgwgYUwHQYDVR0O
BBYEFNpeghKuPoELWkw8Bkv7zrdvQXO1MEkGA1UdIwRCMECAFNpeghKuPoELWkw8
Bkv7zrdvQXO1oR2kGzAZMRcwFQYDVQQDDA5LT0tVWU8gQ28uLEx0ZIIJAIOA3Fa9
S6fjMAwGA1UdEwQFMAMBAf8wCwYDVR0PBAQDAgEGMA0GCSqGSIb3DQEBCwUAA4IB
AQAKjxT2sSrhBnAPxOoSqbFl294g2KNyRsXljEzyEg/pMkcScMvTx0TJ+CEIXYSj
RSINJPL+Lcc31uyMs4ZwFu68vTAwuA6QlpB1qrQK6Oo7reytuwa72TpAlqQURuG2
MtEHPcLwrlfpXkmc0LhjxliLiVyP2d2sGRH6QfYjVzzwOAqpa4cDtZMCf8WO8tGH
5SI5KhOeyYrzQtST4yJ1ds1NI2G2xdfxhsfiQm5IZrJUga9M9KYZ48mlNU7Fi2Ks
yvrUUQyUejIAqL+FhlhTO/47YELfVWrBYVN06gNNmzfix/mOqU5LdxwRQz2Za6TT
c4FjN1n17i2CYgGLKMlI1bUf
-----END CERTIFICATE-----

</ca>
auth-user-pass

reneg-sec 0

cert C:\\Users\\user\\OpenVPN\\config\\kokuyo-test\\common-client.domain.tld.crt
key C:\\Users\\user\\OpenVPN\\config\\kokuyo-test\\common-client.domain.tld.key
```
