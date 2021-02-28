# 文献
- [DataSync を使用して、プライベートネットワーク経由で異なるリージョンの Amazon EFS ファイルシステム間でデータを転送するにはどうすればよいですか?](https://aws.amazon.com/jp/premiumsupport/knowledge-center/datasync-transfer-efs-cross-region/)
- [AWS DataSync を使用して VPC を離れることなく、オンプレミスから AWS にファイルを転送する](https://aws.amazon.com/jp/blogs/news/transferring-files-from-on-premises-to-aws-and-back-without-leaving-your-vpc-using-aws-datasync/)
- [AWS DataSync を使用して Amazon FSx for Windows File Server に移行する](https://aws.amazon.com/jp/blogs/news/migrate-to-amazon-fsx-for-windows-file-server-using-aws-datasync/)
  - オンプレからの移行

## 以下はそもそも論。DataSyncを選択するときは？
- [よくある質問 AWS DataSync をいつ選択するか](https://aws.amazon.com/jp/datasync/faqs/)
- [クラウドへのデータの移行](https://aws.amazon.com/jp/cloud-data-migration/)
- [ホワイトペーパーMoving to Managed FileSystems](https://d1.awsstatic.com/whitepapers/moving-to-managed-file-systems.pdf?did=wp_card&trk=wp_card)
  - EFSやFSxへのオンラインデータ移行の第一選択肢
- [re:Invent2019 storage migration to AWS](https://d1.awsstatic.com/events/reinvent/2019/REPEAT_1_Developing_a_game_plan_for_storage_migration_to_AWS_STG222-R1.pdf) 
  - Data migration toolsの比較。以下ブログの元ネタ
    - [AWS DataSyncでデータ移行 ～他のサービスと何が違う？～](https://blog.serverworks.co.jp/tech/2020/02/14/lets-use-aws-data-sync/)
  - StorageGatewayとEFSの比較
  - StorageGatewayとDataSyncのハイブリッド（AWSへのアクセス性が欲しいときはストゲも使おう）
