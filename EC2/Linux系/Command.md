# 全般
## ディレクトリ削除
```
Mac
rm -rf ディレクトリ名（内容も含め削除）
```
- https://xtech.nikkei.com/it/atcl/column/15/042000103/052100016/

## RPM
- 文献
  - [個人的によく使うrpmコマンド逆引き](https://dev.classmethod.jp/articles/rpm-reverse-search/)
  - [rpmもyumもURLを直接指定してインストールできる](https://qiita.com/kazinoue/items/d432b858c71618a1c15e)
  - [【yum install】パッケージをインストールする](https://uxmilk.jp/9314)
    - >「yum」は内部で「rpm」を使用しているため、rpmパッケージを直接指定してインストールすることもできます。
    ```
    yum install http://rpmfind.net/linux/centos/6.7/os/x86_64/Packages/tree-1.5.3-3.el6.x86_64.rpm
    ```
  - [【RPMによるパッケージ管理】rpmコマンドの使い方](https://eng-entrance.com/linux-package-rpm)
  - [【初心者にもわかる】rpmとyumの違いと使い分け一通り](https://eng-entrance.com/linux-package-rpm-yum-def)


# CentOS
- [AWS用CentOS8公式イメージ](https://note.com/hironobuu/n/n823e922606dc)
  - **注意！centosのWikiは手入れされていないので使用しない。マケプレで検索した方がいい**このページから辿ってCentOS公式AMIのIDとると良い。
- [CentOS7の最新AMI IDの取得方法考えてみた](https://dev.classmethod.jp/articles/get-centos7-ami-id/)
## UserDataモデル
- [CentOS インスタンスに SSM エージェント を手動でインストールする](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/agent-install-centos.html)
### userdata
```
#!/bin/bash
sudo yum update -y
timedatectl set-timezone Asia/Tokyo
localectl set-locale LANG=ja_JP.UTF8
# SSMagent 停止はお好みで
sudo yum install -y https://s3.ap-northeast-1.amazonaws.com/amazon-ssm-ap-northeast-1/latest/linux_amd64/amazon-ssm-agent.rpm
sudo systemctl stop amazon-ssm-agent
sudo systemctl disable amazon-ssm-agent
# CWagent 停止はお好みで
sudo yum install -y https://amazoncloudwatch-agent.s3.amazonaws.com/redhat/amd64/latest/amazon-cloudwatch-agent.rpm
sudo systemctl stop amazon-cloudwatch-agent.service
sudo systemctl disable amazon-cloudwatch-agent.service
```
### 結果
- タイムゾーン・ロケール
```
[centos@ip-10-123-10-34 ~]$ pwd
/home/centos
[centos@ip-10-123-10-34 ~]$ localectl status
   System Locale: LANG=ja_JP.UTF8
       VC Keymap: us
      X11 Layout: us
[centos@ip-10-123-10-34 ~]$ timedatectl status
      Local time: 土 2020-12-12 21:57:37 JST
  Universal time: 土 2020-12-12 12:57:37 UTC
        RTC time: 土 2020-12-12 12:57:37
       Time zone: Asia/Tokyo (JST, +0900)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```
- SSM
```
[centos@ip-10-123-10-34 /]$ find -name ssm*
...中略
./usr/bin/ssm-agent-worker
./usr/bin/ssm-document-worker
./usr/bin/ssm-cli
./usr/bin/ssm-session-logger
./usr/bin/ssm-session-worker
```

