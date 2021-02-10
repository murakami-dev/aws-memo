# 概要
- EC2にmariaDBインストールするとバージョンが5系だったのでバージョンアップする
- yumのパッケージの勉強になる

# 参考
- https://it.hirokun.net/entry/amazonlinux2-mariadb10_5
- https://onoredekaiketsu.com/upgrad-to-mariadb-10-3/

# 手順
## 現在EC2にインストール済みのものを調べる
```
[root@ip-10-123-10-251 ~]# yum list installed | grep mariadb
mariadb.x86_64                        1:5.5.68-1.amzn2               @amzn2-core
mariadb-libs.x86_64                   1:5.5.68-1.amzn2               installed
mariadb-server.x86_64                 1:5.5.68-1.amzn2               @amzn2-core
```

## バージョンを確認する
```
[root@ip-10-123-10-251 ~]# mysql -V
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1
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
- ariaDB.repo
```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.5/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

### インストール
```
sudo yum install -y mariadb-server mariadb-client MariaDB-shared MariaDB-devel
```

### インストールされたか確認
```
[root@ip-10-123-10-251 /]# yum list installed | grep maria
MariaDB-client.x86_64                 10.5.8-1.el7.centos            @mariadb
MariaDB-common.x86_64                 10.5.8-1.el7.centos            @mariadb
MariaDB-compat.x86_64                 10.5.8-1.el7.centos            @mariadb
MariaDB-devel.x86_64                  10.5.8-1.el7.centos            @mariadb
MariaDB-server.x86_64                 10.5.8-1.el7.centos            @mariadb
MariaDB-shared.x86_64                 10.5.8-1.el7.centos            @mariadb
galera-4.x86_64                       26.4.6-1.el7.centos            @mariadb
```

### バージョン確認など
```
[root@ip-10-123-10-251 /]# sudo systemctl start mariadb
[root@ip-10-123-10-251 /]# sudo systemctl enable mariadb.service
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@ip-10-123-10-251 /]# mysql -V
mysql  Ver 15.1 Distrib 10.5.8-MariaDB, for Linux (x86_64) using readline 5.1
```

