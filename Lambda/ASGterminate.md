# このLambdaでやりたいこと
- ASGのライフサイクルフックで`Terminating:Wait`になったEC2に対し、EC2内に配置したスクリプトを実行させる

# アーキテクチャ
EventBridge -> Lambda -> RunCommand -> RunPowershellscript

## Lambdaを使う理由
- EventBridge -> RunCommandではインスタンスをタグで指定するしかなく、タグが同じEC2の場合、ターミネートするインスタンスだけにRunCommandすることができない

## EC2 インスタンス削除のライフサイクルアクションのイベントデータ
- [EventBridge による Amazon EC2 Auto Scaling の自動化](https://docs.aws.amazon.com/ja_jp/autoscaling/ec2/userguide/cloud-watch-events.html#cloudwatch-event-types)
- インスタンスIDがイベントデータにあるので、それをキーにできる
```
{
  "version": "0",
  "id": "12345678-1234-1234-1234-123456789012",
  "detail-type": "EC2 Instance-terminate Lifecycle Action",
  "source": "aws.autoscaling",
  "account": "123456789012",
  "time": "yyyy-mm-ddThh:mm:ssZ",
  "region": "us-west-2",
  "resources": [
    "auto-scaling-group-arn"
  ],
  "detail": { 
    "LifecycleActionToken":"87654321-4321-4321-4321-210987654321", 
    "AutoScalingGroupName":"my-asg", 
    "LifecycleHookName":"my-lifecycle-hook", 
    "EC2InstanceId":"i-1234567890abcdef0", 
    "LifecycleTransition":"autoscaling:EC2_INSTANCE_TERMINATING", 
    "NotificationMetadata":"additional-info"
  } 
}
```

# 以下は試行錯誤のメモ
## 1 まずはインスタンスIDを拾いたい
### 1-1 大前提：結果はdictで返さないとダメ
- [Runtime.MarshalError](https://teratail.com/questions/237413)
- `return print()`とかにしてたらエラー出た

### 2 インスタンスIDは拾えた
#### コード
```
import json
import boto3

def lambda_handler(event, context):
    instance_id = event["detail"]["EC2InstanceId"]
    return {
        'result': instance_id
    }
```

#### 結果
```
{
  "result": "i-1234567890abcdef0"
}
```

### 3 boto3でsend_commandメソッド
- これは途中経過のコード
```
import json
import boto3

def lambda_handler(event, context):
    client = boto3.client('ssm')
    instance_id = event["detail"]["EC2InstanceId"]
    
    try:
        response = client.send_command(
            InstanceIds = [instance_id],
            DocumentName = "AWS-RunPowerShellScript",
            Parameters = {
                "commands": [
                    ".\\test-en-IIS.ps1"
                    ],
                "workingDirectory": ["C:\\Users\\hiroya"]
            },
        )
        return response
    except:
        print(instance_id)
```

#### エラー1
- `SyntaxError: (unicode error) 'unicodeescape' codec can't decode bytes in position 2-3: truncated \UXXXXXXXX escape`
- 解決策：コード中のパス`\`を`\\`にしないとダメ

#### エラー2 ~~InstanceIDはリスト型じゃないとダメ~~ strでok
- Lambdaのログでは今は<cls str>だけどvalidなのは<cls list>だよ的なエラーが出る
- CWlogsには`InvalidInstanceId`とでる
- exceptのprint(instance_id)ではCWlogsに`i-073d9f1c864140cd8`と出た。ちゃんととれているようだがstr型のようです。
  - list()をやると[i,-,1,2,***]みたいになってダメだった

### エラー3 splitのエラー（参考：結局split不要）
- `[ERROR] TypeError: descriptor 'split' for 'str' objects doesn't apply to a 'list' object`
- exceptのエラーハンドリングもうまくいかない
  - `print(str.split(instance_id))`のstr.splitがダメなよう。
- 原因：`str.split(instance_id)`ではなく、`instance_id.spilt()`で['instance_id']のリスト型になる。splitの使い方が間違ってた
```
import json
import boto3

def lambda_handler(event, context):
    client = boto3.client('ssm')
    instance_id = str.split(event["detail"]["EC2InstanceId"])
    
    try:
        response = client.send_command(
            InstanceIds = [instance_id],
            DocumentName = "AWS-RunPowerShellScript",
            Parameters = {
                "commands": [
                    ".\\test-en-IIS.ps1"
                    ],
                "workingDirectory": ["C:\\Users\\hiroya"]
            },
        )
        return response
    except:
        print(str.split(instance_id))
```

### エラー4 インスタンスIDはstr型が正しい
- CWlogsには`['i-097e92fae6342af14']`とexceptionの分がでている。インスタンスIDはリスト型に出来ていそうなのになぜ？
- エラー文みたら正しくはstr型らしい。
  - `Invalid type for parameter InstanceIds[0], value: ['i-1234567890abcdef0'], type: <class 'list'>, valid types: <class 'str'>`
#### splitは消した
- `An error occurred (InvalidInstanceId) when calling the SendCommand operation`
- ググるとマネージドインスタンスになっていない、インスタンスIDが違う、インスタンスが実行中ではないなど出てきた。
  - マネージドにはなっているしインスタンスIDも合っている。
  - マネコンでターミネートしてるからダメなのでは？　→　希望容量を下げる方法でやってみる
- **->できた（S3にログ出力された）** けどCWlogsにはエラー`Object of type datetime is not JSON serializable`がでる
```
import json
import boto3
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    client = boto3.client('ssm')
    instance_id = event["detail"]["EC2InstanceId"]
    
    try:
        response = client.send_command(
            InstanceIds=[instance_id],
            DocumentName="AWS-RunPowerShellScript",
            Parameters={
                "commands": [
                    ".\\test-en-IIS.ps1"
                    ],
                "workingDirectory": ["C:\\Users\\hiroya"]
            },
        )
        return { 'result' : response }
        
    except Exception as e:
        print(instance_id)
        logger.error(e)
        raise e
```

### エラー5 `Object of type datetime is not JSON serializable`
- エラーの原因は`return response`していたresponseの中身にdatetimeがあり、それがJSON対応していないこと。
  - >これはJSONに日付（date/datetime）の型が定義されていないことが原因で、そのためjsonモジュールはどのように出力すれば良いのかわからず、エラーとなってしまいます。このエラーに対応する方法です。
  - `send_command`のレスポンスに`'ExpiresAfter': datetime(2015, 1, 1),`（この日付以降は無効）などdatetimeがあった。
    - [boto3のsend_commandのReturns](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ssm.html#SSM.Client.send_command)
- 解決策は実行結果をreturnしないこと。
#### コード
```
import json
import boto3
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    client = boto3.client('ssm')
    instance_id = event["detail"]["EC2InstanceId"]
    
    try:
        client.send_command(
            InstanceIds=[instance_id],
            DocumentName="AWS-RunPowerShellScript",
            Parameters={
                "commands": [
                    ".\\test-en-IIS.ps1"
                    ],
                "workingDirectory": ["C:\\Users\\hiroya"]
            },
        )
        print(instance_id)
        
    except Exception as e:
        print(type(instance_id))
        logger.error(e)
        raise e
```
