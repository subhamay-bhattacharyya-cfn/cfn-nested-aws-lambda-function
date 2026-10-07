# CloudFormation Lambda Function Template Repository

<!-- Row 1: Status - Most Important -->
[![Release](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazon&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![Lambda](https://img.shields.io/badge/Lambda-Compute-FF9900?logo=amazon&logoColor=white)](https://aws.amazon.com/lambda/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/26af28965bb1fafbf0d0cdfc4e443c26/raw/cfn-nested-aws-lambda-function.json)](https://gist.github.com/subhamay-bhattacharyya/26af28965bb1fafbf0d0cdfc4e443c26)

This repository contains a reusable CloudFormation nested stack template for deploying AWS Lambda functions with flexible configuration, security best practices, and support for advanced features like VPC integration, DLQ, and Lambda Layers.

## Overview

This is a **nested stack template** designed to be invoked from a parent/root CloudFormation stack. The template supports two deployment modes:

1. **S3-based Code** — Deploy Lambda function code from an S3 bucket
2. **Inline Boilerplate** — Deploy a "Hello World" function without S3 code

The template is stored in this repository and should be uploaded to an S3 bucket for reference by parent stacks.

## Template Files

### CloudFormation Templates

- **`cloudformation/template.yaml`** — Nested template for Lambda function deployment with support for:
  - Python (3.12, 3.13) and Node.js (20.x, 24.x) runtimes
  - S3-based code deployment OR inline boilerplate code
  - Flexible memory (128-10,240 MB) and timeout (1-900 seconds)
  - VPC integration for secure database access
  - CloudWatch Logs with JSON structured logging
  - Dead Letter Queue (DLQ) support for async invocations
  - Lambda Layers attachment
  - Reserved concurrent execution limits
  - Environment variable configuration
  - External IAM role and CloudWatch Log Group management

### Parameter Files

- **`cloudformation/lambda-parameters-dev.json`** — Development (256 MB, no VPC)
- **`cloudformation/lambda-parameters-stag.json`** — Staging (512 MB, VPC enabled)
- **`cloudformation/lambda-parameters-prod.json`** — Production (1024 MB, VPC enabled)

## Template Features

### Lambda Function Template (template.yaml)

- ✅ **Flexible Runtimes** — Python 3.12/3.13 and Node.js 20.x/24.x support
- ✅ **Dual Code Sources** — Deploy from S3 bucket OR use inline boilerplate code
- ✅ **Flexible Configuration** — Configurable memory (128-10240 MB), timeout (1-900 seconds)
- ✅ **VPC Integration** — Optional VPC configuration for secure database access
- ✅ **CloudWatch Logging** — JSON structured logging with configurable retention
- ✅ **Dead Letter Queue** — Support for async invocation failure handling
- ✅ **Lambda Layers** — Attach multiple layers for code sharing and dependencies
- ✅ **Reserved Concurrency** — Control function scaling and cost
- ✅ **Environment Variables** — Pre-configured variables (ENVIRONMENT, LOG_LEVEL, DYNAMODB_TABLE)
- ✅ **External Resource Management** — IAM role and log group created externally
- ✅ **Deterministic Naming** — Project prefix, function name, environment, region

## Parameters

### Basic Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ProjectName` | String | `proj-ztc` | Project name prefix (lowercase, alphanumeric, hyphens only) |
| `LambdaFunctionBaseName` | String | `lambda-function` | Base name for Lambda function |
| `Environment` | String | `devl` | Deployment environment (devl, stag, prod) |

### Runtime Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `Runtime` | String | `python3.12` | Lambda runtime (python3.13, python3.12, nodejs24.x, nodejs20.x) |
| `Handler` | String | `index.handler` | Function handler (e.g., lambda_function.lambda_handler) |
| `MemorySize` | Number | `128` | Memory allocation in MB (128-10240) |
| `Timeout` | Number | `30` | Function timeout in seconds (1-900) |

### Code Deployment

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `S3Bucket` | String | `` | S3 bucket with Lambda code (leave empty for boilerplate code) |
| `S3Key` | String | `` | S3 key for Lambda code zip file |
| `S3ObjectVersion` | String | `` | Optional S3 object version ID |
| `InlineCode` | String | `def lambda_handler(event, context): return {'statusCode': 200, 'body': 'Hello World'}` | Boilerplate code when S3Bucket is empty |

### Environment & Logging

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `EnvironmentNameValue` | String | `development` | Value for ENVIRONMENT variable |
| `LogLevelValue` | String | `INFO` | Value for LOG_LEVEL variable (DEBUG, INFO, WARN, ERROR) |
| `DynamoDBTableName` | String | `` | DynamoDB table name (optional, for DYNAMODB_TABLE env var) |
| `LambdaLogGroup` | String | `` | CloudWatch Log Group name (must be created externally) |

### Advanced Configuration

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ReservedConcurrentExecutions` | Number | `-1` | Reserved concurrency (-1 = no reservation) |
| `EphemeralStorage` | Number | `512` | Ephemeral storage in MB (512-10240) |
| `LambdaLayerArns` | CommaDelimitedList | `` | Lambda Layer ARNs to attach |
| `EnableVPC` | String | `false` | Enable VPC configuration (true/false) |
| `VPCSubnetIds` | List | `` | VPC subnet IDs (required if EnableVPC=true) |
| `VPCSecurityGroupIds` | List | `` | VPC security group IDs (required if EnableVPC=true) |
| `EnableDeadLetterQueue` | String | `false` | Enable DLQ for failed async invocations |
| `DeadLetterQueueArn` | String | `` | SQS queue or SNS topic ARN for DLQ |
| `IAMRoleArn` | String | `` | IAM role ARN for Lambda execution (must be created externally) |

## Outputs

| Output | Description |
|--------|-------------|
| `FunctionName` | Name of the Lambda function |
| `FunctionArn` | ARN of the Lambda function |

## Deployment Modes

### Mode 1: Deploy with S3 Code (Default)

```bash
# Upload code to S3
aws s3 cp lambda-function.zip s3://my-bucket/lambda-code/function.zip

# Create IAM role
aws iam create-role --role-name lambda-exec-role \
  --assume-role-policy-document '{"Version":"2012-10-17",...}'

# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Deploy Lambda with S3 code
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-lambda-stack \
  --parameter-overrides \
    S3Bucket=my-bucket \
    S3Key=lambda-code/function.zip \
    LambdaLogGroup=/aws/lambda/my-function \
    IAMRoleArn=arn:aws:iam::123456789012:role/lambda-exec-role
```

### Mode 2: Deploy with Boilerplate Code (Quick Start)

```bash
# Create IAM role
aws iam create-role --role-name lambda-exec-role \
  --assume-role-policy-document '{"Version":"2012-10-17",...}'

# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Deploy Lambda with inline boilerplate code
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-lambda-stack \
  --parameter-overrides \
    LambdaLogGroup=/aws/lambda/my-function \
    IAMRoleArn=arn:aws:iam::123456789012:role/lambda-exec-role
```

The boilerplate code will be used automatically when `S3Bucket` is empty:

```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello World'
    }
```

## Lambda Naming Convention

Lambda function names follow this pattern:

```bash
{ProjectName}-{LambdaFunctionBaseName}-{Environment}-{Region}
```

**Example:** `proj-ztc-data-processor-devl-us-east-1`

**Note:** Unlike other resources, Lambda function names do not include AWS Account ID since they are scoped to an account and region.

## Prerequisites

Before deploying, create these external resources:

### 1. IAM Execution Role

```bash
aws iam create-role \
  --role-name lambda-execution-role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "lambda.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach basic execution policy
aws iam attach-role-policy \
  --role-name lambda-execution-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Attach VPC execution policy (if using VPC)
aws iam attach-role-policy \
  --role-name lambda-execution-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
```

### 2. CloudWatch Log Group

```bash
aws logs create-log-group \
  --log-group-name /aws/lambda/my-function

aws logs put-retention-policy \
  --log-group-name /aws/lambda/my-function \
  --retention-in-days 7
```

### 3. Upload Lambda Code (if using S3 mode)

```bash
aws s3 cp lambda-function.zip s3://my-bucket/lambda-code/
```

## Best Practices

- ✅ Use externally-managed IAM roles for security isolation
- ✅ Create CloudWatch Log Groups with appropriate retention policies
- ✅ Use S3 versioning for code deployments
- ✅ Enable VPC for Lambda functions that access databases
- ✅ Configure DLQ for async invocations to capture failures
- ✅ Use Lambda Layers for shared dependencies
- ✅ Set appropriate memory allocation based on function performance
- ✅ Monitor Lambda metrics in CloudWatch

## Examples

### Example 1: Python Function with DynamoDB Access

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name user-processor \
  --parameter-overrides \
    ProjectName=myapp \
    LambdaFunctionBaseName=user-processor \
    Environment=prod \
    Runtime=python3.12 \
    Handler=lambda_function.lambda_handler \
    MemorySize=512 \
    S3Bucket=lambda-code-bucket \
    S3Key=user-processor.zip \
    DynamoDBTableName=users-table \
    LambdaLogGroup=/aws/lambda/user-processor \
    IAMRoleArn=arn:aws:iam::123456789012:role/lambda-role \
    EnableVPC=true \
    VPCSubnetIds=subnet-123,subnet-456 \
    VPCSecurityGroupIds=sg-789
```

### Example 2: Node.js Function with Quick Start Boilerplate

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name hello-world-api \
  --parameter-overrides \
    ProjectName=quickstart \
    LambdaFunctionBaseName=hello-api \
    Runtime=nodejs24.x \
    Handler=index.handler \
    LambdaLogGroup=/aws/lambda/hello-api \
    IAMRoleArn=arn:aws:iam::123456789012:role/lambda-role
```

The deployment will use the inline boilerplate code automatically.

## Troubleshooting

### Lambda Won't Deploy
- Verify IAM role ARN is correct: `aws iam get-role --role-name your-role`
- Check CloudWatch Log Group exists: `aws logs describe-log-groups --log-group-name-prefix /aws/lambda/`
- Validate template: `aws cloudformation validate-template --template-body file://template.yaml`

### Function Not Invoking S3 Code
- Verify S3 bucket and key exist: `aws s3 ls s3://bucket/key`
- Check S3 object permissions: `aws s3api head-object --bucket bucket --key key`
- Ensure IAM role has S3 read permissions

### VPC Connection Issues
- Verify subnets and security groups exist in same VPC
- Check security group allows necessary outbound traffic
- For database access, ensure VPC endpoints or NAT gateway configured

## License

MIT
