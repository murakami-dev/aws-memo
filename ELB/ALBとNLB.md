# 文献
- 1.[ELBをよりよく理解するためにリリースの歴史と各機能面から紐解いた](https://dev.classmethod.jp/articles/elb-history-and-specs/)
  - [HTTP/2の特徴 HTTP/1.1との違いについて](https://blog.redbox.ne.jp/http2-cdn.html)
- 2.[ネットワーク視点で見るAWS ELB(Elastic Load Balancing)のタイプ別比較[NLB対応]](https://dev.classmethod.jp/articles/describe-elb-types/)


# 利用シーン
- ALB
  - HTTP(S)のとき
- NLB
  - HTTP以外、または静的IPがいるとき
  - 特徴
    - SGがない（通信制御は各インスタンスで）
    - クライアントIPがそのまま維持される
    - 戻りトラフィックはインスタンスから直接(帰りはロードバランサを介さず、ターゲットからクライアントに直接通信する形)
      - >NLBのターゲットはクライアントへのネットワーク到達性を考慮する必要があります。Internet Facingのロードバランサであれば、ターゲットを配置するVPCサブネットのルーティングテーブルにIGWないしNAT Gateway、Internalのロードバランサであればルーティングテーブルにピア接続やVGWを設定することになるでしょう。
