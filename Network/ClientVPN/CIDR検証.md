
## CIDRの検証
- VPC`172.19.64.0/18`
- ClientVPNを関連付けるサブネット`172.19.80.0/24`
- サブネットのRTB
  - `10.0.0.0/8`
  - `172.16.0.0/12`
- ClientVPVクライアントCIDR`10.93.0.0/18`

## ドキュメントによると
```
クライアント CIDR 範囲は、関連付けられたサブネットが配置されている VPC のローカル CIDR、
または クライアント VPN エンドポイントのルートテーブルに手動で追加されたルートと重複することはできません。
```
https://docs.aws.amazon.com/ja_jp/vpn/latest/clientvpn-admin/what-is.html#what-is-limitations

- clientvpnRTBに`0.0.0.0/0`を追加するのはOK
- クライアント CIDR 範囲がサブネットのRTBのルートと重複はOK（上記例はOKだった）。
