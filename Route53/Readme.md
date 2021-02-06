# Blackbelt
- [Amazon Route 53 Resolver](https://d1.awsstatic.com/webinars/jp/pdf/services/20191016_AWS_Blackbelt_Route53_Resolver.pdf)
- [Amazon Route 53 Hosted Zone](https://d1.awsstatic.com/webinars/jp/pdf/services/20191105_AWS_Blackbelt_Route53_Hosted_Zone_A.pdf)

# 用語の整理
## DNS一般
- ネームサーバー
- フルサービスリゾルバー
- スタブリゾルバー
- フォワーダー
- 反復問合せと再帰問合せ
![image](https://user-images.githubusercontent.com/60077121/102025914-41bef600-3dde-11eb-9205-425d81ae1ffa.png)

### digコマンド見方
- [DNSド素人がdigコマンドとRoute 53を使って、DNSについてあれこれ学んでみた](https://dev.classmethod.jp/articles/dig-route53-begginer/)
  - ANSWER SECTIONの数字はTTL値。8.8.8.8は比較的短い。SERVER: 8.8.8.8#53がDNSサーバ
```
murakami@HW-1191:~$ dig @8.8.8.8 stg-development.work

; <<>> DiG 9.16.1-Ubuntu <<>> @8.8.8.8 stg-development.work
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26957
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;stg-development.work.          IN      A

;; ANSWER SECTION:
stg-development.work.   59      IN      A       54.178.250.120
stg-development.work.   59      IN      A       52.196.110.217

;; Query time: 112 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Feb 06 16:34:01 JST 2021
;; MSG SIZE  rcvd: 81
```

## Route53
- Rooute53 Hosted Zone
- Route53 Resolver
- Route53 Resolver for Hybrid Clouds
![image](https://user-images.githubusercontent.com/60077121/102026284-4edce480-3de0-11eb-8c14-c20f73ccb35f.png)

