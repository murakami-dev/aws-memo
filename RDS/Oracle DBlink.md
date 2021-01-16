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

## データベースリンク作成
```
SQL> CREATE DATABASE LINK MYDB
  2    CONNECT TO admin IDENTIFIED BY ******
  3    USING '(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=database-2.ckqtks0xjen9.ap-northeast-1.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=MYDB)))';

Database link created.
```
