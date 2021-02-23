# 文献
- [ファイアウォールとWAF、IPSの違いは？初心者にもわかりやすく解説](https://www.wafcharm.com/blog/difference-between-firewall-waf-ips/)
  - >・Firewall：通信をIPアドレスやポート番号で見て通過・遮断を判断する
  - >・WAF：Webアプリケーション層の通信の中身を見て通過・遮断を判断する
  - >・IPS：OSやネットワークを行き来する通信を監視し、不正な通信や変更を防御する
    - >似たようなシステムにIDS（Intrusion Detection System）というものがあり、通信を監視して不正な通信や攻撃と思われる通信がないかを随時監視します。このIDSの場合は問題のありそうな通信を検知（Detection）するだけですが、さらに一歩踏み込んで防止（Prevention）まで行うのがIPSです。
    - >・NIPS（Network-based IPS）：ネットワーク型IPS
      - >→ネットワークの境界に設置され、そこを通過する通信を監視・防御の対象とする
    - >・HIPS（Host-based IPS）：ホスト型IPS
      - >→サーバ等のコンピュータに配置され、そのサーバが送受信する通信を監視・防御の対象とする
    
    
    
