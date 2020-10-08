# イメージビルダー作成時にVPCエンドポイント、プライベートサブネット設定を試す

## VPCエンドポイントの作成
  
https://docs.aws.amazon.com/ja_jp/appstream2/latest/developerguide/creating-streaming-from-interface-vpc-endpoints.html
![image](https://user-images.githubusercontent.com/60077121/94648628-e5dfe780-032d-11eb-8902-9d8f0c40884a.png)

## イメージビルダーインスタンスの作成
- 上記で作成したVPCエンドポイントを指定
- イメージビルダーインスタンスのENIは~~プライベートサブネットに配置。インターネットアクセスは有効化せず、NATGWから外に出るように設定。~~  パブリックサブネットに配置（プライベートに置くのがベストプラクティスだが面倒。以下失敗の理由）。
![image](https://user-images.githubusercontent.com/60077121/94655956-5c82e200-033a-11eb-92bf-cf6621718b54.png)


### PC->イメージビルダーインスタンスに接続（失敗）
そもそもprivate-subnetにENIあるので、SGどうこうの設定ではなく、ENIまでたどり着けない。
![image](https://user-images.githubusercontent.com/60077121/94653423-83d7b000-0336-11eb-9a7f-b4774ac05673.png)  
※PC->踏み台Windows->イメージビルダーインスタンスも無理。　
**結論：イメージビルダーインスタンスをプライベートサブネットに配置するにはClientVPNなど接続元から当該プライベートサブネットにアクセスできる手段ありきのとき。**

- 表
- イメージビルダーについてパターンわけ

| ストリーミングエンドポイント | デフォルトインターネットアクセス | 結果  |
----|----|---- 
| インターネット | チェックあり（IGW） | OK |
| インターネット | チェックなし（NATGW） | ClientVPNなどないとイメージビルダーに接続できない。ユーザーはインターネット経由でフリートに接続可能 |
| VPCエンドポイント | チェックあり（IGW） | イメージビルダーへの接続はできる。ユーザーはインターネット経由でフリートに接続できない |
| VPCエンドポイント | チェックなし（NATGW） | ClientVPNなどないとイメージビルダーに接続できない。ユーザーはインターネット経由でフリートに接続できない |

