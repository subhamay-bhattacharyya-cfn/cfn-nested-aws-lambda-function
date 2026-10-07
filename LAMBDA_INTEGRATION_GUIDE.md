# Lambda Function CloudFormation Template - Integration Guide

## Overview

A new nested CloudFormation template has been created for deploying Lambda functions with flexible configuration, following the same patterns and standards as the existing DynamoDB template.

## Files Created

### Template Files
- **`cloudformation/lambda-template.yaml`** - Main Lambda function template
  - Creates Lambda function with configurable runtime, memory, timeout
  - Optional VPC integration for database access
  - CloudWatch Logs integration with configurable retention
  - Support for Dead Letter Queues (DLQ) for async invocations
  - Optional Lambda Layers attachment
  - Basic IAM execution role (or use custom role via parameter)

### Parameter Files
- **`cloudformation/lambda-parameters-dev.json`** - Development environment defaults
- **`cloudformation/lambda-parameters-stag.json`** - Staging environment defaults
- **`cloudformation/lambda-parameters-prod.json`** - Production environment defaults

### Documentation
- **`cloudformation/LAMBDA_README.md`** - Comprehensive usage guide

## Function Naming Convention

```
{ProjectName}-{LambdaFunctionBaseName}-{Environment}-{AWS::Region}
```

**Examples:**
- `proj-ztc-data-processor-devl-us-east-1` (development)
- `proj-ztc-data-processor-stag-us-east-1` (staging)
- `proj-ztc-data-processor-prod-us-east-1` (production)

**Note:** Unlike the DynamoDB template, Lambda function names don't include AWS Account ID since Lambda functions are already scoped to an account+region.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                  Parent CloudFormation Stack                 │
│  (References nested DynamoDB and Lambda templates via URL)   │
└──────────────────┬──────────────────┬───────────────────────┘
                   │                  │
        ┌──────────▼──────────┐      │
        │  DynamoDB Table     │      │
        │  (nested stack)     │      │
        │  Outputs:           │      │
        │  - TableArn         │      │
        │  - TableName        │      │
        │  - StreamArn        │      │
        └─────────────────────┘      │
                                     │
                        ┌────────────▼──────────────┐
                        │   Lambda Function        │
                        │   (nested stack)         │
                        │   Inputs:                │
                        │   - S3Bucket/Key         │
                        │   - Runtime              │
                        │   - Memory/Timeout       │
                        │   - VPC config           │
                        │   Outputs:               │
                        │   - FunctionArn          │
                        │   - FunctionName         │
                        │   - LogGroupName         │
                        └──────────────────────────┘
```

## Integration with DynamoDB Template

### Shared Patterns
Both templates follow identical patterns:

| Aspect | DynamoDB | Lambda |
|--------|----------|--------|
| **Naming Convention** | ProjectName + BaseName + Environment + Region + [AccountId] | ProjectName + BaseName + Environment + Region |
| **Parameter Files** | `*-dev.json`, `*-stag.json`, `*-prod.json` | `lambda-parameters-dev.json`, etc. |
| **Outputs** | Table resources exported | Function resources exported |
| **Environment Support** | devl, stag, prod | devl, stag, prod |

### Using Together in Parent Stack

**Parent Stack Template Example:**

```yaml
Resources:
  # Deploy nested DynamoDB stack
  DynamoDBStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/templates/cloudformation/template.yaml
      Parameters:
        ProjectName: proj-ztc
        Environment: !Ref Environment
        # ... other DynamoDB parameters

  # Deploy nested Lambda stack
  LambdaStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/my-bucket/templates/cloudformation/lambda-template.yaml
      Parameters:
        ProjectName: proj-ztc
        Environment: !Ref Environment
        S3Bucket: !Ref LambdaCodeBucket
        S3Key: lambda-functions/processor.zip
        EnvironmentNameValue: !Ref Environment
        DynamoDBTableName: !GetAtt DynamoDBStack.Outputs.TableName
        EnableVPC: 'true'
        VPCSubnetIds: subnet-12345,subnet-67890
        VPCSecurityGroupIds: sg-abcdef

Outputs:
  DynamoDBTableArn:
    Value: !GetAtt DynamoDBStack.Outputs.TableArn

  LambdaFunctionArn:
    Value: !GetAtt LambdaStack.Outputs.FunctionArn

  LambdaLogGroup:
    Value: !GetAtt LambdaStack.Outputs.LogGroupName
```

## Quick Start

### Prerequisites

1. **Create CloudWatch Log Group** (externally managed):
```bash
LOG_GROUP_NAME="/aws/lambda/proj-ztc-data-processor-devl-us-east-1"

aws logs create-log-group \
  --log-group-name $LOG_GROUP_NAME \
  --region us-east-1

