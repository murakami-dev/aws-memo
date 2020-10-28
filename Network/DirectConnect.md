
- オンプレ-DXロケーション間の接続を「コネクション」という
- [LOA-CFA は、AWS に接続するための認可であり、クロスネットワーク接続 (クロスコネクト) を確立するためにコロケーションプロバイダーまたはネットワークプロバイダーで必要になります](https://docs.aws.amazon.com/ja_jp/directconnect/latest/UserGuide/getting_started.html)

## 文献
- 1.[Direct Connect接続タイプとVIF作成パターンをまとめてみた](https://dev.classmethod.jp/articles/direct-connect-connection-pattern-vif/)

## VIF分類（文献1参照）
- 標準VIF（VIFの所有者は自分）
- ホスト型VIF（VIFの所有者は他アカウント（APNパートナー等））
- ホスト型接続のVIF（このパターンだけ1接続1VIF、帯域は狭い）


## 流れ
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
