# 文献
- [AWS Tools for PowerShell Documentation](https://docs.aws.amazon.com/powershell/index.html)
  - リファレンスが辞書的に良い感じ
- [ユーザデータ内のスクリプトが実行されないときに確認する事項](https://awsjp.com/AWS/Faq/c/Script-in-userdata-not-execute-553A.html)

# 実際に走ったユーザーデータを確認する
- [インスタンスユーザーデータの使用](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/instancedata-add-user-data.html)
  - ページ下部のコマンドをインスタンスにログインして実行。ローカル環境から確認するすべもあるよう。
  - **マネコンから見えるユーザーデータはあくまで入力された情報であって実際に走ったものと一致するわけではなさそう**

## ユーザーデータのログデータを確認する
- ` C:\ProgramData\Amazon\EC2Launch\log\userdataexecute.log `だったかな
  - ProgramDataは隠しファイルなので注意
  - https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/WindowsGuide/ec2-windows-user-data.html

### よくあるエラー
- こんなのが出たらsysprepがうまくいっていない?
  - >Sysprep を使用しない場合、メタデータの問題が発生し、メタデータのユーザーデータログに次のエラーが表示される可能性があります。
    - https://aws.amazon.com/jp/premiumsupport/knowledge-center/cloudformation-helper-scripts-windows/
  - [ユーザデータ内のスクリプトが実行されないときに確認する事項](https://awsjp.com/AWS/Faq/c/Script-in-userdata-not-execute-553A.html)
```
2020/12/07 17:17:13Z: Failed to get metadata: The result from http://169.254.169.254/latest/user-data was empty
2020/12/07 17:17:17Z: Unable to execute userdata: Userdata was not provided
```
