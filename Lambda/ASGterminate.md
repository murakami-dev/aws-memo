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

#### エラー InstanceIDはリスト型じゃないとダメ
- 今は<cls str>だけどvalidなのは<cls list>だよ的なエラーが出る
- CWlogsには`InvalidInstanceId`とでる
- list()をやると[i,-,1,2,***]みたいになってダメだった
- exceptのprint(instance_id)ではCWlogsに`i-1234567890abcdef0`と出た。なんで？
