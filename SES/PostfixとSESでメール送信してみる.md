# 目的
- SESでメール送信する手順を知る
- ![image](https://user-images.githubusercontent.com/60077121/111266810-6725ff00-866e-11eb-9797-c7498d1c482c.png)


# 文献
- [SESのSMTPインターフェースでメール送信](https://blog.serverworks.co.jp/tech/2020/02/08/ses-smtp-interface/)
  - >サンドボックス環境にいる場合は、送信先メールアドレスにも制限がかかります。
  - 認証済みアドレスにしか送信できない

# 手順
## 送信先のメールアドレス認証（サンドボックス環境のみ）
- サンドボックス環境なので送信先のメールアドレスを認証
- ![image](https://user-images.githubusercontent.com/60077121/111264309-99cdf880-866a-11eb-8ee8-d06405e1dfe9.png)

## 送信元ドメインの認証
- 個別のメールアドレスでメール認証しても良いが、ドメイン認証すると当該ドメインのすべてのメールアドレスが認証される。
  - [Amazon SES でのドメインの検証](https://docs.aws.amazon.com/ja_jp/ses/latest/DeveloperGuide/verify-domains.html)
  - >ドメイン全体を検証すると、そのドメインのすべての E メールアドレスを検証することになるため、そのドメインの E メールアドレスを個別に検証する必要はありません。
- Route53にレコードが追加される。`Generate DKIM`にチェック入れるとDKIMレコードも追加される
- ![image](https://user-images.githubusercontent.com/60077121/111265055-e239e600-866b-11eb-9321-3b9539ea78ca.png)

## Amazon SES での E メールの送信方法
- [Amazon SES での E メールの送信方法](https://docs.aws.amazon.com/ja_jp/ses/latest/DeveloperGuide/choose-email-sending-method.html)
  - コンソール（テスト時）
  - SMTP インターフェイスまたは API
### Postfix設定
- [Amazon SES と Postfix の連携](https://docs.aws.amazon.com/ja_jp/ses/latest/DeveloperGuide/postfix.html#send-email-postfix)
- 基本は文献通りに

### postfix追加設定
```
[root@ip-10-123-10-6 new]# sudo postconf -e "relayhost = [email-smtp.ap-northeast-1.amazonaws.com]:587" \
> "smtp_sasl_auth_enable = yes" \
> "smtp_sasl_security_options = noanonymous" \
> "smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd" \
> "smtp_use_tls = yes" \
> "smtp_tls_security_level = encrypt" \
> "smtp_tls_note_starttls_offer = yes"
```

### SMTP認証情報の取得
- [Amazon SES SMTP 認証情報の取得](https://docs.aws.amazon.com/ja_jp/ses/latest/DeveloperGuide/smtp-credentials.html)

### SMTP認証上の設定
- `/etc/postfix/sasl_passwd`を新規作成
```
[email-smtp.us-west-2.amazonaws.com]:587 SMTPUSERNAME:SMTPPASSWORD
```
### SMTP 認証情報を含む hashmap データベースファイルを作成します
```
[root@ip-10-123-10-6 postfix]# sudo postmap hash:/etc/postfix/sasl_passwd
```

### Postfix が CA 証明書の場所を認識できるようにします
- >（Amazon SES サーバー証明書を検証するために必要です）。

```
[root@ip-10-123-10-6 postfix]# sudo postconf -e 'smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt'
```

### 設定反映、メール送信
```
[root@ip-10-123-10-6 postfix]# sudo postfix start
postfix/postfix-script: starting the Postfix mail system
[root@ip-10-123-10-6 postfix]# sudo postfix reload
postfix/postfix-script: refreshing the Postfix mail system
[root@ip-10-123-10-6 postfix]# sendmail -f muser@stg-xxxxxxxxxxx.work xxxxx@serverworks.co.jp
From: muser <muser@stg-xxxxxxxxx.work>
Subject: Amazon SES Test
This message was sent using Amazon SES.
.
```

### 届いた
![image](https://user-images.githubusercontent.com/60077121/111266625-26c68100-866e-11eb-8a4b-8da8a196d2d7.png)








