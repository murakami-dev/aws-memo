# やりたいこと
- EC2をオリジンに設定する
- ゆくゆくはS3もオリジンにしてみる
- ログをとる
  - CfoudFrontのログ
  - Apacheログをfluentd使って収集
- ES使って可視化したい
- CWlogsをS3にエクスポートしたい。定期に出来るのか？
- 一部のパスに署名付きクッキーで閲覧制限（別ページで）

# 構成
![image](https://user-images.githubusercontent.com/60077121/108626120-e2dac480-7491-11eb-80b0-cdf35032caca.png)


# 文献
- [料金](https://aws.amazon.com/jp/cloudfront/pricing/)
- [ディストリビューションを作成または更新する場合に指定する値](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html)
- [CloudFrontで特定のパスへのアクセスをリダイレクトさせる](https://dev.classmethod.jp/articles/cloudfront-redirect/)
- [CloudFront の Cache Policy と Origin Request Policy を理解する](https://qiita.com/t-kigi/items/6d7cfccdb629690b8d29)
- [ポリシーの使用](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/working-with-policies.html)
# 手順
- EC2作成
- CloudFront設定
  - CNAMEに独自ドメインを設定
  - **独自ドメイン使うので証明書はCloudFront.netは使えない。ACMのもの使う。しかもバージニア** 
    - [Amazon CloudFront の代替ドメイン名(CNAMEs)設定に SSL 証明書が必須になりました](https://dev.classmethod.jp/articles/201904_enhancing-domain-security-on-cloudfront/)
- Route53設定

## EC2、Route53
- 自分のドメイン→EC2はできている

## バージニア北部のACMで証明書リクエスト
- [代替ドメイン名 (CNAME) を追加してカスタム URL を使用する](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/CNAMEs.html)
- CFではバージニア北部の証明書しかカスタム証明書適用できない
- Route53でCFのエイリアスレコード登録したければ、CFのCNAMEにレコード名（独自ドメイン名）が設定されていることが必要
  - [Route 53 エイリアスリソースレコードセットを作成するときに、優先エイリアスターゲットを選択できないのはなぜですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/route-53-no-targets/)
- CFのCNAMEに独自ドメイン設定するためにはカスタム証明書を設定することが必須
  - >代替ドメイン名（CNAME）をCloudFrontディストリビューションに追加するには、ドメイン名を使用するための承認を検証する信頼できる証明書を添付する必要があります。
- **つまりバージニア北部でのACM登録は必須**

## Route53でCFのエイリアスを設定
## CFの設定
#### 重要：今回は証明書を適用していないEC2をオリジンにしたので`Origin Protocol Policy`は`HTTP Only`にすること
### その他
- 独自ドメインでアクセスしたい場合、CNAMEには独自ドメインを設定すること。その場合、証明書はCFのは使えない。ACM使うこと。
- 証明書はACMの証明書を使用すること
  - **証明書は『米国東部（バージニア北部）リージョン』のACMに格納されている必要があります** 
#### ログ記録（スタンダードログ）はON
- スタンダードログとリアルタイムログがある
- [標準ログ (アクセスログ) の設定および使用](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/AccessLogs.html)
- [リアルタイムログ出力をサポートしたCloudFrontを試してみた](https://dev.classmethod.jp/articles/cloudfront-realtimelogs/)
  
  
  
  
  
