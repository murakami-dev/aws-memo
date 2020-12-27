
### 文献（基本は項目ごとに掲載する、ここには未作成の文献をメモ的に載せる）

- [InstanceMetaDataV2を分かりやすく解説してみる](https://blog.serverworks.co.jp/tech/2019/11/27/imdsv2/)
- [Windows EC2インスタンスのハイバネーション(休止)を試してみた](https://dev.classmethod.jp/articles/ec2-windows-support-hibernation/)

# AMI
- AMIに含まれるもの
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-instances-and-amis.html
>AMI に含まれるものとして、ウェブサーバー、関連する静的コンテンツ、動的ページ用のコードが考えられます。この AMI からインスタンスを起動すると、ウェブサーバーが起動し、アプリケーションはリクエストを受け付け可能な状態になります。

- ストレージ情報も含まれる（AMIにはEBSスナップショットが紐づいている）
>AMI の説明に、ルートデバイスのタイプ (ebs または instance store) が明記されています。

- AMIタイプ
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ComponentsAMIs.html#storage-for-the-root-device
  - パブリックか特定のアカウントに起動許可を与えるか
  - Amazon EC2 instance store-backed インスタンスは停止できない（他の違いは上記参照）
  - 課金は秒で計算される
  
## Linux AMI 仮想化タイプ
- HVMが推奨
>2 つの仮想化タイプ (準仮想化 (PV) およびハードウェア仮想マシン (HVM)) のどちらかを使用します。PV AMI と HVM AMI の主な違いは、起動の方法と、パフォーマンス向上のための特別なハードウェア拡張機能 (CPU、ネットワーク、ストレージ) を利用できるかどうかという点です。

## AMI検索方法
- https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/finding-an-ami.html

## 暗号化されたEBS-backed AMIの挙動
- https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/AMIEncryption.html
- もともとのAMIのEBSが暗号化されていれば、起動後も暗号化はされる。
- 下記いろんなパターンがドキュメントにある。

| 元々のAMIに紐づいたスナップショットが暗号化されている | 自己所有のAMIである |起動時に`Encrypted`を指定する | 起動時に`KmsKeyId`を指定する | 結果 |
----|---- |---- |---- |---- 
| 〇 | 〇 | 〇（×でも一緒） | 〇 | `KmsKeyId`で指定したCMKで暗号化される |
| × | 〇 | × | - | 暗号化されない（オーソドックスなパターン）|

## ルートデバイスボリューム
- instance store-backed AMI または Amazon EBS-backed AMI
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/RootDeviceStorage.html
### 永続的ルートボリュームへの変更
- `DeleteOnTermination`（終了に合わせて削除）をfalseにする。
>デフォルトでは、Amazon EBS-backed AMI のルートボリュームは、インスタンスを終了すると削除されます。インスタンスの終了後もボリュームが永続化するように、デフォルトの動作を変更できます。デフォルトの動作を変更するには、ブロックデバイスマッピングを使用して、DeleteOnTermination 属性を false に設定します。

## Amazon Linux
- [Amazon Linuxパッケージリポジトリ](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#package-repository)
- >Amazon Linux はパッケージ管理に RPM と yum を使用しています。これは、新しいアプリケーションをインストールする最も簡単な方法です。
- >Amazon Linux 2 は EPEL リポジトリを使用するように設定されていません。EPEL は、リポジトリのパッケージのほか、サードパーティ製のパッケージを提供します。AWS はサードパーティ製のパッケージをサポートしていません。EPEL レポジトリは、次のコマンドを使って有効にできます。

### リポジトリ
- 保管場所
  - [Amazon Linux 2のyumリポジトリ構造を読み取る](https://dev.classmethod.jp/articles/amazon-linux-2-yum-repository/)
  - [Amazon Linux 2のExtras Library(amazon-linux-extras)を使ってみた](https://dev.classmethod.jp/articles/how-to-work-with-amazon-linux2-amazon-linux-extras/)
  - [あらためてEPELリポジトリの使い方をまとめてみた](https://qiita.com/yamada-hakase/items/fdf9c276b9cae51b3633)

# インスタンスタイプ
## 汎用
- https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/general-purpose-instances.html
- A1
  - AWS Graviton プロセッサ（Armベース）
  - >AWS Graviton プロセッサには 64 ビットの Arm Neoverse コアと AWS が設計したカスタムシリコンが搭載されており、パフォーマンスおよびコストを最適化します。
  - [AWS Graviton プロセッサ](https://aws.amazon.com/jp/ec2/graviton/)
  - [[AWS] Graviton2 プロセッサを搭載した新しいEC2インスタンスタイプ T4g が利用可能になりました](https://dev.classmethod.jp/articles/aws-ec2-t4g-powered-by-graviton2/)
- M5、M5a インスタンス
- M6g インスタンスと M6gd インスタンス
  - AWS Graviton2 プロセッサを搭載
- t2,t3
![image](https://user-images.githubusercontent.com/60077121/103152020-fdacf780-47c6-11eb-9bb5-024ea783a1d2.png)
![image](https://user-images.githubusercontent.com/60077121/103167795-95672000-4871-11eb-924e-0cf82c8e0b21.png)


### バーストパフォーマンス
- ネットワークパフォーマンスで`最大`とつくものはクレジットによるバーストパフォマンスであることを示す
  - つまり`最大10Gbps`ならバースト状態で10Gbpsということ
- t系のインスタンスはCPUパフォーマンスもクレジットによる。
  - ベースライン以下の時にクレジットがたまる
  - CWを使って現在のクレジットの状態をみれる
![image](https://user-images.githubusercontent.com/60077121/103168127-e8da6d80-4873-11eb-926d-98fcac9bf0b8.png)

## インスタンスタイプを変更する

## インスタンスの購入オプション
→特出ししてまとめる

### テナンシー
- ひとつのハードウェアには複数のゲストOSが立つことを思い出そう
- [EC2 インスタンスのテナント属性を理解する](https://dev.classmethod.jp/articles/ec2-tenancy/)
  - 共有ハードウェアインスタンス：デフォルト
  - 専用ハードウェアインスタンス：ハードウェア専有インスタンス。`Dedicated Instance`。Dedicated Instance が稼働しているホスト上には、それを利用している AWS アカウントが起動した別のインスタンスが動作することはあっても、他の AWS アカウントの起動したインスタンスが動作することはない。
  - 専有ホスト：`Dedicated Host`。他の AWS アカウントのインスタンスはもちろん、利用者の AWS アカウントの別のインスタンスが起動されることもありません。

# インスタンスのライフサイクル
## シャットダウン操作
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingInstanceInitiatedShutdownBehavior

## 休止
- [新機能 – EC2インスタンスの休止](https://aws.amazon.com/jp/blogs/news/new-hibernate-your-ec2-instances/)
  - >休止プロセスはそのインスタンスのメモリ状態を保管し、また休止したタイミングで設定されていたプライベートIPアドレスとEIPを保持します。
  - >休止の指示を受け取ったインスタンスは、ルートEBSボリュームに格納されたファイルにメモリ状態を書き出し、（事実上）自分自身をシャットダウンします。休止できるインスタンスのAMI, ルートEBSボリュームは**暗号化されている必要があります。**暗号化することで、メモリの内容がEBSボリュームにコピーされる際に機密データを保護することができます。**インスタンスが休止状態の間、お支払いいただくのはEBSボリュームとアタッチされたEIPの料金だけです。停止状態と同じく、時間当たりのインスタンス料金は発生しません。**

# インスタンスの設定
## ソフトウェアの管理
### ユーザーの管理
- [Amazon Linux インスタンスでユーザーアカウントを管理する](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/managing-users.html)
  - デフォルトのユーザー名はここを参照する
  - RHELは`ec2-user`または`root`
### プロセッサのステート制御
- CPUには一般的にCステートとPステートがあり、Cがアイドル状態でCxの番号が大きくなるほどアイドル状態となる（C0はほぼON）
- >C ステートは、C0 (コアがアクティブで、命令を実行している最も浅い状態) から始まる番号が付けられ、C6 (コアの電源がオフになっている最も深いアイドル状態) まで移行します。P ステートはコアに希望するパフォーマンス (CPU 周波数) を制御します。P ステートは、P0 (コアが Intel Turbo Boost Technology を使用して可能であれば周波数を上げることができる最高パフォーマンスの設定) から始まる番号が付けられ、P1 (最大限のベースライン周波数をリクエストする P ステート) から P15 (最小限の周波数) まで移行します。
- >デフォルトの C ステートおよび P ステートの設定は、ほとんどの作業負荷に対して最適なパフォーマンスを提供します。
- ただし一部のインスタンスタイプではこれをOSから（ユーザー側で）制御できる。
- >AWS Graviton プロセッサには組み込みの省電力モードがあり、固定周波数で動作します。そのため、オペレーティングシステムが C ステートと P ステートを制御する機能は提供されていません。
- https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/processor_state_control.html

### CPUオプション
- [CPU性能について](http://www.fujitsu-webmart.com/pc/webmart/ui2505.jsp#:~:text=%E3%82%B9%E3%83%AC%E3%83%83%E3%83%89%E3%81%A8%E3%81%AF%E3%80%81%E5%90%8C%E6%99%82%E3%81%AB%E5%87%A6%E7%90%86,%E5%8A%B9%E7%8E%87%E3%81%8C%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%81%BE%E3%81%99%E3%80%82&text=%E4%BE%8B%E3%81%88%E3%81%B0%E3%80%814%E3%81%A4%E3%81%AE%E3%82%B3%E3%82%A2%E3%82%92,%E3%81%99%E3%82%8B%E6%A9%9F%E8%83%BD%E3%81%AE%E3%81%93%E3%81%A8%E3%81%A7%E3%81%99%E3%80%82)
- インスタンスタイプごとにCPUコア、スレッド（ほぼ2）が決まっている（vCPU = CPUコア数 x スレッド数）
- これをユーザー側で**起動時のみ**設定できる。ただし減らす方向にのみ
- 追加・割引はない。
- やるメリットないと思う

#### AWS Summit2019
- [Amazon EC2 パフォーマンス Deep Dive](https://pages.awscloud.com/rs/112-TZM-766/images/B1-06.pdf)
  - これめちゃ詳しい。理解したい

## インスタンスメタデータ
- インスタンス内部から当該インスタンスのメタデータを取得できる。
  - [インスタンスメタデータのカテゴリ](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/instancedata-data-categories.html)
- [待望のアプデ EC2インスタンスメタデータサービスv2がリリースされてSSRF脆弱性等への攻撃に対するセキュリティが強化されました！](https://dev.classmethod.jp/articles/ec2-imdsv2-release/)
  - 分かりやすい
  - デフォルトではv1,v2両方メタデータにアクセスできる
  - トークンが必要な設定にするとv1ではアクセスできなくなる
  

## GPU
- [GPUとは？CPUとの違いや性能と活用](https://www.kagoya.jp/howto/rentalserver/gpu1/)

# モニタリング

# ネットワーク

# セキュリティ

# ストレージ

# リソースとタグ



これまとめたい
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-launch-templates.html
