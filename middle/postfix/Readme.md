# main.cf
## hostnameなどの定義
- myhostnameが指定されていない場合、postfixはシステムのホスト名を取得する。システムのホスト名は完全修飾ホスト名を返さない場合もあるのでmyhostnameは指定推奨。
- mydomainが未定の場合、postfixはmyhostnameの最初の構成要素を除いたものをmaydomainとして設定する（mail.stg-development.workならstg-development.work）
  - 面倒なのでmydomainも指定しよう
- myoriginは・・・
- mydestinationはどのメールを受け付けるかの設定。デフォルトではpostfixを実行しているサーバあてのみ（`$myhostname, localhost.$mydomain, localhost`）
  - `$mydomain`も加えて自分のドメインあては全部受け取るようにしよう。
```
myhostname = mail.stg-development.work
mydomain = stg-development.work
#myorigin = $myhostname
#myorigin = $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
#mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
#       mail.$mydomain, www.$mydomain, ftp.$mydomain
```

## リレー制御
### mynetworks
- どのclientサーバにメールリレーを許可するか
- 基本はmynetworks_styleで設定。mynetworksでは個々のIP書く場合、CIDR表記したい場合、ネットワーク外のサーバにもリレーさせたい場合に指定
- mynetworksが設定されている場合はこれが優先。
```
#mynetworks_style = class #自分と同じクラスのアドレス帯はリレーOK。広くなりすぎるので非推奨。
#mynetworks_style = subnet # コメントアウトされてるけど、デフォルト。同じサブネットのサーバはリレーOK
#mynetworks_style = host  # 自分だけリレーOK


#mynetworks = 168.100.189.0/28, 127.0.0.0/8
#mynetworks = $config_directory/mynetworks
#mynetworks = hash:/etc/postfix/network_table
```

### relay_domains
- デフォルトの内容
```
#relay_domains = $mydestination
```
#### 説明
- 以下はmain.cfに記載されている英文説明抜粋
  - 信頼されたネットワーク(IP address matches $mynetworks)からのメールなら受ける
  - 信頼されていないネットワークの場合、relay_domainsに記載のネットワークなら受ける
    - ただし、`$inet_interfaces`や `$mydestination`などが宛先の場合は`relay_domains`に記載がなくても受けるよ

### relay_recipients_maps
- オプション設定
- 有効にすると/etc/postfix/relay_recipientsにある宛先しかリレーしないよ
```
# REJECTING UNKNOWN RELAY USERS
#
# The relay_recipient_maps parameter specifies optional lookup tables
# with all addresses in the domains that match $relay_domains.
#
# If this parameter is defined, then the SMTP server will reject
# mail for unknown relay users. This feature is off by default.
#
# The right-hand side of the lookup tables is conveniently ignored.
# In the left-hand side, specify an @domain.tld wild-card, or specify
# a user@domain.tld address.
#
#relay_recipient_maps = hash:/etc/postfix/relay_recipients
```
### relayhost
- メールゲートウェイがある場合はこれを指定する
- **NATGW経由で配信する場合、これの指定は必要か？**
```
#relayhost = $mydomain
#relayhost = [gateway.my.domain]
#relayhost = [mailserver.isp.tld]
#relayhost = uucphost
#relayhost = [an.ip.add.ress]
```

### transport
- /etc/postfix/transport
- オプション項目であり、上記ファイルはコメントアウトされている。
- >The relayhost parameter specifies the default host to send mail to when no entry is matched in the optional transport(5) table
  - transportで指定がない場合は、relayhostパラメタで指定されたホストへリレーする。

## chroot
- セキュリティ対策のひとつ。
- ルートディレクトリ配下ではなく特定のディレクトリをルートにみたてるもの
- デフォルトは以下。有効になっている。
```
queue_directory = /var/spool/postfix
```

### こうなってる（キューの仕組みに関わる。後述）
```
[ec2-user@ip-10-123-10-6 postfix]$ pwd
/var/spool/postfix
[ec2-user@ip-10-123-10-6 postfix]$ ls
active  corrupt  deferred  hold      maildrop  private  saved
bounce  defer    flush     incoming  pid       public   trace
```

# master.cf
- 以下は冒頭だけ
- `private`:アクセスをpostfixシステムに限定。inetではパブリックアクセスの`n`を指定
- `maxproc`:最大プロセス数。なにも指定しない場合はmain.cfの`defaul_process_limit`が使われる
```
# ==========================================================================
# service type  private unpriv  chroot  wakeup  maxproc command + args
#               (yes)   (yes)   (no)    (never) (100)
# ==========================================================================
smtp      inet  n       -       n       -       -       smtpd
#smtp      inet  n       -       n       -       1       postscreen
#smtpd     pass  -       -       n       -       -       smtpd
#dnsblog   unix  -       -       n       -       0       dnsblog
#tlsproxy  unix  -       -       n       -       0       tlsproxy
#submission inet n       -       n       -       -       smtpd
```

# キューの管理
- すべてのメッセージはキューを通過する。心臓部。必ず理解。
- キューマネージャはincoming,active,deferred,hold,corruptという5つのキューを管理
- `var/spool/postfix`配下にある以下。
  - `incoming`:active
  - `active`
  - `deferred`
  - `hold`
  - `corrupt`




