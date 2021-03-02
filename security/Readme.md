- セキュリティの全体把握に有益な文書・セッションまとめる
- 概要はここに書くが、混み入った話は各ページに作成すること

# AWSにおけるクラウドコンプライアンスの実践 ―セキュリティ担当者が理解すべき説明責任のポイント―
- https://pages.awscloud.com/rs/112-TZM-766/images/D2-06.pdf
- AWS Summit2019
- NIST Cybersecurity Frameworkの話
  - IPAで翻訳版を入手可能
  - >世界各地の政府、産業界、組織において、NIST サイバーセキュリティフレームワーク(CSF) がサイバーセキュリティの推奨ベースラインであり、システムのサイバーセキュリティリスク管理およびレジリエンスの改善に有効であるという認識が徐々に高まっています。
- NIST Cybersecurity FrameworkホワイトペーパーはCSFの各コア機能に対してAWSのソリューションを紹介する資料
  - https://d1.awsstatic.com/whitepapers/ja_JP/compliance/NIST_Cybersecurity_Framework_CSF.pdf 
    - CSFのコア機能：各コア機能にはサブカテゴリがある
    - 識別
    - 防御
    - 検知
    - 対応
    - 復旧
- 政府機関向け「アマゾン ウェブ サービス（AWS）」対応セキュリティリファレンスの話
  - H30に政府が出した`政府機関等の情報セキュリティ対策のための統一基準群`にこたえる形でAPNパートナーとAWSJが出した文書。NISTにどう対応しているか。
  - https://aws.amazon.com/jp/compliance/nisc-security-guideline/


# AWS環境における脅威検知と対応
- https://pages.awscloud.com/rs/112-TZM-766/images/A1-02.pdf
- AWS Summit2019
- 脅威検知の手法：4つのサービスの有効化
  - CloudTrailでログ収集
  - Configでログ収集
  - GuardDutyでログから検知
  - SecurityHubで検知
- NISTのコア機能別考え方
  - ![image](https://user-images.githubusercontent.com/60077121/109420857-ed5c0780-7a17-11eb-97bf-2c36fbe62a34.png)

# AWSクラウドセキュリティ
- セキュリティ関連のホワイトペーパー、情報などがまとまっている
- https://aws.amazon.com/jp/security/?nc=sn&loc=0

# セキュリティの柱 - AWS Well-Architected フレームワーク
- ホワイトペーパーのひとつ。日本語でhtmlでまとまっているので良い
- https://docs.aws.amazon.com/ja_jp/wellarchitected/latest/security-pillar/welcome.html

# AWSホワイトペーパーセキュリティベストプラクティス
- [AWSホワイトペーパーセキュリティベストプラクティス](https://d1.awsstatic.com/whitepapers/ja_JP/Security/AWS_Security_Best_Practices.pdf)

# WAFってなに
- [WAFとはbyIPA](https://www.ipa.go.jp/files/000017312.pdf)

# AWS上のセキュリティ対策をどういう順序でやっていけばいいか
- https://speakerdeck.com/cmusudakeisuke/awsshang-falsesekiyuriteidui-ce-wodouiushun-xu-deyatuteikebaiika
- ![image](https://user-images.githubusercontent.com/60077121/109730302-a418dd00-7bfc-11eb-95d6-56db6705a9ae.png)
- ![image](https://user-images.githubusercontent.com/60077121/109730381-be52bb00-7bfc-11eb-85f7-e50cebd26136.png)
- ![image](https://user-images.githubusercontent.com/60077121/109730453-d6c2d580-7bfc-11eb-827f-2e0776d72fcc.png)

# Best Practices for Security, Identity, & Compliance
- これもセキュリティの柱みたいな構成で、各ホワイトペーパーを紹介

