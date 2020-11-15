# WAFについて
- 文献
  - [AWS WAFを完全に理解する WAFの基礎からv2の変更点まで](https://dev.classmethod.jp/articles/fully-understood-aws-waf-v2/)
  - [AWS再入門2019 AWS WAF編](https://dev.classmethod.jp/articles/aws-relearning2019-aws-waf/)
    - v1のWAFについての内容ですが概ね同じであるため問題ありません。
  - [「AWS上のセキュリティ対策をどういう順序でやっていけばいいか」という話をしました~Developers.IO 2019 Security登壇資料~](https://dev.classmethod.jp/articles/how-to-aws-security-with-3rd-party-solutions/)
  - [AWSセキュリティベストプラクティスを実践するに当たって適度に抜粋しながら解説・補足した内容を共有します](https://dev.classmethod.jp/articles/explanation-aws-security-best-practices/)
    - ホワイトペーパー「セキュリティベストプラクティス」まとめ
  - [WAFがなぜWebアプリケーションのセキュリティ対策に必要なのか？理由を徹底解説](https://www.shadan-kun.com/blog/measure/1395/)
  - [Web Application Firewall 読本](https://www.ipa.go.jp/security/vuln/waf.html)

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
  - v1ではルール10個までだったが、v2では1つのWebACLに1500Web ACL Capacity Unit (WCU)までルールを組み込むことが可能
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

## マネージドルール
**ブログ曰くCore Rule Set (CRS)とAmazon IP Reputationで良さそう**
- Baseline Rule Groups: 一般的な脅威からの一般的な保護
  - Core Rule Set (CRS): 一般的なルール。OWASPやCVEに記載されているもの等が含まれている。
  - Admin Protection: 公開されている管理ページの保護。
  - Known Bad Inputs: 脆弱性を発見/悪用するリクエストの検知。
- Use-Case Specific Rule Groups: 用途別の保護
  - SQL Database: SQLインジェクション等のSQLデータベースに対する攻撃からの保護
  - LINUX operating system: LFIなどLinuxOSへの攻撃の保護。POSIXOSルールと合わせて利用。
  - POSIX Operating System: Linux、FreeBSD等POSIX OSの保護。
  - Windows Operating System: Windows OSの保護。
  - PHP Application: PHPアプリの保護。
  - WordPress Application: WordPress固有の脆弱性に対する攻撃の保護。
- IP Reputation Rule Groups: IP評価ルール。
  - Amazon IP Reputation: AWSの持つ悪意あるボット等のIPアドレスからの保護。

## どう使うか
```
導入してしばらくはCountモードとして動作させ、誤検知がないか等をログで確認し、問題ないと判断したらBlockにするといいです。
お手軽なのはCSCのOWASP Top 10対応ルールが全部盛りなので、これを利用することを推奨します。
CSCルールの場合には、同じくCSC社提供のWafCharmを併用するとセキュリティが強化されつつ環境に合わせた調整が簡単に行なえます。
```
※CSCルールのブログは`AWS WAFを完全に理解する WAFの基礎からv2の変更点まで`のブログの末尾にリンクあり
