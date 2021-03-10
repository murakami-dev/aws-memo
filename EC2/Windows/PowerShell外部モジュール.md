
# やりたいこと
- 外部モジュールのインポート

# 文献
- [PowerShell のモジュール詳解とモジュールへのコマンドレット配置手法を考える](https://tech.guitarrapc.com/entry/2013/12/03/014013)
- [PowerShell モジュールをインストールする](https://docs.microsoft.com/ja-jp/powershell/scripting/developer/module/installing-a-powershell-module?view=powershell-7.1)
  - 公式
- [PowerShellで、モジュールとしてfunctionを外部ファイルに定義する](http://taeisheauton4programming.blogspot.com/2018/07/powershellfunction.html)
  - モジュールのインポートはこれを参考にした
- [関数を PowerShell プロンプトで実行する](https://www.vwnet.jp/Windows/PowerShell/2016100401/UseFunctionInPsPrompt.htm) 
  - モジュールとして使用する以外にも方法はありそう

# インポートするモジュール
- https://tech.guitarrapc.com/entry/2013/10/07/083837

# 手順
- `%ProgramFiles%\WindowsPowerShell\Modules`（全ユーザが参照するディレクトリ）に`New-ZipCompression`フォルダ作成。フォルダ名は関数名と同じじゃないとダメ。
- 上記フォルダの中に`New-ZipCompression.psm1`を作成し、上記ページ内のfunctionをコピペ。
- PowerShellで`Import-Module New-ZipCompression`して`Get-Module -Name New-ZipCompression`

![image](https://user-images.githubusercontent.com/60077121/110041635-bffcba00-7d87-11eb-9623-40bb75e936d0.png)

### コマンドが使えるようになっているか確認
```
PS C:\Users\hiroya> Get-Command New-ZipCompression

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        New-ZipCompression                                 0.0        New-ZipCompression


PS C:\Users\hiroya> Get-Command New-ZipCompress

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        New-ZipCompress                                    0.0        PS-Zip
```



# 以下は旧内容
# インポートするモジュール
- https://github.com/guitarrapc/PowerShellUtil/tree/master/PS-Zip

# 手順
## モジュールの配置
- GitHubのモジュールをgit cloneして対象サーバのPSModulePath 環境変数のディレクトリに保存
  - https://docs.microsoft.com/ja-jp/powershell/scripting/developer/module/installing-a-powershell-module?view=powershell-7.1
- 今回は`%ProgramFiles%\WindowsPowerShell\Modules`に配置した（全ユーザが参照するディレクトリ）

## インポート
- 配置しただけで使えるようになるらしいけど、使えなかった場合は明示的に`Import-Module`すること（文献1）
```
PS C:\Users\hiroya> Import-Module PS-Zip -PassThru -Verbose
VERBOSE: Loading module from path 'C:\Users\hiroya\Documents\WindowsPowerShell\Modules\PS-Zip\PS-Zip.psm1'.
VERBOSE: Importing function 'New-ZipCompress'.
VERBOSE: Importing function 'New-ZipExtract'.

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        PS-Zip                              {New-ZipCompress, New-ZipExtract}
```

![image](https://user-images.githubusercontent.com/60077121/109170980-a3c7ae80-77c4-11eb-8f35-0ce9fa46e5f8.png)
