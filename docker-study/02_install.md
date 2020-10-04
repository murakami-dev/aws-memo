
```
ubuntu@ip-10-7-30-83:~$ sudo apt-get update
```

- 必要なパッケージのインストール
```
ubuntu@ip-10-7-30-83:~$ sudo apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```
- GPGキー（ファイルが改竄されていないことを確認するために使う）を追加する
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```


```
ubuntu@ip-10-7-30-83:~$ sudo add-apt-repository \
> "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
> $(lsb_release -cs) \
> stable"
```
- 実行結果
```
Hit:1 http://ap-northeast-1.ec2.archive.ubuntu.com/ubuntu focal InRelease
Get:2 http://ap-northeast-1.ec2.archive.ubuntu.com/ubuntu focal-updates InRelease [111 kB]
Get:3 https://download.docker.com/linux/ubuntu focal InRelease [36.2 kB]       
Get:4 http://ap-northeast-1.ec2.archive.ubuntu.com/ubuntu focal-backports InRelease [98.3 kB]
Get:5 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [3684 B]
Hit:6 http://security.ubuntu.com/ubuntu focal-security InRelease
Fetched 250 kB in 1s (473 kB/s)
Reading package lists... Done
```
- アップグレード
```
ubuntu@ip-10-7-30-83:~$ sudo apt-get update
Get:1 http://ap-northeast-1.ec2.archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://ap-northeast-1.ec2.archive.ubuntu.com/ubuntu focal-updates InRelease [111 kB]
Get:3 http://ap-northeast-1.ec2.archive.ubuntu.com/ubuntu focal-backports InRelease [98.3 kB]
Hit:4 https://download.docker.com/linux/ubuntu focal InRelease                 
Hit:5 http://security.ubuntu.com/ubuntu focal-security InRelease           
Fetched 475 kB in 0s (972 kB/s)
Reading package lists... Done
```
- DockerEngine他一式をダウンロード
```
ubuntu@ip-10-7-30-83:~$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

- rootユーザしかdocker使用できないので設定変更
```
sudo gpasswd -a ubuntu docker
```

- 確認
```
docker --version
```
