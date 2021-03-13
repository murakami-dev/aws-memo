# 目的
- postfix使ってみる
- 25番ポートの申請の意味を理解する

# 文献
- [【第1回】【EC2】PostfixでシンプルなSMTPサーバを構築してみる](https://blog.serverworks.co.jp/build-smtp-server)
- [Amazon Linux 2上のPostfixでローカルユーザにメール配送してみた](https://dev.classmethod.jp/articles/ec2-postfix-local-delivery/)
- [Amazon EC2 Eメール送信ベストプラクティス](https://dev.classmethod.jp/articles/ec2-send-email-best-practice/)
- 

# 手順
## Route53
- ![image](https://user-images.githubusercontent.com/60077121/109775372-c6364d80-7c44-11eb-940b-a17bbe0f9957.png)

## SGで25あける

## インストール
### インストールされているか確認
- Available Packages以下にあるのでインストールはされていない模様
- Amazon Linux2ならインストールされているとのこと
```
[ec2-user@ip-10-123-10-6 ~]$ sudo yum list postfix
Red Hat Update Infrastructure 3 Client Configuration Server 8                               15 kB/s | 3.6 kB     00:00
Red Hat Enterprise Linux 8 for x86_64 - AppStream from RHUI (RPMs)                          43 MB/s |  26 MB     00:00
Red Hat Enterprise Linux 8 for x86_64 - BaseOS from RHUI (RPMs)                             45 MB/s |  28 MB     00:00
Available Packages
postfix.x86_64                                  2:3.3.1-12.el8_3.1                                  rhel-8-baseos-rhui-rpms
```

### インストール
```
[ec2-user@ip-10-123-10-6 ~]$ sudo yum install postfix
Last metadata expiration check: 0:01:20 ago on Wed 03 Mar 2021 08:04:37 AM UTC.
Dependencies resolved.
===========================================================================================================================
 Package               Architecture         Version                            Repository                             Size
===========================================================================================================================
Installing:
 postfix               x86_64               2:3.3.1-12.el8_3.1                 rhel-8-baseos-rhui-rpms               1.5 M
Installing dependencies:
 libicu                x86_64               60.3-2.el8_1                       rhel-8-baseos-rhui-rpms               8.8 M

Transaction Summary
===========================================================================================================================
Install  2 Packages

Total download size: 10 M
Installed size: 36 M
Is this ok [y/N]: y
Downloading Packages:
(1/2): postfix-3.3.1-12.el8_3.1.x86_64.rpm                                                 9.6 MB/s | 1.5 MB     00:00
(2/2): libicu-60.3-2.el8_1.x86_64.rpm                                                       33 MB/s | 8.8 MB     00:00
---------------------------------------------------------------------------------------------------------------------------
Total                                                                                       34 MB/s |  10 MB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                   1/1
  Installing       : libicu-60.3-2.el8_1.x86_64                                                                        1/2
  Running scriptlet: libicu-60.3-2.el8_1.x86_64                                                                        1/2
  Running scriptlet: postfix-2:3.3.1-12.el8_3.1.x86_64                                                                 2/2
  Installing       : postfix-2:3.3.1-12.el8_3.1.x86_64                                                                 2/2
  Running scriptlet: postfix-2:3.3.1-12.el8_3.1.x86_64                                                                 2/2
  Verifying        : libicu-60.3-2.el8_1.x86_64                                                                        1/2
  Verifying        : postfix-2:3.3.1-12.el8_3.1.x86_64                                                                 2/2

Installed:
  libicu-60.3-2.el8_1.x86_64                               postfix-2:3.3.1-12.el8_3.1.x86_64

Complete!
```
## 設定
### インストール後に`/etc/postfix`の設定ファイルをバックアップ
```
[ec2-user@ip-10-123-10-6 postfix]$ ls
access     dynamicmaps.cf    generic        main.cf        master.cf        postfix-files    relocated  virtual
canonical  dynamicmaps.cf.d  header_checks  main.cf.proto  master.cf.proto  postfix-files.d  transport
[ec2-user@ip-10-123-10-6 postfix]$ sudo cp -a /etc/postfix/main.cf /etc/postfix/main.cf.`date +"%Y%m%d"` //日付でバックアップ
[ec2-user@ip-10-123-10-6 postfix]$ ls
access     dynamicmaps.cf    generic        main.cf           main.cf.proto  master.cf.proto  postfix-files.d  transport
canonical  dynamicmaps.cf.d  header_checks  main.cf.20210303  master.cf      postfix-files    relocated        virtual
```

### 設定変更箇所（viした後に:set numberしてるので冒頭は行番号）
**`#`外すこと**
- 1
```
94 myhostname = mail.stg-development.work
```
- 2
```
 102 mydomain = stg-development.work
```
- 3
```
132  inet_interfaces = all
135 #inet_interfaces = localhost  //#つける
```
- 4
```
183 #mydestination = $myhostname, localhost.$mydomain, localhost
184  mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
```    
- 5
```
438 home_mailbox = Maildir/
```

### 設定確認
- error, warningがなければOK
```
[ec2-user@ip-10-123-10-6 postfix]$ postconf -n
alias_database = hash:/etc/aliases
alias_maps = hash:/etc/aliases
command_directory = /usr/sbin
compatibility_level = 2
daemon_directory = /usr/libexec/postfix
data_directory = /var/lib/postfix
debug_peer_level = 2
debugger_command = PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin ddd $daemon_directory/$process_name $process_id & sleep 5
home_mailbox = Maildir/
html_directory = no
inet_interfaces = all
inet_protocols = all
mail_owner = postfix
mailq_path = /usr/bin/mailq.postfix
manpage_directory = /usr/share/man
meta_directory = /etc/postfix
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
mydomain = stg-development.work
myhostname = mail.stg-development.work
newaliases_path = /usr/bin/newaliases.postfix
queue_directory = /var/spool/postfix
readme_directory = /usr/share/doc/postfix/README_FILES
sample_directory = /usr/share/doc/postfix/samples
sendmail_path = /usr/sbin/sendmail.postfix
setgid_group = postdrop
shlib_directory = /usr/lib64/postfix
smtp_tls_CAfile = /etc/pki/tls/certs/ca-bundle.crt
smtp_tls_CApath = /etc/pki/tls/certs
smtp_tls_security_level = may
smtpd_tls_cert_file = /etc/pki/tls/certs/postfix.pem
smtpd_tls_key_file = /etc/pki/tls/private/postfix.key
smtpd_tls_security_level = may
unknown_local_recipient_reject_code = 550
```
### 起動
```
$ sudo systemctl restart postfix
$ systemctl status postfix
```

## メールの確認
### Maildirディレクトリは勝手に作られる
```
[root@ip-10-123-10-6 muser]# ls
Maildir
[root@ip-10-123-10-6 muser]# cd Maildir/
[root@ip-10-123-10-6 Maildir]# ls
cur  new  tmp
```

### メールの中身確認
- 文字化けしている
```
[root@ip-10-123-10-6 new]# cat 1614763728.Vca02I40252eM466516.ip-10-123-10-6.ap-northeast-1.compute.internal
Return-Path: <hiroya.murakami@serverworks.co.jp>
X-Original-To: muser@stg-development.work
Delivered-To: muser@stg-development.work
Received: from mta52.mta.hdems.com (mta52.mta.hdems.com [3.112.23.19])
        by mail.stg-development.work (Postfix) with ESMTPS id 6E778805820
        for <muser@stg-development.work>; Wed,  3 Mar 2021 09:28:48 +0000 (UTC)
Received: from mo.hdems.com (unknown [10.5.84.126])
        by mta52.mta.hdems.com ('HDEMS') with ESMTPSA id 4Dr7wc1ht1zlfbkD
        for <muser@stg-development.work>; Wed,  3 Mar 2021 09:28:48 +0000 (UTC)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=serverworks.co.jp;
        s=HENNGE; t=1614763728; h=Subject:Message-Id:Date:From;
        bh=4CTuFU8IdFEgRLpG0zaC+kby4PolNWk2KdM50+4zaw0=;
        b=MPX4dVNHTT0n1aQHhA96bDAgNExCe8t0+e8W4x0qDnxaDD9V1Khd2VeCfTg5gB6jEyyzSt3Bm3
         VBWRqSL28m3zf8iing0l2CDFk33AOndcJT9p50yVCbO9ulZvV5kByIrDKNGTDJ8Ga4noS0hRJFox
         VoowRTL/dpwOAQZiu4CDE=
X-HDEMS-MO-TENANT: serverworks.co.jp
Received: from mail-io1-f72.google.com (mail-io1-f72.google.com. [209.85.166.72])
        by gwsmtp.prod.mo.hdems.com with ESMTPS id gwsmtpd-trans-d36ec035-f4e2-4a74-bcb8-fcb89932466d
        for <muser@stg-development.work>;
        Wed, 03 Mar 2021 09:28:42 +0000
Received: by mail-io1-f72.google.com with SMTP id x6so7639737ioj.6
        for <muser@stg-development.work>; Wed, 03 Mar 2021 01:28:42 -0800 (PST)
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=serverworks.co.jp; s=serverworks;
        h=mime-version:from:date:message-id:subject:to;
        bh=4CTuFU8IdFEgRLpG0zaC+kby4PolNWk2KdM50+4zaw0=;
        b=eYNEEfdqt64K17GcZyP2v8RYqMoK0mZ96h94jw1vMYLJd7VZ7lrUTdoUOhZ6YsP2dj
         6Z1FLP8IVaWXYJTmSenxeCh2iqhQ+ZnrIoODKh9pDrbxQET22Qzh8WpQnCa3ujqPwcp4
         laEzWbYFm0DDZHIRDCCbZqTcNWi1ED2EvgoOQ=
X-Google-DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
        d=1e100.net; s=20161025;
        h=x-gm-message-state:mime-version:from:date:message-id:subject:to;
        bh=4CTuFU8IdFEgRLpG0zaC+kby4PolNWk2KdM50+4zaw0=;
        b=PCj8zBP4E736k8/VX4ZiufGHXZQ993M5cm8WPcdjlEy7uCNUBkLXdLBGCXbwJMZUAv
         Emf9ZnTu1e7ZpsTHr2/S9T4oiM2pCZlH2n5euYKUcEjfi7NFkCMnD7BBVxrs7vEk/JR6
         xAb1i/W/Lr4a7uBkMAtk0V2R0Oc7NqzhEjIHPkkae1ANQ+XnV4wgfv6DQ9WOZ2k2M9db
         OsgptXy8lmODT1oxRXu2OwNWCeGikgsqnYXKBGi5uckUJA6VjiiiC0xv92cBG1+0GAYi
         MML3L8dNKJztscSJBav/Qqf7gYGVbKJ70zLHHBWNqV9JyXGV7LJMGNxTgE5PFjVDYBiD
         UWRQ==
X-Gm-Message-State: AOAM530VOBzjdJFcZ+ILBncfl3HqzuUyRNrY9A/zZZZgv/TDAbd+wq7j
        ilqF8YfSCr8OYkqiYakkcRMojmcMJofeFrPwwFh/2Q3ZPg3K+eQWPBldRyIcQZlxGoM1C8ZZLrP
        +yxTul5PhNLxLUtxiRmawzfx6ujFfRWWTTlhhKc5PiqiYMtRyaF2GRg0Pk6Jbhkg4x7rhBWXWvb
        QCfv8CY6lFELoaVk6kdm96K2pdkZrKoN4rlWY6jAg1LI5X7LhTF+Y2LQnLu2UMcEWtz1s0SlGV6
        WJ6TfFuiRnR4WW2vZhD/HmVAVCCRoi7l8P7OvnqY1AVDV+fgOuNVxhzeb2d
X-Received: by 2002:a05:6e02:19c5:: with SMTP id r5mr20478109ill.171.1614763720942;
        Wed, 03 Mar 2021 01:28:40 -0800 (PST)
X-Google-Smtp-Source: ABdhPJxozHsYXl1kJioiXXzATdgjpB3zLc0nm1jFzwLbdM35Yi7J+zcpLfsTQjgfC0K2ccimmS+cIWjX0Y1mRAGl0EA=
X-Received: by 2002:a05:6e02:19c5:: with SMTP id r5mr20478105ill.171.1614763720697;
 Wed, 03 Mar 2021 01:28:40 -0800 (PST)
MIME-Version: 1.0
From: =?UTF-8?B?5p2R5LiK5Y2a5ZOJ?= <hiroya.murakami@serverworks.co.jp>
Date: Wed, 3 Mar 2021 18:28:30 +0900
Message-ID: <CANTS3obDHP-0skVAaPHiVxjYxY=5VPadZ522F_uW_8Z-7wwzfw@mail.gmail.com>
Subject: =?UTF-8?B?c3ViamVjdC3ku7blkI0=?=
To: muser@stg-development.work
Content-Type: multipart/alternative; boundary="000000000000f803e605bc9e7907"

--000000000000f803e605bc9e7907
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: base64

aGVsbG8t44GT44KT44Gr44Gh44GvDQo=
--000000000000f803e605bc9e7907
Content-Type: text/html; charset="UTF-8"
Content-Transfer-Encoding: base64

PGRpdiBkaXI9Imx0ciI+aGVsbG8t44GT44KT44Gr44Gh44GvPC9kaXY+DQo=
--000000000000f803e605bc9e7907--
```



