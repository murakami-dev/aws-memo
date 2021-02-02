# 文献
- 1.[オリジンアクセスアイデンティティを使用して Amazon S3 コンテンツへのアクセスを制限する](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- 2.[S3+CloudFrontでS3のURLにリダイレクトされてしまう場合の対処法](https://dev.classmethod.jp/articles/s3-cloudfront-redirect/)
# 要件
以下があればS3はブロックパブリックアクセスOK
- オリジンの編集から以下を設定
  - `Restrict Bucket Access`
  - `Origin Access Identity`
  - `Grant Read Permissions on Bucket`
- `Default Root Object`の指定。拡張子含む
- オリジン名にリージョンを追加（バケット作成から24時間以内の場合）
  - 文献2参照
