# WAFについて
- 文献
#### [AWS WAFを完全に理解する WAFの基礎からv2の変更点まで](https://dev.classmethod.jp/articles/fully-understood-aws-waf-v2/)
#### [AWS再入門2019 AWS WAF編](https://dev.classmethod.jp/articles/aws-relearning2019-aws-waf/)
    - v1のWAFについての内容ですが概ね同じであるため問題ありません。
#### [「AWS上のセキュリティ対策をどういう順序でやっていけばいいか」という話をしました~Developers.IO 2019 Security登壇資料~](https://dev.classmethod.jp/articles/how-to-aws-security-with-3rd-party-solutions/)
#### [AWSセキュリティベストプラクティスを実践するに当たって適度に抜粋しながら解説・補足した内容を共有します](https://dev.classmethod.jp/articles/explanation-aws-security-best-practices/)
#### [WAFがなぜWebアプリケーションのセキュリティ対策に必要なのか？理由を徹底解説](https://www.shadan-kun.com/blog/measure/1395/)
#### [Web Application Firewall 読本](https://www.ipa.go.jp/security/vuln/waf.html)

## 概要
- 純粋なファイアウォールと重複した役割(L3/L4の保護機能)を持っているわけではない
- L7に対してのみ動作します。
- IDS/IPSとも役割が異なります。

## 一般用語としてのWAF
```
ファイアウォールやIDS/IPSは、主にネットワーク層やオペレーションシステム層などを監視し、不正な通信を検知します。
つまり、ネットワークやオペレーションシステムとは別の層にある、Webアプリケーションに対してのサイバー攻撃を防御することはできないのです。
```
![image](https://user-images.githubusercontent.com/60077121/99158708-1d0d2b00-2719-11eb-8332-8e0382a788dc.png)

- EC2で動作するものもマーケットプレイスで購入できる
  - F5
  - Imperva
  - Barracuda

## 用語
- WebACL
  - 1つのWAFの設定の塊
  - CloudFront / ALB / API Gatewayに割り当てる単位
  - 1つのリソースに対して1つのWebACLのみ割り当て可能
  - 複数のリソースに対して同じWebACLを割り当て可能
  - 複数のルール/ルールグループを内包する
- ルール
  - リクエストに対する条件のまとまり
  - 例えば指定したIPにマッチするか、特定のクエリストリングがあるか等
  - ルールグループ
  - ルールをまとめたもの
  - 事前に定義可能
- アクション
  - リクエストをルールに照らし合わせた結果をどう扱うかの設定
  - ALLOW / BLOCK / COUNTがある
  - ルール毎にアクションがあり、すべてのルールに当たらなければWebACLに設定されたDefault Actionに沿って処理される

[テスト](#### [Web Application Firewall 読本](https://www.ipa.go.jp/security/vuln/waf.html))
