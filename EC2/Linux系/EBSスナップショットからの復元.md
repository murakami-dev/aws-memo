# 目的
- 空っぽのEC2を複数起動した状態
- EC2Aにミドルウェアインストールして、EBSスナップショットを取得し、EC2 Bにアタッチしたい。
  - EC2 Bでもミドルウェア入れるのめんどくさいから

# 文献
- [以前のスナップショットを使用した Amazon EBS ボリュームの置き換え](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ebs-restoring-volume.html)

# 手順（ルートボリューム置き換え）
Linux2で実施
- EC2 Aのボリュームからスナップショット作成
- スナップショットからボリューム作成
- **EC2 BのボリュームIDとマウント情報をメモ**
  - >ボリュームページで、置き換えるボリュームのチェックボックスをオンにします。[説明] タブで [アタッチ済み情報] を探し、 ボリュームのデバイス名 (ルートボリュームの場合は、/dev/sda1 または /dev/xvda、あるいは /dev/sdb または xvdb など) とインスタンスの ID を書き留めます。
```
こんな感じ
i-02fbb4a0b68dc095c (linux-child):/dev/xvda (attached)
vol-08947b202733517f1
```
- EC2 B停止
- EC2 Bのボリュームをデタッチ

## 新ボリュームをアタッチ
### このときメモしたように`/dev/xvda`のデバイスを指定。
- ![image](https://user-images.githubusercontent.com/60077121/111425366-6e184480-8736-11eb-8d68-1a0b18529d2f.png)
#### こうすることでルートデバイスに勝手にアタッチされる（マウント不要）
- ![image](https://user-images.githubusercontent.com/60077121/111426105-791fa480-8737-11eb-8fcd-e9194ec3f363.png)
- すでにマウントされている
- ![image](https://user-images.githubusercontent.com/60077121/111426259-bc7a1300-8737-11eb-8a86-35f73eccebef.png)

# 秘密鍵が異なるEC2のルートボリューム置き換えたらどうなるか
## EC2 Aを起動
- 秘密鍵はadditional-task.pem
## EC2 Bを起動
- 秘密鍵はmurakami.pem
## EC2 BのルートボリュームをEC2 Aのものに置き換え
- その後EC2 Bの`authorized_keys`見ると、どっちの公開鍵もあった
- ![image](https://user-images.githubusercontent.com/60077121/111429165-f3eabe80-873b-11eb-92b9-aff6f3da9ce4.png)




