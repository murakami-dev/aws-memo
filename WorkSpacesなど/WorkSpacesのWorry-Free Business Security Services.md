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

- ログインスクリプトは
- 配信スクリプトはインストーラを共有フォルダに配置して、スクリプトを使用し各クライアントがDLする方法
  - https://docs.trendmicro.com/ja-jp/smb/worry-free-business-security-services-56/section_agent_install/agent_install_download_pkg_overview/agent_install_deployment_script.aspx
