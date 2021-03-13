# 目的
- 01で設定したSMTPサーバからメールを送ったが届かず。dkimとSPFをやってみる

# 結果
- 以下の設定をしてもコネクションタイムアウトになってメール送信できなかった。多分25番アウトバウンド空いていないから
- `/var/log/maillog`
```
connect to aspmx5.googlemail.com[209.85.146.26]:25: Connection timed out
```
- 25番で制限されている場合も上記のようなログはでるとのこと
- 下記のブログでは制限解除の申請してる
- https://www.chocoyoung.com/post-559

# SPF
- [送信ドメインを認証するためのSPFレコードに詳しくなろう](https://sendgrid.kke.co.jp/blog/?p=3509)
### こんな感じで登録した
- ![image](https://user-images.githubusercontent.com/60077121/110047165-b9bf0b80-7d90-11eb-9c2e-ee8f62e7e8c6.png)

# dkim
- 1.[CentOS Postfix 迷惑メール判定されないDKIM設定（複数ドメイン）](https://rin-ka.net/centos-postfix-dkim/#toc5)
- 2.[OpenDKIM で迷惑メール判定を回避する](https://qiita.com/bezeklik/items/a3619b5abc01ab38bec4)
  - これも見やすい
## OPENdkimをインストール
### EPELインストール
- 割愛

### dkimインストール
```
 sudo yum install opendkim -y
```
- /etc/opendkimにkeysディレクトリ出来るのでその中にドメイン名のディレクトリ作成し、その中に鍵を作成したい
```
[ec2-user@ip-10-123-10-6 opendkim]$ ls
keys  KeyTable  SigningTable  TrustedHosts
```
### Perl Getopt::Long モジュールのインストール
- opendkimでキーペア（秘密鍵／公開鍵）を作成する際にPerl Getopt::Long モジュールが必要になります。  
  - https://rin-ka.net/centos-postfix-dkim/#toc5
- 普通にモジュールなしでやろうとしたらエラーになる
```
[root@ip-10-123-10-6 keys]# opendkim-genkey -D /etc/opendkim/keys/stg-development.work/ -d stg-development.work -s default --verbose
Can't locate Getopt/Long.pm in @INC (you may need to install the Getopt::Long module) (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5) at /usr/sbin/opendkim-genkey line 25.
BEGIN failed--compilation aborted at /usr/sbin/opendkim-genkey line 25.
```

- インストール
```
[root@ip-10-123-10-6 keys]# yum -y install perl-Getopt-Long.noarch
```

## key作成
```
[root@ip-10-123-10-6 keys]# opendkim-genkey -D /etc/opendkim/keys/stg-development.work/ -d stg-development.work -s default --verbose
opendkim-genkey: generating private key
opendkim-genkey: private key written to default.private
opendkim-genkey: extracting public key
opendkim-genkey: DNS TXT record written to default.txt
```

### dkimレコード確認
```
[root@ip-10-123-10-6 stg-development.work]# ls
default.private  default.txt
[root@ip-10-123-10-6 stg-development.work]# cat default.txt
default._domainkey      IN      TXT     ( "v=DKIM1; k=rsa; "
          "p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDeJA2Bu9HQMX4yS/N2DXffzUdnlPWCxvmsv6YYdukfaSVmiNycXygZ3KSA0/xGnz+2GqOyEbC4VLg4MWO6c14PBZy4T95YaUBMUbww2JU4LwhcHGgDOacEJVwze2brmCR9uRlHXug0v8xbhHwpNaWFDNtt/f3ZxuZf9v9pJjMFNQIDAQAB" )  ; ----- DKIM key default for stg-development.work
```

## Route53登録
- これでうまくいった
- ![image](https://user-images.githubusercontent.com/60077121/109927656-e1b65c80-7d07-11eb-9505-cb689152e60e.png)


## opendkimの設定
- 文献1のとおり
```
# モードの変更 s:送信時に署名する v:受信時に確認する
Mode    sv

# OpenDKIMのソフトウェアバージョン秘匿
SoftwareHeader no

#KeyfileではなくKeyTableを利用する。
#KeyFile        /etc/opendkim/keys/default.private　←　コメントアウト
KeyTable        refile:/etc/opendkim/KeyTable　←　#を外して有効化

#　署名するドメインの指定
SigningTable    refile:/etc/opendkim/SigningTable　←　#を外して有効化

#　外部ホストの特定
#　署名を付けるかの判断
ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts　←　#を外して有効化

#内部ホストの特定
#　署名を付けるかの判断
InternalHosts   refile:/etc/opendkim/TrustedHosts　←　#を外して有効化
```
### `Mode`
- >Modeパラメータのデフォルト設定は、メール受信時の確認のみの”v”になっています。今回はメール送信時の署名を行うために ”s” を追加します。

### SoftwareHeader no
- 項目なかったので割愛
- >デフォルト”yes”だとメールヘッダーにバージョン情報が記載されます。バージョン情報を表示するとセキュリティリスクになるので非表示 “no” にします。

### KeyTable
- ドメインに対応する秘密鍵をマッピングする。マルチドメイン扱う場合はkeytable必須。
- ひとつのドメインでも可なので、テーブル使うことにした。

### SigningTable
- >署名をするメールのドメインと公開鍵ファイルの情報をマッピングするための設定です。
- 書式は`*@[ドメイン名] [セレクタ名]._domainkey.[ドメイン名]`
- `vi /etc/opendkim/SigningTable`
```
*@stg-development.work default._domainkey.stg-development.work
```

### ExternalIgnoreList, InternalHosts
- リレーサーバ経由するときに設定。文献参照

## 秘密鍵の所有者をopndkimに変更（文献にはないが必須）
- これをやらないと`can't load key from /etc/opendkim/keys/stg-development.work/default.private: Permission denied`と`/var/log/maillog`にでる
```
[root@ip-10-123-10-6 stg-development.work]# chown -R opendkim:opendkim /etc/opendkim/keys/stg-development.work/
[root@ip-10-123-10-6 stg-development.work]# ls -l
total 8
-rw-------. 1 opendkim opendkim 891 Mar  4 07:09 default.private
-rw-------. 1 opendkim opendkim 324 Mar  4 07:09 default.txt
```

## OpenDKIMの起動
```
# systemctl start opendkim
# systetemctl enable opendkim
```

## postfixとの連携
- 文献参照

## 追加設定
- main.cfのinetを`ipv4`にしないと以下のエラーが`/var/log/maillog`にでる
  - http://wpress.biz/blog/2016/09/18/postfix%E3%81%A7gmail%E3%81%AB%E9%80%81%E4%BF%A1%E3%81%A7%E3%81%8D%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%99%E3%82%8B/
```
connect to alt2.aspmx.l.google.com[2607:f8b0:4023:1006::1b]:25: Network is unreachable
```


