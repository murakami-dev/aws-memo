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
![image](https://user-images.githubusercontent.com/60077121/108596589-5533a200-73c9-11eb-82ec-2011ba9a84f0.png)

# 文献
- [料金](https://aws.amazon.com/jp/cloudfront/pricing/)
- [ディストリビューションを作成または更新する場合に指定する値](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html)
- [CloudFrontで特定のパスへのアクセスをリダイレクトさせる](https://dev.classmethod.jp/articles/cloudfront-redirect/)
- [CloudFront の Cache Policy と Origin Request Policy を理解する](https://qiita.com/t-kigi/items/6d7cfccdb629690b8d29)
- [ポリシーの使用](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/working-with-policies.html)
# 手順

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

## CloudFront設定
- 以下は詳細を勉強したい
- [ディストリビューションを作成または更新する場合に指定する値](https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html)
### 各項目
#### Origin Domain Name
- パブリックDNS名を入れる
#### Origin Path
- >オプション。 CloudFrontがAmazonS3バケット内のディレクトリまたはカスタムオリジンからコンテンツをリクエストするようにする場合は、ここに/で始まるディレクトリ名を入力します。 CloudFrontは、リクエストをオリジンに転送するときに、オリジンドメイン名の値にディレクトリ名を追加します（例：myawsbucket / production）。ディレクトリ名の末尾に/を含めないでください。
#### Enable Origin Shield
- [Amazon CloudFrontにオリジンへのリクエストを軽減するキャッシュレイヤー(Origin Shield)が追加されました](https://dev.classmethod.jp/articles/amazon-cloudfront-support-cache-layer-origin-shield/)
- >Origin Shield を利用すると、オリジンとの通信は Origin Shield に集約されます。 オリジンへのリクエストが発生するのは、Origin Shield にキャッシュがない場合だけです。
- >各リージョナルエッジキャッシュが個別にオリジンにリクエストしなくなるため、オリジンの負荷が軽減
- >エッジローケーション 〜 リージョナルエッジキャッシュ 〜 Origin Shield(オリジンのそば) 間は CloudFront のグローバルネットワークを利用するためネットワークパフォーマンスが向上
- ![image](https://user-images.githubusercontent.com/60077121/108588508-f8ba8d80-739c-11eb-9893-8e146cece093.png)

#### Origin ID

#### Origin Custom Headers

### Default Cache Behavior Settings
#### Cache Policy

#### Origin Request Policy



- 独自ドメインでアクセスしたい場合、CNAMEには独自ドメインを設定すること。その場合、証明書はCFのは使えない。ACM使うこと。
- 証明書はACMの証明書を使用すること
  - **証明書は『米国東部（バージニア北部）リージョン』のACMに格納されている必要があります**
- 


  
  
  
  
  
