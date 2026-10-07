# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **CloudFormation template repository** that provides a reusable nested stack template for deploying AWS Lambda functions with flexible configuration and security best practices. The template follows the nested stack pattern and is designed to be referenced by parent CloudFormation stacks.

**Key characteristics:**

- Nested CloudFormation template for Lambda function deployment (referenced via `TemplateURL`)
- Dual deployment modes: S3-based code OR inline boilerplate code
- Support for Python (3.12, 3.13) and Node.js (20.x, 24.x) runtimes
- Configurable memory, timeout, VPC integration, and concurrency
- CloudWatch Logs integration with external log group management
- Dead Letter Queue (DLQ) support for async invocations
- Lambda Layers support for dependencies
- External IAM role management for security
- Automated semantic versioning and releases
- AWS OIDC authentication for CI/CD deployments

## Project Structure

```text
cloudformation/
├── template.yaml                   # Nested template: Lambda function creation
├── lambda-parameters-dev.json      # Parameters for development environment
├── lambda-parameters-stag.json     # Parameters for staging environment
├── lambda-parameters-prod.json     # Parameters for production environment
├── parameters.json                 # Legacy parameters file
└── stack-config.json               # Stack configuration

.github/workflows/
├── ci.yaml                         # Validates, deploys, and cleans up templates
├── release.yaml                    # Semantic release on push to main
└── create-branch.yaml              # Auto-create feature branches from issues

scripts/plugins/
├── release.config.js               # Semantic-release configuration
└── (other release plugins)         # Custom commit analysis, notes generation

.claude/
├── settings.json                   # Claude Code workspace settings
└── settings.local.json             # Local overrides

.devcontainer/
└── devcontainer.json               # Dev container setup (Node.js 20)

package.json                        # Dependencies: semantic-release, commitizen
README.md                           # Template documentation and usage examples
CLAUDE.md                           # This file
```

## Development Commands

### Install dependencies

```bash
npm ci
```

### Trigger semantic release (usually automatic on main)

```bash
npm run release
```

### Commit with conventional commit format

```bash
npx cz commit
```

Select `feat`, `fix`, or `chore` type. Only `feat` and `fix` trigger releases.

## Key Architecture Concepts

### Nested Stack Pattern

This repo provides a **nested stack template** — a template that is referenced from parent/root CloudFormation stacks via `TemplateURL`. The template is self-contained and exports outputs for cross-stack references.

- **Parent stack** calls: `AWS::CloudFormation::Stack` with `TemplateURL` pointing to S3
- **Nested template** outputs values via `Outputs` section with `Export`
- Parent retrieves outputs via `!GetAtt LambdaStack.Outputs.OutputKey`

### Function Naming Convention

Function names follow a deterministic pattern driven by parameters:

```bash
{ProjectName}-{LambdaFunctionBaseName}-{Environment}-{Region}[-{CiSuffix}]
```

Examples:
- Standard: `proj-ztc-data-processor-devl-us-east-1`
- With CI suffix: `proj-ztc-data-processor-devl-us-east-1-ci-123`

This ensures:

- Consistency across deployments
- Environment isolation
- Predictable naming for infrastructure automation
- Unique names for CI/CD ephemeral deployments (via optional CiSuffix)

