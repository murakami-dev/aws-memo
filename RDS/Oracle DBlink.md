# やりたいこと
- 2つのRDS Oracle間でDBlinkしてdumpファイルの受け渡し
- RDSからEC2へのdumpファイルの転送

# 文献
- 1.[データベースタスク-データベースリンクの調整](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/Appendix.Oracle.CommonDBATasks.Database.html#Appendix.Oracle.CommonDBATasks.DBLinks)
  - >各 DB インスタンスのセキュリティグループは他の DB インスタンスの受信と送信を許可する必要があります。
- 2.[Oracle Data Pump を使用したインポート](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/Oracle.Procedural.Importing.html#Oracle.Procedural.Importing.DataPump)
  - オンプレやEC2 <---> RDSまたはRDS <---> RDSのデータインポートが可能
  - >ダンプファイルを転送するには、Amazon S3 バケットを使用するか、2 つのデータベース間のデータベースリンクを使用します。
  - >full モードではインポートしないでください。
    - >Amazon RDS for Oracle では、管理ユーザー SYS または SYSDBA へのアクセスは許可されていないため、full モードでインポートしたり、Oracle 管理のコンポーネントのスキーマをインポートしたりすると、Oracle データディレクトリが損傷し、データベースの安定性に影響を及ぼす可能性があります。
- 3.[RDS for Oracle環境でData Pumpを利用する](https://dev.classmethod.jp/articles/data-pump-on-rds-for-oracle/)
- 4.[RDS for Oracle へ Data Pump インポートする前にやっておくべき 3 つのこと](https://dev.classmethod.jp/articles/3-tasks-before-import-oracle-rds-with-datapump/)
- 5.[Data Pump のダンプファイルをRDS for Oracle インスタンスへ転送する](https://dev.classmethod.jp/articles/transfer-data-pump-file-to-rds-instace/)
- 6.[export](https://oracle-chokotto.com/ora_export.html)
  - ClientであるEC2にdumpファイルをエクスポート。expdpはRDS内にdumpが保存されるのでOS入れないため取り出せない。

# やったこと
## RDS作成
- database-1
- database-2

## Windows EC2から接続
### database-1 テーブル作成
```
SQL> CREATE TABLE emp
  2   (
  3   empno VARCHAR2(10) NOT NULL,
  4   empname VARCHAR2(50),
  5   gender_f NUMBER(1,0)
  6   )
  7  ;

Table created.
```
### database-1 行挿入
```
SQL> INSERT INTO emp
  2   VALUES ('001', '山田太郎', '1');

1 row created.
```
### database-1 テーブル一覧
```
SQL> SELECT TABLE_NAME FROM USER_TABLES;

TABLE_NAME
--------------------------------------------------------------------------------
EMP
```

## DBMS_DATAPUMP を使用してダンプファイルを作成する（エラー）
- エラーが出る
```
SQL> DECLARE
  2    v_hdnl NUMBER;
  3  BEGIN
  4    v_hdnl := DBMS_DATAPUMP.OPEN( operation => 'EXPORT', job_mode => 'SCHEMA', job_name=>null);
  5    DBMS_DATAPUMP.ADD_FILE(
  6      handle    => v_hdnl,
  7      filename  => 'sample.dmp',
  8      directory => 'DATA_PUMP_DIR',
  9      filetype  => dbms_datapump.ku$_file_type_dump_file);
 10    DBMS_DATAPUMP.ADD_FILE(
 11      handle    => v_hdnl,
 12      filename  => 'sample_exp.log',
 13      directory => 'DATA_PUMP_DIR',
 14      filetype  => dbms_datapump.ku$_file_type_log_file);
 15    DBMS_DATAPUMP.METADATA_FILTER(v_hdnl,'SCHEMA_EXPR','IN (''admin'')');
 16    DBMS_DATAPUMP.START_JOB(v_hdnl);
 17  END;
 18  /
DECLARE
*
ERROR at line 1:
ORA-39001: invalid argument value
ORA-06512: at "SYS.DBMS_SYS_ERROR", line 79
ORA-06512: at "SYS.DBMS_DATAPUMP", line 4929
ORA-06512: at "SYS.DBMS_DATAPUMP", line 5180
ORA-06512: at line 5
```
- しかしdumpファイルは作成されている模様
  - [RDS for Oracle環境でData Pumpを利用する](https://dev.classmethod.jp/articles/data-pump-on-rds-for-oracle/)
```
SQL> SELECT * FROM TABLE(RDSADMIN.RDS_FILE_UTIL.LISTDIR('DATA_PUMP_DIR')) ORDER BY MTIME;

FILENAME
--------------------------------------------------------------------------------
TYPE         FILESIZE MTIME
---------- ---------- ---------
sample.dmp
file             4096 16-JAN-21

sample_exp.log
file               78 16-JAN-21

datapump/
directory        4096 16-JAN-21
```


## データベースリンク作成
```
SQL> CREATE DATABASE LINK MYDB
  2    CONNECT TO admin IDENTIFIED BY ******
  3    USING '(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=database-2.ckqtks0xjen9.ap-northeast-1.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=MYDB)))';

Database link created.
```

## DBMS_FILE_TRANSFER を使用して、エクスポートされたダンプファイルを移行先の DB インスタンスにコピーする
```
SQL> BEGIN
  2    DBMS_FILE_TRANSFER.PUT_FILE(
  3      source_directory_object       => 'DATA_PUMP_DIR',
  4      source_file_name              => 'sample.dmp',
  5      destination_directory_object  => 'DATA_PUMP_DIR',
  6      destination_file_name         => 'sample_copied.dmp',
  7      destination_database          => 'MYDB' );
  8  END;
  9  /

PL/SQL procedure successfully completed.
```

# RDS -> EC2へファイルの受け渡し
## まず`tnsnames.ora`を作成
- `tnsnames.ora`とはDB接続時に記載を省略できるもの（Linuxのhostsファイルみたいな）
  - https://qiita.com/ghogho-seki/items/ac58466b693a01ad6464
- ユーザー環境変数に`TNS_ADMIN`を`C:\OracleInstantClient`で登録
- ※後述**tnsnames.oraなくてもいける**
```
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = XXXX.rds.amazonaws.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ORCL)
    )
  )
```

## 実行
```
C:\Users\Administrator>exp admin/XXXXXX(パスワード)@ORCL file=C:\usrexp.dmp owner=admin

Export: Release 19.0.0.0.0 - Production on Sun Jan 17 00:41:32 2021
Version 19.9.0.0.0

Copyright (c) 1982, 2020, Oracle and/or its affiliates.  All rights reserved.


Connected to: Oracle Database 19c Standard Edition 2 Release 19.0.0.0.0 - Production
Version 19.9.0.0.0
Export done in US7ASCII character set and AL16UTF16 NCHAR character set
server uses AL32UTF8 character set (possible charset conversion)

About to export specified users ...
. exporting pre-schema procedural objects and actions
. exporting foreign function library names for user ADMIN
. exporting PUBLIC type synonyms
. exporting private type synonyms
. exporting object type definitions for user ADMIN
About to export ADMIN's objects ...
. exporting database links
. exporting sequence numbers
. exporting cluster definitions
. about to export ADMIN's tables via Conventional Path ...
. . exporting table                            EMP          1 rows exported
. . exporting table           SYS_EXPORT_SCHEMA_01          6 rows exported
. exporting synonyms
. exporting views
. exporting stored procedures
. exporting operators
. exporting referential integrity constraints
. exporting triggers
. exporting indextypes
. exporting bitmap, functional and extensible indexes
. exporting posttables actions
. exporting materialized views
. exporting snapshot logs
. exporting job queues
. exporting refresh groups and children
. exporting dimensions
. exporting post-schema procedural objects and actions
. exporting statistics
Export terminated successfully without warnings.
```
![image](https://user-images.githubusercontent.com/60077121/104828171-acfe5b00-58a9-11eb-8a24-a04fd189a77b.png)

### EC2側のインバウンドでRDSからの通信は許可必要なし
- EC2 -> RDSの通信が確立しているので必要なし。（プライベートサブネットのEC2がアウトバウンドだけでインターネット出れるのと同じ）

### `tnsnames.ora`なくてもいける
```
C:\Users\Administrator>exp admin/Hoge1234@database-1.ckqtks0xjen9.ap-northeast-1.rds.amazonaws.com:1521/ORCL file=C:\test04.dmp owner=admin
```
![image](https://user-images.githubusercontent.com/60077121/104828293-159a0780-58ab-11eb-9f8f-39368fb5134c.png)

# 落書きしていない画像のこしておく
![image](https://user-images.githubusercontent.com/60077121/104828459-e7b5c280-58ac-11eb-874a-67e5c04f7492.png)

