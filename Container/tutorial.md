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
### 作成したクラスターインスタンス
```
MacBook-Pro:~ murakamihiroya$ aws ecs describe-container-instances \
>     --cluster test \
>     --container-instances c7b9537c6a534e26b1dd17d3d9e13a4c
{
    "containerInstances": [
        {
            "containerInstanceArn": "arn:aws:ecs:ap-northeast-1:358455993290:container-instance/test/c7b9537c6a534e26b1dd17d3d9e13a4c",
            "ec2InstanceId": "i-0d6f8d9a4508ea6f1",
            "version": 6,
            "versionInfo": {
                "agentVersion": "1.48.1",
                "agentHash": "e9b600d2",
                "dockerVersion": "DockerVersion: 19.03.6-ce"
            },
            "remainingResources": [
                {
                    "name": "CPU",
                    "type": "INTEGER",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 2038
                },
                {
                    "name": "MEMORY",
                    "type": "INTEGER",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 159
                },
                {
                    "name": "PORTS",
                    "type": "STRINGSET",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "80",
                        "51678",
                        "51679"
                    ]
                },
                {
                    "name": "PORTS_UDP",
                    "type": "STRINGSET",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 0,
                    "stringSetValue": []
                }
            ],
            "registeredResources": [
                {
                    "name": "CPU",
                    "type": "INTEGER",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 2048
                },
                {
                    "name": "MEMORY",
                    "type": "INTEGER",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 459
                },
                {
                    "name": "PORTS",
                    "type": "STRINGSET",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 0,
                    "stringSetValue": [
                        "22",
                        "2376",
                        "2375",
                        "51678",
                        "51679"
                    ]
                },
                {
                    "name": "PORTS_UDP",
                    "type": "STRINGSET",
                    "doubleValue": 0.0,
                    "longValue": 0,
                    "integerValue": 0,
                    "stringSetValue": []
                }
            ],
            "status": "ACTIVE",
            "agentConnected": true,
            "runningTasksCount": 1,
            "pendingTasksCount": 0,
            "attributes": [
                {
                    "name": "ecs.capability.secrets.asm.environment-variables"
                },
                {
                    "name": "ecs.capability.branch-cni-plugin-version",
                    "value": "a21d3a41-"
                },
                {
                    "name": "ecs.ami-id",
                    "value": "ami-0ba238c6cfa8b1e42"
                },
                {
                    "name": "ecs.capability.secrets.asm.bootstrap.log-driver"
                },
                {
                    "name": "ecs.capability.task-eia.optimized-cpu"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.none"
                },
                {
                    "name": "ecs.capability.ecr-endpoint"
                },
                {
                    "name": "ecs.capability.docker-plugin.local"
                },
                {
                    "name": "ecs.capability.task-cpu-mem-limit"
                },
                {
                    "name": "ecs.capability.secrets.ssm.bootstrap.log-driver"
                },
                {
                    "name": "ecs.capability.efsAuth"
                },
                {
                    "name": "ecs.capability.full-sync"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.30"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.31"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.32"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.fluentd"
                },
                {
                    "name": "ecs.capability.firelens.options.config.file"
                },
                {
                    "name": "ecs.availability-zone",
                    "value": "ap-northeast-1a"
                },
                {
                    "name": "ecs.capability.aws-appmesh"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.awslogs"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.24"
                },
                {
                    "name": "ecs.capability.task-eni-trunking"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.25"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.26"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.27"
                },
                {
                    "name": "com.amazonaws.ecs.capability.privileged-container"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.28"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.29"
                },
                {
                    "name": "ecs.cpu-architecture",
                    "value": "x86_64"
                },
                {
                    "name": "ecs.capability.firelens.fluentbit"
                },
                {
                    "name": "com.amazonaws.ecs.capability.ecr-auth"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.20"
                },
                {
                    "name": "ecs.os-type",
                    "value": "linux"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.21"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.22"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.23"
                },
                {
                    "name": "ecs.capability.task-eia"
                },
                {
                    "name": "ecs.capability.private-registry-authentication.secretsmanager"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.syslog"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.awsfirelens"
                },
                {
                    "name": "ecs.capability.firelens.options.config.s3"
                },
                {
                    "name": "com.amazonaws.ecs.capability.logging-driver.json-file"
                },
                {
                    "name": "ecs.capability.execution-role-awslogs"
                },
                {
                    "name": "ecs.vpc-id",
                    "value": "vpc-0178ec2ab606c2bc2"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.17"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.18"
                },
                {
                    "name": "com.amazonaws.ecs.capability.docker-remote-api.1.19"
                },
                {
                    "name": "ecs.capability.docker-plugin.amazon-ecs-volume-plugin"
                },
                {
                    "name": "ecs.capability.task-eni"
                },
                {
                    "name": "ecs.capability.firelens.fluentd"
                },
                {
                    "name": "ecs.capability.efs"
                },
                {
                    "name": "ecs.capability.execution-role-ecr-pull"
                },
                {
                    "name": "ecs.capability.task-eni.ipv6"
                },
                {
                    "name": "ecs.capability.container-health-check"
                },
                {
                    "name": "ecs.subnet-id",
                    "value": "subnet-0b7eac18a11d4eb1d"
                },
                {
                    "name": "ecs.instance-type",
                    "value": "t3.nano"
                },
                {
                    "name": "com.amazonaws.ecs.capability.task-iam-role-network-host"
                },
                {
                    "name": "ecs.capability.container-ordering"
                },
                {
                    "name": "ecs.capability.cni-plugin-version",
                    "value": "55b2ae77-2020.09.0"
                },
                {
                    "name": "ecs.capability.env-files.s3"
                },
                {
                    "name": "ecs.capability.secrets.ssm.environment-variables"
                },
                {
                    "name": "ecs.capability.pid-ipc-namespace-sharing"
                },
                {
                    "name": "com.amazonaws.ecs.capability.task-iam-role"
                }
            ],
            "registeredAt": 1606607178.315,
            "attachments": [],
            "tags": []
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
