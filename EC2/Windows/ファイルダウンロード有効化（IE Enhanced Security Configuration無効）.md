- [Internet Explorer でファイルのダウンロードを行えるように EC2 Windows インスタンスを設定するにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ec2-windows-file-download-ie/)
>Windows Server 用 Amazon マシンイメージ (AMI) では、デフォルトで Internet Explorer セキュリティ強化設定が有効になります。これは、**サーバー上でのウェブブラウジングはベストプラクティスではないためです。**Internet Explorer セキュリティ強化設定では、Internet Explorer を使用したファイルのダウンロードが無効になります。
インターネットからツールをダウンロードしてインストールする場合は、セキュリティ設定を変更してダウンロードを有効にできます。
注意: EC2 Windows インスタンスでダウンロードを有効にする場合は、信頼できるソースからのみファイルをダウンロードしてください。ソフトウェアのダウンロードとインストール後に Internet Explorer セキュリティ強化の構成を再度有効にすることをお勧めします。
