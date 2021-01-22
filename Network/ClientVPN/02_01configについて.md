# Configファイルの書き方について
- [Reference manual for OpenVPN 2.4](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/)

# 前提
- 相互認証かつAD認証で検証する
- `config`
```
client
dev tun
proto udp
remote asdfa.cvpn-endpoint-0b3****c0cf7.prod.clientvpn.ap-northeast-1.amazonaws.com 443
remote-random-hostname
resolv-retry infinite
nobind
remote-cert-tls server
cipher AES-256-GCM
verb 3
<ca>
-----BEGIN CERTIFICATE-----
MIIDPjCCAiagAwIBAgIJAIOA3Fa9S6fjMA0GCSqGSIb3DQEBCwUAMBkxFzAVBgNV
*****中略******
c4FjN1n17i2CYgGLKMlI1bUf
-----END CERTIFICATE-----

</ca>
auth-user-pass

reneg-sec 0

cert C:\\Users\\user\\OpenVPN\\config\\***\\common-client.domain.tld.crt
key C:\\Users\\user\\OpenVPN\\config\\***\\common-client.domain.tld.key
```

# ovpnファイルに記載するオプション
## `-inactive n x`
- n秒間経ったら切断
- xはオプション。合計入力/出力トラフィックxバイト未満の通信が行われたら切断（xバイト以上の通信をしていたらn秒経っても切断しない）

> –inactive n [bytes]
> Causes OpenVPN to exit after n seconds of inactivity on the TUN/TAP device. The time length of inactivity is measured since the last incoming or outgoing tunnel packet. The default value is 0 seconds, which disables this feature.If the optional bytes parameter is included, exit if less than bytes of combined in/out traffic are produced on the tun/tap device in n seconds.
> In any case, OpenVPN’s internal ping packets (which are just keepalives) and TLS control packets are not considered “activity”, nor are they counted as traffic, as they are used internally by OpenVPN and are not an indication of actual user activity.

### ClientVPNのログで調査できる
- ingress-bytes — 接続の受信 (インバウンド) バイト数。この値は、ログ内で定期的に更新されます。
- egress-bytes — 接続の送信 (アウトバウンド) バイト数。この値は、ログ内で定期的に更新されます。
```
{
 "connection-log-type": "connection-attempt",
 "connection-attempt-status": "successful",
 "connection-reset-status": "NA",
 "connection-attempt-failure-reason": "NA",
 "connection-id": "cvpn-connection-abc123abc123abc12",
 "client-vpn-endpoint-id": "cvpn-endpoint-aaa111bbb222ccc33",
 "transport-protocol": "udp",
 "connection-start-time": "2020-03-26 20:37:15",       <---接続開始時刻
 "connection-last-update-time": "2020-03-26 20:37:15",
 "client-ip": "10.0.1.2",
 "common-name": "client1",
 "device-type": "mac",
 "device-ip": "98.247.202.82",
 "port": "50096",
 "ingress-bytes": "0",   <---受信バイト数
 "egress-bytes": "0",    <---送信バイト数
 "ingress-packets": "0",
 "egress-packets": "0",
 "connection-end-time": "NA"   <---接続終了時刻
}
```
