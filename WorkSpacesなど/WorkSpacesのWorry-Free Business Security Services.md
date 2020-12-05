# 文献
- [WorkSpacesにバンドルされるセキュリティソフト(WFBS)の管理コンソールについて](https://blog.serverworks.co.jp/tech/2020/04/30/workspaces-wfbs/)
- [管理ガイド](https://files.trendmicro.com/jp/ucmodule/vbbiz/100/biz10_ag_r1.pdf)

# 流れ
- ※ウイルスバスタービジネスセキュリティの体現版でやってみる
  - https://www.trendmicro.co.jp/business/products/vbbss/
  
## 管理コンソールのログイン情報が来る
- アカウント名
- パスワード
- コンソールログインURL
- >ログイン後、WorkSpacesへのエージェントの展開の詳細については、[コンソール]> [管理]> [ツール]を開きます。
とくる。

## エージェントのインストール
- 各WSへエージェントをインストールする。p.40くらいから
  - >すべてのインストール方法で、インストール先エンドポイントのローカル管理者権限が必要となります。
  - ![image](https://user-images.githubusercontent.com/60077121/100849201-918bec00-34c5-11eb-8ab1-19d3f6b186d1.png)

### クライアントはインストーラをDLする必要がある。
- ログインスクリプトは
- 配信スクリプトはインストーラを共有フォルダに配置して、スクリプトを使用し各クライアントがDLする方法
  - https://docs.trendmicro.com/ja-jp/smb/worry-free-business-security-services-56/section_agent_install/agent_install_download_pkg_overview/agent_install_deployment_script.aspx

- インストーラの実行には、というか常にインターネットに通信できる環境が必要

# ポリシーの設定
## 対象とサービスの設定
次のプラットフォームで不正変更防止サービスを有効にする

- Windows servers

## 検索設定

| 設定項目 | 設定値 | 備考 |
----|----|---- 
| 検索方法   |  スマートスキャン（推奨） | スマートスキャンまたは従来型スキャンから選択。スマートスキャンでは、インターネットクラウドに格納されている不正プログラム対策シグネチャおよびスパイウェア対策シグネチャが利用され、従来型ではローカルのものが使用される |
| リアルタイム検索 | 有効 |ファイルを受信、開く、ダウンロード、コピー、または変更したときに、セキュリティ上のリスクがあるかファイルを検索する |
| 予約検索 | 無効 |月1回/週1回/毎日、曜日、時刻が選択可能 |

- [スマートスキャンの通信先](https://success.trendmicro.com/jp/solution/000237288)作成中
- >ディスク容量(Guide p.57)
  - >従来型スキャン: 1.5GB 以上 (2GB を推奨)
  - >スマートスキャン: 500MB

## 挙動監視
| 設定項目 | 設定値 | 備考 |
------------- | ------------- | -------------
| 不正プログラム挙動ブロック  |  既知および潜在的な脅威のブロック |  |
| ランサムウェア対策 | 有効 | ・不正なファイル暗号化や変更から文書を保護<br>・不審なプログラムによって変更されたファイルを自動的にバックアップして復元<br>・ランサムウェアに関連付けられていることの多いプロセスをブロック<br>・プログラム検査を有効にして不正な実行可能ファイルを検出およびブロック|
| 脆弱性対策 | 有効 |脆弱性攻撃に関連する異常な挙動を示すプログラムを終了 |
| Intuit™ QuickBook™保護 |無効 | |
| イベント監視 |無効 |不正プログラム挙動ブロックによる保護ではなくイベントごとに許可/ブロックを設定する要件がある場合のみ有効|

- Intuit™ QuickBookhはアメリカの会計ソフトのファイル
  - https://docs.trendmicro.com/ja-jp/smb/worry-free-business-security-services-56/section_groups_sec_settings/section_config_win/be_monit_config_wfbs-svc.aspx
- イベント監視の制御イメージ
  - ![image](https://user-images.githubusercontent.com/60077121/101235935-18340980-3710-11eb-8b2a-54c018ec4d6f.png)


## URLフィルタリング
![image](https://user-images.githubusercontent.com/60077121/101042691-ce88d900-35c0-11eb-8c25-702c28b74d78.png)

### URLフィルタリング　トラブル
- [ChromeまたはEdgeでHTTPSサイトがブロックされない場合がある](https://success.trendmicro.com/jp/solution/000154688)

## WEBレピュテーション
- Webレピュテーションは、トレンドマイクロのWebセキュリティデータベースを利用して、クライアントがアクセスしようとしているURLのレピュテーションをチェックする。
- 安全性と誤検出のリスクを考慮し、デフォルトの`中`とする。

> ### セキュリティレベル
> 
> |レベル | 危険| 極めて不審| 不審|
>  ------------- | ------------- | ------------- | -------------
> | 高  | ブロック  |ブロック |ブロック|
> | 中  | ブロック | ブロック|閲覧可|
> |低 | ブロック  |閲覧可|閲覧可|
> - 危険：詐欺サイトや脅威の既知の送信元であることが確認されました
> - 極めて不審: 詐欺サイトまたは脅威の送信元である可能性があります
> - 不審: スパムメールに関連付けられている、またはセキュリティ侵害の可能性があります

### その他の設定
| 設定項目  | 設定値 |備考|
 ------------- | ------------- |-------------
| 未評価のURL | 無効  |有効の場合トレンドマイクロによる評価が完了していないWebサイトをブロックします。安全なページも含まれることから無効とします|
| ブラウザ脆弱性対策 | 有効  |有効の場合不正なスクリプトを含むWebサイトをブロックします|

## 情報漏洩
機密データを定義し、データの転送を監視またはブロックします（有効化には再起動が必要）
- [設定方法](https://success.trendmicro.com/jp/solution/1119387)
- https://success.trendmicro.com/jp/solution/1312344
  - 上記のサイトの「ビジネスセキュリティ」に以下のPDFがある
    - [情報漏えい対策リスト]
- ![image](https://user-images.githubusercontent.com/60077121/101235900-d5723180-370f-11eb-97c5-29280d951f44.png)



# その他参考
- [ウイルスバスター ビジネスセキュリティサービス を運用するために必要な通信ポート](https://success.trendmicro.com/jp/solution/1310552)
- [Active Directoryとの自動同期方法について](https://success.trendmicro.com/jp/solution/1118182)
- [Active Directory 構造の手動同期による設定方法](https://success.trendmicro.com/jp/solution/1115789)
