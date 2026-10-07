# CloudFormation DynamoDB Table Template Repository

<!-- Row 1: Status - Most Important -->
[![Release](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/actions/workflows/release.yaml/badge.svg)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/issues)&nbsp;[![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/commits)

<!-- Row 2: Code Quality -->
[![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Commits](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/commits)

<!-- Row 3: Tech Stack -->
[![CloudFormation](https://img.shields.io/badge/CloudFormation-IaC-orange?logo=amazon&logoColor=white)](https://aws.amazon.com/cloudformation/)&nbsp;[![DynamoDB](https://img.shields.io/badge/DynamoDB-NoSQL-brightgreen?logo=amazon&logoColor=white)](https://aws.amazon.com/dynamodb/)&nbsp;[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757?logo=anthropic&logoColor=white)](https://claude.ai/)

<!-- Row 4: Repository Info -->
[![Files](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)&nbsp;[![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function)](https://github.com/subhamay-bhattacharyya-cfn/cfn-nested-aws-lambda-function/releases)

<!-- Row 5: Custom Metrics -->
[![Custom Endpoint](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/26af28965bb1fafbf0d0cdfc4e443c26/raw/cfn-nested-aws-lambda-function.json)](https://gist.github.com/subhamay-bhattacharyya/26af28965bb1fafbf0d0cdfc4e443c26)

This repository contains a nested CloudFormation template for deploying DynamoDB tables with flexible configuration, security best practices, and support for advanced features like streams, indexes, and encryption.

## Overview

This is a **nested stack template** designed to be invoked from a parent/root CloudFormation stack. The template is stored in this repository and should be uploaded to an S3 bucket for reference by parent stacks.

## Template Files

### CloudFormation Templates

- **`templates/dynamodb-table.yaml`** — Nested template for DynamoDB table creation with support for:
  - Flexible billing modes (on-demand or provisioned)
  - Custom partition and sort keys
  - Local Secondary Indexes (LSI)
  - Global Secondary Indexes (GSI)
  - DynamoDB Streams
  - Point-in-time recovery
  - TTL (Time-to-Live)
  - KMS encryption

### Parameter Files

- **`parameters/dynamodb-dev.json`** — Development environment parameters
- **`parameters/dynamodb-staging.json`** — Staging environment parameters
- **`parameters/dynamodb-prod.json`** — Production environment parameters

## Template Features

### DynamoDB Table Template (dynamodb-table.yaml)

- ✅ **Flexible Billing** — On-demand (PAY_PER_REQUEST) or provisioned capacity
- ✅ **Custom Key Schema** — Configurable partition and sort keys with multiple data types
- ✅ **Local Secondary Indexes** — LSI support for alternative sort keys on the same partition key
- ✅ **Global Secondary Indexes** — GSI with optional sort key and independent provisioning
- ✅ **DynamoDB Streams** — Enable change data capture with multiple view types
- ✅ **Point-in-Time Recovery** — Restore tables to any point in time
- ✅ **TTL (Time-to-Live)** — Automatic item expiration
- ✅ **KMS Encryption** — Support for customer-managed or AWS-managed keys
- ✅ **Smart Table Naming** — Project prefix, account ID, environment, region with optional CI suffix
- ✅ **Data Retention** — Automatic retention policy (no deletion on stack removal)

## Parameters

### Table Naming & Environment

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `ProjectName` | String | `proj-ztc` | Project name prefix (lowercase, alphanumeric, hyphens only) |
| `TableBaseName` | String | `dynamodb-table` | Base name for the DynamoDB table |
| `Environment` | String | `devl` | Deployment environment (devl, stag, prod) |
| `CiSuffix` | String | `""` | Optional CI suffix to append to table name (e.g., pipeline ID) |

### Billing & Throughput

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `BillingMode` | String | `PAY_PER_REQUEST` | Billing mode (PAY_PER_REQUEST or PROVISIONED) |
| `ProvisionedReadCapacity` | Number | `5` | Read capacity units (1-40000, only for PROVISIONED mode) |
| `ProvisionedWriteCapacity` | Number | `5` | Write capacity units (1-40000, only for PROVISIONED mode) |

### Primary Key Configuration

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `PartitionKeyName` | String | `PK` | Name of the partition key attribute |
| `PartitionKeyType` | String | `S` | Data type: S (String), N (Number), or B (Binary) |
| `SortKeyName` | String | `SK` | Name of the sort key (leave empty for no sort key) |
| `SortKeyType` | String | `S` | Data type: S (String), N (Number), or B (Binary) |

### Encryption & Security

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `KmsKey` | String | `SB-KMS` | KMS key for encryption (name, alias, or ARN; empty for AWS-managed key) |

### Streams & CDC

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `EnableStreams` | String | `false` | Enable DynamoDB Streams (true or false) |
| `StreamViewType` | String | `NEW_AND_OLD_IMAGES` | Stream info: KEYS_ONLY, NEW_IMAGE, OLD_IMAGE, or NEW_AND_OLD_IMAGES |

### Backup & Recovery

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `EnablePointInTimeRecovery` | String | `false` | Enable point-in-time recovery (true or false) |

### TTL Configuration

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `EnableTTL` | String | `false` | Enable TTL for automatic item expiration (true or false) |
| `TTLAttributeName` | String | `ExpirationTime` | Name of the TTL attribute |

### Local Secondary Indexes

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `LSI1Enabled` | String | `false` | Enable Local Secondary Index 1 (requires sort key on base table) |
| `LSI1AttributeName` | String | `LSI1SK` | Sort key attribute name for LSI1 |
| `LSI1AttributeType` | String | `S` | Data type: S (String), N (Number), or B (Binary) |

### Global Secondary Indexes

| Parameter | Type | Default | Description |
| ----------- | ------ | --------- | ------------- |
| `GSI1Enabled` | String | `false` | Enable Global Secondary Index 1 |
| `GSI1PartitionKeyName` | String | `GSI1PK` | Partition key attribute name for GSI1 |
| `GSI1PartitionKeyType` | String | `S` | Data type: S (String), N (Number), or B (Binary) |
| `GSI1SortKeyName` | String | `""` | Optional sort key attribute name for GSI1 |
| `GSI1SortKeyType` | String | `S` | Data type: S (String), N (Number), or B (Binary) |
| `GSI1ReadCapacity` | Number | `5` | Read capacity for GSI1 (only for PROVISIONED mode) |
| `GSI1WriteCapacity` | Number | `5` | Write capacity for GSI1 (only for PROVISIONED mode) |

## Outputs

### DynamoDB Table Template Outputs

| Output | Type | Description |
| ----------- | ------ | ------------- |
| `TableName` | String | Name of the created DynamoDB table (exported for cross-stack reference) |
| `TableArn` | String | ARN of the created DynamoDB table (exported for cross-stack reference) |
| `StreamArn` | String | ARN of the DynamoDB Stream (only when EnableStreams is true) |

## Usage

### 1. Upload Template to S3

```bash
aws s3 cp templates/dynamodb-table.yaml s3://your-cfn-bucket/templates/dynamodb-table.yaml
```

### 2. Reference from Parent Stack

In your parent/root CloudFormation template:

```yaml
DynamoDBTableNestedStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: https://s3.amazonaws.com/your-cfn-bucket/templates/dynamodb-table.yaml
    Parameters:
      ProjectName: !Ref ProjectName
      TableBaseName: users-table
      Environment: !Ref Environment
      BillingMode: PAY_PER_REQUEST
      PartitionKeyName: UserID
      PartitionKeyType: S
      SortKeyName: CreatedAt
      SortKeyType: S
      EnableStreams: "true"
      EnablePointInTimeRecovery: "true"
      EnableTTL: "false"
      GSI1Enabled: "true"
      GSI1PartitionKeyName: Email
      GSI1PartitionKeyType: S
    Tags:
      - Key: Environment
        Value: !Ref Environment

Outputs:
  TableName:
    Value: !GetAtt DynamoDBTableNestedStack.Outputs.TableName
  TableArn:
    Value: !GetAtt DynamoDBTableNestedStack.Outputs.TableArn
  StreamArn:
    Value: !GetAtt DynamoDBTableNestedStack.Outputs.StreamArn
```

### 3. Deploy Using AWS CLI

#### Example 1: Basic On-Demand Table

```bash
aws cloudformation create-stack \
  --stack-name myapp-dynamodb-dev \
  --template-body file://templates/dynamodb-table.yaml \
  --parameters file://parameters/dynamodb-dev.json
```

#### Example 2: Provisioned Capacity with GSI

```bash
aws cloudformation create-stack \
  --stack-name myapp-dynamodb-prod \
  --template-body file://templates/dynamodb-table.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=TableBaseName,ParameterValue=orders \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=BillingMode,ParameterValue=PROVISIONED \
    ParameterKey=ProvisionedReadCapacity,ParameterValue=100 \
    ParameterKey=ProvisionedWriteCapacity,ParameterValue=100 \
    ParameterKey=PartitionKeyName,ParameterValue=OrderID \
    ParameterKey=PartitionKeyType,ParameterValue=S \
    ParameterKey=SortKeyName,ParameterValue=OrderDate \
    ParameterKey=SortKeyType,ParameterValue=S \
    ParameterKey=GSI1Enabled,ParameterValue=true \
    ParameterKey=GSI1PartitionKeyName,ParameterValue=CustomerID \
    ParameterKey=GSI1PartitionKeyType,ParameterValue=S \
    ParameterKey=GSI1SortKeyName,ParameterValue=OrderDate \
    ParameterKey=GSI1SortKeyType,ParameterValue=S \
    ParameterKey=GSI1ReadCapacity,ParameterValue=50 \
    ParameterKey=GSI1WriteCapacity,ParameterValue=50
```

#### Example 3: With Streams and Point-in-Time Recovery

```bash
aws cloudformation create-stack \
  --stack-name myapp-dynamodb-events \
  --template-body file://templates/dynamodb-table.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=TableBaseName,ParameterValue=events \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=BillingMode,ParameterValue=PAY_PER_REQUEST \
    ParameterKey=PartitionKeyName,ParameterValue=EventID \
    ParameterKey=PartitionKeyType,ParameterValue=S \
    ParameterKey=SortKeyName,ParameterValue=Timestamp \
    ParameterKey=SortKeyType,ParameterValue=N \
    ParameterKey=EnableStreams,ParameterValue=true \
    ParameterKey=StreamViewType,ParameterValue=NEW_AND_OLD_IMAGES \
    ParameterKey=EnablePointInTimeRecovery,ParameterValue=true
```

#### Example 4: With TTL and LSI

```bash
aws cloudformation create-stack \
  --stack-name myapp-dynamodb-sessions \
  --template-body file://templates/dynamodb-table.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=TableBaseName,ParameterValue=sessions \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=BillingMode,ParameterValue=PAY_PER_REQUEST \
    ParameterKey=PartitionKeyName,ParameterValue=SessionID \
    ParameterKey=PartitionKeyType,ParameterValue=S \
    ParameterKey=SortKeyName,ParameterValue=UserID \
    ParameterKey=SortKeyType,ParameterValue=S \
    ParameterKey=EnableTTL,ParameterValue=true \
    ParameterKey=TTLAttributeName,ParameterValue=ExpiresAt \
    ParameterKey=LSI1Enabled,ParameterValue=true \
    ParameterKey=LSI1AttributeName,ParameterValue=CreatedAt \
    ParameterKey=LSI1AttributeType,ParameterValue=N
```

#### Example 5: With Custom KMS Encryption

```bash
aws cloudformation create-stack \
  --stack-name myapp-dynamodb-secure \
  --template-body file://templates/dynamodb-table.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=TableBaseName,ParameterValue=sensitive-data \
    ParameterKey=Environment,ParameterValue=prod \
    ParameterKey=BillingMode,ParameterValue=PROVISIONED \
    ParameterKey=ProvisionedReadCapacity,ParameterValue=10 \
    ParameterKey=ProvisionedWriteCapacity,ParameterValue=10 \
    ParameterKey=PartitionKeyName,ParameterValue=DataID \
    ParameterKey=PartitionKeyType,ParameterValue=S \
    ParameterKey=KmsKey,ParameterValue=arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012
```

#### Example 6: With CI Suffix

```bash
aws cloudformation create-stack \
  --stack-name myapp-dynamodb-ci \
  --template-body file://templates/dynamodb-table.yaml \
  --parameters \
    ParameterKey=ProjectName,ParameterValue=myapp \
    ParameterKey=TableBaseName,ParameterValue=test-table \
    ParameterKey=Environment,ParameterValue=test \
    ParameterKey=CiSuffix,ParameterValue=$CI_PIPELINE_ID \
    ParameterKey=BillingMode,ParameterValue=PAY_PER_REQUEST \
    ParameterKey=PartitionKeyName,ParameterValue=PK \
    ParameterKey=PartitionKeyType,ParameterValue=S
```

#### Monitoring Stack Creation

```bash
# Wait for stack creation to complete
aws cloudformation wait stack-create-complete --stack-name myapp-dynamodb-dev

# Get stack outputs
aws cloudformation describe-stacks \
  --stack-name myapp-dynamodb-dev \
  --query 'Stacks[0].Outputs' \
  --output table

# Get table details
TABLE_NAME=$(aws cloudformation describe-stacks \
  --stack-name myapp-dynamodb-dev \
  --query 'Stacks[0].Outputs[?OutputKey==`TableName`].OutputValue' \
  --output text)

aws dynamodb describe-table --table-name $TABLE_NAME
```

## Table Naming Convention

The template generates table names using the following pattern:

**Without CI Suffix:**

```bash
{ProjectName}-{TableBaseName}-{AccountId}-{Environment}-{Region}
```

Example: `myapp-users-table-123456789012-devl-us-east-1`

**With CI Suffix:**

```bash
{ProjectName}-{TableBaseName}-{AccountId}-{Environment}-{Region}-{CiSuffix}
```

Example: `myapp-users-table-123456789012-devl-us-east-1-pipeline-12345`

## Best Practices Implemented

- ✅ KMS encryption enabled by default (AWS-managed key)
- ✅ Flexible billing modes (on-demand for dev, provisioned for production)
- ✅ Point-in-time recovery support for disaster recovery
- ✅ DynamoDB Streams for change data capture
- ✅ TTL support for automatic data expiration
- ✅ LSI and GSI support for flexible querying patterns
- ✅ Smart table naming with project prefix, account ID, environment, and region
- ✅ Optional CI suffix support for ephemeral test deployments
- ✅ Data retention policy (table not deleted when stack is removed)
- ✅ Automatic tagging for resource management
- ✅ Export values for cross-stack references

## License

MIT