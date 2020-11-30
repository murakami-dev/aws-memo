# コンテナって?
- [基礎から応用までじっくり学ぶECS Fargateを利用したコンテナ環境構築](https://dev.classmethod.jp/articles/developers-io-2020-connect-kaji-ecs-fargate/)
# ECS,EKSの存在意義
手作業でのコンテナ操作の非効率性の解消
- ECS,EKS:コンテナオーケストレーションサービス
- EC2,Fargate:実行環境
- ECR:レポジトリ
![image](https://user-images.githubusercontent.com/60077121/100527891-52863e00-321a-11eb-90fe-d301ebb06105.png)

# タスク定義とサービスの概要
- 各コンテナはタスク定義を参照する
　　- クラスターはタスク定義を参照してタスクを実行する（**タスク定義に基づいてインスタンス化される**）
- サービスはタスクをコピーして維持する
- **クラスター内では複数のタスクが実行される。そのタスク実行のインフラ管理戦略がキャパシティプロバイダー。**
![image](https://user-images.githubusercontent.com/60077121/100529951-9fc0da80-322f-11eb-9673-11d073402a32.png)


# ECSクラスター
- https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/clusters.html
- >Amazon ECS クラスターは、タスクまたはサービスの論理グループです。 EC2 を使用してタスクまたはサービスを実行している場合、クラスターはコンテナインスタンスのグループ化でもあります。 キャパシティープロバイダーを使用している場合、クラスターはキャパシティープロバイダーの論理的なグループでもあります。

## クラスターの作成
- マネコンで設定できるのは一般的なもののみ
  - >たとえば、このウィザードを使用してコンテナインスタンス AMI ID をカスタマイズすることはできません。このウィザードでサポートされない要件がある場合には、https://github.com/awslabs/ecs-refarch-cloudformation のリファレンスアーキテクチャの使用を検討します。
- 選択できるのは3つ
  - ネットワークのみ（Fargate用）
  - Linux＋ネットワーク
  - Windows+ネットワーク
- インスタンスタイプやストレージ、VPC、SG、キーペア、オンデマンドorスポット　など設定

### キャパシティプロバイダー
- ドキュメント読んでもわからないので以下。
  - [Capacity Providerとは？ECSの次世代スケーリング戦略を解説する](https://dev.classmethod.jp/articles/regrwoth-capacity-provider/)
  - >Capacity Providerとは「ECSにおけるタスク実行のインフラをより柔軟に設定する新しい仕組み」です。
  - ![image](https://user-images.githubusercontent.com/60077121/100529589-4e631c00-322c-11eb-9ec9-1f0dba6756f3.png)

### クラスターの Auto Scaling

# タスク定義パラメータ
## 必須
- family:名前のようなもの。これとリビジョン（バージョンみたいな）でタスク定義が特定される
- コンテナ定義 ( containerDefinitions )
  - name
    - タスク定義で複数のコンテナをリンクする時はこの`name`で指定する
  - image:使用するコンテナのイメージ
    - >Docker Hub レジストリのイメージはデフォルトで使用できます。
  - memory
    - >コンテナに適用されるメモリの量 (MiB 単位)。コンテナは、ここで指定したメモリを超えようとすると、強制終了されます。タスク内のすべてのコンテナ用に予約されるメモリの合計量は、タスクの memory 値より小さくする必要があります (指定されている場合)。
  - memoryReservation
    - >コンテナ用に予約するメモリのソフト制限 (MiB 単位)。システムメモリが競合している場合、Docker はコンテナメモリをこのソフト制限に維持しようとします。

## オプション（詳細パラメータ）
- logConfiguration 
  - 作成中ブログみたい
- environment:平文。機密情報には向かない
- secrets：機密情報はこちら
- networkMode
  - 作成中。Dockerを学んでから
- dependsOn


# ログ
- [ECS/Fargateでログ出力先をカスタマイズできる「FireLens」機能がリリースされました](https://dev.classmethod.jp/articles/ecs-firelens/)
  - logConfiguration にログ出力の設定を記述します。logDriver に awslogs を指定することで、ログ出力先をCloudWatch Logsに設定することができます。
