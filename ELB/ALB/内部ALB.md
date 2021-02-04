# 文献
- [内部 ALB の HTTPS 通信を試してみた](https://dev.classmethod.jp/articles/internal-alb-https/)

# やったこと
## 1.VPC内外からの名前解決
- プライベートサブネットにEC2作成。いったんEIPつけてWEBサーバー機能もたせる
- privateなsubnetに内部ALB作成。上記EC2ぶらさげる
- ローカルから内部ALBドメイン名前解決できない。パブリックサブネットの別EC2からはアクセス可能であること確認

### ローカルから名前解決不可
```
C:\Users\user>nslookup internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com
サーバー:  UnKnown
Address:  172.20.10.1

*** UnKnown が internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com を見つけられません: Non-existent domain
```

### VPC内EC2ならOK
```
[ec2-user@ip-10-123-10-212 ~]$ dig internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.amzn2.2 <<>> internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 43016
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com. IN A

;; ANSWER SECTION:
internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com. 48 IN A 10.123.20.26
internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com. 48 IN A 10.123.21.153

;; Query time: 0 msec
;; SERVER: 10.123.0.2#53(10.123.0.2)
;; WHEN: Thu Feb 04 22:56:54 UTC 2021
;; MSG SIZE  rcvd: 126
```
### curl
```
[ec2-user@ip-10-123-10-212 ~]$ curl internal-test-internal-707224996.ap-northeast-1.elb.amazonaws.com
<html>
<body>
<h1>API No.1</h1>
</body>
</html>
```

## やったこと2 パブリックDNSにレコード登録
- パブリックなRoute53ホストゾーンのレコードにinternal ALB登録,www.stg-****として登録
- ローカルからの名前解決でプライベートIP返ってくる
```
C:\Users\user>nslookup www.stg-development.work
サーバー:  UnKnown
Address:  172.20.10.1

権限のない回答:
名前:    www.stg-development.work
Addresses:  10.123.20.26
          10.123.21.153
```
