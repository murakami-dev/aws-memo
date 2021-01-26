- [lsofコマンド入門](https://qiita.com/hypermkt/items/905139168b0bc5c28ef2)

- 3128で待ち受けているサービスを調べる
```
sudo lsof -i:3128
 (出力例)COMMAND  PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
               squid   7236 squid   11u  IPv6  29959      0t0  TCP *:squid (LISTEN)
```
