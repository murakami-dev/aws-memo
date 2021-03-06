# 文献
- [プライベート Amazon EC2 インスタンスが Amazon Linux、Ubuntu、または RHEL で実行中です。再起動中も持続する EC2 インスタンスに静的 DNS サーバーを割り当てる方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-static-dns-ubuntu-debian/)
  - `/etc/dhcp/dhclient.conf`を編集する方法
  - (こっちの方が強い)インターフェイスごとの設定ファイル (/etc/sysconfig/network-scripts/ifcfg-*) の中で、カスタム DNS サーバーを指定する方法
- [Amazon EC2(Linux)のネットワーク設定でハマったときに見るメモ](https://dev.classmethod.jp/articles/ec2-linux-network-conf-tips/)
  - `/etc/sysconfig/network-scripts/ifcfg-eth0`の`PEERDNS=yes`をnoに変更しないとダメ？

# 手順
-  /etc/dhcp/dhclient.confに以下を追加
```
supersede domain-name-servers 8.8.8.8;
```

- 2.再起動`sudo reboot`
- 3./etc/resolv.confが更新される
```
options timeout:2 attempts:5
; generated by /usr/sbin/dhclient-script
search ap-northeast-1.compute.internal
nameserver 8.8.8.8
```

# 各設定ファイルのデフォルトの内容
## /etc/sysconfig/network-scripts/ifcfg-eth0
```
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
TYPE=Ethernet
USERCTL=yes
PEERDNS=yes
DHCPV6C=yes
DHCPV6C_OPTIONS=-nw
PERSISTENT_DHCLIENT=yes
RES_OPTIONS="timeout:2 attempts:5"
DHCP_ARP_CHECK=no
```

## /etc/resolv.conf
- Route53リゾルバーを向いている
```
options timeout:2 attempts:5
; generated by /usr/sbin/dhclient-script
search ap-northeast-1.compute.internal
nameserver 10.123.0.2
```

##  /etc/dhcp/dhclient.conf
```
timeout 300;
```

## /etc/nsswitch.conf
```
[ec2-user@ip-10-123-10-212 ~]$ sudo cat /etc/nsswitch.conf
#
# /etc/nsswitch.conf
#
# An example Name Service Switch config file. This file should be
# sorted with the most-used services at the beginning.
#
# The entry '[NOTFOUND=return]' means that the search for an
# entry should stop if the search in the previous entry turned
# up nothing. Note that if the search failed due to some other reason
# (like no NIS server responding) then the search continues with the
# next entry.
#
# Valid entries include:
#
#       nisplus                 Use NIS+ (NIS version 3)
#       nis                     Use NIS (NIS version 2), also called YP
#       dns                     Use DNS (Domain Name Service)
#       files                   Use the local files
#       db                      Use the local database (.db) files
#       compat                  Use NIS on compat mode
#       hesiod                  Use Hesiod for user lookups
#       [NOTFOUND=return]       Stop searching if not found so far
#

# To use db, put the "db" in front of "files" for entries you want to be
# looked up first in the databases
#
# Example:
#passwd:    db files nisplus nis
#shadow:    db files nisplus nis
#group:     db files nisplus nis

passwd:     sss files
shadow:     files sss
group:      sss files

#hosts:     db files nisplus nis dns
hosts:      files dns myhostname

# Example - obey only what nisplus tells us...
#services:   nisplus [NOTFOUND=return] files
#networks:   nisplus [NOTFOUND=return] files
#protocols:  nisplus [NOTFOUND=return] files
#rpc:        nisplus [NOTFOUND=return] files
#ethers:     nisplus [NOTFOUND=return] files
#netmasks:   nisplus [NOTFOUND=return] files

bootparams: nisplus [NOTFOUND=return] files

ethers:     files
netmasks:   files
networks:   files
protocols:  files
rpc:        files
services:   files sss

netgroup:   nisplus sss

publickey:  nisplus

automount:  files nisplus
aliases:    files nisplus
```
