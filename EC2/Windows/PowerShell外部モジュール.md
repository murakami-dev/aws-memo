
# やりたいこと
- 外部モジュールのインポート

# 文献
- [PowerShell のモジュール詳解とモジュールへのコマンドレット配置手法を考える](https://tech.guitarrapc.com/entry/2013/12/03/014013)
- [PowerShell モジュールをインストールする](https://docs.microsoft.com/ja-jp/powershell/scripting/developer/module/installing-a-powershell-module?view=powershell-7.1)

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
