# 概要
Suricata Rule書いた時の挙動

# 手順
自分のブログ参照

# 文献
- [AWS Network Firewallを分かりやすく解説してみる](https://blog.serverworks.co.jp/all-about-aws-network-firewall)
- [[新サービス] VPC 向け AWS マネージドファイアウォールサービス「AWS Network Firewall」がリリースされました](https://dev.classmethod.jp/articles/aws-network-firewall/)


# 結果
## ルールに記載した文
```
drop tcp any any -> any any (msg:"TCP traffic detected"; sid:200001; rev:1;)
```

## Clientの結果
```
[ec2-user@ip-10-123-10-212 ~]$ curl 3.81.95.61
curl: (7) Failed to connect to 3.81.95.61 port 80: Connection timed out
```

## ログ
```
{
    "firewall_name": "test",
    "availability_zone": "us-east-1a",
    "event_timestamp": "1614379684",
    "event": {
        "timestamp": "2021-02-26T22:48:04.413411+0000",
        "flow_id": 1935816832339683,
        "event_type": "alert",
        "src_ip": "54.199.234.1",
        "src_port": 47922,
        "dest_ip": "10.11.0.26",
        "dest_port": 80,
        "proto": "TCP",
        "alert": {
            "action": "blocked",
            "signature_id": 200001,
            "rev": 1,
            "signature": "TCP traffic detected",
            "category": "",
            "severity": 3
        }
    }
}
```