# Set retention policy (7 days for dev)
aws logs put-retention-policy \
  --log-group-name $LOG_GROUP_NAME \
  --retention-in-days 7 \
  --region us-east-1
```

2. **Create IAM Execution Role** (externally managed):
```bash
aws iam create-role \
  --role-name lambda-execution-role-devl \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "lambda.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach basic execution policy (includes CloudWatch Logs)
aws iam attach-role-policy \
  --role-name lambda-execution-role-devl \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Get the role ARN for deployment
ROLE_ARN=$(aws iam get-role --role-name lambda-execution-role-devl \
  --query 'Role.Arn' --output text)
```

3. **Upload Lambda Code to S3**:
```bash
# Create bucket if needed
aws s3 mb s3://my-lambda-code-bucket --region us-east-1

# Upload function code
aws s3 cp lambda-function.zip s3://my-lambda-code-bucket/lambda-functions/processor.zip
```

### 1. Deploy to Development

```bash
cd cloudformation

# Validate template
aws cloudformation validate-template \
  --template-body file://lambda-template.yaml

# Deploy Lambda function with IAM role and log group
aws cloudformation deploy \
  --template-file lambda-template.yaml \
  --stack-name proj-ztc-lambda-processor-devl \
  --parameter-overrides \
    file://lambda-parameters-dev.json \
    IAMRoleArn=$ROLE_ARN \
    LogGroupName=$LOG_GROUP_NAME \
  --region us-east-1
```

### 2. Deploy to Staging with VPC

```bash
# Create staging role
aws iam create-role \
  --role-name lambda-execution-role-stag \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "lambda.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

STAG_ROLE_ARN=$(aws iam get-role --role-name lambda-execution-role-stag \
  --query 'Role.Arn' --output text)

# Deploy with VPC config
aws cloudformation deploy \
  --template-file lambda-template.yaml \
  --stack-name proj-ztc-lambda-processor-stag \
  --parameter-overrides \
    file://lambda-parameters-stag.json \
    IAMRoleArn=$STAG_ROLE_ARN \
    VPCSubnetIds="subnet-12345,subnet-67890" \
    VPCSecurityGroupIds="sg-abcdef" \
  --region us-east-1
```

### 3. Deploy to Production with Existing Role

```bash
# Reference existing production role
PROD_ROLE_ARN="arn:aws:iam::123456789012:role/prod-lambda-execution-role"

aws cloudformation deploy \
  --template-file lambda-template.yaml \
  --stack-name proj-ztc-lambda-processor-prod \
  --parameter-overrides \
    file://lambda-parameters-prod.json \
    IAMRoleArn=$PROD_ROLE_ARN \
  --region us-east-1
```

## Parameter Reference

### Required Parameters
- **S3Bucket**: S3 bucket containing Lambda code
- **S3Key**: S3 key for Lambda deployment package
- **IAMRoleArn**: ARN of existing IAM role for Lambda execution (must be created beforehand)

### Key Optional Parameters

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `ProjectName` | `proj-ztc` | Function name prefix |
| `LambdaFunctionBaseName` | `lambda-function` | Function name component |
| `Environment` | `devl` | Environment label |
| `Runtime` | `python3.12` | Lambda runtime |
| `MemorySize` | `128` | Memory allocation (MB) |
| `Timeout` | `30` | Timeout (seconds) |
| `EnableVPC` | `false` | VPC integration |
| `EnableDeadLetterQueue` | `false` | DLQ for failed invocations |

## Environment Variables

The template automatically configures:

```
ENVIRONMENT=development|staging|production
LOG_LEVEL=DEBUG|INFO|WARN|ERROR
DYNAMODB_TABLE={table_name}           (if configured)
```

### Lambda Code Access

**Python:**
```python
import os

def lambda_handler(event, context):
    env = os.getenv('ENVIRONMENT')
    table = os.getenv('DYNAMODB_TABLE')
    # ...
```

**Node.js:**
```javascript
exports.handler = async (event, context) => {
    const env = process.env.ENVIRONMENT;
    const table = process.env.DYNAMODB_TABLE;
    // ...
};
```

## VPC Configuration

When integrating Lambda with DynamoDB in a VPC:

1. **Enable VPC:**
```bash
--parameter-overrides \
  EnableVPC=true \
  VPCSubnetIds="subnet-12345,subnet-67890" \
  VPCSecurityGroupIds="sg-abcdef"
```

2. **Configure DynamoDB VPC Endpoint** (if needed):
```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-12345 \
  --service-name com.amazonaws.us-east-1.dynamodb \
  --route-table-ids rtb-12345
