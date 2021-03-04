# 目的
- yumなどのリポジトリについて整理

# 全体
- yumリポジトリの中にrpmリポジトリは入っている、epelは入っていない。（RHELで検証）

# yum
- https://www.atmarkit.co.jp/ait/articles/1608/29/news019.html
## 現在EC2にインストール済みのものを調べる
```
[root@ip-10-123-10-251 ~]# yum list installed | grep mariadb
[root@ip-10-123-10-251 ~]#
```

## 既存のリポジトリを確認する
```
[root@ip-10-123-10-251 ~]# yum repolist
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
repo id                                                   repo name                                                      status
!amzn2-core/2/x86_64                                      Amazon Linux 2 core repository                                 23,094
amzn2extra-docker/2/x86_64                                Amazon Extras repo for docker                                      36
repolist: 23,130
```

- RHEL
```
[ec2-user@ip-10-123-10-6 ~]$ yum repolist
repo id                                       repo name
rhel-8-appstream-rhui-rpms                    Red Hat Enterprise Linux 8 for x86_64 - AppStream from RHUI (RPMs)
rhel-8-baseos-rhui-rpms                       Red Hat Enterprise Linux 8 for x86_64 - BaseOS from RHUI (RPMs)
rhui-client-config-server-8                   Red Hat Update Infrastructure 3 Client Configuration Server 8
```


### yumがもつリポジトリは`/etc/yum.repos.d`で管理されている
```
[root@ip-10-123-10-251 ~]# cd /etc/yum.repos.d
[root@ip-10-123-10-251 yum.repos.d]# ls
amzn2-core.repo  amzn2-extras.repo
```

### ここにMariaDBレポジトリを追加
```
sudo vi /etc/yum.repos.d/MariaDB.repo
```
- MariaDB.repo
```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.5/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```



# epel
- [CentOS、RHEL、または Amazon Linux を実行している Amazon EC2 インスタンスの EPEL リポジトリを有効にするにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-enable-epel/)
- [あらためてEPELリポジトリの使い方をまとめてみた](https://qiita.com/yamada-hakase/items/fdf9c276b9cae51b3633)
  - >Red Hat Enterprise Linux (RHEL) 系Linuxディストリビューション向けオプションパッケージ群だ。LinuxのメディアやYumリポジトリに含まれないパッケージ入手先の第１候補になる。



