# 文献
https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/example-policies-s3.html#iam-policy-ex0


# EC2から特定のバケットへアクセスするための権限
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",  #これがないとaws s3 lsできない
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",               # これがないとaws s3 ls s3://my_bucket/できない
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::my-bucket"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::my-bucket/*"
        }
    ]
}
```
