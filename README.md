# AWS Fargate 環境構築サンプル

`ecs-cli` で運用するAWS Fargateの環境を構築するためのサンプルテンプレートになります。

## create network
### vpc
```
$ aws cloudformation create-stack --stack-name fargate-sample-vpc --template-body file://cloudformation/network/vpc.yml
```
### security group
```
$ aws cloudformation create-stack --stack-name fargate-sample-sg --template-body file://cloudformation/network/securitygroup.yml
```

## initialize ecs-cli
### setup aws credentials
```
$ ecs-cli configure profile --profile-name fargate-sample --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY
```
### ecs-cli config
```
$ ecs-cli configure --cluster fargate-sample-cluster --region ap-northeast-1 --default-launch-type FARGATE --config-name fargate-sample
```
### setup default profile
```
$ ecs-cli configure profile default --profile-name fargate-sample
```

## setup ecs
### create ecs cluster
```
$ ecs-cli up --cluster fargate-sample-cluster --vpc [vpc id] --subnets [subnet ids] --security-group fargate-sample-sg --launch-type FARGATE --region ap-northeast-1 --ecs-profile fargate-sample
e-sg --launch-type FARGATE --region ap-northeast-1 --ecs-profile fargate-sample
```

### create alb
```
$ aws cloudformation create-stack --stack-name fargate-sample-alb --template-body file://cloudformation/alb.yml
```

## deploy
### deploy
```
$ ecs-cli compose --project-name fargate-sample -f compose/docker-compose.common.yml -f compose/docker-compose.dev.yml --ecs-params compose/ecs-params.dev.yml --cluster fargate-sample-cluster service up --deployment-max-percent 200 --deployment-min-healthy-percent 50 --target-group-arn [target group arn] --container-name frontend --container-port 80 --launch-type FARGATE --health-check-grace-period 120 --create-log-groups --timeout 10
```