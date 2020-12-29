# 文献
- 1[AWS環境上にADサーバを立てて、複数アカウントにシングル・サインオンする【ADFS】](https://qiita.com/Yuki_BB3/items/6d5db6486d915e8b5c5a)
- 2.[ADFS利用による複数AWSアカウントへのSSO](https://dev.classmethod.jp/articles/adfs-sso-multi-aws-account/)
  - 2-1.[Active Directory資産を活用したAWS Management ConsoleへのSSO](https://dev.classmethod.jp/articles/adfs-aws-sso/)
    - このブログもADFS構築の手順ブログ
- 3.[AWS Managed Microsoft AD環境でADFSサーバーを構築する](https://dev.classmethod.jp/articles/setup-adfs-server-on-aws-microsoft-ad/)
  - 3-1.[AWS Managed Microsoft ADでグループ管理サービスアカウント(gMSA)を作成する](https://dev.classmethod.jp/articles/how-to-create-gmsa-on-microsoft-ad/)
    - グループ管理サービスアカウント（gMSA）って何？が解説

# 文献１に剃って進めていく
- ADの構築
- ADFSの追加
- IISの追加

## `AWS OU`というOUを作成し、中に`AWS RO Group`というグループを作成する

## IISでSSL証明書
- >ADFSを用いたSSOの実装にはHTTPS通信を使います。よって、サーバにSSL証明書をインストールする必要があります。自身で認証局の証明書を持っている方はその証明書をインストールし、使用してください。ここでは、所謂オレオレ証明書（自己署名の証明書）を使用する手順を紹介します。
![image](https://user-images.githubusercontent.com/60077121/103264629-996c7c80-49ee-11eb-98f6-ca2a8a0a3b67.png)

## ADFSの設定
- >サービスアカウントの指定 ADFSがADとやり取りするサービスアカウントを指定します。 今回はAdministratorアカウントを利用していますが、本番環境では最小限の権限に絞ることが望ましいです。
  - 文献2-1より

### KDSキーの作成
```
PS C:\Windows\system32> Add-KdsRootKey -EffectiveTime ((Get-Date).addhours(-10))

Guid
----
d707c5d4-xxxx-xxxx-xxxx-xxxxxxxxxxxx


PS C:\Windows\system32> New-ADServiceAccount -Name adfsadmin -DNSHostName EC2AMAZ-MYAD
PS C:\Windows\system32>
```
![image](https://user-images.githubusercontent.com/60077121/103267540-0d118800-49f5-11eb-85ae-1becbbe9b98b.png)

## メタデータの取得
- 文献2-1のようにやる
- EC2のブラウザからアクセスし取得。`https://＜ADFSのURL＞/FederationMetadata/2007-06/FederationMetadata.xml`
![image](https://user-images.githubusercontent.com/60077121/103267681-67124d80-49f5-11eb-968d-2e8b231aaa83.png)

### ハマった
- メタデータを解析できません(`Could not parse metadata`)というエラーが出た。
- IEの互換表示設定をしてから保存すると解決した
![image](https://user-images.githubusercontent.com/60077121/103295144-b2008500-4a36-11eb-9474-a189f8afff7f.png)

#### 以下は意味なかった（参考）
- 文字コードの問題
  - [エラー: メタデータを解析できませんでした。](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/troubleshoot_saml.html#troubleshoot_saml_issuer-metadata)
- `file`コマンドで(with BOM)と出たのでこれを削除
- `nkf`コマンドは`brew install nkf`でインストール
```
MacBook-Pro:AWSアイコン murakamihiroya$ file FederationMetadata.xml 
FederationMetadata.xml: news or mail text, UTF-8 Unicode (with BOM) text, with CRLF line terminators
MacBook-Pro:AWSアイコン murakamihiroya$ nkf --overwrite --oc=UTF-8 FederationMetadata.xml 
MacBook-Pro:AWSアイコン murakamihiroya$ file FederationMetadata.xml 
FederationMetadata.xml: news or mail text, ASCII text, with CRLF line terminators
```
## ADFS管理
![image](https://user-images.githubusercontent.com/60077121/103297083-e5dda980-4a3a-11eb-9cba-0866458336de.png)

## 規則の追加
- ここからは文献2-1に沿って設定

