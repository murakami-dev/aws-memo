# やりたいこと
- RDSの初期マスターユーザの権限とMariaDB rootユーザの権限比較

## RDSの初期マスターユーザ
- https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/UsingWithRDS.MasterAccounts.html
  - >SELECT、INSERT、UPDATE、DELETE、CREATE、DROP、RELOAD、PROCESS、REFERENCES、INDEX、ALTER、SHOW DATABASES、CREATE TEMPORARY TABLES、LOCK TABLES、EXECUTE、REPLICATION CLIENT、CREATE VIEW、SHOW VIEW、CREATE ROUTINE、ALTER ROUTINE、CREATE USER、EVENT、TRIGGER ON *.* WITH GRANT OPTION、REPLICATION SLAVE (Amazon RDS MySQL バージョン 5.6、5.7、8.0、Amazon RDS MariaDB の場合のみ)

```
MariaDB [(none)]> show grants;
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Grants for admin@%                                                                                                                                                                                                                                                                                                                                                                                 |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO `admin`@`%` IDENTIFIED BY PASSWORD '*805F66B2BC924362325A6BB599F9DBA3E3FB44E8' WITH GRANT OPTION |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## rootの権限
- すべて
```
MariaDB [(none)]> show grants;
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
```

## 結論
- マスターユーザとrootユーザーは同じではない。**以下は良い記事**
  - https://qiita.com/thaim/items/4ad87859334399eebc04
- SUPER権限やFILE権限（インスタンス上のファイルにアクセスする）などがない
  - SUPERがない
```
MariaDB [(none)]> SELECT user, Super_priv FROM mysql.user;
+----------+------------+
| user     | Super_priv |
+----------+------------+
| rdsadmin | Y          |
| admin    | N          |
| fuga     | N          |
+----------+------------+
3 rows in set (0.01 sec)

MariaDB [(none)]> UPDATE mysql.user SET Super_priv='Y' WHERE user='admin';
ERROR 1054 (42S22): Unknown column 'ERROR (RDS): SUPER PRIVILEGE CANNOT BE GRANTED OR MAINTAINED' in 'field list'
MariaDB [(none)]>
```


# 文献
- [公式:GRANT](https://mariadb.com/kb/en/grant/)
- https://tadtadya.com/mariadb-install-the-latest-version-on-linux-centos-ubuntu/

# 手順
## インストール
- EC2にインストール
```
[ec2-user@ip-10-123-10-251 ~]$ sudo yum install mariadb mariadb-server
```

- 確認
```
[ec2-user@ip-10-123-10-251 ~]$ rpm -qa | grep maria
mariadb-server-5.5.68-1.amzn2.x86_64
mariadb-5.5.68-1.amzn2.x86_64
mariadb-libs-5.5.68-1.amzn2.x86_64
```

- 起動
```
[ec2-user@ip-10-123-10-251 ~]$ sudo systemctl start mariadb
[ec2-user@ip-10-123-10-251 ~]$ sudo systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-02-10 15:57:26 UTC; 14s ago
  Process: 2406 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
  Process: 2323 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 2405 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           tq2405 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           mq2590 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql...

Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mariadb-prepare-db-dir[2323]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mysqld_safe[2405]: ...
Feb 10 15:57:24 ip-10-123-10-251.ap-northeast-1.compute.internal mysqld_safe[2405]: ...
Feb 10 15:57:26 ip-10-123-10-251.ap-northeast-1.compute.internal systemd[1]: ...
Hint: Some lines were ellipsized, use -l to show in full.
```

## 初期設定
### 初期設定が必要な理由
- >インストール直後のMariaDBは、ユーザー・パスワードなしでログインできたり、設定がほぼない状態だったり、ザルです。
- >ログインユーザー・パスワード無しで、『mysql』と打つだけで見れちゃいます。infomation_schemaまでイジれてしまっては、セキュリティはないも同然です。
```
[ec2-user@ip-10-123-10-251 ~]$ mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit;
Bye
```

### 文字コード変更
- https://qiita.com/iamdaisuke/items/adc561e057a69afebad8
```
[ec2-user@ip-10-123-10-251 ~]$ sudo vi vi /etc/my.cnf
```


  
