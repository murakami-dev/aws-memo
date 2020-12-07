# 文献
- 1.[【図解/AD】ドメイン参加の仕組みと手順,メリット,参加できない時の注意点～](https://milestone-of-se.nesuke.com/sv-advanced/activedirectory/join-to-domain/)
- 2.[EC2インスタンス上にActive Directory環境を構築する際のTIPS](https://dev.classmethod.jp/articles/tips-for-build-active-directory-on-ec2/)
  - ドメインコントローラーのフォワーダーにはAmazonProvidedDNSを設定すること
- 3.[SSMを利用し、起動したWindowsServerを自動でドメイン参加させる](https://blog.serverworks.co.jp/tech/2017/02/21/windowsserver-ssm-domain/)

# ADに参加しているか確認する方法
- クライアントで以下のコマンド
```
PS C:\Users\hiroya> whoami /fqdn
CN=hiroya,CN=Users,DC=murakami,DC=com
```

# インスタンス（クライアント側）にログインしてやる方法
- 文献1,2
  - クライアントからの参加手順
  - DNSサーバ変更→システムからADへ参加
  - >あらかじめ配られたドメインユーザの ID / パスワードを使ってログインします。ドメインの指定方法はユーザ名の前に『ドメイン名\』をつけるか、ユーザ名の後に『@ドメイン名』をつけます。

# RunCommandで参加させる方法もある
- [AWS Systems Manager を使用して、実行中の EC2 Windows インスタンスを、AWS Directory Service のドメインに参加させる方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-systems-manager-dx-domain/)
  - IAMロールに注意が必要。`AmazonSSMDirectoryServiceAccess`がいる

## 上記をユーザーデータでやってみる
- 参考→文献3
- 注意
  - `ssm:CreateAssociation`のポリシーが必要（ManagedInstanceCoreにもEC2RoleforSSMにもない）
  - 以下のようなポリシーを別途アタッチした。
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ssm:CreateAssociation",
            "Resource": "*"
        }
    ]
  }
  ```
### 実行結果
```
PS C:\Windows\system32> Set-DefaultAWSRegion -Region ap-northeast-1
>> Set-Variable -name instance_id -value (Invoke-Restmethod -uri http://169.254.169.254/latest/meta-data/instance-id)
PS C:\Windows\system32> $instance_id
i-0964a3ce9bccaf0d6
PS C:\Windows\system32> $params = @{
>>     InstanceId   = $instance_id;
>>     Name = 'AWS-JoinDirectoryServiceDomain';
>>     Parameter    = @{
>>         directoryId    = 'd-95671ad6a3';
>>         directoryName  = 'murakami.com';
>>         dnsIpAddresses = @('10.123.10.31');
>>     };
>> }
>> New-SSMAssociation @params


Name                  : AWS-JoinDirectoryServiceDomain
InstanceId            : i-0964a3ce9bccaf0d6
Date                  : 12/7/2020 3:25:21 PM
Status.Name           : Associated
Status.Date           : 12/7/2020 3:25:21 PM
Status.Message        : Associated with AWS-JoinDirectoryServiceDomain
Status.AdditionalInfo :
```
