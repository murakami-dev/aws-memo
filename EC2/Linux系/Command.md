
# zip化
```
zip -r [zip後の名前(.zip不要)] /path/zip化したいフォルダ
```
- カレントディレクトリのすべてのファイルをサブディレクトリも含めてeasy-rsa.zipとして保存。
  - >サブディレクトリも含めて圧縮したい場合は、「-r」オプションを指定します。例えば、カレントディレクトリのファイルをサブディレクトリ下のファイルも含めて圧縮したい場合は、「zip -r ZIPファイル名 *」のように指定します
  - https://www.atmarkit.co.jp/ait/articles/1607/25/news021.html
```
[ec2-user@radius easy-rsa]$ zip -r easy-rsa *
```
