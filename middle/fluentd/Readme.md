# やりたいこと
- fluentdの使い方を学ぶ

# 文献
- [fluentdと定番プラグインのインストール](https://www.atmarkit.co.jp/ait/articles/1403/05/news012_2.html)
  - インプットプラグイン`tail`やアウトプットプラグインを解説
- [fluentdでOSのいろんなログをまとめてS3に出力する設定考えてみた](https://dev.classmethod.jp/articles/fluentd-settings-with-some-os-logs/)]
- [fluentdでapacheのログに時間が出力されなくて苦労したので解決策と原理をまとめてみた](https://dev.classmethod.jp/articles/fluentd-cant-output-time-with-apache-format/)
- [fluentdのbuffer周りで注意すべき点](https://qiita.com/kazuhiro-iwi/items/071100d200b243db9bd1)

# 手順
## まずは初期設定
- タイムゾーン変更
- タイムゾーン変更したらアパッチを再起動して反映しよう`systemctl restart httpd`
## インストール
- AmazonLinux2なら以下
  - `curl -L https://toolbelt.treasuredata.com/sh/install-amazon2-td-agent4.sh | sh`
  - https://docs.fluentd.org/installation/install-by-rpm#step1-install-from-rpm-repository
## 設定ファイルを準備（これはやらない方がいい。confのバックアップとるならcpでやろう）
- >デフォルトでは/etc/td-agent/td-agent.confが読み込まれます。このデフォルトのファイルに設定を追加してもいいのですが、新たに設定ファイルを作って管理する場合は、以下のコマンドを用います。

```
[ec2-user@ip-10-123-10-251 ~]$ td-agent --setup ~/fluent
Installed /home/ec2-user/fluent/fluent.conf.
```

## 設定ファイルを編集
- 詳細は下部とドキュメントを参照
- 今回
```
# add by murakami
<source>
  @type tail
  <parse>
    @type none
  </parse>
  path /var/log/httpd/access_log
  tag httpd_access_log
  pos_file /var/log/td-agent/tmp/access.log.pos
  @id forward_input
</source>

<filter **>
  @type record_transformer
  <record>
    hostname ${hostname}
  </record>
</filter>

<match httpd_access_log>
  @type s3
  s3_bucket ec2-logs-913926916566
  s3_region ap-northeast-1
  path "fluentd-logs/#{Socket.gethostname}"   ##{Socket.gethostname}でホスト名がとれる
  s3_object_key_format %{path}%{time_slice}_%{index}.%{file_extension}
  <format>
    @type json
  </format>
  <buffer time>
    @type file
    timekey 60   #60秒ごとに転送
    timekey_wait 60
    path /var/log/td-agent/s3 # チャンクファイルのたまる場所（多分）
  </buffer>
</match>
```

## fluentdをrootで起動するよう設定（オプション）
- [fluentd(td-agent)をrootで起動させる方法](https://qiita.com/katuemon/items/7105c24271c07ce7b412)
- `vim /usr/lib/systemd/system/td-agent.service`
- 以下をrootに変更
```
[Service]
User=td-agent   ←rootにする
Group=td-agent  ←rootにする
LimitNOFILE=65536
Environment=LD_PRELOAD=/opt/td-agent/lib/libjemalloc.so
Environment=GEM_HOME=/opt/td-agent/lib/ruby/gems/2.7.0/
Environment=GEM_PATH=/opt/td-agent/lib/ruby/gems/2.7.0/
Environment=FLUENT_CONF=/etc/td-agent/td-agent.conf       ←ここを/home/ec2-user/fluent/fluent.confに変えたら良いかと思うけどうまくいかなかった。
Environment=FLUENT_PLUGIN=/etc/td-agent/plugin
Environment=FLUENT_SOCKET=/var/run/td-agent/td-agent.sock
Environment=TD_AGENT_LOG_FILE=/var/log/td-agent/td-agent.log
Environment=TD_AGENT_OPTIONS=
EnvironmentFile=-/etc/sysconfig/td-agent
PIDFile=/var/run/td-agent/td-agent.pid
RuntimeDirectory=td-agent
Type=forking
# XXX: Fix fluentd executables path
ExecStart=/opt/td-agent/bin/fluentd --log $TD_AGENT_LOG_FILE --daemon /var/run/td-agent/td-agent.pid $TD_AGENT_OPTIONS
ExecStop=/bin/kill -TERM ${MAINPID}
ExecReload=/bin/kill -HUP ${MAINPID}
```

## 起動
```
[root@ip-10-123-10-251 td-agent]# sudo systemctl restart td-agent.service
Warning: td-agent.service changed on disk. Run 'systemctl daemon-reload' to reload units.
[root@ip-10-123-10-251 td-agent]# systemctl daemon-reload
[root@ip-10-123-10-251 td-agent]# sudo systemctl restart td-agent.service
[root@ip-10-123-10-251 td-agent]#
```

```
2021-02-22 07:11:14 +0000 [error]: config error file="/etc/td-agent/td-agent.conf" error_class=Fluent::ConfigError error="<parse> section is required."
```


# 設定ファイルなどの解説
## 概要
- https://docs.fluentd.org/configuration/config-file
  - source directives determine the input sources
  - match directives determine the output destinations
  - filter directives determine the event processing pipelines
  - system directives set system-wide configuration
  - label directives group the output and filter for internal routing
  - @include directives include ot

## `source`ディレクティブ
- [Input Plugins](https://docs.fluentd.org/input)
- type…入力プラグインを指定します。tailなど
- path…入力対象のファイルを指定します。
- tag…ログにつけるタグを指定します。
- pos_file…最後に読み込んだファイルの位置をこのファイルに記録します。
  - > pos_fileの設定を行い、前回どこまで読み込んだかを記録しておくことで、fluentdは起動後、前回読み込んだ位置から継続して続きを読み込むことができます。これによりログの収集漏れなく管理することが可能になります。
- format…パーサプラグインを指定します。ここではsyslogを指定しています。

## `match`ディレクティブ
- アウトプットプラグインの中にデフォルトでS3が入っている
  - https://docs.fluentd.org/output/s3

### bufferディレクティブ
- [Config: Buffer Section](https://docs.fluentd.org/configuration/buffer-section)
  - 結局これ見る方が分かりやすい
- [fluentdのbuffer周りで注意すべき点](https://qiita.com/kazuhiro-iwi/items/071100d200b243db9bd1)
- [fluentdメモ - (4) 設定ファイル調査 Buffer編](https://qiita.com/tomotagwork/items/ef51fc60adbb2ca8db92)
- matchの中にある、ファイルの出力頻度を制御するディレクティブ
  - buffer は Chunk の集合
  - Chunk は 1 つの blob に連結されたイベントの集合
  - それぞれの Chunk は file か memory で管理される
  - buffer は Chunk を 2 つの場所に分けて管理
    - stage と呼ばれる Chunk を入力ソース(Event)で埋めていく場所
    - queue と呼ばれる送信前のチャンクが待機している場所
  - 新しく作成された全てのチャンクはstage に入れられ、時間内に queue に移動される (その後転送される)
  - stagedの状態からenqueuedの状態へ移行する処理がflushである
- ![image](https://user-images.githubusercontent.com/60077121/108700622-36a8e480-754a-11eb-97ce-c5015950bd59.png)
- ![image](https://user-images.githubusercontent.com/60077121/108711350-94442d80-7558-11eb-976c-735e80aad5ae.png)

#### Buffer Plugin Type
- memory(デフォルト)とfileがある
```
<buffer>
  @type file
</buffer>
```

#### Chunk Keys
- <buffer xxxxx>の部分のこと。
- チャンクのグループ分けを制御。
- 指定しない場合、すべてのインプットは同じチャンクに入る
```
<buffer ARGUMENT_CHUNK_KEYS>
  # ...
</buffer>
```
  
#### Parameters
- チャンクキーにtimeを使用するとtime関連のparam使える
  - timekey : 指定したx秒ごとにチャンクをフラッシュ
  - timekey_wait : 指定したx秒後にデータを転送

#### Buffering Parameters
chunk_limit_size : チャンクの最大容量。fileの場合256MB

## 初期の`/etc/td-agent/td-agent.conf`
```
####
## Output descriptions:
##

# Treasure Data (http://www.treasure-data.com/) provides cloud based data
# analytics platform, which easily stores and processes data from td-agent.
# FREE plan is also provided.
# @see http://docs.fluentd.org/articles/http-to-td
#
# This section matches events whose tag is td.DATABASE.TABLE
<match td.*.*>
  @type tdlog
  @id output_td
  apikey YOUR_API_KEY

  auto_create_table
  <buffer>
    @type file
    path /var/log/td-agent/buffer/td
  </buffer>

  <secondary>
    @type file
    path /var/log/td-agent/failed_records
  </secondary>
</match>

## match tag=debug.** and dump to console
<match debug.**>
  @type stdout
  @id output_stdout
</match>

####
## Source descriptions:
##

## built-in TCP input
## @see http://docs.fluentd.org/articles/in_forward
<source>
  @type forward
  @id input_forward
</source>

## built-in UNIX socket input
#<source>
#  type unix
#</source>

# HTTP input
# POST http://localhost:8888/<tag>?json=<json>
# POST http://localhost:8888/td.myapp.login?json={"user"%3A"me"}
# @see http://docs.fluentd.org/articles/in_http
<source>
  @type http
  @id input_http
  port 8888
</source>

## live debugging agent
<source>
  @type debug_agent
  @id input_debug_agent
  bind 127.0.0.1
  port 24230
</source>

####
## Examples:
##

## File input
## read apache logs continuously and tags td.apache.access
#<source>
#  @type tail
#  @id input_tail
#  <parse>
#    @type apache2
#  </parse>
#  path /var/log/httpd-access.log
#  tag td.apache.access
#</source>

## File output
## match tag=local.** and write to file
#<match local.**>
#  @type file
#  @id output_file
#  path /var/log/td-agent/access
#</match>

## Forwarding
## match tag=system.** and forward to another td-agent server
#<match system.**>
#  @type forward
#  @id output_system_forward
#
#  <server>
#    host 192.168.0.11
#  </server>
#  # secondary host is optional
#  <secondary>
#    <server>
#      host 192.168.0.12
#    </server>
#  </secondary>
#</match>

## Multiple output
## match tag=td.*.* and output to Treasure Data AND file
#<match td.*.*>
#  @type copy
#  @id output_copy
#  <store>
#    @type tdlog
#    apikey API_KEY
#    auto_create_table
#    <buffer>
#      @type file
#      path /var/log/td-agent/buffer/td
#    </buffer>
#  </store>
#  <store>
#    @type file
#    path /var/log/td-agent/td-%Y-%m-%d/%H.log
#  </store>
#</match>
```




