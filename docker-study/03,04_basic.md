- コマンド・オプションはp.83,90

- 起動
  - `--name`はコンテナ名。重複NG。`--docker rm`で削除する必要あり。
  - `httpd`はイメージ名。`2.4`は制作者のつけるタグ。特に指定しなければ`latest`になる。
  - `-p 8080:80`は設定必須。ホスト（EC2）の8080番をコンテナの80番につなげるという意味。ブラウザ->host->コンテナなのでブラウザで検索する際は`:8080`になる。
  - `-v`はマウント。ホストのカレントディレクトリ`$PWD`(環境変数)をコンテナの`/usr/local/apache2/htdocs/`にマウントしている
  - `-d`はデタッチモード。これを指定することでコンテナはバックグラウンドで動作してくれる。デタッチなのでPC入力はコンテナではなくホストに対して有効になる。
  - `-i`はインタラクティブモード。標準入出力とエラー出力をコンテナに渡してくれる。
  - `-t`で疑似端末を割り当てる。ctrlキーなどの効果がコンテナにも反映される。
```
docker run -dit --name my-apache-app -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4
```

  - `docker run`は以下の操作と同義
```
docker pull httpd:2.4 #dockerイメージhttpdのタグ2.4をホスト内にDL
docker create --name my-apache-app -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4 #dockerコンテナを作成
docker start my-apache-app
```

- 確認
- 
```
docker ps
```

- p.71　3-4 index.htmlを作る
  - nanoでの編集
```
nano index.html
```

- p.74　3-5 コンテナの停止と再開
  - コンテナの停止
  - `docker stop コンテナID`でもOK
```
docker stop my-apache-app
```

- コンテナの再開
```
docker start my-apache-app
```

- p.76　3-6 ログの確認
  - ログの確認。ここではApacheのログが表示される（これはコンテナ内の設定ファイルで定義されている）
```
docker logs my-apache-app
```

- p.76　3-7 コンテナの破棄
  - コンテナの破棄と破棄後の確認
```
docker stop my-apache-app
docker rm my-apache-app
docker ps -a
```

- p.77　3-8 イメージの破棄
  - イメージの確認
```
docker image ls
```

- イメージの破棄
```
docker image rm httpd:2.4
```

- p.101　デタッチとアタッチの切り替えを試す
  - `-d`を指定せずに実行すると、アタッチモード（フォアグラウンドで動作）で起動
  - ログが表示される。デタッチするにはctrl+p,ctrl+q
```
docker run -it --name my-apache-app -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4
```
- アタッチするには以下。
```
docker attach my-apache-app
```

- p.105　停止中のコンテナでシェルを実行する（あまり使わないやり方、-dを使用しないパターン）
  - dockerコンテナ内でシェル操作が可能になる
  - ctrl+p,ctrl+qまたは`exit`でデタッチして元のホストに戻れる
  - `docker run`で起動したものを`exit`するとコンテナの状態は`Exited`になる。
```
ubuntu@ip-10-7-30-83:~$ docker run --name my-apache-app -it httpd:2.4 /bin/bash
root@5f1882aeacc9:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs	modules
```

- p.107　実行中のコンテナでシェルを実行する
  - 次は`-d`を実行して稼働中
```
docker run --name my-apache-app -dit -p 8080:80 -v "$PWD":/usr/local/apache2/htdocs/ httpd:2.4
```
  - `docker ps`で確認するとステータスはアップになっている。

- `docker exec`でシェル起動。
  - 稼働中のコンテナの場合はこれ。exitしても稼働中のまま
```
docker exec -it my-apache-app /bin/bash
```

- p.110　Go言語をコンパイルする
```
docker run --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp golang:1.13 go build -v

ls

./myapp

docker ps -a
```


