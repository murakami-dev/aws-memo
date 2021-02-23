# やりたいこと
- IDS/IPSのOSSであるSuricataをインストールして最低限の検知を行いたい

# 文献
- [Doc](https://suricata.readthedocs.io/en/latest/what-is-suricata.html)
- [トラフィックミラーリング用のオープンソースツールの使用](https://docs.aws.amazon.com/ja_jp/vpc/latest/mirroring/tm-example-open-source.html)
  - インストール
- [Dockerの仮想ネットワーク内でOSSのIDS(suricata)を試す](https://koimedenshi.hatenablog.com/entry/2019/02/23/090000)
  - セットアップの参考に
  
  
# 手順
## インストール（管理者で）
- 
```
amazon-linux-extras install -y epel
```

- インストール
```
yum install -y suricata
```

## Basic setup
- [2. Quickstart guide](https://suricata.readthedocs.io/en/latest/quickstart.html)
- まずは自身のIPを把握
```
[root@ip-10-11-0-49 suricata]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 0e:35:98:e7:8b:55 brd ff:ff:ff:ff:ff:ff
    inet 10.11.0.49/24 brd 10.11.0.255 scope global dynamic eth0
       valid_lft 2641sec preferred_lft 2641sec
    inet6 fe80::c35:98ff:fee7:8b55/64 scope link
       valid_lft forever preferred_lft forever
```

### suricata.yamlの設定
- >suricataの設定変更は/etc/suricata/suricata.yamlを編集する．
- >色々と細かい設定ができるが，ここでは何かしらの攻撃アクセスを検知できれば良いので，最低限の設定として検査対象のIPアドレスを指定する．
- >検査対象のIPアドレスはsuricata.yamlのHOME_NETおよびEXTERNAL_NETにより制限できる．
- >suricataはHOME_NETに指定されたIPを保護するネットワークとして認識し，EXTERNAL_NETを外部ネットワークとして認識する．

#### 初期のsuricata.yaml（抜粋、冒頭部分だけ）
- ネットワークの定義部分
```
vars:
  # more specific is better for alert accuracy and performance
  address-groups:
    HOME_NET: "[192.168.0.0/16,10.0.0.0/8,172.16.0.0/12]"

    EXTERNAL_NET: "!$HOME_NET"
```

- ネットワークインターフェイスの定義部分
  - これで基本的なIDSモードにはなるらしい
```
af-packet:
    - interface: eth0     # この部分がip addrで調べたインターフェイス名と一致する必要がある
      cluster-id: 99
      cluster-type: cluster_flow
      defrag: yes
      use-mmap: yes
      tpacket-v3: yes
```

### update
- suricataルールをアップデートするコマンド
- `/var/lib/suricata/rules`にルールが作成される（量が多いので見ても意味ない）
```
[root@ip-10-11-0-49 suricata]# suricata-update
23/2/2021 -- 03:33:27 - <Info> -- Using data-directory /var/lib/suricata.
23/2/2021 -- 03:33:27 - <Info> -- Using Suricata configuration /etc/suricata/suricata.yaml
23/2/2021 -- 03:33:27 - <Info> -- Using /etc/suricata/rules for Suricata provided rules.
23/2/2021 -- 03:33:27 - <Info> -- Found Suricata version 4.1.10 at /usr/sbin/suricata.
23/2/2021 -- 03:33:27 - <Info> -- Loading /etc/suricata/suricata.yaml
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol ikev2
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol dhcp
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol tftp
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol krb5
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol ntp
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol modbus
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol enip
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol smb
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol dnp3
23/2/2021 -- 03:33:27 - <Info> -- Disabling rules for protocol nfs
23/2/2021 -- 03:33:27 - <Info> -- No sources configured, will use Emerging Threats Open
23/2/2021 -- 03:33:27 - <Info> -- Fetching https://rules.emergingthreats.net/open/suricata-4.1.10/emerging.rules.tar.gz.
 100% - 2768755/2768755
23/2/2021 -- 03:33:28 - <Info> -- Done.
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/app-layer-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/decoder-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/dnp3-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/dns-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/files.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/http-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/ipsec-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/kerberos-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/modbus-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/nfs-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/ntp-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/smb-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/smtp-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/stream-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Loading distribution rule file /etc/suricata/rules/tls-events.rules
23/2/2021 -- 03:33:28 - <Info> -- Ignoring file rules/emerging-deleted.rules
23/2/2021 -- 03:33:31 - <Info> -- Loaded 28492 rules.
23/2/2021 -- 03:33:32 - <Info> -- Disabled 79 rules.
23/2/2021 -- 03:33:32 - <Info> -- Enabled 0 rules.
23/2/2021 -- 03:33:32 - <Info> -- Modified 0 rules.
23/2/2021 -- 03:33:32 - <Info> -- Dropped 0 rules.
23/2/2021 -- 03:33:32 - <Info> -- Enabled 128 rules for flowbit dependencies.
23/2/2021 -- 03:33:32 - <Info> -- Creating directory /var/lib/suricata/rules.
23/2/2021 -- 03:33:32 - <Info> -- Backing up current rules.
23/2/2021 -- 03:33:32 - <Info> -- Writing rules to /var/lib/suricata/rules/suricata.rules: total: 28492; enabled: 21105; added: 28492; removed 0; modified: 0
23/2/2021 -- 03:33:32 - <Info> -- Testing with suricata -T.
23/2/2021 -- 03:33:37 - <Info> -- Done.
```

### アラートの設定
```
# Add a rule to match all UDP traffic
echo 'alert udp any any -> any any (msg:"UDP traffic detected"; sid:200001; rev:1;)' > /var/lib/suricata/rules/suricata.rules
```

### 設定ファイルの反映
```
# Start suricata listening on eth0 in daemon mode
suricata -c /etc/suricata/suricata.yaml -k none -i eth0 -D
```

## パケットを送信してみる
- 違うサーバ（パケット送信元）でnetcat をインストール
```
[ec2-user@ip-10-123-10-212 ~]$ sudo yum install nmap-ncat.x86_64
```

- パケット送信
  - Suricata(3.81.xx)の10000ポートに`-u`でUDPパケット送る
  - SuricataではSG空けておこう
```
[ec2-user@ip-10-123-10-212 ~]$ nc -u 3.81.95.61 10000
yeah
Ncat: Connection refused.
```

## Suricataでログを確認
- `/var/log/suricata/fast.log`
```
02/23/2021-04:50:18.504494  [**] [1:200001:1] UDP traffic detected [**] [Classification: (null)] [Priority: 3] {UDP} 54.199.234.1:37859 -> 10.11.0.26:10000
```
