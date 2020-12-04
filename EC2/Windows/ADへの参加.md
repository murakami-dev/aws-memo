# 文献
- [【図解/AD】ドメイン参加の仕組みと手順,メリット,参加できない時の注意点～](https://milestone-of-se.nesuke.com/sv-advanced/activedirectory/join-to-domain/)
  - クライアントからの参加手順
  - DNSサーバ変更→システムからADへ参加
  - >あらかじめ配られたドメインユーザの ID / パスワードを使ってログインします。ドメインの指定方法はユーザ名の前に『ドメイン名\』をつけるか、ユーザ名の後に『@ドメイン名』をつけます。
- [EC2インスタンス上にActive Directory環境を構築する際のTIPS](https://dev.classmethod.jp/articles/tips-for-build-active-directory-on-ec2/)
  - ドメインコントローラーのフォワーダーにはAmazonProvidedDNSを設定すること
  
# ADに参加しているか確認する方法
- クライアントで以下のコマンド
```
PS C:\Users\hiroya> whoami /fqdn
CN=hiroya,CN=Users,DC=murakami,DC=com
```

# RunCommandで参加させる方法もある
- [AWS Systems Manager を使用して、実行中の EC2 Windows インスタンスを、AWS Directory Service のドメインに参加させる方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-systems-manager-dx-domain/)
  - IAMロールに注意が必要。`AmazonSSMDirectoryServiceAccess`がいる
  
