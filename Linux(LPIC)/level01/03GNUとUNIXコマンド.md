<img width="478" alt="スクリーンショット 2020-10-18 19 33 22" src="https://user-images.githubusercontent.com/60077121/96365028-d4de0580-1178-11eb-896f-be712126da52.png">

- echo[文字列または$変数名]
- env
  - 環境変数を表示

- 内部コマンド:シェル自体に組み込まれているもの（cdなど）
- 外部コマンド:独立したプログラムとして存在するもの（lsはbin/lsを実行する外部コマンド）
  - 外部コマンドの場合は、シェルはそのコマンドがどこに置かれているのかを、環境変数PATHに指定されたディレクトリを順に調べて見つけ出します。コマンドが置かれたディレクトリを環境変数PATHに追加することを「パスを通す」といいます。
  - `PATH=$PATH:/opt/bin`は`opt/bin`をPATHに追加している
  - カレントディレクトリにコマンドがある場合はパスが通っていなくても使えるが、**その場合にカレントディレクトリを意味する`./`をいれる**
  
  <img width="459" alt="スクリーンショット 2020-10-18 21 19 25" src="https://user-images.githubusercontent.com/60077121/96367247-a156a780-1187-11eb-89d4-ec1245b18e2e.png">

- クォーテーション（文字列と判断）
- ダブルクォーテーション（文字列だけど変数があれば展開）
```
[ec2-user@ip-10-7-0-55 ~]$ echo "カレントディレクトリは、$(pwd)です。"
カレントディレクトリは、/home/ec2-userです。
```

- メタキャラクタ
<img width="453" alt="スクリーンショット 2020-10-18 21 30 53" src="https://user-images.githubusercontent.com/60077121/96367508-3ad28900-1189-11eb-8bf8-d3c0eacf9509.png">

- パイプ
- 例：dmesgすると出力が画面から溢れるので、lessで一画面ずつ表示。
```
[ec2-user@ip-10-7-0-55 ~]$ dmesg | less
```
- lessの使い方
<img width="449" alt="スクリーンショット 2020-10-18 21 37 03" src="https://user-images.githubusercontent.com/60077121/96367657-1a56fe80-118a-11eb-9390-61e31c8ab264.png">

- tee
  - 標準出力したものをファイルにも書き込む
  - 上書きしないで追記する場合は`-a`
```
[ec2-user@ip-10-7-0-55 ~]$ ls -l | tee test
合計 6720
-rw-rw-r-- 1 ec2-user ec2-user 4990572  8月  9 04:22 Agent-Core-amzn2-20.0.0-877.x86_64.rpm
drwxrwxr-x 2 ec2-user ec2-user     116  8月  2 15:46 custom_folder
drwxrwxr-x 9 ec2-user ec2-user     323  8月  2 15:41 easy-rsa
-rw-rw-r-- 1 ec2-user ec2-user 1884621  8月  1 04:02 get-pip.py
-rw-rw-r-- 1 ec2-user ec2-user       0 10月 18 12:40 test
[ec2-user@ip-10-7-0-55 ~]$ cat test
合計 6720
-rw-rw-r-- 1 ec2-user ec2-user 4990572  8月  9 04:22 Agent-Core-amzn2-20.0.0-877.x86_64.rpm
drwxrwxr-x 2 ec2-user ec2-user     116  8月  2 15:46 custom_folder
drwxrwxr-x 9 ec2-user ec2-user     323  8月  2 15:41 easy-rsa
-rw-rw-r-- 1 ec2-user ec2-user 1884621  8月  1 04:02 get-pip.py
-rw-rw-r-- 1 ec2-user ec2-user       0 10月 18 12:40 test
```

- リダイレクト
- `>`は標準出力せずファイルに書き込む
- `>>`は上書きせずに追記

- grep 文字列 ファイル名（ファイルから特定の文字列を検索して標準出力）
- `grep "lpic" < target.txt > result.txt`はtarget.txtからlpicを検索して標準出力せずに結果をresultに書き込む

- 特定の文字列が現れるまで入力を続けるには、リダイレクト記号の「<<」を使います。これはヒアドキュメントと呼ばれます。
- 次の例では、終了文字として「EOF」を指定し、「EOF」が入力されるまでファイルへの入力を続けます。
  - `$ cat > sample.txt << EOF`
  
<img width="460" alt="スクリーンショット 2020-10-18 21 56 10" src="https://user-images.githubusercontent.com/60077121/96368085-c13c9a00-118c-11eb-8261-d4c072cbc9a5.png">

## エディタ
<img width="467" alt="スクリーンショット 2020-10-18 22 09 39" src="https://user-images.githubusercontent.com/60077121/96368399-a4a16180-118e-11eb-922b-5c7c80eaaa23.png">

<img width="464" alt="スクリーンショット 2020-10-18 22 10 23" src="https://user-images.githubusercontent.com/60077121/96368451-d6b2c380-118e-11eb-8518-8b38c7a5db31.png">

<img width="482" alt="スクリーンショット 2020-10-18 22 10 28" src="https://user-images.githubusercontent.com/60077121/96368492-0a8de900-118f-11eb-9be3-677b8b7d1e0a.png">

<img width="459" alt="スクリーンショット 2020-10-18 22 10 40" src="https://user-images.githubusercontent.com/60077121/96368497-17aad800-118f-11eb-8997-5d4250e7cbe6.png">


