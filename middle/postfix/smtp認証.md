# 目的
- SMTP認証の理解

# 概要

# 文献
- [pamを使う設定](https://futuremix.org/2009/11/postfix-smtp_auth-pam)
  - サーバのユーザの認証を使いまわせるので便利

# （pam）設定後の挙動
- 認証とかなくメール送れた（ローカルユーザどうしだけど）
```
[root@ip-10-123-10-6 new]# echo "test by auth" | mail -s "title" -r muser@stg-development.work test@stg-development.work
[root@ip-10-123-10-6 new]#
```
