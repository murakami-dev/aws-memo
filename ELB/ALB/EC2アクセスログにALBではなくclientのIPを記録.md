# 目標
- client -> ALB -> EC2の場合、EC2のアクセスログにはALBのIP残る。これにclientのIPを残す

# 文献
- 1.[ELB アクセスログにクライアント IP アドレスをキャプチャする方法を教えてください。](https://aws.amazon.com/jp/premiumsupport/knowledge-center/elb-capture-client-ip-addresses/)
  - Linux前提
- 2.[IIS10でX-Forwarded-Forヘッダーに対応する方法は？](https://snap.hamazo.tv/e8401460.html)
  - IISはこのブログが分かりやすい
- 3.[ELB + EC2( IIS 8.5 ) でのIP制限](https://qiita.com/kt_higa/items/2f100e5fcbf163bf5e36)
  - IISの場合
- 4[IIS アクセスログの分析](https://jp.alibabacloud.com/help/doc-detail/84813.htm)

# 手順
基本、文献2のとおりでいける。<br>リクエストヘッダーにカスタムフィールドを挿入する
- 以下、特徴
  - X-Forwarded-Forヘッダーに対応するとログを出力しているフォルダが変わる
  - ファイル名も`u_ex190401.log`　→　`u_ex190401_x.log`と変わる

# ログ
- 見方は文献4参照
- ALBはプライベートIPで記録される`10.123.11.180`
- 末尾にX-Forwarded-For
```
#Software: Microsoft Internet Information Services 10.0
#Version: 1.0
#Date: 2021-02-13 09:03:11
#Fields: date time s-ip cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) cs(Referer) sc-status sc-substatus sc-win32-status time-taken X-FORWARDED-FOR
2021-02-13 09:03:45 10.123.10.218 GET / - 80 - 10.123.11.180 Mozilla/5.0+(Windows+NT+10.0;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/88.0.4324.150+Safari/537.36 - 304 0 0 7 218.xx.xx.xx
2021-02-13 09:03:45 10.123.10.218 GET / - 80 - 10.123.11.180 Mozilla/5.0+(Windows+NT+10.0;+Win64;+x64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/88.0.4324.150+Safari/537.36 - 304 0 0 3 218.xx.xx.xx
```

# 関係ないけどヘルスチェックに合格しなかった
## 不合格になったとき
- ALBのリスナー
  - 80 -> 443へリダイレクト
  - 443で受ける
- ターゲットグループ
  - ヘルスチェックプロトコルはHTTP
  - パスは`/`
  - HTTPバージョン1

## 合格したとき1
- 単純にHTTP80で受ける設定にしたら合格した。

## 合格したとき2
- ヘルスチェック用のファイルを作成し、`/hcheck.html`のパスへは80で受ける設定にした
![image](https://user-images.githubusercontent.com/60077121/107862308-4d3aa600-6e8f-11eb-9925-bb8dce820078.png)
