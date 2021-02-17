# やること
RDS VS Auroraのメモ

# 詳しくは文献（あくまでメモ）
- [AuroraかRDSどちらを選ぶべきか比較する話をDevelopers.IO 2019 in OSAKAでしました](https://dev.classmethod.jp/articles/developers-io-2019-in-osaka-aurora-or-rds/#toc-9)

# インスタンスタイプ
- Auroraはt系とr系（メモリ最適化）のみ
  - > Amazon Aurora は、2 種類のインスタンスクラス (メモリ最適化、バーストパフォーマンス) をサポートしています。
  - >https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html
  
# 料金
https://aws.amazon.com/jp/rds/pricing/

|  | インスタンス料金/h | ストレージ/GB・月 | IO料金/100万リクエスト |
----|---- |---- |---- 
| RDS posgre m5.large（マルチAZ） | 0.494USD |0.276USD | \- |
| RDS posgre r5.large（マルチAZ） | 0.60USD |0.276USD| \- |
| Aurora r5.large（リードレプリカ1台とした場合） | 0.7USD(0.35*2) | 0.12USD |0.24USD |

- r5 -> vCPU: 2 Memory: 16 GiB
- m5 -> vCPU: 2 Memory: 8 GiB


