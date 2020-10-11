# 4
![image](https://user-images.githubusercontent.com/60077121/95654612-17319200-0b3c-11eb-970f-5833d0929163.png)

# 6
![image](https://user-images.githubusercontent.com/60077121/95654666-7e4f4680-0b3c-11eb-8c1d-6799cf3dedd0.png)
- 送りたいもの=データ
- L4でデータを制御データでカプセル化→セグメント
- L3ではパケット
- L2ではフレーム
- L1ではビット

# 10 L1の機器
- リピータ
  - リピータは弱まったりノイズが入った信号を、増幅や整形して、元の信号と同じ強さ、同じ形に直す。
- ハブ
  - 多くの機器を繋ぐ時に使用される。リピータの役割も持つ
  
# 11 L1 ネットワークトポロジ
- ノードはコンピュータやネットワーキングデバイス。リンクはメディア。
- ノードは点、リンクは直線で表記される。

# 12 L2 フレームの伝送制御
![image](https://user-images.githubusercontent.com/60077121/95655061-bad07180-0b3f-11eb-8141-e87ceb0860ec.png)
![image](https://user-images.githubusercontent.com/60077121/95655083-d5a2e600-0b3f-11eb-8ae7-b816b7716882.png)

# 13 L2 アドレッシング
- MACアドレスは物理アドレスのひとつ
- MACアドレスはNICにつけられたアドレス。
- `ipconfig /all`でも`phyisical address`と表示される
- MACアドレスは48ビット。16進数で12桁で表示される
- 先頭の24ビット、つまり16進数だと6桁目までをベンダコード

- MACアドレスはNICに紐づいているので正確なホストの宛先ではない。この欠点をL3の論理アドレスが補完する。

# 14
![image](https://user-images.githubusercontent.com/60077121/95655255-eb64db00-0b40-11eb-8f4d-e7cec84b9a1e.png)

- プリアンプルは、「これはフレームですよ。プリアンブルが終わるとデータが始まりますよ」という前触れの部分

- イーサネットはフレームが全員に届く。MACアドレスが自分宛でないものは破棄する（繋がっているすべてのノードへフレームを送りつけるブロードキャスト型で、CSMA/CDでアクセス制御をしてる）

# 16 IEEE802.5やFDDI
- ブロードキャスト型のイーサネットとは違い、トークンパッシングで制御する

![image](https://user-images.githubusercontent.com/60077121/95655429-329f9b80-0b42-11eb-98a1-b95f95215d1e.png)

# 17 ブリッジとスイッチ
- ブリッジ
- 宛先MACアドレスを読み取る(ブリッジの向こう側へ渡すか判断する)
- ただし、ブリッジではブロードキャストを止めることができない
![image](https://user-images.githubusercontent.com/60077121/95655521-fcaee700-0b42-11eb-9b50-51f698492b89.png)

# 18 ブリッジとスイッチ
- レイヤ２スイッチはスイッチングハブという
- スイッチもブリッジ同様、MACアドレスによるフィルタリングを行う
- ブリッジと異なる点は、スイッチは通すか通さないかの2択ではなく誰あてに送信するかも判断する

- ストアアンドフォワード方式
![image](https://user-images.githubusercontent.com/60077121/95655671-ed7c6900-0b43-11eb-8a57-f5ee855d875b.png)

# 19 L3 ルータ
- スイッチがMACアドレスに基づいてスイッチングを行うのに対し、ルータはレイヤ３の論理アドレスに基づいてスイッチングを行う。
- ルータは宛先に対して最適な経路を選択する能力
- ルータはブロードキャストをフィルタリングする（#26で言及）

# 20 IP
- L3のプロトコルはIP
- L1,L2ではWANならば、各種回線やPPP。LANならばUTPやイーサネット（実際の接続によりかわる）

- IPのカプセル化
![image](https://user-images.githubusercontent.com/60077121/95656016-58c73a80-0b46-11eb-8b62-9a85cd920c91.png)

- IPヘッダは20バイト(160ビット)で、オプションを最大40バイト(320ビット)つけることが可能
![image](https://user-images.githubusercontent.com/60077121/95656040-7d231700-0b46-11eb-8a18-cf9223352f75.png)

# 23
- プライベートIP
![image](https://user-images.githubusercontent.com/60077121/95656240-f2dbb280-0b47-11eb-8044-b46b72e864c2.png)

# 25 DHCP
![image](https://user-images.githubusercontent.com/60077121/95656436-629e6d00-0b49-11eb-89dc-e7fdf0e94575.png)

# 26 L3 ARP（アドレス解決プロトコル）
- 宛先IPアドレスから宛先MACアドレスがわかる

# 27 DNSとNetBIOS
- LANならNetBIOS over TCP/IPで宛先IPアドレスがわかる（プライベートDNSのようなもの）

# 28 L3 ルータ
- ルータは接続するネットワークごとにポートを有し、各ポートにはIPアドレスがある
![image](https://user-images.githubusercontent.com/60077121/95656836-e9544980-0b4b-11eb-9037-56a03549dfbe.png)

※ここではルータはブロードキャストを通さないのに、なぜARPで離れたLANのMACアドレスを知れるのかについて解説。割愛

# 32 ルーティングプロトコル
- ネットワークの区別→AS
- AS間のルーティング用→EGP
- AS内部のルーティング用→IGP　（internalと覚える）
![image](https://user-images.githubusercontent.com/60077121/95657110-a85d3480-0b4d-11eb-9c99-cdae5a2c4356.png)

- メトリック→最小の値を持つものを最適ルートとする

# 35 ping
- pingが成功したのに繋がらないのはレイヤ４以上に障害があることを示すしpingが失敗したならレイヤ３以下に障害があることになる。

# 39 TCPコネクション
<img width="629" alt="スクリーンショット 2020-10-11 6 51 52" src="https://user-images.githubusercontent.com/60077121/95665791-81732280-0b8e-11eb-8bc0-57986c0176ea.png">
<img width="561" alt="スクリーンショット 2020-10-11 6 54 34" src="https://user-images.githubusercontent.com/60077121/95665807-a8315900-0b8e-11eb-91b0-d8bc220ec101.png">

- スリーウェイハンドシェイク
<img width="486" alt="スクリーンショット 2020-10-11 6 55 19" src="https://user-images.githubusercontent.com/60077121/95665822-c4cd9100-0b8e-11eb-869a-260ea0b0ecdb.png">

# シーケンスと確認応答
<img width="714" alt="スクリーンショット 2020-10-11 7 05 10" src="https://user-images.githubusercontent.com/60077121/95666027-a6689500-0b90-11eb-88cc-90eb0602579a.png">

# 43 ポート
<img width="585" alt="スクリーンショット 2020-10-11 7 17 20" src="https://user-images.githubusercontent.com/60077121/95666141-dfedd000-0b91-11eb-88e7-5d64150e0d7e.png">

# 44 UDP
- UDPヘッダはTCPヘッダの約４割
- UDPはソケットを確立している時間が短時間（#45で言及）
<img width="625" alt="スクリーンショット 2020-10-11 7 19 06" src="https://user-images.githubusercontent.com/60077121/95666198-52f74680-0b92-11eb-92df-4d1676f3588d.png">

#45 netstat
- ローカルのどのポートでどこと通信しているか確認する
- `netstat -na`-aで全て、-nでDNS逆引きを省略（-aだけだと時間かかる）。
  - https://www.atmarkit.co.jp/fnetwork/netcom/netstat/netstat.html

# 47 NAPT
- GIPがひとつだけでも複数機器をLANにぶら下げられるのはNAPTのおかげ
- NAPTはプライベートIPだけでなく送信元ポート番号も変換してくれる。送り返された際は変換したポートあてにデータが来るので記憶しておいた変換元の機器に送り返す
- 静的NAPTはLAN内でWEBサーバを公開している際など、80番をずっと記憶してくれる機能のこと
<img width="555" alt="スクリーンショット 2020-10-11 7 45 50" src="https://user-images.githubusercontent.com/60077121/95666565-d8c8c100-0b95-11eb-9da3-6af17d64f822.png">

# 50 L6
- レイヤ６はデータの変換、圧縮、暗号化を行う。
- レイヤ６の機能により、異なるハードウェアやOSでのデータ形式の違いを考えなくてもよいデータ転送が可能になる。

### 53くらいからのtelnetは割愛

# 56 FTP
- ポートは20,21を使う
- FTPはOSごとの差異を意識することなく使用できるクライアント・サーバ型
  - （余談）PI（プロトコルインタープリタ）：プログラム言語をコンピュータがわかる言葉に翻訳するソフト
  - DTP:データトランスファープロトコル
<img width="485" alt="スクリーンショット 2020-10-11 8 06 35" src="https://user-images.githubusercontent.com/60077121/95666844-c9974280-0b98-11eb-85ed-06d501ab9297.png">

# 64 DNS
- Aレコードはドメイン名とIPアドレスの対応を記述
- NSレコードはそのドメインとそのドメインの下のドメインを管理するネームサーバ
- 複数の名前を設定するのがCNAMEレコード（別名みたいな）
- SOAはゾーン情報の管理パラメータ
- MXはそのドメインのメールサーバを定義するためのレコード。優先値が設定されている。優先値の低いメールサーバを最初に使用する。

# 65 DNS
- DNSリゾルバはホストが持つDNS問い合わせ用のソフトウェア
- 再帰問い合わせ
  - クライアントからネームサーバへの問い合わせで使われる。リゾルバはネームサーバに完全な回答を問い合わせる。
- 反復問い合わせ
  - 他のゾーンのドメイン名を聞かれた場合は他のネームサーバに問い合わせる
<img width="1004" alt="スクリーンショット 2020-10-11 8 30 03" src="https://user-images.githubusercontent.com/60077121/95667152-0f560a00-0b9d-11eb-8b00-0270233dc087.png">

# 68 ゾーン転送
- プライマリのDNSサーバがセカンダリのDNSサーバに対してゾーン情報の同期を行うこと
- SOAレコードのシリアル値をみて、プライマリに対してゾーン情報を要求するか決める

# 71 HTTP
<img width="855" alt="スクリーンショット 2020-10-11 8 48 18" src="https://user-images.githubusercontent.com/60077121/95667312-2e559b80-0b9f-11eb-9ac3-979e2162eb2f.png">

# 73 HTTPメッセージヘッダ
- 代表的なもの
  - Host (Request/End-to-End):唯一必須のヘッダ。宛先サーバのドメイン名
  - User-Agent (Request/End-to-End):送信元のブラウザの種類
<img width="899" alt="スクリーンショット 2020-10-11 8 59 25" src="https://user-images.githubusercontent.com/60077121/95667383-3eba4600-0ba0-11eb-94a3-610c31dc52e6.png">

# 75 Cookie
- サーバはステートレスなのでクライアントにCookieをつくるよう要求しセッション状態を保つ
- サーバはクライアントにSet-Cookieメッセージヘッダを送る（ネットスケープ社独自）
- クライアントからCookieの内容を送る時は「Cookie」メッセージヘッダ
<img width="524" alt="スクリーンショット 2020-10-11 9 13 25" src="https://user-images.githubusercontent.com/60077121/95667513-29deb200-0ba2-11eb-9d3b-754bd30aace8.png">

- Cookieの例（名前のみ必須であとはサーバ側が決める）
```
Set-Cookie：userid = 123 ; Expire = Thursday, 30-Jun-2005 00:00:00 GMT ; path = /net/ ; domain = www.3min.net ; username = prof;
```
  - 名前(userid) … 123
  - 有効期限（Expire）… 2005/06/30(木) 0時0分0秒
  - パス属性（path） … /net/
  - サーバドメイン名（domain）… www.3min.net
  - その他サーバが処理に必要な識別情報(username) … prof
  
- もしCookieの中身にExpireがあった場合、ファイルとして保存される。
- Expireがない場合はファイルとして保存されずにブラウザが閉じるまでしか使われない。これはセッションCookieと呼ばれる。
- XSS対策にはセッションCookieにするのが定石

# 77 認証（ただし現在はSSLが主流）
- Basic認証（基本認証）
  - サーバはWWW-Authenticateメッセージヘッダをクライアントに送信し、クライアントはユーザ名とパスワードをAuthorizationリクエストヘッダをつけて通信する。
  - 基本認証の欠点はパスワードがそのまま送られてしまう。BASE64でコード化されるが、戻すのが簡単
- ダイジェスト認証
  - ハッシュ関数を使ってハッシュ値にして通信する
<img width="481" alt="スクリーンショット 2020-10-11 9 31 52" src="https://user-images.githubusercontent.com/60077121/95667720-bf7b4100-0ba4-11eb-98bd-7c7ad9df2b7b.png">
