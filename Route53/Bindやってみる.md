# 概要
- Bind使ってみてDNS理解を深めたい

# 目標
- Bindをつかって自分のドメインのゾーン情報作成し、プライべートIPでアクセスさせる
- 当然今はALBのパブリックIPを返す。
```
[root@ip-10-123-10-204 ~]# dig stg-development.work

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.2 <<>> stg-development.work
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44270
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;stg-development.work.          IN      A

;; ANSWER SECTION:
stg-development.work.   60      IN      A       52.196.110.217
stg-development.work.   60      IN      A       54.178.250.120

;; Query time: 7 msec
;; SERVER: 10.123.0.2#53(10.123.0.2)
;; WHEN: Thu Feb 11 07:16:59 UTC 2021
;; MSG SIZE  rcvd: 81
```

# 文献
- [bind導入ガイド](https://rfs.jp/server/bind)
- [BINDの設定(named.conf)](http://cos.linux-dvr.biz/archives/87)
  - このブログシリーズ良い
- [@IT BINDで作るDNSサーバ](BINDで作るDNSサーバ)
  - これ古いのでだめ

# 手順
## yumにbindあるか調べる
- あるみたい
```
[ec2-user@ip-10-123-10-204 ~]$ yum search bind
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
============================== N/S matched: bind ===============================
bind.x86_64 : The Berkeley Internet Name Domain (BIND) DNS (Domain Name System)
            : server
bind-chroot.x86_64 : A chroot runtime environment for the ISC BIND DNS server,
                   : named(8)
bind-devel.x86_64 : Header files and libraries needed for BIND DNS development
bind-dyndb-ldap.x86_64 : LDAP back-end plug-in for BIND
bind-export-devel.x86_64 : Header files and libraries needed for BIND export
                         : libraries
bind-libs.x86_64 : Libraries used by the BIND DNS packages
bind-libs.i686 : Libraries used by the BIND DNS packages
bind-license.noarch : License of the BIND DNS suite
bind-lite-devel.x86_64 : Lite version of header files and libraries needed for
                       : BIND DNS development
```

## インストール
```
[ec2-user@ip-10-123-10-204 ~]$ sudo su -
[root@ip-10-123-10-204 ~]# yum install -y bind
```

## `named.conf`を確認
- 初期の状態
- 各項目の意味はブログ参照
```
[root@ip-10-123-10-204 ~]# cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//
// See the BIND Administrator's Reference Manual (ARM) for details about the
// configuration located in /usr/share/doc/bind-{version}/Bv9ARM.html

options {
        listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { localhost; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.root.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

## `named.conf`に以下の情報追加
### `stg-~`のゾーン情報はfile "stg-development.work"を見てね
```
zone "stg-development.work" IN {
    type master;
    file "stg-development.work";
    allow-update { none; };
};
```
### VPC内からクエリを受け付けるための設定
- `listen-on port 53`,`allow-query`
```
options {
        listen-on port 53 { 127.0.0.1; 10.123.0.0/16;};
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { localhost; 10.123.0.0/16;};
```

## 正引き用設定ファイルの作成
- named.confのdirectory配下に作成する
- 今回は`var/named/stg-development.work`
  - 
```
$TTL      60
@         IN SOA   ns.stg-development.work. root.stg-development.work. (
                   2015123105 ; Serial
                   28800      ; Refresh
                   14400      ; Retry
                   3600000    ; Expire
                   300 )       ; Minimum

          IN NS    ns-1522.awsdns-62.org.       ;←ここにns.stg-development.work.書いたらエラーになった。存在しないから？でも＠ITには自分自身を書くと書いてある
          IN A     10.123.10.251
```

## サービスの起動
- サービス起動後に設定ファイル編集した場合はrestart必要
```
[root@ip-10-123-10-204 named]# systemctl start named
[root@ip-10-123-10-204 named]# systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-02-12 00:02:05 UTC; 4min 48s ago
  Process: 5970 ExecStop=/bin/sh -c /usr/sbin/rndc stop > /dev/null 2>&1 || /bin/kill -TERM $MAINPID (code=exited, status=0/SUCCESS)
  Process: 5984 ExecStart=/usr/sbin/named -u named -c ${NAMEDCONF} $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 5981 ExecStartPre=/bin/bash -c if [ ! "$DISABLE_ZONE_CHECKING" == "yes" ]; then /usr/sbin/named-checkconf -z "$NAMEDCONF"; else echo "Checking of zone files is disabled"; fi (code=exited, status=0/SUCCESS)
 Main PID: 5987 (named)
   CGroup: /system.slice/named.service
           mq5987 /usr/sbin/named -u named -c /etc/named.conf

Feb 12 00:04:59 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ari.delta.aridns.net.au/A/IN': 2a01:88...:1#53
Feb 12 00:04:59 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ari.delta.aridns.net.au/AAAA/IN': 2a01...:1#53
Feb 12 00:04:59 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ari.beta.aridns.net.au/AAAA/IN': 2001:...24#53
Feb 12 00:04:59 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ari.alpha.aridns.net.au/AAAA/IN': 2001...:2#53
Feb 12 00:04:59 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ari.alpha.aridns.net.au/AAAA/IN': 2001...:2#53
Feb 12 00:05:00 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ns-1903.awsdns-45.co.uk/AAAA/IN': 2600...:1#53
Feb 12 00:05:00 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ns-1903.awsdns-45.co.uk/AAAA/IN': 2600...:1#53
Feb 12 00:05:00 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ns-1903.awsdns-45.co.uk/AAAA/IN': 2600...:1#53
Feb 12 00:05:00 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: network unreachable resolving 'ns-1903.awsdns-45.co.uk/AAAA/IN': 2600...:1#53
Feb 12 00:05:08 ip-10-123-10-204.ap-northeast-1.compute.internal named[5987]: client @0x7f687ee97350 45.148.122.159#50648 (sl): query (cache) 'sl/A...enied
Hint: Some lines were ellipsized, use -l to show in full.
```

## DNSクライアントからdig
### 別のEC2起動
### resolv.confを編集
- ※resolv.confだけでは再起動後元に戻るはず（EC2のGithub参照）
```
; generated by /usr/sbin/dhclient-script
search ap-northeast-1.compute.internal
options timeout:2 attempts:5
nameserver 10.123.10.204
```

### dig
- プライベートIP返ってきた
```
[root@ip-10-123-10-130 ~]# dig stg-development.work

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.2 <<>> stg-development.work
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 54329
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;stg-development.work.          IN      A

;; ANSWER SECTION:
stg-development.work.   60      IN      A       10.123.10.251

;; AUTHORITY SECTION:
stg-development.work.   60      IN      NS      ns-1522.awsdns-62.org.

;; Query time: 0 msec
;; SERVER: 10.123.10.204#53(10.123.10.204)
;; WHEN: Fri Feb 12 00:02:10 UTC 2021
;; MSG SIZE  rcvd: 100
```

### curl
- webサイトも見れた
```
[root@ip-10-123-10-130 ~]# curl stg-development.work
<html>
<body>
<h1>Hello Web02</h1>
</body>
</html>
```
