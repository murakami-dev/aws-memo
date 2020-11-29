# ECSチュートリアル
- https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/getting-started-ecs-ec2.html
## タスク定義
```
{
    "containerDefinitions": [
        {
            "entryPoint": [
                "sh",
                "-c"
            ],
            "portMappings": [
                {
                    "hostPort": 80,
                    "protocol": "tcp",
                    "containerPort": 80
                }
            ],
            "command": [
                "/bin/sh -c \"echo '<html> <head> <title>Amazon ECS Sample App</title> <style>body {margin-top: 40px; background-color: #333;} </style> </head><body> <div style=color:white;text-align:center> <h1>Amazon ECS Sample App</h1> <h2>Congratulations!</h2> <p>Your application is now running on a container in Amazon ECS.</p> </div></body></html>' >  /usr/local/apache2/htdocs/index.html && httpd-foreground\""
            ],
            "cpu": 10,
            "memory": 300,
            "image": "httpd:2.4",
            "name": "simple-app"
        }
    ],
    "family": "console-sample-app-static"
}
```

## ステップ 2: クラスターを作成する
EC2のインスタンスタイプやストレージ、VPC設定
### 作成したクラスター
```
MacBook-Pro:~ murakamihiroya$ aws ecs describe-clusters --clusters test
{
    "clusters": [
        {
            "clusterArn": "arn:aws:ecs:ap-northeast-1:アカウント番号:cluster/test",
            "clusterName": "test",
            "status": "ACTIVE",
            "registeredContainerInstancesCount": 1,
            "runningTasksCount": 1,
            "pendingTasksCount": 0,
            "activeServicesCount": 1,
            "statistics": [],
            "tags": [],
            "settings": [
                {
                    "name": "containerInsights",
                    "value": "disabled"
                }
            ],
            "capacityProviders": [],
            "defaultCapacityProviderStrategy": []
        }
    ],
    "failures": []
}
```

## ステップ 3: サービスを作成する
- タスク定義やクラスターを指定して紐付ける必要あり
- ELB設定やASG設定もここ
### 作成したサービス
```
MacBook-Pro:~ murakamihiroya$ aws ecs describe-services --cluster test --services test-service
{
    "services": [
        {
            "serviceArn": "arn:aws:ecs:ap-northeast-1:358455993290:service/test/test-service",
            "serviceName": "test-service",
            "clusterArn": "arn:aws:ecs:ap-northeast-1:358455993290:cluster/test",
            "loadBalancers": [],
            "serviceRegistries": [],
            "status": "ACTIVE",
            "desiredCount": 1,
            "runningCount": 1,
            "pendingCount": 0,
            "launchType": "EC2",
            "taskDefinition": "arn:aws:ecs:ap-northeast-1:358455993290:task-definition/console-sample-app-static:1",
            "deploymentConfiguration": {
                "maximumPercent": 200,
                "minimumHealthyPercent": 100
            },
            "deployments": [
                {
                    "id": "ecs-svc/7737401298906576628",
                    "status": "PRIMARY",
                    "taskDefinition": "arn:aws:ecs:ap-northeast-1:358455993290:task-definition/console-sample-app-static:1",
                    "desiredCount": 1,
                    "pendingCount": 0,
                    "runningCount": 1,
                    "createdAt": 1606607602.561,
                    "updatedAt": 1606607617.556,
                    "launchType": "EC2"
                }
            ],
            "events": [
                {
                    "id": "f5c5ee42-61cc-4f30-933b-681b3575925d",
                    "createdAt": 1606607617.564,
                    "message": "(service test-service) has reached a steady state."
                },
                {
                    "id": "4d2ae579-2c84-4e66-837c-3dbff32c7ee5",
                    "createdAt": 1606607617.563,
                    "message": "(service test-service) (deployment ecs-svc/7737401298906576628) deployment completed."
                },
                {
                    "id": "50958417-4ce1-459b-88f0-9c9eaf2df8e4",
                    "createdAt": 1606607608.22,
                    "message": "(service test-service) has started 1 tasks: (task 9e5105c832224347961f717d2c8a5705)."
                }
            ],
            "createdAt": 1606607602.561,
            "placementConstraints": [],
            "placementStrategy": [
                {
                    "type": "spread",
                    "field": "attribute:ecs.availability-zone"
                },
                {
                    "type": "spread",
                    "field": "instanceId"
                }
            ],
            "schedulingStrategy": "REPLICA",
            "createdBy": "arn:aws:iam::358455993290:user/hiroya",
            "enableECSManagedTags": true,
            "propagateTags": "NONE"
        }
    ],
    "failures": []
}
```

## ステップ 4: サービスを表示する
![image](https://user-images.githubusercontent.com/60077121/100528515-2b7f3a80-3221-11eb-8d7a-0263c67a117f.png)
