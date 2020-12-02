# IP制限
- [送信元 IP に基づいて AWS へのアクセスを拒否する](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference_policies_examples_aws_deny-ip.html)
- 挙動としてはマネコンへのログインは可能。ただし何のリソースも見れない

# ユーザー作成時のメール送信
![image](https://user-images.githubusercontent.com/60077121/100893111-59ed6600-34fe-11eb-8250-7f242a831970.png)

# MFA強制
- [AWS: MFA で認証された IAM ユーザーがセキュリティ認証情報ページで自分の認証情報を管理できるようにする](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/reference_policies_examples_aws_my-sec-creds-self-manage.html)
  - >このポリシー例では、サインイン時のパスワードのリセットをユーザーに許可していません。新しいユーザーおよびパスワードの期限が切れているユーザーは、これを行う場合があります。この操作を許可するには、iam:ChangePassword と iam:GetAccountPasswordPolicy をステートメント DenyAllExceptListedIfNoMFA に追加します。ただし、IAM ではこのようなアクセス許可をお勧めしません。ユーザーが MFA なしで自分のパスワードを変更できるようにすると、セキュリティ上のリスクが生じる可能性があります。
- ★[IAMユーザにMFA設定を強制するにあたりiam:ListUsersが必須では無くなった話](https://blog.serverworks.co.jp/tech/2019/04/02/mysecuritycredentials/)

## 利用の流れ（★ブログ参照）
- IAMポリシー作成（下記またはドキュメント参照）
- IAMユーザー作成
  - 初回ログイン時のパスワード変更はチェックしない（MFAログインしていない時点でのパスワード変更は非推奨、以下のポリシーでは変更できない）
- IAMユーザログイン
- 右上のマイセキュリティ資格情報からMFA登録
- 再ログイン
- これでやっとパスワード変更できるしリソース見れる。

## デフォルトのポリシー
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowViewAccountInfo",
            "Effect": "Allow",
            "Action": [
                "iam:GetAccountPasswordPolicy",
                "iam:GetAccountSummary",       
                "iam:ListVirtualMFADevices"
            ],
            "Resource": "*"
        },       
        {
            "Sid": "AllowManageOwnPasswords",
            "Effect": "Allow",
            "Action": [
                "iam:ChangePassword",
                "iam:GetUser"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnAccessKeys",
            "Effect": "Allow",
            "Action": [
                "iam:CreateAccessKey",
                "iam:DeleteAccessKey",
                "iam:ListAccessKeys",
                "iam:UpdateAccessKey"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnSigningCertificates",
            "Effect": "Allow",
            "Action": [
                "iam:DeleteSigningCertificate",
                "iam:ListSigningCertificates",
                "iam:UpdateSigningCertificate",
                "iam:UploadSigningCertificate"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnSSHPublicKeys",
            "Effect": "Allow",
            "Action": [
                "iam:DeleteSSHPublicKey",
                "iam:GetSSHPublicKey",
                "iam:ListSSHPublicKeys",
                "iam:UpdateSSHPublicKey",
                "iam:UploadSSHPublicKey"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnGitCredentials",
            "Effect": "Allow",
            "Action": [
                "iam:CreateServiceSpecificCredential",
                "iam:DeleteServiceSpecificCredential",
                "iam:ListServiceSpecificCredentials",
                "iam:ResetServiceSpecificCredential",
                "iam:UpdateServiceSpecificCredential"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnVirtualMFADevice",
            "Effect": "Allow",
            "Action": [
                "iam:CreateVirtualMFADevice",
                "iam:DeleteVirtualMFADevice"
            ],
            "Resource": "arn:aws:iam::*:mfa/${aws:username}"
        },
        {
            "Sid": "AllowManageOwnUserMFA",
            "Effect": "Allow",
            "Action": [
                "iam:DeactivateMFADevice",
                "iam:EnableMFADevice",
                "iam:ListMFADevices",
                "iam:ResyncMFADevice"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
        },
        {
            "Sid": "DenyAllExceptListedIfNoMFA",
            "Effect": "Deny",
            "NotAction": [
                "iam:CreateVirtualMFADevice",
                "iam:EnableMFADevice",
                "iam:GetUser",
                "iam:ListMFADevices",
                "iam:ListVirtualMFADevices",
                "iam:ResyncMFADevice",
                "sts:GetSessionToken"
            ],
            "Resource": "*",
            "Condition": {
                "BoolIfExists": {
                    "aws:MultiFactorAuthPresent": "false"
                }
            }
        }
    ]
}
```

