

## Tips
### VPN設定にある「静的ルートタブ」について
- ルート伝播をしない場合は、サブネットのルートテーブルとVPNの「静的ルート」どちらも編集する必要がありそう。
```
ルートテーブルでルート伝播を有効にしていない場合、ルートテーブルで手動でルートを更新し、更新された静的 IP プレフィックスを Site-to-Site VPN 接続に反映する必要があります。
```
https://docs.aws.amazon.com/ja_jp/vpn/latest/s2svpn/vpn-edit-static-routes.html

- ルート伝播をする場合は、VPNの「静的ルート」を設定すれば、サブネットのルートテーブルにルートが伝播されるよう。
```
静的ルーティングでは、Site-to-Site VPN 接続の状態が UP であるときに、VPN 設定に指定した静的 IP プレフィックスがルートテーブルに伝播されます。
同様に、動的なルーティングでは、Site-to-Site VPN 接続の状態が UP のときに、カスタマーゲートウェイから BGP でアドバタイズされたルートがルートテーブルに伝播されます。
```
https://docs.aws.amazon.com/ja_jp/vpn/latest/s2svpn/SetUpVPNConnections.html#vpn-configure-routing


### そもそもBGPはTCP
https://www.infraexpert.com/study/tea5.htm

### 静的と動的の判断
- 基本は動的を使う
>使用可能な場合は BGP に対応したデバイスを使用することをお勧めします。BGP プロトコルは安定したライブ状態検出チェックが可能であり、1 番目のトンネル停止時の 2 番目の VPN トンネルへのフェイルオーバーに役立ちます。BGP をサポートしていないデバイスでも、ヘルスチェックを実行することによって、必要時に 2 番目のトンネルへのフェイルオーバーを支援できます。
>https://docs.aws.amazon.com/ja_jp/vpn/latest/s2svpn/VPNRoutingTypes.html#vpn-static-dynamic

- Transit Gatewayのルーティング仕様を分かりやすく解説してみる
  - https://blog.serverworks.co.jp/tech/2020/06/30/transit-gateway-routing/

### トンネルのフェイルオーバー
- そもそも2本あるトンネルのうちひとつはスタンバイ状態
```
仮想プライベートゲートウェイの場合、ゲートウェイ上のすべての Site-to-Site VPN 接続にまたがる 1 つのトンネルが選択されます。複数のトンネルを使用するには、トランジットゲートウェイ上の Site-to-Site VPN 接続でサポートされる Equal Cost Multipath (ECMP) について検討することをお勧めします。詳細については、Amazon VPC トランジットゲートウェイの「トランジットゲートウェイ」を参照してください。ECMP は、仮想プライベートゲートウェイの Site-to-Site VPN 接続ではサポートされていません。
https://docs.aws.amazon.com/ja_jp/vpn/latest/s2svpn/VPNRoutingTypes.html
```
- 自動でフェイルオーバーするがCGW側で2本のトンネルを使用するような設定が必要
```
Site-to-Site VPN 接続には 2 つのトンネルがあるため、VPC の可用性が向上します。AWS でデバイス障害が発生した場合、VPN 接続は自動的に 2 番目のトンネルにフェールオーバーして、アクセスが中断されないようにします。ときどき、AWS は、仮想プライベートゲートウェイで定期メンテナンスを実行します。これにより、VPN 接続の 2 つのトンネルの 1 つが短時間無効になる場合があります。このメンテナンスの実行中、VPN 接続は自動的に 2 番目のトンネルにフェイルオーバーします。したがって、カスタマーゲートウェイを設定するときは、両方のトンネルを設定することが重要です。
```


https://www.atmarkit.co.jp/ait/articles/0011/27/news001_3.html
