# AWS CloudFormation Commands with AWS CLI

`aws cloudformation`

```bash
aws sts get-caller-identity

## if not configured, run this command and set user identity
aws configure

## validate the template
aws cloudformation validate-template --template-body file://vpc-deploy.yml

## create a stack
aws cloudformation create-stack --stack-name test-vpc-stack \
    --template-body file://vpc-deploy.yml \
    --parameters ParameterKey=KeyName,ParameterValue=my-ec2-key

## monitoring the stack
aws cloudformation describe-stacks --stack-name test-vpc-stack \
    --query 'Stacks[0].StackStatus'

## get stack outputs
aws cloudformation describe-stacks --stack-name test-vpc-stack \
    --query 'Stacks[0].Outputs'

## get stack status
aws cloudformation describe-stacks --stack-name test-vpc-stack \
    --query 'Stacks[0].StackStatus'

## delete the stack
aws cloudformation delete-stack --stack-name test-vpc-stack

## create change-set to preview changes before updating stack
aws cloudformation create-change-set --stack-name test-vpc-stack --change-set-name change-set-1 --template-body file://test-network.yaml

## execute change-set after review
aws cloudformation execute-change-set --change-set-name change-set-2 --stack-name test-stack-vpc

aws cloudformation delete-change-set --change-set-name change-set-3 --stack-name test-stack-vpc

## detect drift in the stack resources
aws cloudformation detect-stack-drift --stack-name test-stack-vpc

## When creating stacks with templates that include IAM resources
aws cloudformation create-stack --stack-name dev-env-stack \
    --template-body file://template.yaml \
    --capabilities CAPABILITY_NAMED_IAM

aws cloudformation create-change-set --stack-name teejo-stack \
    --capabilities CAPABILITY_NAMED_IAM --change-set-name add-iam-roles-ec2-instance \  --template-body file://main.yaml

## passing parameters at runtime
aws cloudformation create-stack --stack-name test-stack --parameters ParameterKey="environment",ParameterValue="dev" --template-url https://<BUCKET_NAME>.s3.<REGION>.amazonaws.com

```
