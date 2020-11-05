
## EC2にAD入れてAD Connector用のユーザ作成
- サーバマネージャからADをインストール
- 再起動
- しばらく待ってから再度RDP
- ユーザ作成（Domain AdminsとAD Connector用のユーザ）

- 以下の「権限をサービスアカウントに委任する」を参照)
  - https://docs.aws.amazon.com/ja_jp/directoryservice/latest/admin-guide/prereq_connector.html
![image](https://user-images.githubusercontent.com/60077121/98249118-3b588580-1fb9-11eb-8c5b-30419ef01cfd.png)

## AD Connector作成

## 証明書をACMへアップロード

