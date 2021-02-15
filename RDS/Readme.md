## 主な設定
- DB インスタンス識別子:DB インスタンスの名前を入力します。この名前は、AWS アカウントが現在の AWS リージョンで所有しているすべての DB インスタンスにおいて一意である必要があります。
- マスターユーザ名とパスワード

## DB インスタンスサイズ
- db.t3.smallなど。cpu,メモリ(Gib RAM),EBS(Mbps)
- [Amazon RDS DB インスタンスを変更する](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html)
  - 様々なインスタンスの変更とダウンタイムの関係

## ストレージ
- タイプ
  - gp2
    - [Tips：RDSでのIOクレジットバランスの考え方（汎用SSDボリューム）](https://dev.classmethod.jp/articles/rds_gp2_iocreditbarance/)
  - プロビジョンドIOPS
  - マグネティック
- ストレージ割り当て(GiB)
- 自動スケーリング、最大ストレージ
  - [【DBのディスクサイズ管理が簡単に】RDSのストレージがストレージの自動スケーリングをサポートしました！](https://dev.classmethod.jp/articles/rds-storage-auto-scaling/)
  - ストレージ割り当て20GiBで最大ストレージしきい値1,000GiBの場合、20の10%<5GiBのため、スケーリングする容量は5GiB
  - >追加のストレージは、次のうちいずれか大きい方の増分です。/5 GiB/現在割り当てられているストレージの 10%
  
  - ダウンタイム
    - >ほとんどの場合、ストレージのスケーリングには停止が必要ではなく、サーバーのパフォーマンスを低下させません。
    - >インスタンスでストレージの最適化が完了してから 6 時間後まではストレージをこれ以上変更することはできません。
    - >ただし、特別なケースとして、SQL Server DB インスタンスを所有しているが、2017 年 11 月以降にストレージ設定を変更していない場合があります。この場合、割り当てられたストレージを増やすように DB インスタンスを変更すると、数分の短い停止時間が発生する場合があります。
    - https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_PIOPS.StorageTypes.html#USER_PIOPS.ModifyingExisting

  - 上記の理由（6時間ルール）で、ワークロードの増加が予想できている場合は手動でスケールするのがベストプラクティス
  - >データロードの増加が予想される場合は (RDS DB インスタンスの現在のストレージサイズの 20% を超えるロードなど)、データロードの前に RDS DB ストレージのサイズを手動で増加させることがベストプラクティスです。
  - [ストレージの Auto Scaling を有効化したのに Amazon RDS DB インスタンスのストレージ容量が少ない、または DB インスタンスが Storage-Full 状態なのはなぜですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/rds-autoscaling-low-free-storage/)

 - ストレージ拡張の所要時間
   - RDSオラクルでシングル構成、r5.xlargeで20->50GiBに変更で20分かかった。→数十分かかるが以下のとおりワークロードに依るので不明。
   - https://aws.amazon.com/jp/premiumsupport/knowledge-center/rds-stuck-modifying/

## 可用性と耐久性
### マルチAZシングルAZ
- マルチAZのフェイルオーバーのダウンタイムは1,2分
  - [RDS マルチ AZ 配置](https://aws.amazon.com/jp/rds/features/multi-az/) 
  - >Amazon Aurora の場合 1 分間以内 (Maria DB Connector/J を使用する場合わずか 30 秒)、その他のデータベースエンジンでは 1～2 分間です
- プライマリAZの変更はマルチAZ構成の状態で行う必要あり。シングルAZ構成でAZ変えたい場合は一旦マルチAZ構成に。
- ![image](https://user-images.githubusercontent.com/60077121/107112763-6941b480-689d-11eb-92f8-0a0ab6479174.png)

### レプリケーション
  - レプリケーションなしだとこんな感じの出方
  - ![image](https://user-images.githubusercontent.com/60077121/99684651-096d1600-2ac5-11eb-83ed-8d556ceb1edd.png)

## 接続
- VPC
- サブネットグループ（必ずAZをまたぐように設定しないとダメ）
- SG
- AZ（マルチAZなしを選んだ場合のみ選択する）
- port`1521`

### ネットワークの変更-VPC,AZ変更には再起動必要
- [Amazon RDS DB インスタンスの VPC を変更するにはどうすれば良いですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/change-vpc-rds-db-instance/)
  - DB インスタンスの VPC を変更すると、インスタンスがあるネットワークから別のネットワークに移動したときにインスタンスが再起動します。
- [[小ネタ] RDSのMulti-AZ構成を切り替える手順](https://dev.classmethod.jp/articles/how-to-change-rds-multiaz-configuration/)
- [[小ネタ] RDSのSingle-AZ構成でAvailability Zoneを変更する方法](https://dev.classmethod.jp/articles/rds-singleaz-change-az/)
  - AZ-cでシングル構成で構築したRDSをマルチAZ構成に変更→フェイルオーバー強制実行でプライマリAZをcからaに変更→シングル構成に変更

- 同じVPCにあるサブネットグループへの変更は不可
![image](https://user-images.githubusercontent.com/60077121/99568391-b3d83100-2a12-11eb-82be-49b018f59320.png)
```
申し訳ありません。DB インスタンス database-1 の変更のリクエストが失敗しました。
You cannot move DB instance database-1 to subnet group tech-kadai-subnetgroup. The specified DB subnet group and DB instance are in the same VPC. Choose a DB subnet group in different VPC than the specified DB instance and try again.
```
- 違うVPCのサブネットグループならOK
![image](https://user-images.githubusercontent.com/60077121/99569152-a0799580-2a13-11eb-92b7-180d448c9a93.png)


# 追加設定
## パラメータグループ
- https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html
`DB エンジンの設定を管理するには、DB インスタンスをパラメータグループに関連付けます。`

- パラメータグループを特に指定せずRDS作成するとデフォルトのパラメータグループが適用されるが、これは後から変更できず、変更するにはパラメータグループの変更（RDS再起動有）が必要になる
- [RDSのデフォルトパラメータグループの罠](https://qiita.com/ssssasaki/items/f4ab5aacaf1b724ffc50)
- [Amazon RDS DB パラメーターグループの値を変更する方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/rds-modify-parameter-group-values/)
- [DB パラメータグループのパラメータの変更](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html#USER_WorkingWithParamGroups.Modifying)
```
ユーザー定義の DB パラメータグループのパラメータ値は変更できますが、デフォルトの DB パラメータグループのパラメータ値を変更することはできません。
```
- デフォルトのパラメータグループは変更できないので、デフォルトと同じ内容のパラメータグループを割り当てた方が良さそう
![image](https://user-images.githubusercontent.com/60077121/98461421-4b59aa80-21ef-11eb-9a6d-c35464b969f6.png)
![image](https://user-images.githubusercontent.com/60077121/98461557-6842ad80-21f0-11eb-903d-0993c2dfbef7.png)

### Tips:パラメータグループをcsvに出力する方法
- jqインストールしてexeを環境変数PATHのディレクトリに置く。
  - https://stedolan.github.io/jq/
- 以下のコマンドをGit Bashで。iconvがあるから
  - `aws rds describe-db-parameters --db-parameter-group-name xxxxxxxx | jq-win64.exe -r '["Name","Values","Allowed values","Modifiable","Source","Apply type","Data type","Description","ApplyMethod","MinimumEngineVersion"], (.Parameters[] | [.ParameterName,.ParameterValue,.AllowedValues,.IsModifiable,.Source,.ApplyType,.DataType,.Description,.ApplyMethod,.MinimumEngineVersion]) | @csv' > param.csv`
- 以下のサイトでcsv -> マークダウンも
- https://tableconvert.com/

## オプショングループ
- https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_WorkingWithOptionGroups.html
```
Amazon RDS では、新しい DB インスタンスごとに空のデフォルトオプショングループが用意されます。このデフォルトオプショングループを変更することはできませんが、デフォルトオプショングループからその設定を引き継いで新しいオプショングループを作成することはできます。
```
- 例
  - `S3_INTEGRATION`:S3統合。Amazon RDS for Oracle DB インスタンスと Amazon S3 バケットの間でファイルを転送することができます。
  - `Timezone`:Oracle DB インスタンスで使用するシステムのタイムゾーンを変更することができます。

- パラメータグループとオプショングループは後からでも変更可能。ただし動的
![image](https://user-images.githubusercontent.com/60077121/98460610-2e21dd80-21e9-11eb-9fc2-667819bb1cae.png)


## バックアップ
(https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- 自動バックアップと手動スナップショット
- 自動・手動バックアップをコピーすることで手動スナップショットとなる。コピーは無期限保持だし、アカウント間、リージョン間で共有可能
- 復元
  - [Aurora MySQL の スナップショットの復元、ポイントインタイムリカバリ、バックトラックの違いを簡単にまとめてみた](https://dev.classmethod.jp/articles/aurora-mysql-snapshot-point-in-time-backtrack/)
  - スナップショットから復元
    - スナップショットを元に新たなクラスターを作成する
    - [RDSインスタンスをスナップショットからリストアする際に、元のインスタンスと同じエンドポイントにする方法](https://dev.classmethod.jp/articles/tsnote-rds-restore-endpoint-001/)
      - エンドポイントを変えないために既存のRDSの識別子を変えてから、復元時に元の識別子を指定する
      - >同一アカウント内、同一リージョン、同一識別子であれば、エンドポイントは同じ
  - ポイントインタイムリカバリ
    - >RDS は、DB インスタンスのトランザクションログを 5 分ごとに Amazon S3 にアップロードします。DB インスタンスの復元可能な直近の時間を調べるには、AWS CLI の describe-db-instances コマンドを使用し、DB インスタンスの [LatestRestorableTime] フィールドに返される値を参照します。Amazon RDS コンソールで各 DB インスタンスの復元可能な直近の時間を表示するには、[自動バックアップ] を選択します。
### 自動バックアップ
- 自動バックアップの有効・無効が選択できる。
- 保持期間はデフォルト7日。MAX35日。0にすると無効（非推奨）。** 0 から 0 以外の値、0 以外の値から 0 に変更した場合、機能停止が発生します。**
- バックアップウィンドウのデフォルトは`13:00–21:00 UTC`。自分で選択することもできる
  - 毎日行われる。頻度の変更は不可のよう。
  - バックアップウィンドウに割り当てられた時間より長い時間がバックアップに必要な場合、ウィンドウが終了した後もバックアップが完了するまでバックアップが継続します。
  - 週 1 回のメンテナンス時間とバックアップウィンドウは重複できません。
- **インスタンスを削除すると自動バックアップも削除される**
  - >Amazon RDS で自動スナップショットが削除されるのは、その保持期間の終了時、DB インスタンスの自動スナップショットを無効にしたとき、および DB インスタンスを削除したときのいずれかです。
- 増分バックアップ
- **1リージョン最大40保持**
- パラメータグループまたはオプショングループについての情報は含まれません。

### 手動スナップショット
- 保持期間の設定はない。ずっと保存される
- **シングルAZの場合、スナップショットとっている間、数秒から数分停止する。SQL Serverの場合マルチAZでも停止発生する**
  - PostgreSQL DB インスタンスの場合、ログに記録されていないテーブルのデータはスナップショットから復元されないことがある。
  - https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_CreateSnapshot.html

## 暗号化
- 通信の暗号化
  - デフォルトでTLS通信なっている
  - マネコン画面で証明書、期限がみれる
  - 期限が切れたらマネコン左下のアップデートに表示されるので、DBの変更から実施する
- 保管データの暗号化
  - 起動時に設定する（途中変更不可）

## メンテナンス
https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Maintenance.html
- 週次で行われる
- メンテナンス中はパフォーマンス落ちることがある。**まれにマルチAZフェイルオーバーが必要なことがある**
- マルチAZのときはスタンバイでメンテナンス→スタンバイがプライマリに昇格→プライマリでメンテナンス
  - [DB インスタンスのメンテナンス](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_UpgradeDBInstance.Maintenance.html)
- 停止を伴うものあるし稀にメンテナンスウィンドウ以外に発生する場合もある
  - [RDSのメンテナンス基礎知識（パッチ適用）](https://tech-dive.xyz/2019/02/15/post-2144/)★これは良いブログ
  - >一部のメンテナンス項目では、Amazon RDS が DB インスタンス を少しの間オフラインにする必要があります。リソースをオフラインにする必要があるメンテナンス項目には、必要なオペレーティングシステムやデータベースのパッチが含まれます。セキュリティやインスタンスの信頼性に関連するパッチのみ、必須のパッチ適用として自動的にスケジューリングされます。このようなパッチは頻繁に発生するものではありません (通常数ヵ月ごとに一度です)。またお客様のメンテナンスウィンドウのごく一部以外を使用する必要があることは稀なはずです。
- メンテナンスはウィンドウ内に開始されるが、ウィンドウ内で終わるとは限らない
  - >メンテナンス時間は、保留中のオペレーションを開始する時刻を指定しますが、オペレーションの合計所要時間は制限されません。メンテナンスオペレーションは、メンテナンスウィンドウが終了するまでに完了するかどうかは保証されておらず、指定終了時間を超える場合もあります。
- 止めたくない場合はマルチAZ。こうすればダウンタイムはフェイルオーバー時の1,2分に抑えられる。
  - [必要な Amazon RDS メンテナンスの実行時におけるダウンタイムを最小限に抑えるにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/rds-required-maintenance/)
- マイナーバージョン自動アップグレードの有効化
- メンテナンスウィンドウ

## Performance Insights
- 有効・無効
- 保持期間

## モニタリング
- [RDS のモニタリングの概要についてまとめた](https://dev.classmethod.jp/articles/rds-monitoring-summary/)**これはいい記事**
- 詳細モニタリングを有効にするかどうか


# 料金

- ストレージの課金は日割り計算
>Q: Amazon RDS の利用料金は、どのようにして課金され、請求されますか?
使用料金は従量課金制となっており、最低料金やセットアップ料金はありません。以下の内容に基づき、請求が行われます。

>DB インスタンス時間 – 消費された DB インスタンスのクラス (例: db.t2.micro、db.m4.large など) に基づいています。1 時間に満たない DB インスタンスの利用は、1 時間として請求されます。
ストレージ (/GB/月) – DB インスタンスに対してプロビジョニングしたストレージ容量。プロビジョニングしたストレージ容量を当月にスケールした場合、請求は日割り料金となります。
I/O リクエスト/月 – ストレージ I/O リクエストの合計 (Amazon RDS Magnetic ストレージおよび Amazon Aurora のみ)
プロビジョンド IOPS/月 – 消費 IOPS とは無関係な、プロビジョンド IOPS レート (Amazon RDS プロビジョンド IOPS (SSD) ストレージのみ)
バックアップストレージ – バックアップストレージは、自動化されたデータベースバックアップや、お客様の作成したデータベーススナップショットに関連付けられたストレージです。バックアップ保持期間を増やしたり、追加のデータベーススナップショットを取得したりすれば、お客様のデータベースが使用するバックアップストレージは増大します。
データ転送 – DB インスタンスのインターネット経由のデータ受信および送信です。


