# 専用線ハンズオンラボの知見も入れたい
- オンプレ-DXロケーション間の接続を「コネクション」という
- コネクション（物理的な接続）の中に複数の論理的な接続であるVLAN（VLANにはそれぞれVLANIDがある）。**コネクションとVLANは1対多**
  - https://www.furukawa.co.jp/fitelnet/product/f200/setting/detail/amazon_vpc_3.html
  - ![image](https://user-images.githubusercontent.com/60077121/100759086-90aa7a00-3433-11eb-9a1a-ea3cd2651a7d.png)
- [LOA-CFA は、AWS に接続するための認可であり、クロスネットワーク接続 (クロスコネクト) を確立するためにコロケーションプロバイダーまたはネットワークプロバイダーで必要になります](https://docs.aws.amazon.com/ja_jp/directconnect/latest/UserGuide/getting_started.html)

# 文献
- 1.[Direct Connect接続タイプとVIF作成パターンをまとめてみた](https://dev.classmethod.jp/articles/direct-connect-connection-pattern-vif/)
- 2.[やさしいDirect Connect の構築](https://oji-cloud.net/2019/12/18/post-3759/)
  - パートナー経由（パートナーが接続を作成）。接続を承諾するパターン
- 3.[AWS Direct Connect経由でEC2にtracerouteを打つ](https://blog.serverworks.co.jp/traceroute-via-directconnect)]
  - ピアIPの部分に後述
- 4.[ダイレクトコネクトの図解と本番の構成](https://blog.hiroakis.net/posts/6/index.html)

# まずは絵で理解
![image](https://user-images.githubusercontent.com/60077121/100880729-708cc080-34f0-11eb-9daa-49966b9c5b46.png)
- 例：`169.254.202.0/30`のネットワークの中でAWS-オンプレを繋ぐルータが同じネットワークを構成する。
- `169.254.202.1/30`（Amazon側）も`169.254.202.2/30`（オンプレ側）も第4オクテット`0`の`169.254.202.0/30`と同じ意味。


# VIF分類（文献1参照）
- 標準VIF（VIFの所有者は自分）
- ホスト型VIF（VIFの所有者は他アカウント（APNパートナー等））
- ホスト型接続のVIF（このパターンだけ1接続1VIF、帯域は狭い）


# 流れ（実際にやったこと）
- 接続の作成（クラシックモードで良い）
  - 名前
  - **ロケーション**
  - ポート速度（1Gbps or 10Gbps）
  - パートナーを通じて接続するかどうか
  - 上記がyesの場合、パートナー名
- 接続が作成されたらLOA-CFAのダウンロード
  - Letter of Authorization and Connecting Facility Assignment（承認書および接続機能の割り当て）
  - rackの情報などが掲載
  - これをパートナー等に共有
- VGW作成、VPCにアタッチ（TGWの場合は別）
- DXGWの作成（名前とAS番号デフォ64512）
- DXGWの関連付けからVGWを選択
- VIF作成
  - VLAN
  - BGP ASN
  - ルーターのピアIP（オプション、後述）
  - Amazon側ルータピアIP（同上）
  - BGP認証キー（オプション、これがMD5）
  - ジャンボフレーム（無効が無難そう）

### 以下はおそらくパートナー経由
- 接続の依頼
- LOA
- VIFの払い出し依頼(完了するとVIFがマネコンに表示)
- VIFの承諾（**VGWかDXGWを選ぶ。やり直し不可**）
- オンプレルータの設定(これができるとVIFの状態がDOWNから改善される)

# 気にすること
- APNパートナー等へVIFの払い出し依頼は必要か（AWS-DXロケーション間）
  - 物理接続（回線敷設）の後、論理接続（VIF）を準備
  - **VIF払い出し依頼の際はアカウントIDが必要**
  - VIF払い出しまでは1か月程度必要（APNパートナーによる）
- DX切り替えの日
- 複数VPCを使う可能性があるならDXGW使おう（DXGW自体は無料）

## ASN

## ピアIP
- 基本は入力しない。**リンクローカルアドレス`169.254.0.0/16`から払い出される**
  - 変にプライベートIP入れると、VPCのCIDRと被ったとき、オンプレから重複したアドレス帯へ通信できなくなる。（個人slack参照）
- 冒頭で掲載した絵が分かりやすい（文献2の絵）
- [AWS Direct Connect経由でEC2にtracerouteを打つ](https://blog.serverworks.co.jp/traceroute-via-directconnect)
  - ![image](https://user-images.githubusercontent.com/60077121/100758519-f21e1900-3432-11eb-961f-ce72692064a5.png)
- >(*1)ピア (peer)：対等の立場で通信を行うノード、または通信相手のことを指します。
  - https://www.nic.ad.jp/ja/basics/terms/p2p.html

## ジャンボフレーム（基本は無効）
- 有効にすると大きいサイズのパケット送信できるので早くなる。
- 有効にするためには確認事項が多い。すべての中間デバイスのMTUが9001に対応しているか確認する必要あり
  - >ルーター、スイッチ、エンドデバイス (ネットワークカードなど) を含むすべてのノードがジャンボフレームをサポートしていることを確認します。VPN の Amazon Elastic Compute Cloud (Amazon EC2) インスタンスもジャンボフレームをサポートしている必要があります。そうしないと、互換性のないデバイスがフレームをドロップまたはフラグメント化する可能性が高いため、ネットワークパフォーマンスが低下する可能性があります。ジャンボフレームは、DX から伝播されたルートにのみ適用されます。仮想プライベートゲートウェイを指す静的ルートをルートテーブルに追加すると、静的ルート経由でルーティングされたトラフィックは 1500 MTU を使用して送信されます。
  - [Direct Connect でジャンボフレームを使用するタイミングを決定するにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/dx-use-jumbo-frames-with-direct-connect/)
- ジャンボフレームに対応していないコネクション（接続）でジャンボフレーム有効のVIFを作成すると30秒くらい通信とまる
  - >仮想インターフェイスのMTUを8500（ジャンボフレーム）または9001（ジャンボフレーム）に設定すると、ジャンボフレームをサポートするように更新されていない場合、基盤となる物理接続が更新される可能性があります。接続を更新すると、接続に関連付けられているすべての仮想インターフェイスのネットワーク接続が最大30秒間中断されます。
  - https://docs.aws.amazon.com/directconnect/latest/UserGuide/set-jumbo-frames-vif.html
- VPCピアリングも対応していない

# 作成中
- MD5の共有
- BGPパラメータ
  - Timer値：推奨は10秒/40秒
    - 10秒に1回のkeepaliveで40秒でタイムアウト。peerがダウンして約1分後に切り替わるイメージ。
  - BFDはONが推奨。
    - ms単位でのkeepaliveなのでダウンして即切り替わる
    - BGPタイマーとBFDを組み合わせてダウン時の切り替わりが早くなる模様。
