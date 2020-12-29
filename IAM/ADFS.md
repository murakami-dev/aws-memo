# 文献
- 1[AWS環境上にADサーバを立てて、複数アカウントにシングル・サインオンする【ADFS】](https://qiita.com/Yuki_BB3/items/6d5db6486d915e8b5c5a)

# 文献１に剃って進めていく
- ADの構築
- ADFSの追加
- IISの追加

## `AWS OU`というOUを作成し、中に`AWS RO Group`というグループを作成する

## IISでSSL証明書
- >ADFSを用いたSSOの実装にはHTTPS通信を使います。よって、サーバにSSL証明書をインストールする必要があります。自身で認証局の証明書を持っている方はその証明書をインストールし、使用してください。ここでは、所謂オレオレ証明書（自己署名の証明書）を使用する手順を紹介します。
![image](https://user-images.githubusercontent.com/60077121/103264629-996c7c80-49ee-11eb-98f6-ca2a8a0a3b67.png)

## ADFSの設定
