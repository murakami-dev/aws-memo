# 目的
- サイズの大きなファイルをEC2へ。
- EC2はRHEL
- 下部に帯域制限

# 手順
## ローカルに10GBくらいの重いファイルを作成
- [Windows 10対応：巨大サイズのファイルを簡単に作る（fsutilコマンド編）](https://www.atmarkit.co.jp/ait/articles/0209/28/news002.html)
- [AWS CLI での高レベル (S3) コマンドの使用](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-managing-objects-copy)
```
C:\Users\user>fsutil file createnew testfile 10000000000
ファイル C:\Users\user\testfile が作成されました

C:\Users\user>aws s3 cp C:\Users\user\testfile s3://bucket-name
upload: .\testfile to s3://bucket-name/testfile
```
## EC2へCLI入れる
- [Linux での AWS CLI バージョン 2 のインストール、更新、アンインストール](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-linux.html)
```
[ec2-user@ip-10-123-10-167 ~]$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
### unzip入れる
```
[ec2-user@ip-10-123-10-167 ~]$ sudo yum install unzip -y
```
### CLI入れる
```
[ec2-user@ip-10-123-10-167 ~]$ unzip awscliv2.zip
[ec2-user@ip-10-123-10-167 ~]$ sudo ./aws/install
```

### IAMロールでS3へのアクセス権付与

## S3からEC2へ
```
[ec2-user@ip-10-123-10-167 ~]$ aws s3 cp s3://bucket-name/testfile /home/ec2-user
[ec2-user@ip-10-123-10-167 ~]$ ls -l
total 9801704
drwxr-xr-x. 3 ec2-user ec2-user          78 Mar  9 23:02 aws
-rw-rw-r--. 1 ec2-user ec2-user    36929936 Mar 10 09:18 awscliv2.zip
-rw-rw-r--. 1 ec2-user ec2-user 10000000000 Mar 10 09:04 testfile
```

# 帯域制限をする手順
- [AWS CLI S3 Configuration](https://docs.aws.amazon.com/cli/latest/topic/s3-config.html)
- [AWS CLIがS3とのトラフィックを帯域制御できるようになりました](https://dev.classmethod.jp/articles/rate-limit-aws-cli-s3-transfer/)

## 帯域制限の設定コマンド
- ファイル送信元端末で以下のコマンド実行
- `default`はCLIの`config`ファイルのプロファイルに該当する部分。異なるプロファイルを使用する場合は変更すること。
```
aws configure set default.s3.max_bandwidth 10MB/s
```
### 上記コマンドに成功するとconfigに以下が追記される
```
[default]
region = ap-northeast-1
s3 =
    max_bandwidth = 10MB/s
```

## あとはやるだけ
- time使ったら転送時間測れるらしい

```
time aws s3 cp test.dat s3://BUCKET/default
```

