
## CWagentインストール
- SSMを利用するバターン
- CLIを利用するパターン
- https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-on-EC2-Instance.html  

### SSMを利用
- CloudWatch AgentのWindows版を試す
  - https://dev.classmethod.jp/articles/cloudwatch-agent-windows/
  - RunCommandで`AWS-ConfigureAWSPackage`を選択する
  - Nameには`AmazonCloudWatchAgent`
  - 最後にターゲットと出力から出力結果を確認すること
  - ![image](https://user-images.githubusercontent.com/60077121/96211715-05bf0e80-0fb0-11eb-9065-8b966fa31c2e.png)

  
### CLIを利用
#### Windows(CLIが入っていないので入れること)
- Windowsサーバに入ってインストールする場合は以下
  - https://hacknote.jp/archives/50483/

### 注意
- インストールしてconfigファイルを作成しただけではダメ。起動すること。
    - https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-commandline-fleet.html

- 起動にはPowerShellを管理者として実行して以下コマンドを入力
    - ※WindowsはスペースNGなので”で囲む
    - https://www.itmedia.co.jp/help/tips/windows/w0140.html
```
cd "C:\Program Files\Amazon\AmazonCloudWatchAgent"
& "C:\Program Files\Amazon\AmazonCloudWatchAgent\amazon-cloudwatch-agent-ctl.ps1" -a fetch-config -m ec2 -s -c file:config.json
```

- ※以下のようなエラーは管理者権限で実行していないから
`error: Cannot open・・・`

- ※以下のようなエラーはWindows特有。エージェントの起動には30秒以上かかるがWinではデフォルトで止まるようになっている。これを解消するには以下の手順（再起動含む）が必要。
```
https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/monitoring/troubleshooting-CloudWatch-Agent.html#CloudWatch-Agent-troubleshooting-Windows-start
error: Cannot start service AmazonCloudWatchAgent on computer '.'.
```

## CWagent config編集

### Windows(セッションマネージャーでいける)
```
cd "C:\Program Files\Amazon\AmazonCloudWatchAgent"
.\amazon-cloudwatch-agent-config-wizard.exe
```

- cloudwatchlogsに出力するファイルのパスを聞かれるとき、`C:\inetpub\logs\LogFiles\W3SVC2\**.log`のようにすると、パラメータストアには`"C:\\inetpub\\logs\\LogFiles\\W3SVC2\\**.log"`と登録される


## 参考
```
PS C:\Program Files\Amazon\AmazonCloudWatchAgent> .\amazon-cloudwatch-agent-config-wizard.exe
=============================================================
= Welcome to the AWS CloudWatch Agent Configuration Manager =
=============================================================
On which OS are you planning to use the agent?
1. linux
2. windows
default choice: [2]:

Trying to fetch the default region based on ec2 metadata...
Are you using EC2 or On-Premises hosts?
1. EC2
2. On-Premises
default choice: [1]:

Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]:

Which port do you want StatsD daemon to listen to?
default choice: [8125]

What is the collect interval for StatsD daemon?
1. 10s
2. 30s
3. 60s
default choice: [1]:

What is the aggregation interval for metrics collected by StatsD daemon?
1. Do not aggregate
2. 10s
3. 30s
4. 60s
default choice: [4]:

Do you have any existing CloudWatch Log Agent configuration file to import for migration?
1. yes
2. no
default choice: [2]:

Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]:

Do you want to monitor cpu metrics per core? Additional CloudWatch charges may apply.
1. yes
2. no
default choice: [1]:

Do you want to add ec2 dimensions (ImageId, InstanceId, InstanceType, AutoScalingGroupName) into all of your metrics if the info is available?
1. yes
2. no
default choice: [1]:

Would you like to collect your metrics at high resolution (sub-minute resolution)? This enables sub-minute resolution for all metrics, but you can customize for specific metrics in the output json file.
1. 1s
2. 10s
3. 30s
4. 60s
default choice: [4]:

Which default metrics config do you want?
1. Basic
2. Standard
3. Advanced
4. None
default choice: [1]:

Current config as follows:
{
        "metrics": {
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "LogicalDisk": {
                                "measurement": [
                                        "% Free Space"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "Memory": {
                                "measurement": [
                                        "% Committed Bytes In Use"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 10,
                                "service_address": ":8125"
                        }
                }
        }
}
Are you satisfied with the above config? Note: it can be manually customized after the wizard completes to add additional items.
1. yes
2. no
default choice: [1]:

Do you want to monitor any customized log files?
1. yes
2. no
default choice: [1]:

Log file path:
C:\inetpub\logs\LogFiles\W3SVC2\**.log
Log group name:
default choice: [**.log]
test-daimatsu
Log stream name:
default choice: [{instance_id}]

Do you want to specify any additional log files to monitor?
1. yes
2. no
default choice: [1]:
2
Do you want to monitor any Windows event log?
1. yes
2. no
default choice: [1]:

Windows event log name:
default choice: [System]

Do you want to monitor VERBOSE level events for Windows event log System ?
1. yes
2. no
default choice: [1]:

Do you want to monitor INFORMATION level events for Windows event log System ?
1. yes
2. no
default choice: [1]:

Do you want to monitor WARNING level events for Windows event log System ?
1. yes
2. no
default choice: [1]:

Do you want to monitor ERROR level events for Windows event log System ?
1. yes
2. no
default choice: [1]:

Do you want to monitor CRITICAL level events for Windows event log System ?
1. yes
2. no
default choice: [1]:

Log group name:
default choice: [System]

Log stream name:
default choice: [{instance_id}]

In which format do you want to store windows event to CloudWatch Logs?
1. XML: XML format in Windows Event Viewer
2. Plain Text: Legacy CloudWatch Windows Agent (SSM Plugin) Format
default choice: [1]:

Do you want to specify any additional Windows event log to monitor?
1. yes
2. no
default choice: [1]:
2
Saved config file to config.json successfully.
Current config as follows:
{
        "logs": {
                "logs_collected": {
                        "files": {
                                "collect_list": [
                                        {
                                                "file_path": "C:\\inetpub\\logs\\LogFiles\\W3SVC2\\**.log",
                                                "log_group_name": "test-daimatsu",
                                                "log_stream_name": "{instance_id}"
                                        }
                                ]
                        },
                        "windows_events": {
                                "collect_list": [
                                        {
                                                "event_format": "xml",
                                                "event_levels": [
                                                        "VERBOSE",
                                                        "INFORMATION",
                                                        "WARNING",
                                                        "ERROR",
                                                        "CRITICAL"
                                                ],
                                                "event_name": "System",
                                                "log_group_name": "System",
                                                "log_stream_name": "{instance_id}"
                                        }
                                ]
                        }
                }
        },
        "metrics": {
                "append_dimensions": {
                        "AutoScalingGroupName": "${aws:AutoScalingGroupName}",
                        "ImageId": "${aws:ImageId}",
                        "InstanceId": "${aws:InstanceId}",
                        "InstanceType": "${aws:InstanceType}"
                },
                "metrics_collected": {
                        "LogicalDisk": {
                                "measurement": [
                                        "% Free Space"
                                ],
                                "metrics_collection_interval": 60,
                                "resources": [
                                        "*"
                                ]
                        },
                        "Memory": {
                                "measurement": [
                                        "% Committed Bytes In Use"
                                ],
                                "metrics_collection_interval": 60
                        },
                        "statsd": {
                                "metrics_aggregation_interval": 60,
                                "metrics_collection_interval": 10,
                                "service_address": ":8125"
                        }
                }
        }
}
Please check the above content of the config.
The config file is also located at config.json.
Edit it manually if needed.
Do you want to store the config in the SSM parameter store?
1. yes
2. no
default choice: [1]:

What parameter store name do you want to use to store your config? (Use 'AmazonCloudWatch-' prefix if you use our managed AWS policy)
default choice: [AmazonCloudWatch-windows]
AmazonCloudWatch-test-daimatsu
Trying to fetch the default region based on ec2 metadata...
Which region do you want to store the config in the parameter store?
default choice: [ap-northeast-1]

Which AWS credential should be used to send json config to parameter store?
1. ASIA5JSSKBHLNO4RWVDF(From SDK)
2. Other
default choice: [1]:

Successfully put config to parameter store AmazonCloudWatch-test-daimatsu.
Please press Enter to exit...
```
