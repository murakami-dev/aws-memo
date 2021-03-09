# 目的
- ALBにWAFのACLが関連づいているかCLIで調べたい
  - マネコンが信用できないとき

# 文献
- https://awscli.amazonaws.com/v2/documentation/api/latest/reference/wafv2/get-web-acl-for-resource.html

# 手順
### ALBのARNを入れる
```
C:\Users\user>aws wafv2 get-web-acl-for-resource --resource-arn arn:aws:elasticloadbalancing:ap-northeast-1:913926916566:loadbalancer/app/test/xxxxxxxxxxx
{
    "WebACL": {
        "Name": "test",
        "Id": "43ef7340-e5a2-41f0-a053-d72783b71d8d",
        "ARN": "arn:aws:wafv2:ap-northeast-1:913926916566:regional/webacl/test/43ef7340-e5a2-41f0-a053-d72783b71d8d",
        "DefaultAction": {
            "Allow": {}
        },
        "Description": "",
        "Rules": [],
        "VisibilityConfig": {
            "SampledRequestsEnabled": true,
            "CloudWatchMetricsEnabled": true,
            "MetricName": "test"
        },
        "Capacity": 0,
        "ManagedByFirewallManager": false
    }
}
```
#### 何も関連づいていないときはnullが返る（文献より）
```
C:\Users\user>aws wafv2 get-web-acl-for-resource --resource-arn arn:aws:elasticloadbalancing:ap-northeast-1:913926916566:loadbalancer/app/test/xxxxx

C:\Users\user>
```
