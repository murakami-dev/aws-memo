# このページでは全体像をまとめる
## AWS Summit2019
### AWS 環境における脅威検知と対応
資料:https://pages.awscloud.com/rs/112-TZM-766/images/A1-02.pdf

- 4つのサービスを有効化しよう
-- GuardDuty：不正なIPからアクセス検知、トロイの木馬
-- CloudTrail:（ログを記録）APIコールを監視
-- Config:（ログを記録）リソースの構成変更を記録（SGのポート開放）
-- SecurityHub：セキュリティサービス（サードパーティ含む）を一元管理



- セキュリティの全体像（NIST Cybersecurity Framework）
<img width="1127" alt="スクリーンショット 2020-09-30 8 21 55" src="https://user-images.githubusercontent.com/60077121/94626481-20c62900-02f6-11eb-9a15-66b42bdee3e9.png">
※https://d1.awsstatic.com/whitepapers/compliance/JP_Whitepapers/NIST_Cybersecurity_Framework_CSF_JP.pdf

- セキュリティワークショップで疑似攻撃など試せる
https://awssecworkshops.jp/

