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

## 自動化修復アクションを実装する
- [ちなみにオリジナルのカスタムアクションをつくる方法はこちら](https://aws.amazon.com/jp/blogs/news/automated-response-and-remediation-with-aws-security-hub/)
### 事前構築
- ServiceCatalogを使ったカスタムアクションの自動デプロイ（手順は以下のクラメソブログがとっても良い記事）
  - AWS Security Hub の自動化された応答と修復
    - https://aws.amazon.com/jp/solutions/implementations/aws-security-hub-automated-response-and-remediation/
  - [神機能]Security HubでCISの違反を自動修復する仕組みが提供されたので試したりちょっと深堀りして理解してみた
    - https://dev.classmethod.jp/articles/security-hub-cis-auto-remediation/
    
- 以下リンクの`ステップ1.スタックを起動します`のCFnを対象のリージョンで流す
  - https://docs.aws.amazon.com/solutions/latest/aws-security-hub-automated-response-and-remediation/deployment.html
- 作成されたIAMグループ（サービスカタログの管理者、使用者）に使用するIAMユーザーを加える
  - スイッチロールを使用する場合はサービスカタログのポートフォリオの設定を変更して該当ロールを加える。デフォルトではポートフォリオにアクセス権限あるのはIAMグループの`SO0111-SHARR_Catalog_Admin_ap-northeast-1`,`SO0111-SHARR_Catalog_User_ap-northeast-1`
- サービスカタログの製品（管理者メニューじゃない方）から、製品名CISを選択し製品の起動を選択。
- 名前を入力（グルーバルに一意）。バージョンを選択して次へ
- パラメータは全て`DISABLE`にする。`ENABLE`にするとセキュリティ違反になったリソースが直ちに矯正される。
- **SNS通知は有効にしない、ゼッタイ、メール500通きた**
- 冒頭リンクの`手順4.ソリューションのアクセス許可を展開する`のCFn流す

### 修復アクション
- 検出結果で修復したい事項をチェック。リソースごとに表示されている
![image](https://user-images.githubusercontent.com/60077121/96569524-37afd800-1304-11eb-828b-3298d365b953.png)

- アクションからデプロイしたカスタムアクションを実行。押した瞬間CloudWatchイベントに送信され修復されるので注意
  - **ここで誤って違うアクション選択しないようデプロイするアクションを取捨選択できないか**
![image](https://user-images.githubusercontent.com/60077121/96569694-6af26700-1304-11eb-91f2-c3c4a2cb87c3.png)

- デフォルトのSGのルールが削除された(これには1分もかかってない、たぶんSGだから)
![image](https://user-images.githubusercontent.com/60077121/96574221-2073e900-130a-11eb-8d58-592eff3bb0cc.png)

### どうなっているのか（わかる範囲で書く）
- CFnについて
- `aws-sharr-deploy.template`
- `SC-913926916566-pp-egzwu2lwyewj6`
  - サービスカタログの製品をデプロイしたときに流れたCFn。CloudwatchイベントとそのターゲットであるLambdaが作成されている。
- `CISPermissions`
  - IAMロールが作成されている。このIAMロールがLambdaの実行ロールになる。
  - 例えばCIS43のSGに関する項目なら、`SO0111_CIS43_memberRole_ap-northeast-1`というSG触れるロールができる。それをLambdaがassumeroleしてクロスアカウントでAPIコールする仕組み。
- CloudWatchイベントが作成される
![image](https://user-images.githubusercontent.com/60077121/96591390-2c6ba500-1322-11eb-8ef2-d8f4355df853.png)



