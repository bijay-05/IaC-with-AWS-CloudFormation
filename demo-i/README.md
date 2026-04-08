# Multi Environment AWS Architecture

## Constant

| Services                |
| ----------------------- |
| S3 bucket for templates |
| ECR                     |

## Environment specific services

| Services                           |
| ---------------------------------- |
| VPC                                |
| Subnet                             |
| Route Tables                       |
| Internet Gateways                  |
| Public IP                          |
| Security Groups                    |
| EC2                                |
| IAM Roles                          |
| AWS System manager Parameter Store |
| RDS                                |

## Push Image to ECR

```bash
aws ecr get-login-password --region <AWS_REGION> | docker login --username AWS --password-stdin <ACCOUNTID>.dkr.ecr.<AWS_REGION>.amazonaws.com

docker build -t events/web .

docker tag events/web:latest <ACCOUNTID>.dkr.ecr.<AWS_REGION>.amazonaws.com/events/web:latest

docker push <ACCOUNTID>.dkr.ecr.<AWS_REGION>.amazonaws.com/events/web:latest
```

## Create stacks with templates

```bash

aws configure --profle test-profile
aws configure list-profiles

aws s3 cp /path/to/template/file.yaml s3://<BUCKETNAME>/file.yaml --profile random-profile

aws cloudformation create-stack --stack-name test-network-stack --template-url https://<BUCKETNAME.s3.ap-south-1.amazonaws.com/file.yaml --region ap-southeast-1 --profile test-profile

aws cloudformation list-stacks --profile test-profile
aws cloudformation delete-stack --stack-name test-network-stack --profile test-profile

aws cloudformation help
aws cloudformation create-stack help
```
