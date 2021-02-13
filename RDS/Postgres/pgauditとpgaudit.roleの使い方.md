# 目標
- pgaudit.roleの意味を知る

# 概要
- pgaudit.roleは一部のオブジェクト（DBやテーブル、列など）だけの監査ログを残したい場合に、対象のオブジェクトと当該ロールを紐づけて設定するもの。
- 以下の検証は`log_statement`が`none`で行った。

|組み合わせ  | pgaudit.log | pgaudit.role | 結果 |
| ------------- | ------------- | ------------- | ------------- | 
| 1  | none | 設定なし | 記録されない（試していないけど） |
| 2  | none  | rds_pgaudit | 記録されない |
| 3 | all | 設定なし | 記録される（検証済み） |
| 4 | all| rds_pgaudit |記録された。特にテーブルとroleの紐づけ設定（`ALTER DATABASE`）していなくても。 |

# 文献
- [PostgreSQL を実行している Amazon RDS DB インスタンスを監査するために pgaudit 拡張機能を使用する方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/rds-postgresql-pgaudit/)

# 本来のroleの使い方
- 文献のとおりにやる
## テーブル作成
- role:設定なし
- pgaudit.log : none
  - >パラメータグループレベルでは pgaudit.log パラメータを none に設定しておきます。
```
postgres=> CREATE TABLE Staff
postgres-> (id    CHAR(4)    NOT NULL,
postgres(> name   TEXT       NOT NULL,
postgres(> age    INTEGER    ,
postgres(> PRIMARY KEY (id));
CREATE TABLE
postgres=> INSERT INTO Staff VALUES ('0001', 'DB起動後', 28);　←これはログに残らない（log_statementもnoneなので）
INSERT 0 1
```
## role作成（OS上）
- OS上でもロール作成する必要あり
```
postgres=> CREATE ROLE rds_pgaudit;
CREATE ROLE
```

## パラメタグループでrole設定
- role:rds_pgaudit
  - 即時適用される

## もろもろ
```
postgres=> show shared_preload_libraries;
 shared_preload_libraries
--------------------------
 rdsutils,pgaudit
(1 row)

postgres=> CREATE EXTENSION pgaudit;
CREATE EXTENSION
postgres=> show pgaudit.role;
 pgaudit.role
--------------
 rds_pgaudit
(1 row)

postgres=> show pgaudit.log
postgres-> ;
 pgaudit.log
-------------
 none
(1 row)
```

## DB作成・ロール紐づけ
```
postgres=> create database test_database;
CREATE DATABASE
postgres=> ALTER DATABASE test_database set pgaudit.log='All';    ←これで紐づけ
ALTER DATABASE
```

- DB切替
```
postgres=> \c test_database
Password for user postgres:
psql (9.2.24, server 12.5)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
You are now connected to database "test_database" as user "postgres".
```
- table作成
```
test_database=> CREATE TABLE test_table
test_database-> (id    CHAR(4)    NOT NULL,
test_database(> name   TEXT       NOT NULL,
test_database(> age    INTEGER    ,
test_database(> PRIMARY KEY (id));
CREATE TABLE

test_database=> INSERT INTO test_table VALUES ('0002', 'test_databaseにaudit設定後', 28);　←これはログに残る
INSERT 0 1
```

### ロール紐づいていない方のDBに切替
```
test_database=> \c postgres
Password for user postgres:
psql (9.2.24, server 12.5)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
You are now connected to database "postgres" as user "postgres".
postgres=> INSERT INTO Staff VALUES ('0003', 'audit設定していないDBにINSERT', 28);　←これはログに残らない
INSERT 0 1
```

### もう一回ロール紐づいている方に切り替えてINSERT
```
postgres=> \c test_database
Password for user postgres:
psql (9.2.24, server 12.5)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
You are now connected to database "test_database" as user "postgres".
test_database=> INSERT INTO test_table VALUES ('0004', 'audit設定している方に再INSERT', 28);       ←これはログに残る
INSERT 0 1
```

## CWlogsに残っていたログ
```
2021-02-13T00:51:53.000+09:00	2021-02-12 15:51:53 UTC:10.123.10.212(57116):postgres@test_database:[28213]:LOG: AUDIT: SESSION,2,1,WRITE,INSERT,,,"INSERT INTO test_table VALUES ('0002', 'test_databaseにaudit設定後', 28);",<not logged>
2021-02-13T00:57:44.000+09:00	2021-02-12 15:57:44 UTC:10.123.10.212(57184):postgres@test_database:[4269]:LOG: AUDIT: SESSION,1,1,WRITE,INSERT,,,"INSERT INTO test_table VALUES ('0004', 'audit設定している方に再INSERT', 28);",<not logged>
```
