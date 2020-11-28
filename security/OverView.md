# このページでは全体像をまとめる
## 文献
- [AWSホワイトペーパーセキュリティベストプラクティス](https://d1.awsstatic.com/whitepapers/ja_JP/Security/AWS_Security_Best_Practices.pdf)
- [WAFとはbyIPA](https://www.ipa.go.jp/files/000017312.pdf)

## AWS Summit2019
### AWS 環境における脅威検知と対応
資料:https://pages.awscloud.com/rs/112-TZM-766/images/A1-02.pdf

- 4つのサービスを有効化しよう
  - GuardDuty：不正なIPからアクセス検知、トロイの木馬
  - CloudTrail:（ログを記録）APIコールを監視
  - Config:（ログを記録）リソースの構成変更を記録（SGのポート開放）
  - SecurityHub：セキュリティサービス（サードパーティ含む）を一元管理



- セキュリティの全体像（NIST Cybersecurity Framework）
<img width="1127" alt="スクリーンショット 2020-09-30 8 21 55" src="https://user-images.githubusercontent.com/60077121/94626481-20c62900-02f6-11eb-9a15-66b42bdee3e9.png">
※https://d1.awsstatic.com/whitepapers/compliance/JP_Whitepapers/NIST_Cybersecurity_Framework_CSF_JP.pdf  

- セキュリティワークショップで疑似攻撃など試せる
https://awssecworkshops.jp/

### Edge Services を利用した DDoS 防御の構成（AWS WAF/Shield）
資料：https://pages.awscloud.com/rs/112-TZM-766/images/B1-03.pdf

- まずはDDos対策のホワイトペーパー読もう
  - https://d1.awsstatic.com/whitepapers/Security/DDoS_White_Paper.pdf

- Layer3,4,7の攻撃ってなんだ
 - <img width="927" alt="スクリーンショット 2020-09-30 8 56 44" src="https://user-images.githubusercontent.com/60077121/94628451-fd51ad00-02fa-11eb-9c69-da7a2cebfa70.png">

- WAF,Shieldだけじゃだめ。
  - <img width="940" alt="スクリーンショット 2020-09-30 8 56 04" src="https://user-images.githubusercontent.com/60077121/94628507-2114f300-02fb-11eb-95e6-9fbf33b4218d.png">
  - <img width="931" alt="スクリーンショット 2020-09-30 8 56 12" src="https://user-images.githubusercontent.com/60077121/94628542-3853e080-02fb-11eb-916e-2f35adfa4062.png">