**Note:** Unlike other AWS resources, Lambda function names do NOT include Account ID (they're scoped to account+region already). The CiSuffix is optional and typically used for CI/CD testing to create unique function names without colliding with permanent deployments.

### Dual Deployment Modes

The template supports two ways to provide Lambda function code:

1. **S3 Mode (Default)** — Code stored in S3 bucket
   - Provide `S3Bucket` and `S3Key` parameters
   - For production deployments with versioned code
   - Code lifecycle managed separately from CloudFormation

2. **Inline Mode (Quick Start)** — Boilerplate code in template
   - Leave `S3Bucket` empty
   - Uses default "Hello World" Python function
   - For testing, prototypes, or quick deployments
   - Code: `def lambda_handler(event, context): return {'statusCode': 200, 'body': 'Hello World'}`

### CommaDelimitedList Parameters

The template uses CloudFormation's `CommaDelimitedList` type for parameters that need to accept multiple values:

- `VPCSubnetIds` — Accepts comma-delimited subnet IDs (e.g., `'subnet-123,subnet-456'`)
- `VPCSecurityGroupIds` — Accepts comma-delimited security group IDs (e.g., `'sg-abc,sg-def'`)
- `LambdaLayerArns` — Accepts comma-delimited Lambda Layer ARNs

**How it works:**
1. Parameters are passed as quoted comma-delimited strings in CLI or JSON files
2. CloudFormation automatically converts them to arrays/lists
3. Conditions check if these parameters are non-empty before using them
4. Only applied to the function when their associated feature is enabled (e.g., VPC config only applies if EnableVPC=true and both subnet/security group IDs are provided)

### IAM Role Management

The template uses a **hybrid approach** for IAM role management:

**CI/CD Deployments (with CiSuffix):**
- Lambda execution role is **automatically created** by the template
- Includes basic Lambda execution permissions (`AWSLambdaBasicExecutionRole`)
- Role is ephemeral: deleted when stack is deleted
- No prerequisites needed for role creation

**Production Deployments (without CiSuffix):**
- Lambda execution role must be **created externally** and provided via `IAMRoleArn` parameter
- Better security isolation and compliance
- Lifecycle independence (role can outlive the function)
- Team separation of concerns (platform team creates roles, application team creates functions)

**CloudWatch Log Groups:**
- Can be provided externally via `LambdaLogGroup` parameter
- Optional: leave empty if not using CloudWatch logging

This separation provides:

- **Security**: Production roles are externally managed and vetted
- **Flexibility**: CI/CD deployments don't need pre-created roles
- **Compliance**: Enterprise patterns with role isolation and lifecycle management
- **Ephemeral Testing**: CI suffix deployments are fully self-contained

## Key Files to Understand

### `cloudformation/template.yaml`

**Purpose:** Creates a Lambda function with flexible configuration

**Key inputs:**

- `ProjectName` (default: `proj-ztc`): Project prefix
- `LambdaFunctionBaseName` (default: `lambda-function`): Base function name
- `Environment`: Environment label (devl, stag, prod)
- `CiSuffix` (optional): Suffix for CI/CD deployments (appended to function name; ignored if auto-creating role)
- `Runtime`: Lambda runtime (python3.13, python3.12, nodejs24.x, nodejs20.x)
- `S3Bucket` / `S3Key`: Code location (leave empty for inline boilerplate)
- `LambdaLogGroup`: External CloudWatch Log Group name (optional)
- `IAMRoleArn` (default: placeholder): External IAM role ARN for Lambda execution (required if no CiSuffix; must be overridden with actual role ARN)

**Key outputs:**

- `FunctionName`: Function name (exported for parent stack)
- `FunctionArn`: Function ARN (exported for parent stack)

**Features:**

- Conditional code source: S3 OR inline boilerplate
- Configurable memory (128-10240 MB) and timeout (1-900 seconds)
- Optional VPC integration for database access
- Optional DLQ for async invocation failure handling
- Optional Lambda Layers attachment
- Reserved concurrent execution limits
- Environment variables (ENVIRONMENT, LOG_LEVEL, DYNAMODB_TABLE)
- JSON structured logging to CloudWatch

### `cloudformation/lambda-parameters-*.json`

**Purpose:** Environment-specific parameter overrides

**Structure:**

- `lambda-parameters-dev.json`: Development (256 MB, no VPC, no DLQ)
- `lambda-parameters-stag.json`: Staging (512 MB, VPC enabled, DLQ enabled)
- `lambda-parameters-prod.json`: Production (1024 MB, VPC enabled, DLQ enabled)

**Usage:**

```bash
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --parameter-overrides file://cloudformation/lambda-parameters-dev.json \
  S3Bucket=my-bucket \
  S3Key=lambda-code/function.zip
```

### `.github/workflows/ci.yaml`

**Triggered on:**

- Manual workflow_dispatch (anytime)
- Pull requests (any branch)
- Pushes to `feature/**` and `bug/**` branches

**Path filter:** Only runs if changes to `cloudformation/`, `.github/workflows/ci.yaml`

**Phases:**

1. **Validation:** `aws cloudformation validate-template` on template
2. **Deployment:** Creates Lambda stack in CI environment
3. **Cleanup:** Destroys stack for ephemeral testing

**Environment setup:**

- Reads config from GitHub environment variables: `AWS_REGION`, `AWS_ACCOUNT_ID`, `OIDC_ROLE_NAME`, `CFN_TEMPLATES_S3_BUCKET`
- Uses AWS OIDC for keyless authentication via `aws-actions/configure-aws-credentials`
- Requires GitHub environment `ci` with OIDC trust configured

### `.github/workflows/release.yaml`

**Triggered:** On push to main

**Process:**

1. Analyze commits (conventional format: `feat:`, `fix:`, `BREAKING CHANGE:`)
2. Generate release notes
3. Update CHANGELOG.md
4. Create GitHub release and tag
5. Commit version bump

**Release rules:**

- `feat:` → MINOR bump (0.1.0 → 0.2.0)
- `fix:` → PATCH bump (0.1.0 → 0.1.1)
- `BREAKING CHANGE:` → MAJOR bump (0.1.0 → 1.0.0)
- Other commits → no release

## Testing & Validation

### Manual template validation

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
```

### Manual stack deployment (with S3 code)

```bash
# Create prerequisites first
aws iam create-role --role-name lambda-exec-role \
  --assume-role-policy-document '{"Version":"2012-10-17",...}'

aws logs create-log-group --log-group-name /aws/lambda/my-function

# Upload code
aws s3 cp lambda-function.zip s3://my-bucket/lambda-code/

# Deploy Lambda function
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-lambda-dev \
  --parameter-overrides \
    file://cloudformation/lambda-parameters-dev.json \
    S3Bucket=my-bucket \
    S3Key=lambda-code/lambda-function.zip \
    LambdaLogGroup=/aws/lambda/my-function \
    IAMRoleArn=arn:aws:iam::123456789012:role/lambda-exec-role
```

### Quick deployment (with inline boilerplate)

```bash
# Create prerequisites
aws iam create-role --role-name lambda-exec-role \
  --assume-role-policy-document '{"Version":"2012-10-17",...}'

aws logs create-log-group --log-group-name /aws/lambda/my-function

# Deploy with inline boilerplate code
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-lambda-dev \
  --parameter-overrides \
    LambdaLogGroup=/aws/lambda/my-function \
    IAMRoleArn=arn:aws:iam::123456789012:role/lambda-exec-role
```

The CI workflow (ci.yaml) runs the full cycle automatically on PR, then cleans up.

## AWS Credentials & Environment Variables

**GitHub environment variables required in `ci` environment:**

- `AWS_REGION`: CloudFormation deployment region
- `AWS_ACCOUNT_ID`: AWS account to deploy into
- `OIDC_ROLE_NAME`: IAM role name for OIDC trust (uses `arn:aws:iam::{ACCOUNT_ID}:role/{ROLE_NAME}`)
- `CFN_TEMPLATES_S3_BUCKET`: S3 bucket where templates are stored

**OIDC setup:** The CI workflow uses AWS OIDC for keyless auth. The GitHub OIDC provider must trust the specified role.

## Conventional Commits & Release Flow

This repo enforces conventional commits to drive semantic versioning:

```bash
npx cz commit
```

Commit types:

- `feat: add support for X` → triggers MINOR release
- `fix: correct behavior of Y` → triggers PATCH release
- `chore: update deps` → no release
- `docs: clarify README` → no release

Only commits to `main` trigger releases. Feature branches use this format but releases happen on merge to main.

## When Modifying Templates

1. **Edit the template YAML** in `cloudformation/template.yaml`
2. **Update parameter files** in `cloudformation/lambda-parameters-*.json` if new parameters added
3. **Test locally** with `aws cloudformation validate-template`
4. **Create a PR** with conventional commit message (e.g., `feat: add memory size parameter`)
5. **CI validates and deploys** to dev environment automatically
6. **Merge to main** → release workflow creates version tag and GitHub release

## Guidelines for Working with This Template

### When to Update `template.yaml`

- ✅ Adding new Lambda configuration parameters
- ✅ Changing function properties (memory, timeout, layers, etc.)
- ✅ Updating runtimes or handler configuration
- ✅ Modifying VPC or logging configuration
- ✅ Fixing bugs or improving conditions

### When to Update Parameter Files

- ✅ Changing environment-specific values (memory, timeout)
- ✅ Adding environment-specific VPC or DLQ configuration
- ✅ Updating S3 bucket or code paths
- ✅ Modifying environment variables per environment

### When to Update README.md

- ✅ Adding documentation for new parameters
- ✅ Adding new deployment examples
- ✅ Clarifying usage or best practices
- ✅ Updating architecture diagrams or explanations

### When NOT to Create New Template Files

- ❌ Don't create separate lambda templates
- ❌ Don't create environment-specific templates
- Use parameter files and conditions instead

## Dev Container

Pre-configured with:

- Node.js 20
- GitHub Copilot extension

Use via VS Code: `code --remote-container-url <repo-url>`

## Current Branch

Main branch is the release branch. Feature work branches from here and merges back via PR. Branch naming follows: `{type}/CFN-{issue-number}-{slug}` (e.g., `feature/CFN-42-add-memory-parameter`).

## Key Decisions

1. **Single Template File**: All Lambda configuration in one `template.yaml` for easier maintenance
2. **Hybrid IAM Role Management**: Auto-create roles for CI/CD (with CiSuffix), externally-manage for production (without CiSuffix)
3. **Dual Deployment Modes**: Support both S3-based and inline boilerplate code for flexibility
4. **Python/Node.js Only**: Limited runtimes to most commonly-used (Python 3.12/3.13, Node.js 20.x/24.x)
5. **Parameterized Configuration**: Environment-specific values in parameter files, not hardcoded
6. **CommaDelimitedList Parameters**: Use CloudFormation's CommaDelimitedList type for multi-value parameters (VPC subnets, security groups, Lambda Layer ARNs)
7. **Conditional VPC Configuration**: VPC only configured when EnableVPC=true AND both subnet and security group IDs are non-empty
8. **CiSuffix for Ephemeral Deployments**: Optional suffix for CI/CD testing creates unique function names without colliding with permanent deployments
9. **Semantic Versioning**: Automated releases on main branch based on conventional commits
