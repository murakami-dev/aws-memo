# AWS Security Hubについてまとめる

## 参考文献集

- 【祝リリース】Security HubがGAになったので特徴と使い方まとめてみた
  - https://dev.classmethod.jp/articles/security-hub-ga/
- Security Hubの概要とユースケース
  - https://dev.classmethod.jp/articles/security-hub-usecase/
- Security HubとDeep Securityを連携してみた
  - https://dev.classmethod.jp/articles/security-hub-deep-security/


## Blackbelt
### 6つのポイント
- 1.デプロイのポイント
  - 全リージョンと全AWSアカウントに対し有効にする  
  - リソースの構成情報を評価するためにAWS Config が有効化され、記録開始していること
  - メンバーアカウントも監視すること
    - メンバーアカウントが多い場合はスクリプトもあるよ

- 2.セキュリティツールと統合する
  - AWS セキュリティサービス
    - GuardDuty
    - Inspector
    - Macie
    - IAM Access Analyzer 
    - Firewall Manager
    - Systems Manager Patch Manager
    - Config カスタムルール検出結果
  - パートナー製品との統合例
    - https://www.slideshare.net/AmazonWebServicesJapan/20201013-aws-black-belt-online-seminar-aws-security-hub#31
    - 例えばslack
      - https://aws.amazon.com/jp/blogs/security/enabling-aws-security-hub-integration-with-aws-chatbot/
  - Amazon Security Finding Format (ASFF)
    - Amazon Security Finding Format (ASFF) でに沿った Findings を生成する
    - Security Hub に Findings を 送信し、Security Hub の一元的な View で対応

- 3.セキュリティ基準の有効化
  - AWS Foundational Security Best Practices v1.0.0
    - AWSセキュリティ専門家により定義された統制項目。セキュリティベストプラクティスに沿わないAWSアカウントやリソースを検知する
  - CIS AWS Foundations Benchmark v1.2.0
    - Center for Internet Security が定義した要件の一部に対してチェックをする
  - PCI DSS v3.2.1
    - クレジットカード情報を保存・処理・転送する組織が従うセキュリティ標準であるPCI DSS要件の一部に対してチェックする
    
- 4.セキュリティ検出結果を取り扱う
  - CRITICAL 及び HIGH の検出結果から優先的に対応しよう
  - 対応するときは検出結果をクリックした後の画面の、改善アクションのガイダンスが提供
  
  - インサイト
    - インサイトは「グループ化条件(Group By)」 フィルターによって生成
    - 例えば、リソースタイプ（AWS リソース毎に検出結果を集約）、AWS アカウント ID（マルチアカウント環境においてAWSアカウント毎）
    
  - ワークフローステータス（Backlogのステータスみたいな）
  - 新規、通知しただけ、抑制（対応の必要なし）、対応済み
  
  - BatchUpdateFindings API
  - API使ってワークフローステータスを更新したり、Note（対応内容メモ）つけたりできる
  
- 5.対応を自動化する（カスタムアクション）
  - カスタムアクションの作成（CloudWatchイベントのような設定）
  - 作成済みのカスタムアクションは保存され、他の検出結果に対しても定義できる
  - カスタムアクションのサンプル例
    - https://www.slideshare.net/AmazonWebServicesJapan/20201013-aws-black-belt-online-seminar-aws-security-hub#53
    
- 6.コスト管理
- 設定→使用でコストの予測が可能。30日間の無料期間に見よう。
  - コスト最適化するならセキュリティチェック項目を選別すること。
  - 例：IAMに関する項目でIAMはグローバルリソースだけど、Configを全リージョンで有効にしていないので無効にする、とか。
  
## slackに通知するワークショップ
https://security-hub-workshop.awssecworkshops.com/
