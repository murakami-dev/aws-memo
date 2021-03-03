# 目的
- メール送信の一般的事項まとめ
- EC2メール送信の概要

# メール送信の仕組み
- 送信側アウトバウンドで25ブロック
- **中継**・受信側で審査
- ![image](https://user-images.githubusercontent.com/60077121/109823184-0c0e0880-7c7b-11eb-922b-a946c2898a95.png)
- https://blastmail.jp/blog/mail-delivery/send-and-receive-mail

## DNSの動き
- https://ms.repica.jp/column/026/

### 送信者側
- sample@arara.comにメール送りたい
- arara.comのメールサーバは？と問合せ→mail.arara.comだよ（MXレコード）
- mail.arara.comのIPは？→xx.xx.x.xだよ（Aレコード）
- つまりメール受信側でMXレコードとAレコードの設定は必須
- ![image](https://user-images.githubusercontent.com/60077121/109824071-d3bafa00-7c7b-11eb-8191-95f5083642c2.png)

### TXTレコード
- >TXT RECORDは、コメント欄です。ゾーンファイルのリソースコードを見たDNSは、TXT RECORDに特別なコメントが書かれていたら、「このドメインに紐づいているのはこのIPアドレスだよー」という情報だけを返答するのではなく、「こんなことも書かれていたよー」と、コメントをつけます。
- >迷惑メール対策として重視されている、送信ドメイン認証技術である「SPF」や「DKIM」は、このTXT RECORDに文字列を記載します。これで、「このメールの配信元はなりすましじゃないよー」と証明するわけですね！

#### SPFレコード -Sender Policy Framework
- https://ms.repica.jp/column/06/
- >SPFは、メール送信元IPアドレスを送信側のDNSサーバへあらかじめ登録することで、メールを受信した受信側メールサーバが怪しいメールでないかを送信側のDNSサーバに確認する仕組みです。
- ![image](https://user-images.githubusercontent.com/60077121/109824839-8c813900-7c7c-11eb-9ca9-ebc3345aecdf.png)

#### DKIM -DomainKeys Identified Mail
- 送信メールサーバーがEメールに署名を付加しDNSと組み合わせて正当性を検証する仕組みです。受信メールサーバーはメールの署名を基にDNSサーバーへ問合せを行い、メールの正当性を検証します。


# Amazon EC2 Eメール送信ベストプラクティス
- 1.[Amazon EC2 Eメール送信ベストプラクティス](https://dev.classmethod.jp/articles/ec2-send-email-best-practice/)
- 2.[AWSからのメール送信](https://www.slideshare.net/AmazonWebServicesJapan/aws-30934799)
  - ![image](https://user-images.githubusercontent.com/60077121/109827921-82ad0500-7c7f-11eb-97a0-f31225a9dd5a.png)
- 3-1.[EC2からメールを送信したいです。何かAWSに申請する必要はありますか？](https://support.serverworks.co.jp/hc/ja/articles/115008163087)
  - 25アウトバウンドの制限解除
  - 逆引き申請も行う。レシーバーはIPアドレス（アクセスログでIP分かる）でdigしてドメインが引けるかチェックする。
  - 上記を行うために送信者側DNSでmail.example.comの正引きも登録すること。 **NATGW経由でメールを送信するEC2の場合、NATGWのEIPを正引き**
- 3-2.[EC2からメール送信するためにメール送信制限解除申請と逆引き設定申請を行いたいです。どうすればいいでしょうか？](https://support.serverworks.co.jp/hc/ja/articles/115008315608)
  - `xx.xx.xx.xx : mail.example.com`と記載例にあるのでこれをRoute53でやっておけば、AWS側でどこぞのDNSに`xx.xx.xx.xx -> mail.example.com`の逆引き登録してくれるのだろう
- 3.3[ELB配下のEC2からメール送信する場合のEC2のDNS登録はどのようにしたらいいですか？](https://support.serverworks.co.jp/hc/ja/articles/115006282408)
  - >EC2からメール送信を行う場合は、EC2もELBとは別にサブドメインを設け、Aレコードに登録をお願いいたします。送信元MTA（EC2）のDNS登録は、smtp、mail、sendのようなメール送信をしていることがわかるサブドメインでの登録を推奨いたします。
  - EC2にサブドメインのDNSつけて、AレコードでNATGWのEIP登録する



