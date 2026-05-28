# IaC-with-AWS-CloudFormation

This repository documents how to set up infrastructure on AWS with IaC tool CloudFormation.

## Core Topics of CloudFormation

1. Template: The `yml/json` file where you define your resources
2. Stack: A collection of AWS resources created from a template
3. Change Sets: A preview of what changes CloudFormation will apply before executing them.
4. Drift Detection: A way to check if your actual resources differ from what's defined in the template.

## Keywords in CloudFormation Template

1. **AWSTemplateFormatVersion**: Specifies the CloudFormation template version

2. **Description**: A human-readable description of what the template does.

3. **Metadata**: Stores arbitrary `JSON/YAML` data about the template or resources, often used with tools like `AWS::CloudFormation::Init` to configure EC2 instances

4. **Parameters**: Allow you to pass values into your template at deployment time. It makes reusable template across environments.

```yml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues: [t2.micro, t3.micro, t3.small]
```

5. **Mappings**: Define key-value mappings, useful for region-specific AMI IDs or configuration values

```yml
Mappings:
  AWSInstanceType2Arch:
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64

  AWSRegionArch2AMI:
    ap-southeast-1:
      HVM64: ami-05fd46f12b86c4a6c
```

6. **Conditions**: Control whether resources are created or properties assigned, based on input. We can set conditions to deploy different resources in dev vs prod environments.

```yml
Conditions:
  CreateProdResources: !Equals [!Ref EnvType, prod]
```

7. **Transform**: Apply macros or reusable snippets. AWS provides built-in macros and supported custom transform as well.

```yml
Transform: AWS::Serverless-2016-10-31
```

8. **Resources**: This is the only mandatory section in a CloudFormation template. It defines the AWS resources (EC2, S3, etc)

```yml
Resources:
  # VPC
  MyVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.10.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: MyVPC
```

9. **Outputs**: Return values after the stack is created, share values across stacks (via cross-stack references) or display important info.

```yml
Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref MyVPC
    Export:
      Name: !Sub ${AWS::StackName}-VPC-ID

  PublicSubnetId:
    Description: Public Subnet ID
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub ${AWS::StackName}-PublicSubnet-ID

  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MyEC2Instance

  InstancePublicIP:
    Description: Public IP address of the EC2 instance
    Value: !GetAtt MyEC2Instance.PublicIp

  WebsiteURL:
    Description: URL for the website
    Value: !Sub "http://${MyEC2Instance.PublicIp}"
```

10. **Rules**: Validate parameter inputs before creating resources

```yml
Rules:
  InstanceTypeRule:
    Assertions:
      - Assert: !Contains [[t2.micro, t2.small], !Ref InstanceType]
        AssertDescription: Must be a valid instance type
```

- Rules run before deployment. They don't create or remove resources. Instead, they validate parameter values (e.g., only allow `t2.micro` as instance type). If a rule fails, the stack won't event start creating.

- Conditions are applied during deployment. They decide whether a resource (or property) should be created/applied based on input parameters (e.g., deploy an RDS database instance only if `EnvType = prod`)

### Deploy via aws cli

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

```

## Miscellaneous But Important

In some cases, you must explicitly acknowledge that your stack template contains certain capabilities in order for CloudFormation to create the stack.

- **CAPABILITY_IAM** And **CAPABILITY_NAMED_IAM**

Some stack templates might include resources that can affect permissions in your AWS account; for example, by creating new IAM users. For those stacks, you must explicitly acknowledge this by specifying one of these capabilities.

The following IAM resources require you to specify either the **CAPABILITY_IAM** or **CAPABILITY_NAMED_IAM** capability.

1. If you have IAM resources, you can specify either capability.

2. If you have IAM resources with custom names, you must specify `CAPABILITY_NAMED_IAM`.

3. If you don't specify either of these capabilities, CloudFormation returns an `InsufficientCapabilities` error.

```markdown
AWS::IAM::AccessKey

AWS::IAM::Group

AWS::IAM::InstanceProfile

AWS::IAM::ManagedPolicy

AWS::IAM::Policy

AWS::IAM::Role

AWS::IAM::User

AWS::IAM::UserToGroupAddition
```
