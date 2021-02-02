# 文献
- まとめ
  - [AWS再入門ブログリレー Amazon CloudFront編](https://dev.classmethod.jp/articles/re-introduction-2020-cloudfront/#toc-1)
  - [AWS再入門ブログリレー Amazon CloudFront 編](https://dev.classmethod.jp/articles/blogrelay2019_cloudfront/)
    - こっちの方が古いけど内容充実
- [マルチオリジン](#マルチオリジン)
  - [CloudFrontでマルチオリジンとCache Behavior設定してみた](https://dev.classmethod.jp/articles/cloudfront-multioriginbehavior/)
  - [CloudFrontで特定のパスへのアクセスをリダイレクトさせる](https://dev.classmethod.jp/articles/cloudfront-redirect/)


# 総論
- オリジンはEC2,ALB,S3などAWSのサービス以外に、WEBサーバもオリジンとして指定することができます。
- セキュリティ対策にACM,AWS Shield（デフォルトで使用可能）、AWS WAFが使用可能。

# マルチオリジン
- ブログ（CloudFrontでマルチオリジンとCache Behavior設定してみた）より
>はじめは1つのオリジンしか登録できませんので、S3-Originを登録します。 
また、Cache BehaviorもPath Patternが Default(*) の1つだけしか登録できないので、このまま先に進みます。
- 同一オリジンで異なるパスパターンで振り分け先を変更する設定も可能

- 手順
  - ディストリビューション作成後、2つめのオリジンを登録
    - Origin Pathにはオリジン内のディレクトリを登録する
    - >`/production`と入力した場合、ユーザーがブラウザに「example.com/index.html」と入力すると、CloudFront は myawsbucket/production/index.html に対するリクエストを Amazon S3 に送信します。
    - https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValuesOriginPath
  - Behaviorからパスパターンを設定
    - https://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValuesPathPattern
- ![image](https://user-images.githubusercontent.com/60077121/100808902-629b5900-3478-11eb-99b8-a648c0d34232.png)


# キャッシュコントロール

## キャッシュファイルの無効化(Invalidation)


# 料金
- [Amazon CloudFront の料金](https://aws.amazon.com/jp/cloudfront/pricing/)
- 料金は以下のオンデマンドの欄にある料金体系ごとに課金される
- エッジロケーションを絞ることで、料金クラスを抑えることができる
![image](https://user-images.githubusercontent.com/60077121/99171321-56f72500-274b-11eb-8a69-8bb6ff3c007d.png)
