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
- **ローカルユーザどうしではメール送れた**
  - [[mailコマンド]Linuxからメールを送る](https://qiita.com/shuntaro_tamura/items/40a7d9b4400f31ec0923)

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

# メールの送信
### 直接メール送れた。
```
[root@ip-10-123-10-6 new]# cat 1615602020.Vca02I51f7eM844642.ip-10-123-10-6.ap-northeast-1.compute.internal
Return-Path: <muser@stg-development.work>
X-Original-To: test@stg-development.work
Delivered-To: test@stg-development.work
Received: by mail.stg-development.work (Postfix, from userid 0)
        id C2CE9805823; Sat, 13 Mar 2021 02:20:20 +0000 (UTC)
DKIM-Filter: OpenDKIM Filter v2.11.0 mail.stg-development.work C2CE9805823
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=stg-development.work; s=default; t=1615602020;
        bh=g3zLYH4xKxcPrHOD18z9YfpQcnk/GaJedfustWU5uGs=;
        h=Date:From:To:Subject:From;
        b=xhvqddByk+WHqUM+VmiuUND8J16r1aMjra0od9k15t6JuvoR22AO4eYKnlgEZxzFB
         pZBUHC5iaIhgGrcn9LPHjd9xM71qFgrZ6IbM4b5/WE0wgZh9KxIIymxbyN6api+I2x
         QntV+2iVMcoIzqwuWy9R/HzdNYOx00V2gWUQRwaA=
Date: Sat, 13 Mar 2021 02:20:20 +0000
From: muser@stg-development.work
To: test@stg-development.work
Subject: title
Message-ID: <604c2164.9Hm2iAl75Nf/0/F2%muser@stg-development.work>
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit

test
```
### 同じサーバ内だけどメールリレーの構文でも送信できた
```
[root@ip-10-123-10-6 new]# echo "smtp-handson" | mail -v -s "Relay Success" -S smtp=smtp://localhost:25 -r muser@stg-development.work test@stg-development.work
Resolving host localhost . . . done.
Connecting to ::1:25 . . .Connecting to 127.0.0.1:25 . . . connected.
220 mail.stg-development.work ESMTP Postfix
>>> HELO ip-10-123-10-6.ap-northeast-1.compute.internal
250 mail.stg-development.work
>>> MAIL FROM:<muser@stg-development.work>
250 2.1.0 Ok
>>> RCPT TO:<test@stg-development.work>
250 2.1.5 Ok
>>> DATA
354 End data with <CR><LF>.<CR><LF>
>>> .
250 2.0.0 Ok: queued as 29F0A805823
>>> QUIT
221 2.0.0 Bye
[root@ip-10-123-10-6 new]# ls
1615601941.Vca02I51f7dM4660.ip-10-123-10-6.ap-northeast-1.compute.internal    1615602367.Vca02I51f7fM269950.ip-10-123-10-6.ap-northeast-1.compute.internal
1615602020.Vca02I51f7eM844642.ip-10-123-10-6.ap-northeast-1.compute.internal
[root@ip-10-123-10-6 new]# cat 1615602367.Vca02I51f7fM269950.ip-10-123-10-6.ap-northeast-1.compute.internal
Return-Path: <muser@stg-development.work>
X-Original-To: test@stg-development.work
Delivered-To: test@stg-development.work
Received: from ip-10-123-10-6.ap-northeast-1.compute.internal (localhost [127.0.0.1])
        by mail.stg-development.work (Postfix) with SMTP id 29F0A805823
        for <test@stg-development.work>; Sat, 13 Mar 2021 02:26:07 +0000 (UTC)
DKIM-Filter: OpenDKIM Filter v2.11.0 mail.stg-development.work 29F0A805823
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=stg-development.work; s=default; t=1615602367;
        bh=HHNALJelevoXjZnpuz9+XeOEE1GVptdeDJek9mNF5l4=;
        h=Date:From:To:Subject:From;
        b=xQniGPmf7XxCzISnXNWunU23AIyrK76yEK5aKw+JiPn6QCWq575ioCYn07GIDrh9v
         fteh+wXqlkFiUB5J7wA9dT5/B9TWToUzmva34pRjpobNZIcZwQdCFxU+yUGiHmHuLM
         J0KlXD7CDlGWnuNha7gmNMg7hjCP8w5Jl4+/3hbI=
Date: Sat, 13 Mar 2021 02:26:07 +0000
From: muser@stg-development.work
To: test@stg-development.work
Subject: Relay Success
Message-ID: <604c22bf.56LEikBhdmV/7+Ia%muser@stg-development.work>
User-Agent: Heirloom mailx 12.5 7/5/10
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit

smtp-handson
```
