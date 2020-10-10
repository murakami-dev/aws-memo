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

