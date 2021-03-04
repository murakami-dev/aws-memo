# 目的
- 01で設定したSMTPサーバからメールを送ったが届かず。dkimとSPFをやってみる

# dkim
## OPENdkimをインストール
### EPELインストール
- 割愛

### dkimインストール
```
 sudo yum install opendkim -y
```
- /etc/opendkimにkeysディレクトリ出来るのでその中にドメイン名のディレクトリ作成し、その中に鍵を作成したい
```
[ec2-user@ip-10-123-10-6 opendkim]$ ls
keys  KeyTable  SigningTable  TrustedHosts
```
### Perl Getopt::Long モジュールのインストール
- opendkimでキーペア（秘密鍵／公開鍵）を作成する際にPerl Getopt::Long モジュールが必要になります。  
  - https://rin-ka.net/centos-postfix-dkim/#toc5
- 普通にモジュールなしでやろうとしたらエラーになる
```
[root@ip-10-123-10-6 keys]# opendkim-genkey -D /etc/opendkim/keys/stg-development.work/ -d stg-development.work -s default --verbose
Can't locate Getopt/Long.pm in @INC (you may need to install the Getopt::Long module) (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5) at /usr/sbin/opendkim-genkey line 25.
BEGIN failed--compilation aborted at /usr/sbin/opendkim-genkey line 25.
```

- インストール
```
[root@ip-10-123-10-6 keys]# yum -y install perl-Getopt-Long.noarch
```

### key作成
```
[root@ip-10-123-10-6 keys]# opendkim-genkey -D /etc/opendkim/keys/stg-development.work/ -d stg-development.work -s default --verbose
opendkim-genkey: generating private key
opendkim-genkey: private key written to default.private
opendkim-genkey: extracting public key
opendkim-genkey: DNS TXT record written to default.txt
```

### dkimレコード確認
```
[root@ip-10-123-10-6 stg-development.work]# ls
default.private  default.txt
[root@ip-10-123-10-6 stg-development.work]# cat default.txt
default._domainkey      IN      TXT     ( "v=DKIM1; k=rsa; "
          "p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDeJA2Bu9HQMX4yS/N2DXffzUdnlPWCxvmsv6YYdukfaSVmiNycXygZ3KSA0/xGnz+2GqOyEbC4VLg4MWO6c14PBZy4T95YaUBMUbww2JU4LwhcHGgDOacEJVwze2brmCR9uRlHXug0v8xbhHwpNaWFDNtt/f3ZxuZf9v9pJjMFNQIDAQAB" )  ; ----- DKIM key default for stg-development.work
```

### Route53登録
- これでうまくいった
- ![image](https://user-images.githubusercontent.com/60077121/109927656-e1b65c80-7d07-11eb-9505-cb689152e60e.png)






