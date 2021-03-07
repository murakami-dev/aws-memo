# 目的
- ハード、OSなどの関係性を知りたい

# 全体像
- ![image](https://user-images.githubusercontent.com/60077121/110229521-68a84680-7f4d-11eb-815e-fa85f6835945.png)

# 連載より
- [RDBMSでも大規模データをあきらめないためには](https://gihyo.jp/admin/serial/01/rdbms/0001)
- 

# 書籍より（絵で見てわかる）
## OS（劇場）
- ハード（CPUやメモリ）を操るためのツール
- アプリやDBMSのOSにおける実行単位はプロセスとスレッド
  - プロセスは舞台
  - スレッドは役者。プロセスの中に何人でもいれる。
  - プロセス（スレッド）はキューに入れられ、メモリを割り当てられCPU上で処理される

- ![image](https://user-images.githubusercontent.com/60077121/110229783-43b4d300-7f4f-11eb-9713-6b63ed98f996.png)
## vmstatの見方

## CPU使用率の内訳
- >CPUの使用率には、3つの状態（SYS、USER、IDLE）があります。SYS（sy）は、カーネルが使用した時間の割合を示しています。Windowsでは、「PrivilegedTime」と表示されます。
- >IDLE（id）です。これは、CPU上で処理されているものがないことを指します。

### CPU使用率の考え方
- 使用率が高い、すなわち遅いではない。使用率が高い＝待ち行列発生率が高い。待ちが発生して初めてCPUが原因で遅いと言える。
- CPU待ち（runqueue、ProcessorQueueLength）が高くなければ、CPU使用率が高くても実害はほとんどないことが理解いただけたでしょう。
- ![image](https://user-images.githubusercontent.com/60077121/110229962-9ba00980-7f50-11eb-8edb-d1edc84d4c5a.png)

### I/O待ちが発生するとCPUを使い切れない
- ![image](https://user-images.githubusercontent.com/60077121/110230063-7d86d900-7f51-11eb-9eed-49e3eaaa6f75.png)

### プロセスの優先度はOS任せでOK
- psコマンドで優先度見れる。niceやreniceで調整しなくてもOK

### CPU使用率が高い時にDBやアプリでできること
- CPU使用率が高いのはメモリからCPUに大量のデータを処理させているから
- ディスク（ストレージ）から読み込むデータが多いとI/Oの増大にもつながる
- ![image](https://user-images.githubusercontent.com/60077121/110230270-2124b900-7f53-11eb-9684-bd6185f0d61e.png)

- DBならインデックスをうまく活用して、「アクセスするブロック数を減らす」つまりデータ量を減らすのが大事
- ![image](https://user-images.githubusercontent.com/60077121/110230323-91333f00-7f53-11eb-8cab-19a19f119302.png)

## マルチコアが活きるのはCPU待ちが発生している時
- CPU使用率がピッタリ100%の時にコアを2つにしても50%になるだけで速さは不変

### 性能指標はレスポンスタイムとスループット（CPUの処理能力）で評価する
- https://codezine.jp/article/detail/9614

### 仮想メモリやスワップ領域を使うと時間かかる
- ページングすると時間かかる
- ![image](https://user-images.githubusercontent.com/60077121/110231200-7f549a80-7f59-11eb-8fe7-262ffee3268b.png)

### ページング情報の見方
- 詳細見るなら`vmstat`要約見るなら`free`
- ![image](https://user-images.githubusercontent.com/60077121/110231252-f853f200-7f59-11eb-8479-08463a827725.png)

## OS性能を見るためのコマンド
- `vmstat`
- `iostat`
- `netstat`
- `top`
- `ps`

- ![image](https://user-images.githubusercontent.com/60077121/110231226-c0e54580-7f59-11eb-954d-92994b5855db.png)

## ストレージ
### ストレージのインターフェイス（接続規格）（I/Oの部分）
- ATA
- SATA(シリアルATA)
  - 伝送をシリアル（一つ）にしたことで転送速度アップ。ATAの伝送はパラレルだった
- SCSI,iSCSI（スカジー）：サーバに使用される。IO中でもバスを手放せるのでプロセスの多いサーバ向き。
