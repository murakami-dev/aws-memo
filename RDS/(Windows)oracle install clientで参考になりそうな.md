# 文献
- [Microsoft Windows（x64）64ビット用のインスタントクライアントダウンロード](https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html)
  - 公式。一番下に分かりにくいが手順の記載もあり
  - [The latest supported Visual C++ downloads](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)
    - 手順の下部にあるリンク。これもインストールしないとダメなよう
- [Windows10(64bit)にOracle Instant ClientをインストールしSQL*Plusを使えるようにする手順](https://souiunogaii.hatenablog.com/entry/OracleInstantClient-Windows10)
  - 手順。
  - ダウンロードしたファイルを**ひとつのファイル直下に配置**して、そのフォルダを環境変数に設定する

# やったこと
## 必要なzipファイルのダウンロード・展開
- ひとつのフォルダ直下に配置するのが必須
- `instantclient-basic-windows.x64-19.9.0.0.0dbru`と`instantclient-sqlplus-windows.x64-19.9.0.0.0dbru`だけでいけた
![image](https://user-images.githubusercontent.com/60077121/104813693-2700f680-584e-11eb-968c-09e6034d7d9f.png)

## システム環境変数の設定
- `Path`で`OracleInstantClient`フォルダを指定
- `TNS_ADMIN`で`OracleInstantClient`フォルダを指定（これ多分不要）
- 文献にある`NLS_LANG`は設定してない
- tnsnames.oraの設定もやってない

## Visual C++ダウンロード
- 文献に従って


## 接続確認
- [Oracle DB インスタンスへの接続](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_ConnectToOracleInstance.html)
```
C:\Users\Administrator>sqlplus admin@(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=database-1.ckqtks0xjen9.ap-northeast-1.rds.amazonaws.com)(PORT=1521))(CONNECT_DATA=(SID=ORCL)))

SQL*Plus: Release 19.0.0.0.0 - Production on Sat Jan 16 13:47:57 2021
Version 19.9.0.0.0

Copyright (c) 1982, 2020, Oracle.  All rights reserved.

Enter password:

Connected to:
Oracle Database 19c Standard Edition 2 Release 19.0.0.0.0 - Production
Version 19.9.0.0.0

SQL>
```
