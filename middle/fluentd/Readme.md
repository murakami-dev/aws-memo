# やりたいこと
- fluentdの使い方を学ぶ

# 文献
- [fluentdと定番プラグインのインストール](https://www.atmarkit.co.jp/ait/articles/1403/05/news012_2.html)
  - インプットプラグイン`tail`やアウトプットプラグインを解説
- [FluentdでS3にログを収集してみる](https://dev.classmethod.jp/articles/fluentd-s3/)
  - 参考になるけどアウトプットの`time_slice_format`は最新バージョンではbufferディレクティブで設定するみたいなので注意
  - >
# 手順
## インストール
- AmazonLinux2なら以下
  - `curl -L https://toolbelt.treasuredata.com/sh/install-amazon2-td-agent4.sh | sh`
  - https://docs.fluentd.org/installation/install-by-rpm#step1-install-from-rpm-repository
## 設定ファイルを準備（オプション）
- >デフォルトでは/etc/td-agent/td-agent.confが読み込まれます。このデフォルトのファイルに設定を追加してもいいのですが、新たに設定ファイルを作って管理する場合は、以下のコマンドを用います。

```
[ec2-user@ip-10-123-10-251 ~]$ td-agent --setup ~/fluent
Installed /home/ec2-user/fluent/fluent.conf.
```

## 設定ファイルを編集
詳細は下部とドキュメントを参照

## fluentdをrootで起動するよう設定
- [fluentd(td-agent)をrootで起動させる方法](https://qiita.com/katuemon/items/7105c24271c07ce7b412)
- 以下をrootに変更
```
[Service]
User=td-agent   ←rootにする
Group=td-agent  ←rootにする
```

## 起動
```
[root@ip-10-123-10-251 td-agent]# sudo systemctl restart td-agent.service
Warning: td-agent.service changed on disk. Run 'systemctl daemon-reload' to reload units.
[root@ip-10-123-10-251 td-agent]# systemctl daemon-reload
[root@ip-10-123-10-251 td-agent]# sudo systemctl restart td-agent.service
[root@ip-10-123-10-251 td-agent]#
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




