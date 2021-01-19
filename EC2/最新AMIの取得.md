# 文献
- 1.[AWS CLIで、最新のAMIの情報を取得する](https://qiita.com/charon/items/18d9d582ea6c8960f274)
  - OverView的な記事
- 2.[CentOS7の最新AMI IDをAWS CLIで取得する方法](https://dev.classmethod.jp/articles/get_latest_centos_ami_id/)
  - CentOSのWikiはメンテされていない。古い
  - CentOSのWikiからプロダクトコードを参照し、CLIで検索しよう
- 3.[Red Hat 社から公式に提供されている Red Hat Enterprise Linux (RHEL) の AMI の ID の見つけ方を教えてください](https://dev.classmethod.jp/articles/tsnote-support-ec2-linux-rhel-001/)

# 手順（作成中）
- やってることはCLIコマンドの`aws ec2 describe-images`
- コマンドのオプションで絞っていく。肝はオプションの使い方だけ
