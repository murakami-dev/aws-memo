# 文献
- [EC2 Windows ServerでのRemote Desktop Service構成パターン](https://dev.classmethod.jp/articles/ec2-windows-remote-desktop-patterns/)
  - >現在サポートされているWindows Serverは、デフォルトの状態では「管理用途に限り2セッションまでRemote Desktopを同時使用できる」様になっています。3人以上のユーザーでWindows ServerへRemote Desktop接続してデスクトップ環境を利用したい場合は追加でRemote Desktop Service CAL(RDS CAL)もしくはRemote Desktop Service SAL(RDS SAL)を購入し、Windows Serverに所定の設定を施す必要があります。
  - >RDS CALは端的に言ってしまうとオンプレ環境専用のライセンスでありAWSをはじめとしたクラウド環境(＝SPLA契約下)では使用できません。クラウド環境ではCALに変わるライセンスとしてSAL(Subscriber Access License)が提供されておりRDS CALの代わりにRDS SALが必要となります。
  - >ワークグループ環境ではDevice RDS CALしか使用できない制約があります。RDS SALはユーザーライセンスのみの提供ですのでAWS環境でRemote Desktop Serviceを使いたい場合はActive Directory環境であることが事実上必須となりますのでご注意ください。
  
  
  
