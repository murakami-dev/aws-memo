※ピンときたやつしかメモしてないので必要時は本を見ること

## 共有ライブラリ
- 共有ライブラリは、libreadline.so.5のように、「lib～.so～」という名前。
- 共有ライブラリファイルは通常、/libあるいは/usr/libディレクトリに配置
```
[ec2-user@ip-10-7-0-55 ~]$ ldd /bin/cat
	linux-vdso.so.1 (0x00007ffe28954000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f0549ff3000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f054a39e000)
```

- ld.soリンカが共有ライブラリを検索する順序は、環境変数LD_LIBRARY_PATHが優先され、次にキャッシュファイルの/etc/ld.so.cacheになり、それからデフォルトのパスである/libと/usr/libになります。
- EC2の場合、以下のようになった
```
[ec2-user@ip-10-7-0-55 ~]$ export
...省略
declare -x PATH="/home/ec2-user/.local/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/ec2-user/.local/bin:/home/ec2-user/bin"
```
<img width="480" alt="スクリーンショット 2020-10-18 18 28 19" src="https://user-images.githubusercontent.com/60077121/96363494-ba535e80-116f-11eb-822d-d2cee274e421.png">

## パッケージ管理
- Linuxでのパッケージ管理は大きく分けて、Debian形式とRPM形式の2種類。
- Debian形式は、Debian系のディストリビューションで利用され、パッケージ管理作業にはdpkgコマンド、APTツールなどが使われます。
- RPM形式は、RedHat系ディストリビューションを中心に利用されている形式です。パッケージ管理作業にはrpmコマンドが使われます。

<img width="454" alt="スクリーンショット 2020-10-18 18 38 43" src="https://user-images.githubusercontent.com/60077121/96363766-2e423680-1171-11eb-8f7a-0078522a59b5.png">

<img width="473" alt="スクリーンショット 2020-10-18 18 39 45" src="https://user-images.githubusercontent.com/60077121/96363791-52057c80-1171-11eb-9725-4ad6db1a85c9.png">

<img width="458" alt="スクリーンショット 2020-10-18 18 41 07" src="https://user-images.githubusercontent.com/60077121/96363821-82e5b180-1171-11eb-97d9-bb17940b5977.png">

## yum
- YUMの設定は、/etc/yum.confと/etc/yum.repos.dディレクトリ以下のファイルで行います
```
[ec2-user@ip-10-7-0-55 ~]$ cat /etc/yum.conf
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log  #ログファイル
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=3
distroverpkg=system-release
timeout=5
retries=7


#  This is the default, if you make this bigger yum won't see if the metadata
# is newer on the remote and so you'll "gain" the bandwidth of not having to
# download the new metadata and "pay" for it by yum not having correct
# information.
#  It is esp. important, to have correct metadata, for distributions like
# Fedora which don't keep old packages around. If you don't like this checking
# interupting your command line usage, it's much better to have something
# manually check the metadata once an hour (yum-updatesd will do this).
# metadata_expire=90m

# PUT YOUR REPOS HERE OR IN separate files named file.repo
# in /etc/yum.repos.d
```
<img width="455" alt="スクリーンショット 2020-10-18 18 44 35" src="https://user-images.githubusercontent.com/60077121/96363902-10290600-1172-11eb-83ca-7ddf072bffe8.png">
