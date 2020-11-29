
- オンプレ-DXロケーション間の接続を「コネクション」という
- [LOA-CFA は、AWS に接続するための認可であり、クロスネットワーク接続 (クロスコネクト) を確立するためにコロケーションプロバイダーまたはネットワークプロバイダーで必要になります](https://docs.aws.amazon.com/ja_jp/directconnect/latest/UserGuide/getting_started.html)

## 文献
- 1.[Direct Connect接続タイプとVIF作成パターンをまとめてみた](https://dev.classmethod.jp/articles/direct-connect-connection-pattern-vif/)

## VIF分類（文献1参照）
- 標準VIF（VIFの所有者は自分）
- ホスト型VIF（VIFの所有者は他アカウント（APNパートナー等））
- ホスト型接続のVIF（このパターンだけ1接続1VIF、帯域は狭い）


## 流れ（実際にやったこと）
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
- 接続の依頼
- LOA
- VIFの払い出し依頼(完了するとVIFがマネコンに表示)
- VIFの承諾（**VGWかDXGWを選ぶ。やり直し不可**）
- オンプレルータの設定(これができるとVIFの状態がDOWNから改善される)

## 気にすること
- APNパートナー等へVIFの払い出し依頼は必要か（AWS-DXロケーション間）
  - 物理接続（回線敷設）の後、論理接続（VIF）を準備
  - **VIF払い出し依頼の際はアカウントIDが必要**
  - VIF払い出しまでは1か月程度必要（APNパートナーによる）
- DX切り替えの日
- 複数VPCを使う可能性があるならDXGW使おう（DXGW自体は無料）
