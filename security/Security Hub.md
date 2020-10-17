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
