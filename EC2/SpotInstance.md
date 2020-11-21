
- 文献
  - [コスト削減に期待！ECS on EC2 でスポットインスタンスの利用を考える](https://dev.classmethod.jp/articles/spotinstances-with-ecs-on-ec2/)
  - [AWS Summit Tokyo 2019 「EC2スポットインスタンスのすべて」](https://pages.awscloud.com/rs/112-TZM-766/images/B1-01.pdf)
  - [Amazon EC2 Auto Scaling によるスポットインスタンス活用講座](https://speakerdeck.com/haruyosh/amazon-ec2-auto-scaling-niyorusupotutoinsutansuhuo-yong-jiang-zuo)
    - 配分戦略`capacity optimized`の良い例あり
  - [Amazon EC2 スポットインスタンスを活用したウェブアプリケーションの構築](https://aws.amazon.com/jp/blogs/news/running-web-applications-on-amazon-ec2-spot-instances/)
    - 配分戦略`lowest-price`のスポットインスタンスプールNについて言及
```
lowest-price : この戦略では、起動時点で最低単価のスポットインスタンスプールから順にインスタンスを起動します。
このとき、安い順に何番目までのスポットプールを使うかを、全体に分散する最低価格のスポットインスタンスプールの数 
(Number of lowest priced Spot Instance pools to diversify across)（N個）に指定します。
このあと、複数のインスタンスタイプの種類を指定するとき、思ったよりも多くのインスタンスタイプが使用されない、という状況を防ぐため、
Nにはある程度大きな値を指定することを推奨します。
```
