
# やること
- [1 か月あたりに記録された設定項目の数を取得することにより、AWS Config の請求を理解する方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/retrieve-aws-config-items-per-month/)
- Configがログを出力しているS3にAthenaでSQLクエリを投げて変更回数として集計する。
- 詳細に変更回数を知るにはこれしかない（Configの高度なクエリでは現在の構成にしか対応していない）
- 変更回数X0.003$が課金

# 手順
## テーブル作成
- クエリエディタに入力（その前にクエリの結果の保存先のバケットが聞かれたら適宜設定する）
```
CREATE EXTERNAL TABLE awsconfig (
         fileversion string,
         configSnapshotId string,
         configurationitems ARRAY < STRUCT < configurationItemVersion : STRING,
         configurationItemCaptureTime : STRING,
         configurationStateId : BIGINT,
         awsAccountId : STRING,
         configurationItemStatus : STRING,
         resourceType : STRING,
         resourceId : STRING,
         resourceName : STRING,
         ARN : STRING,
         awsRegion : STRING,
         availabilityZone : STRING,
         configurationStateMd5Hash : STRING,
         resourceCreationTime : STRING > > 
) 
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe' LOCATION 's3://バケット名/AWSLogs/アカウントID/Config/ap-northeast-1/';
```
## クエリ
- 8月のリソースごとの変更回数多い順
```
SELECT configurationItem.resourceType,
         configurationItem.resourceId,
         COUNT(configurationItem.resourceId) AS NumberOfChanges
FROM sampledb.awsconfig                 ###データベース名.テーブル名にする
CROSS JOIN UNNEST(configurationitems) AS t(configurationItem)
WHERE "$path" LIKE '%ConfigHistory%'
        AND configurationItem.configurationItemCaptureTime >= '2020-08-01T%'   ###日付入力
        AND configurationItem.configurationItemCaptureTime <= '2020-08-31T%'
GROUP BY  configurationItem.resourceType, configurationItem.resourceId
ORDER BY  NumberOfChanges DESC
```
## 結果
![image](https://user-images.githubusercontent.com/60077121/103707614-4d26cb00-4ff2-11eb-835e-2436277f62be.png)
