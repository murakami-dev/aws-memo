

## Tips
### そもそもBGPはTCP
https://www.infraexpert.com/study/tea5.htm

### 静的と動的の判断
- 基本は動的を使う
>使用可能な場合は BGP に対応したデバイスを使用することをお勧めします。BGP プロトコルは安定したライブ状態検出チェックが可能であり、1 番目のトンネル停止時の 2 番目の VPN トンネルへのフェイルオーバーに役立ちます。BGP をサポートしていないデバイスでも、ヘルスチェックを実行することによって、必要時に 2 番目のトンネルへのフェイルオーバーを支援できます。
>https://docs.aws.amazon.com/ja_jp/vpn/latest/s2svpn/VPNRoutingTypes.html#vpn-static-dynamic

- Transit Gatewayのルーティング仕様を分かりやすく解説してみる
  - https://blog.serverworks.co.jp/tech/2020/06/30/transit-gateway-routing/

### トンネルのフェイルオーバー
