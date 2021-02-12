


```
postgres=> CREATE TABLE Staff
postgres-> (id    CHAR(4)    NOT NULL,
postgres(> name   TEXT       NOT NULL,
postgres(> age    INTEGER    ,
postgres(> PRIMARY KEY (id));
CREATE TABLE
postgres=> INSERT INTO Staff VALUES ('0001', 'DB起動後', 28);
INSERT 0 1
postgres=> CREATE ROLE rds_pgaudit;
CREATE ROLE
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

postgres=> create database test_database;
CREATE DATABASE
postgres=> ALTER DATABASE test_database set pgaudit.log='All';
ALTER DATABASE
postgres=> \c test_database
Password for user postgres:
psql (9.2.24, server 12.5)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
You are now connected to database "test_database" as user "postgres".
test_database=> CREATE TABLE test_table
test_database-> (id    CHAR(4)    NOT NULL,
test_database(> name   TEXT       NOT NULL,
test_database(> age    INTEGER    ,
test_database(> PRIMARY KEY (id));
CREATE TABLE
test_database=> INSERT INTO Staff VALUES ('0002', 'test_databaseにaudit設定後', 28);
ERROR:  relation "staff" does not exist
LINE 1: INSERT INTO Staff VALUES ('0002', 'test_databaseにaudit設定...
                    ^
test_database=> INSERT INTO test_table VALUES ('0002', 'test_databaseにaudit設定後', 28);
INSERT 0 1
test_database=> \c mydb
Password for user postgres:
FATAL:  database "mydb" does not exist
Previous connection kept
test_database=> \c postgres
Password for user postgres:
psql (9.2.24, server 12.5)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
You are now connected to database "postgres" as user "postgres".
postgres=> INSERT INTO Staff VALUES ('0003', 'audit設定していないDBにINSERT', 28);
INSERT 0 1
postgres=> \c test_database
Password for user postgres:
psql (9.2.24, server 12.5)
WARNING: psql version 9.2, server version 12.0.
         Some psql features might not work.
SSL connection (cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256)
You are now connected to database "test_database" as user "postgres".
test_database=> INSERT INTO test_table VALUES ('0004', 'audit設定している方に再INSERT', 28);
INSERT 0 1
test_database=>
```
