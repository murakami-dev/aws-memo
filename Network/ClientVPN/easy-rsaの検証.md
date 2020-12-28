# 文献
- [AWS Client VPNクライアント証明書の管理手順を整理してみた](https://dev.classmethod.jp/articles/manage-aws-client-vpn-client-certificates/#toc-16)
https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/client-authentication.html#mutual

# 証明書のパス
任意。
- OPENVPNの場合：C:\Users\user\OpenVPN\config
- クライアントアプリの場合：C:\Users\user\AppData\Roaming\AWSVPNClient\OpenVpnConfigs

# ひとつのCAで複数の証明書を作成する
- ドキュメントにあるサーバ証明書を作成するコマンド`./easyrsa build-server-full server nopass`の`server`部分がサーバ証明書のドメイン。
- 以下のようにドメイン部分を変更すれば証明書は何個でも作れる
- `/pki/issued`配下にサーバ・クライアント証明書が配置される
- `/pki/private`配下にサーバ・クライアント秘密鍵が配置される
```
[ec2-user@ip-10-123-10-212 easyrsa3]$ ls
easyrsa  openssl-easyrsa.cnf  pki  vars.example  x509-types
[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-server-full domain nopass  #ドメイン部分を変えれば何個でも作れる
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating a 2048 bit RSA private key
...........+++
....+++
writing new private key to '/home/ec2-user/easy-rsa/easyrsa3/pki/easy-rsa-13373.46z01K/tmp.D88wyx'
-----
Using configuration from /home/ec2-user/easy-rsa/easyrsa3/pki/easy-rsa-13373.46z01K/tmp.LlizVJ
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'domain'
Certificate is to be certified until Jan 30 11:25:03 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

[ec2-user@ip-10-123-10-212 easyrsa3]$ cd pki
[ec2-user@ip-10-123-10-212 pki]$ ls
ca.crt           index.txt       index.txt.attr.old  issued               private  reqs     safessl-easyrsa.cnf  serial.old
certs_by_serial  index.txt.attr  index.txt.old       openssl-easyrsa.cnf  renewed  revoked  serial
[ec2-user@ip-10-123-10-212 pki]$ cd issued
[ec2-user@ip-10-123-10-212 issued]$ ls -la
total 24
drwx------ 2 ec2-user ec2-user   72 Oct 27 11:25 .
drwx------ 8 ec2-user ec2-user  286 Oct 27 11:25 ..
-rw------- 1 ec2-user ec2-user 4460 Sep 30 13:02 client1.domain.tld.crt
-rw------- 1 ec2-user ec2-user 4552 Oct 27 11:25 domain.crt
-rw------- 1 ec2-user ec2-user 4552 Sep 30 13:02 server.crt
```

# 証明書の運用
- `easy-rsa`以下を保存しておけば、後に別のサーバでも同じCAが使えるし、証明書も`easy-rsa`配下に置かれている。
- ローカルにもってくるときは以下の手順
  - カレントディレクトリのすべてのファイルをサブディレクトリも含めてeasy-rsa.zipとして保存。
  - >サブディレクトリも含めて圧縮したい場合は、「-r」オプションを指定します。例えば、カレントディレクトリのファイルをサブディレクトリ下のファイルも含めて圧縮したい場合は、「zip -r ZIPファイル名 *」のように指定します
  - https://www.atmarkit.co.jp/ait/articles/1607/25/news021.html
```
[ec2-user@radius easy-rsa]$ zip -r easy-rsa *
```
# 証明書の運用（自分の検証環境）
- まずは証明書つくってACMにあげた
- 以下のとおり、証明書の更新、期限管理を試してみる
```
[ec2-user@ip-10-123-10-212 ~]$ git clone https://github.com/OpenVPN/easy-rsa.git
Cloning into 'easy-rsa'...
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 2082 (delta 0), reused 2 (delta 0), pack-reused 2077
Receiving objects: 100% (2082/2082), 11.72 MiB | 7.15 MiB/s, done.
Resolving deltas: 100% (912/912), done.
[ec2-user@ip-10-123-10-212 ~]$ cd easy-rsa/easyrsa3
[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/ec2-user/easy-rsa/easyrsa3/pki


[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-ca nopass
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating RSA private key, 2048 bit long modulus
................+++
...+++
e is 65537 (0x10001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:my-own-clientvpn

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/ec2-user/easy-rsa/easyrsa3/pki/ca.crt


[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-server-full murakami nopass
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating a 2048 bit RSA private key
..+++
.....+++
writing new private key to '/home/ec2-user/easy-rsa/easyrsa3/pki/easy-rsa-11113.PRhfuU/tmp.fqgKnm'
-----
Using configuration from /home/ec2-user/easy-rsa/easyrsa3/pki/easy-rsa-11113.PRhfuU/tmp.82CRSU
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'murakami'
Certificate is to be certified until Mar  5 14:41:59 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-client-full client-murakami nopass
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating a 2048 bit RSA private key
...................................................................................................................................+++
..+++
writing new private key to '/home/ec2-user/easy-rsa/easyrsa3/pki/easy-rsa-11246.fjoL4h/tmp.agwVAK'
-----
Using configuration from /home/ec2-user/easy-rsa/easyrsa3/pki/easy-rsa-11246.fjoL4h/tmp.53ThIR
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client-murakami'
Certificate is to be certified until Mar  5 14:42:50 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

[ec2-user@ip-10-123-10-212 easyrsa3]$ mkdir ~/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/ca.crt ~/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/issued/murakami.crt ~/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/private/murakami.key ~/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/issued/client-murakami.crt ~/custom_folder
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/private/client-murakami.key ~/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cd ~/custom_folder/
[ec2-user@ip-10-123-10-212 custom_folder]$ ls
ca.crt  client-murakami.crt  client-murakami.key  murakami.crt  murakami.key
[ec2-user@ip-10-123-10-212 custom_folder]$ aws acm import-certificate --certificate fileb://murakami.crt --private-key fileb://murakami.key --certificate-chain fileb://ca.crt --region ap-northeast-1
{
    "CertificateArn": "arn:aws:acm:ap-northeast-1:913926916566:certificate/34137012-b29b-4695-9c6d-db2b9c546d76"
}
[ec2-user@ip-10-123-10-212 custom_folder]$ aws acm import-certificate --certificate fileb://client-murakami.crt --private-key fileb://client-murakami.key --certificate-chain fileb://ca.crt --region ap-northeast-1
{
    "CertificateArn": "arn:aws:acm:ap-northeast-1:913926916566:certificate/0779a1a2-33f0-4c23-8f7b-2e12d3881349"
}
```
## 期限の更新

## 証明書の期限管理

## 新しいクライアント証明書の発行
- 認証局が同じの場合、ACMへのアップロードは不要。
  - ClientVPN構築時にクライアント証明書のARNを指定するが、CAが同じならそれ以外のクライアント証明書でも接続できる


## （微妙）一度認証局（CA）をたてたあと、同じサーバで別のCAをたてて証明書を作成することもできる

- 当初、easy-rsaをgit cloneしてきたら`home/ec2-user/easy-rsa/easyrsa3`が作成される
- 2つめのCAをつくる場合、ホームフォルダに適当なディレクトリ作成してやるのが早そう。
```
[ec2-user@ip-10-123-10-212 ~]$ mkdir test
[ec2-user@ip-10-123-10-212 ~]$ ls
custom_folder  easy-rsa  test
[ec2-user@ip-10-123-10-212 ~]$ cd test
[ec2-user@ip-10-123-10-212 test]$ git clone https://github.com/OpenVPN/easy-rsa.git
Cloning into 'easy-rsa'...
remote: Enumerating objects: 2077, done.
remote: Total 2077 (delta 0), reused 0 (delta 0), pack-reused 2077
Receiving objects: 100% (2077/2077), 11.69 MiB | 7.17 MiB/s, done.
Resolving deltas: 100% (910/910), done.
[ec2-user@ip-10-123-10-212 test]$ ls
easy-rsa
[ec2-user@ip-10-123-10-212 test]$ cd easy-rsa/
[ec2-user@ip-10-123-10-212 easy-rsa]$ ls
build      COPYING.md  doc       KNOWN_ISSUES  op_test.orig  README.md             release-keys  wop_test.sh
ChangeLog  distro      easyrsa3  Licensing     op_test.sh    README.quickstart.md  wop_test.bat
[ec2-user@ip-10-123-10-212 easy-rsa]$ cd easy-rsa/easyrsa3
-bash: cd: easy-rsa/easyrsa3: No such file or directory
[ec2-user@ip-10-123-10-212 easy-rsa]$ cd ..
[ec2-user@ip-10-123-10-212 test]$ cd easy-rsa/easyrsa3
[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/ec2-user/test/easy-rsa/easyrsa3/pki


[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-ca nopass
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating RSA private key, 2048 bit long modulus
...+++
......+++
e is 65537 (0x10001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:test

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/ec2-user/test/easy-rsa/easyrsa3/pki/ca.crt


[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-server-full server nopass
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating a 2048 bit RSA private key
...................+++
............................................................+++
writing new private key to '/home/ec2-user/test/easy-rsa/easyrsa3/pki/easy-rsa-13041.yKOXCU/tmp.GHPt43'
-----
Using configuration from /home/ec2-user/test/easy-rsa/easyrsa3/pki/easy-rsa-13041.yKOXCU/tmp.pOgdPU
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Jan 30 10:57:12 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

[ec2-user@ip-10-123-10-212 easyrsa3]$ ./easyrsa build-client-full client1.domain.tld nopass
Using SSL: openssl OpenSSL 1.0.2k-fips  26 Jan 2017
Generating a 2048 bit RSA private key
.......................+++
......+++
writing new private key to '/home/ec2-user/test/easy-rsa/easyrsa3/pki/easy-rsa-13128.6Quoxj/tmp.trTZgM'
-----
Using configuration from /home/ec2-user/test/easy-rsa/easyrsa3/pki/easy-rsa-13128.6Quoxj/tmp.efPKRk
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'client1.domain.tld'
Certificate is to be certified until Jan 30 10:57:33 2023 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

[ec2-user@ip-10-123-10-212 easyrsa3]$ mkdir ~/test/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/ca.crt ~/test/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/issued/server.crt ~/test/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/private/server.key ~/test/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/issued/client1.domain.tld.crt ~/test/custom_folder
[ec2-user@ip-10-123-10-212 easyrsa3]$ cp pki/private/client1.domain.tld.key ~/test/custom_folder/
[ec2-user@ip-10-123-10-212 easyrsa3]$ cd ~/test/custom_folder/
[ec2-user@ip-10-123-10-212 custom_folder]$ ls
ca.crt  client1.domain.tld.crt  client1.domain.tld.key  server.crt  server.key
```


