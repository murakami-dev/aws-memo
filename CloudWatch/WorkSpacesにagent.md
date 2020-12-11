# 文献
- 1[WorkSpacesにCloudWatchAgentを導入してWindowsイベントログを出力してみた](https://dev.classmethod.jp/articles/workspaces-cwa-windows-event-logs/)
- 2[WorkSpacesにCloudWatchAgentを導入してログ転送する方法](https://qiita.com/mo_chiee/items/36b292eed009f16d8b6d)

# 手順
- 基本文献の1でいける
- ただしCWagentのウィザードではオンプレミスを選択すること
- クレデンシャルはシステム環境変数に書く

# 説明
- 文献2にあるように方法は2つある
  - 1.文献1のようにシステム環境編にクレデンシャルを直接書く方法。
    - これならCLIのインストールは不要
  - 2.文献2にあるようにCLIの`config`ファイルにクレデンシャルを置く方法
    - ただし普通`config`ファイルはCドライブにあるがWorkSpaceの場合Dドライブに保持される。**その場合、複製時に消去される**

# 以下はpowershell
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine      Restricted


PS C:\Windows\system32> Set-ExecutionPolicy RemoteSigned -Scope Process

実行ポリシーの変更
実行ポリシーは、信頼されていないスクリプトからの保護に役立ちます。実行ポリシーを変更すると、about_Execution
のヘルプ トピック (http://go.microsoft.com/fwlink/?LinkID=135170)
で説明されているセキュリティ上の危険にさらされる可能性があります。実行ポリシーを変更しますか?
[Y] はい(Y)  [A] すべて続行(A)  [N] いいえ(N)  [L] すべて無視(L)  [S] 中断(S)  [?] ヘルプ (既定値は "N"): y
PS C:\Windows\system32> Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process    RemoteSigned
  CurrentUser       Undefined
 LocalMachine      Restricted


PS C:\Windows\system32> cd "C:\Program Files\Amazon\AmazonCloudWatchAgent\"
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
2
Please make sure the credentials and region set correctly on your hosts.
Refer to http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html
Do you want to turn on StatsD daemon?
1. yes
2. no
default choice: [1]:
2
Do you have any existing CloudWatch Log Agent configuration file to import for migration?
1. yes
2. no
default choice: [2]:

Do you want to monitor any host metrics? e.g. CPU, memory, etc.
1. yes
2. no
default choice: [1]:
2
Do you want to monitor any customized log files?
1. yes
2. no
default choice: [1]:

Log file path:
C:\Windows\System32\winevt\Logs\Security.evtx
Log group name:
default choice: [Security.evtx]

Log stream name:
default choice: [{hostname}]

Do you want to specify any additional log files to monitor?
1. yes
2. no
default choice: [1]:

Log file path:
C:\Windows\System32\winevt\Logs\System.evtx
Log group name:
default choice: [System.evtx]

Log stream name:
default choice: [{hostname}]

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
default choice: [{hostname}]

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
                                                "file_path": "C:\\Windows\\System32\\winevt\\Logs\\Security
                                                "log_group_name": "Security.evtx",
                                                "log_stream_name": "{hostname}"
                                        },
                                        {
                                                "file_path": "C:\\Windows\\System32\\winevt\\Logs\\System.e
                                                "log_group_name": "System.evtx",
                                                "log_stream_name": "{hostname}"
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
                                                "log_stream_name": "{hostname}"
                                        }
                                ]
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
1
What parameter store name do you want to use to store your config? (Use 'AmazonCloudWatch-' prefix if you u
d AWS policy)
default choice: [AmazonCloudWatch-windows]
AmazonCloudWatch-WSAMZN-0E9A53BQ
Which region do you want to store the config in the parameter store?
default choice: [ap-northeast-1]

Which AWS credential should be used to send json config to parameter store?
1. AKIA5JSSKBHLPUNYCN6C(From SDK)
2. Other
default choice: [1]:

Successfully put config to parameter store AmazonCloudWatch-WSAMZN-0E9A53BQ.
Please press Enter to exit...

Program exits now.
PS C:\Program Files\Amazon\AmazonCloudWatchAgent> .\amazon-cloudwatch-agent-ctl.ps1 -a fetch-config -c file
Successfully fetched the config and saved in C:\ProgramData\Amazon\AmazonCloudWatchAgent\Configs\file_confi
Start configuration validation...
2020/12/12 01:37:45 Reading json config file path: C:\ProgramData\Amazon\AmazonCloudWatchAgent\Configs\file
tmp ...
Valid Json input schema.
No csm configuration found.
No metric configuration found.
Configuration validation first phase succeeded
Configuration validation second phase succeeded
Configuration validation succeeded
PS C:\Program Files\Amazon\AmazonCloudWatchAgent> .\amazon-cloudwatch-agent-ctl.ps1 -a status
{
  "status": "stopped",
  "starttime": "",
  "version": "1.247346.1b249759"
}
PS C:\Program Files\Amazon\AmazonCloudWatchAgent> .\amazon-cloudwatch-agent-ctl.ps1 -a start
PS C:\Program Files\Amazon\AmazonCloudWatchAgent> .\amazon-cloudwatch-agent-ctl.ps1 -a status
{
  "status": "running",
  "starttime": "2020-12-12T01:38:38",
  "version": "1.247346.1b249759"
}
PS C:\Program Files\Amazon\AmazonCloudWatchAgent>
```
