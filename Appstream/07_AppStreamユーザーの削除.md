# 文献
- [Amazon AppStream 2.0のユーザーを削除する](https://dev.classmethod.jp/articles/delete-appstream-2-0-user/)
# 手順
## 現在のユーザーを参照
```
C:\Users\user>aws appstream describe-users --authentication-type USERPOOL
{
    "Users": [
        {
            "Arn": "arn:aws:appstream:ap-northeast-1:913926916566:user/userpool/db4bd808-3eb6-4d7c-b3e4-e45162cf9641",
            "UserName": "hiroya.murakami@serverworks.co.jp",
            "Enabled": true,
            "Status": "CONFIRMED",
            "FirstName": "swx",
            "LastName": "swx",
            "CreatedTime": "2020-09-19T12:25:54.951000+09:00",
            "AuthenticationType": "USERPOOL"
        }
    ]
}
```

## ユーザーを削除
```
C:\Users\user>aws appstream delete-user --user-name hiroya.murakami@serverworks.co.jp --authentication-type USERPOOL
```

## ユーザーが削除されていることを確認
```
C:\Users\user>aws appstream describe-users --authentication-type USERPOOL
{
    "Users": []
}
```