```

3. **Attach IAM Policy** for DynamoDB access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/proj-ztc-*"
    }
  ]
}
```

## Monitoring & Troubleshooting

### View Lambda Logs

```bash
# Get log group name from stack output
LOG_GROUP=$(aws cloudformation describe-stacks \
  --stack-name proj-ztc-lambda-processor-devl \
  --query 'Stacks[0].Outputs[?OutputKey==`LogGroupName`].OutputValue' \
  --output text)

# View recent logs
aws logs tail $LOG_GROUP --follow
```

### Check Function Performance

```bash
# Get function metrics from CloudWatch
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Duration \
  --dimensions Name=FunctionName,Value=proj-ztc-data-processor-devl-us-east-1 \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 60 \
  --statistics Average,Maximum
```

### Update Function Code

```bash
# Update S3 object and trigger Lambda update
aws s3 cp lambda-function.zip s3://my-bucket/lambda-functions/processor.zip

# Update stack (CloudFormation detects S3 object version change)
aws cloudformation deploy \
  --template-file lambda-template.yaml \
  --stack-name proj-ztc-lambda-processor-devl \
  --parameter-overrides file://lambda-parameters-dev.json \
  --region us-east-1
```

## Cleanup

```bash
# Delete Lambda stack (removes function and log group)
aws cloudformation delete-stack \
  --stack-name proj-ztc-lambda-processor-devl \
  --region us-east-1

# Monitor deletion
aws cloudformation wait stack-delete-complete \
  --stack-name proj-ztc-lambda-processor-devl \
  --region us-east-1
```

## Best Practices

1. **Code Deployment**
   - Store Lambda code in S3 with versioning enabled
   - Use S3 object versioning for rollback capability
   - Scan code for security vulnerabilities before upload

2. **Environment Configuration**
   - Use separate parameter files per environment
   - Store sensitive values in AWS Secrets Manager
   - Reference from Lambda via SDK or environment variables

3. **Monitoring**
   - Enable CloudWatch Logs (default: enabled)
   - Set appropriate log retention periods
   - Use structured logging (JSON) for easier parsing

4. **Performance**
   - Increase `MemorySize` if frequently timing out
   - Monitor CloudWatch `Duration` metric
   - Use `ReservedConcurrentExecutions` for critical functions

5. **VPC Usage**
   - Use private subnets for database access
   - Configure NAT gateway for external API calls
   - Use VPC endpoints for AWS services

6. **Error Handling**
   - Enable DLQ for async invocations
   - Monitor DLQ for failed messages
   - Implement proper retry logic in code

## Comparison: DynamoDB vs Lambda Templates

| Feature | DynamoDB | Lambda |
|---------|----------|--------|
| Naming includes Account ID | Yes | No |
| VPC support | No | Yes (optional) |
| Requires S3 code bucket | No | Yes |
| Auto-creates IAM role | No | Yes (optional) |
| CloudWatch Logs | Not applicable | Yes (optional) |
| Stream support | Yes (optional) | Not applicable |
| TTL support | Yes (optional) | Not applicable |
| LSI/GSI support | Yes | Not applicable |
| Reserved capacity | Yes (PROVISIONED mode) | Yes (concurrent executions) |

## File Structure

```
cloudformation/
├── template.yaml                      # DynamoDB template
├── lambda-template.yaml               # Lambda template (NEW)
├── LAMBDA_README.md                   # Lambda template docs (NEW)
├── parameters.json                    # Existing DynamoDB dev params
├── lambda-parameters-dev.json         # Lambda dev params (NEW)
├── lambda-parameters-stag.json        # Lambda stag params (NEW)
└── lambda-parameters-prod.json        # Lambda prod params (NEW)

cloudformation/function/               # Old directory (deprecated)
├── parameters.json
├── stack-config.json
└── template.yaml
```

## Next Steps

1. **Customize Parameters**
   - Update S3 bucket names in parameter files
   - Adjust memory/timeout for your function
   - Configure VPC settings if needed

2. **Deploy to Dev**
   - Test deployment to development environment
   - Verify function invocation
   - Check CloudWatch Logs

3. **Add to CI/CD**
   - Update GitHub Actions workflow to deploy Lambda stack
   - Add validation and testing steps
   - Configure production safeguards

4. **Integrate with DynamoDB**
   - Create parent stack referencing both templates
   - Test Lambda-to-DynamoDB connectivity
   - Monitor end-to-end functionality

## Support Resources

- **Template Documentation**: See `cloudformation/LAMBDA_README.md`
- **AWS Lambda Best Practices**: https://docs.aws.amazon.com/lambda/latest/dg/
- **CloudFormation User Guide**: https://docs.aws.amazon.com/cloudformation/
- **Nested Stacks**: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html
