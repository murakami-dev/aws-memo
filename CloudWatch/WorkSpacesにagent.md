# 文献
- 1[WorkSpacesにCloudWatchAgentを導入してWindowsイベントログを出力してみた](https://dev.classmethod.jp/articles/workspaces-cwa-windows-event-logs/)
- 2[WorkSpacesにCloudWatchAgentを導入してログ転送する方法](https://qiita.com/mo_chiee/items/36b292eed009f16d8b6d)

# 手順
- 基本文献の1でいける
- ただしCWagentのウィザードではオンプレミスを選択すること
- クレデンシャルはシステム環境変数に書く

# 説明
- 文献2にあるように方法は2つある
  - 1.文献1のようにシステム環境編にクレデンシャルを直接書く方法。
    - これならCLIのインストールは不要
  - 2.文献2にあるようにCLIの`config`ファイルにクレデンシャルを置く方法
    - ただし普通`config`ファイルはCドライブにあるがWorkSpaceの場合Dドライブに保持される。**その場合、複製時に消去される**
