# Part 99: DevOps และ Infrastructure (Steps 1951-1970)

## บทนำ

DevOps คือวัฒนธรรมและชุดของ practices ที่ช่วยให้ทีม development และ operations ทำงานร่วมกันได้อย่างมีประสิทธิภาพ ในบทนี้เราจะเรียนรู้การใช้ Infrastructure as Code, AWS services สำหรับ JavaScript developers, Serverless computing, Monitoring, และ best practices ต่างๆ

---

## Step 1951: DevOps Culture และ Practices

### CI/CD Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  AWS_REGION: ap-southeast-1

jobs:
  # Job 1: Test
  test:
    name: Test
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run lint
        run: npm run lint
      
      - name: Run type check
        run: npm run type-check
      
      - name: Run unit tests
        run: npm run test:unit -- --coverage
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
  
  # Job 2: Build
  build:
    name: Build
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          NEXT_PUBLIC_API_URL: ${{ vars.API_URL }}
      
      - name: Build Docker image
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest
      
      - name: Save Docker image
        run: docker save myapp:${{ github.sha }} | gzip > image.tar.gz
      
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: docker-image
          path: image.tar.gz
          retention-days: 1
  
  # Job 3: Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: docker-image
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Push to ECR
        run: |
          ECR_REGISTRY=${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG=${{ github.sha }}
          
          docker load < image.tar.gz
          docker tag myapp:$IMAGE_TAG $ECR_REGISTRY/myapp:$IMAGE_TAG
          docker tag myapp:$IMAGE_TAG $ECR_REGISTRY/myapp:staging
          docker push $ECR_REGISTRY/myapp:$IMAGE_TAG
          docker push $ECR_REGISTRY/myapp:staging
      
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster staging-cluster \
            --service myapp-service \
            --force-new-deployment
      
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster staging-cluster \
            --services myapp-service
  
  # Job 4: Deploy to Production
  deploy-production:
    name: Deploy to Production
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Blue/Green Deployment
        run: |
          # สร้าง new task definition
          TASK_DEF=$(aws ecs describe-task-definition \
            --task-definition myapp \
            --query taskDefinition)
          
          NEW_TASK_DEF=$(echo $TASK_DEF | \
            jq --arg IMAGE "$ECR_REGISTRY/myapp:${{ github.sha }}" \
            '.containerDefinitions[0].image = $IMAGE')
          
          NEW_TASK_ARN=$(aws ecs register-task-definition \
            --cli-input-json "$NEW_TASK_DEF" \
            --query 'taskDefinition.taskDefinitionArn' \
            --output text)
          
          # อัปเดต service
          aws ecs update-service \
            --cluster production-cluster \
            --service myapp-service \
            --task-definition $NEW_TASK_ARN
```

---

## Step 1952: Infrastructure as Code - Terraform

### Terraform สำหรับ AWS

```hcl
# infrastructure/main.tf
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # Remote state
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "myapp/terraform.tfstate"
    region         = "ap-southeast-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = "MyApp"
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# Variables
variable "environment" {
  type    = string
  default = "production"
}

variable "aws_region" {
  type    = string
  default = "ap-southeast-1"
}

variable "app_name" {
  type    = string
  default = "myapp"
}

# VPC
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "${var.app_name}-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["ap-southeast-1a", "ap-southeast-1b", "ap-southeast-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  single_nat_gateway = var.environment != "production"
  
  enable_dns_hostnames = true
  enable_dns_support   = true
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "${var.app_name}-${var.environment}"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

# RDS PostgreSQL
resource "aws_db_instance" "postgres" {
  identifier           = "${var.app_name}-postgres"
  engine               = "postgres"
  engine_version       = "15.3"
  instance_class       = "db.t3.medium"
  allocated_storage    = 20
  max_allocated_storage = 100
  storage_encrypted    = true
  
  db_name  = "myapp"
  username = "admin"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window           = "02:00-04:00"
  maintenance_window      = "sun:04:00-sun:06:00"
  
  deletion_protection = var.environment == "production"
  skip_final_snapshot = var.environment != "production"
  
  performance_insights_enabled = true
  
  tags = {
    Name = "${var.app_name}-postgres"
  }
}

# ElastiCache Redis
resource "aws_elasticache_cluster" "redis" {
  cluster_id           = "${var.app_name}-redis"
  engine               = "redis"
  node_type            = "cache.t3.micro"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis7"
  engine_version       = "7.0"
  port                 = 6379
  
  subnet_group_name  = aws_elasticache_subnet_group.main.name
  security_group_ids = [aws_security_group.redis.id]
}

# S3 Bucket
resource "aws_s3_bucket" "assets" {
  bucket = "${var.app_name}-assets-${var.environment}"
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "assets" {
  bucket = aws_s3_bucket.assets.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# CloudFront
resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name              = aws_s3_bucket.assets.bucket_regional_domain_name
    origin_id                = "S3Origin"
    origin_access_control_id = aws_cloudfront_origin_access_control.main.id
  }
  
  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"
  
  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "S3Origin"
    
    forwarded_values {
      query_string = false
      cookies { forward = "none" }
    }
    
    viewer_protocol_policy = "redirect-to-https"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
    compress               = true
  }
  
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
  
  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
```

---

## Step 1953: Pulumi - TypeScript IaC

### Infrastructure ด้วย TypeScript

```typescript
// infrastructure/index.ts
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

const config = new pulumi.Config();
const environment = config.require("environment");
const dbPassword = config.requireSecret("dbPassword");

// VPC
const vpc = new awsx.ec2.Vpc("main-vpc", {
  numberOfAvailabilityZones: 2,
  cidrBlock: "10.0.0.0/16",
  subnetSpecs: [
    { type: awsx.ec2.SubnetType.Public, name: "public" },
    { type: awsx.ec2.SubnetType.Private, name: "private" }
  ],
  tags: { Environment: environment }
});

// ECS Cluster
const cluster = new aws.ecs.Cluster("app-cluster", {
  settings: [{
    name: "containerInsights",
    value: "enabled"
  }]
});

// ECR Repository
const repo = new aws.ecr.Repository("app-repo", {
  imageTagMutability: "MUTABLE",
  imageScanningConfiguration: {
    scanOnPush: true
  }
});

// Build และ push Docker image
const image = new awsx.ecr.Image("app-image", {
  repositoryUrl: repo.repositoryUrl,
  context: "../app",
  platform: "linux/amd64"
});

// Application Load Balancer
const alb = new awsx.lb.ApplicationLoadBalancer("app-alb", {
  subnetIds: vpc.publicSubnetIds
});

// ECS Service
const service = new awsx.ecs.FargateService("app-service", {
  cluster: cluster.arn,
  assignPublicIp: false,
  subnets: vpc.privateSubnetIds,
  taskDefinitionArgs: {
    container: {
      name: "app",
      image: image.imageUri,
      cpu: 256,
      memory: 512,
      essential: true,
      portMappings: [{
        containerPort: 3000,
        protocol: "tcp",
        targetGroup: alb.defaultTargetGroup
      }],
      environment: [
        { name: "NODE_ENV", value: environment },
        { name: "PORT", value: "3000" }
      ],
      secrets: [
        { name: "DATABASE_URL", valueFrom: dbSecretArn },
        { name: "JWT_SECRET", valueFrom: jwtSecretArn }
      ],
      logConfiguration: {
        logDriver: "awslogs",
        options: {
          "awslogs-group": logGroup.name,
          "awslogs-region": aws.config.region!,
          "awslogs-stream-prefix": "app"
        }
      }
    }
  },
  desiredCount: 2
});

// Auto Scaling
const scalingTarget = new aws.appautoscaling.Target("scaling-target", {
  maxCapacity: 10,
  minCapacity: 2,
  resourceId: pulumi.interpolate`service/${cluster.name}/${service.service.name}`,
  scalableDimension: "ecs:service:DesiredCount",
  serviceNamespace: "ecs"
});

// CPU-based scaling
new aws.appautoscaling.Policy("cpu-scaling", {
  name: "cpu-scaling",
  policyType: "TargetTrackingScaling",
  resourceId: scalingTarget.resourceId,
  scalableDimension: scalingTarget.scalableDimension,
  serviceNamespace: scalingTarget.serviceNamespace,
  targetTrackingScalingPolicyConfiguration: {
    predefinedMetricSpecification: {
      predefinedMetricType: "ECSServiceAverageCPUUtilization"
    },
    targetValue: 70.0,
    scaleInCooldown: 300,
    scaleOutCooldown: 60
  }
});

// Outputs
export const albUrl = alb.loadBalancer.dnsName;
export const imageUri = image.imageUri;
export const vpcId = vpc.vpcId;
```

---

## Step 1954: AWS Lambda (Serverless)

### Serverless Functions ด้วย Node.js

```typescript
// functions/api/users.ts
import { APIGatewayProxyHandler, APIGatewayProxyResult } from 'aws-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand, PutCommand, QueryCommand } from '@aws-sdk/lib-dynamodb';
import { z } from 'zod';

const client = new DynamoDBClient({ region: process.env.AWS_REGION });
const docClient = DynamoDBDocumentClient.from(client);
const TABLE_NAME = process.env.TABLE_NAME!;

// Response helper
function response(statusCode: number, body: unknown): APIGatewayProxyResult {
  return {
    statusCode,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Headers': 'Content-Type,Authorization',
      'Access-Control-Allow-Methods': 'GET,POST,PUT,DELETE,OPTIONS'
    },
    body: JSON.stringify(body)
  };
}

// Schema validation
const CreateUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  role: z.enum(['user', 'admin']).default('user')
});

// GET /users/{userId}
export const getUser: APIGatewayProxyHandler = async (event) => {
  const userId = event.pathParameters?.userId;

  if (!userId) {
    return response(400, { error: 'userId is required' });
  }

  try {
    const result = await docClient.send(
      new GetCommand({
        TableName: TABLE_NAME,
        Key: { pk: `USER#${userId}`, sk: 'PROFILE' }
      })
    );

    if (!result.Item) {
      return response(404, { error: 'User not found' });
    }

    return response(200, {
      id: userId,
      name: result.Item.name,
      email: result.Item.email,
      role: result.Item.role,
      createdAt: result.Item.createdAt
    });
  } catch (error) {
    console.error('Error getting user:', error);
    return response(500, { error: 'Internal server error' });
  }
};

// POST /users
export const createUser: APIGatewayProxyHandler = async (event) => {
  if (!event.body) {
    return response(400, { error: 'Request body is required' });
  }

  let data;
  try {
    data = CreateUserSchema.parse(JSON.parse(event.body));
  } catch (error) {
    return response(400, { error: 'Invalid request data', details: error });
  }

  const userId = `usr_${Date.now()}_${Math.random().toString(36).substring(2)}`;
  const now = new Date().toISOString();

  try {
    await docClient.send(
      new PutCommand({
        TableName: TABLE_NAME,
        Item: {
          pk: `USER#${userId}`,
          sk: 'PROFILE',
          id: userId,
          name: data.name,
          email: data.email,
          role: data.role,
          createdAt: now,
          updatedAt: now,
          gsi1pk: `EMAIL#${data.email}`,
          gsi1sk: `USER#${userId}`
        },
        ConditionExpression: 'attribute_not_exists(pk)'
      })
    );

    return response(201, {
      id: userId,
      name: data.name,
      email: data.email,
      role: data.role,
      createdAt: now
    });
  } catch (error: any) {
    if (error.name === 'ConditionalCheckFailedException') {
      return response(409, { error: 'User already exists' });
    }
    console.error('Error creating user:', error);
    return response(500, { error: 'Internal server error' });
  }
};

// Lambda Middleware Pattern
type Handler = APIGatewayProxyHandler;

function withAuth(handler: Handler): Handler {
  return async (event, context, callback) => {
    const token = event.headers?.Authorization?.replace('Bearer ', '');

    if (!token) {
      return response(401, { error: 'Unauthorized' });
    }

    try {
      const payload = verifyJWT(token, process.env.JWT_SECRET!);
      event.requestContext = {
        ...event.requestContext,
        authorizer: { user: payload }
      };
      return handler(event, context, callback);
    } catch {
      return response(401, { error: 'Invalid token' });
    }
  };
}

function withLogger(handler: Handler): Handler {
  return async (event, context, callback) => {
    const start = Date.now();
    console.log('Request:', {
      method: event.httpMethod,
      path: event.path,
      params: event.pathParameters,
      query: event.queryStringParameters
    });

    const result = await handler(event, context, callback);

    console.log('Response:', {
      statusCode: (result as APIGatewayProxyResult)?.statusCode,
      duration: Date.now() - start
    });

    return result;
  };
}

// ใช้ middleware
export const getUserWithAuth = withLogger(withAuth(getUser));
```

---

## Step 1955: Serverless Framework

### serverless.yml Configuration

```yaml
# serverless.yml
service: myapp-api

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs20.x
  region: ap-southeast-1
  stage: ${opt:stage, 'dev'}
  
  memorySize: 512
  timeout: 30
  
  environment:
    NODE_ENV: ${self:provider.stage}
    TABLE_NAME: ${self:service}-${self:provider.stage}-users
    JWT_SECRET: ${ssm:/myapp/${self:provider.stage}/jwt-secret}
    DATABASE_URL: ${ssm:/myapp/${self:provider.stage}/database-url~true}
  
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:GetItem
            - dynamodb:PutItem
            - dynamodb:UpdateItem
            - dynamodb:DeleteItem
            - dynamodb:Query
            - dynamodb:Scan
          Resource:
            - !GetAtt UsersTable.Arn
            - !Sub "${UsersTable.Arn}/index/*"
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:PutObject
            - s3:DeleteObject
          Resource: !Sub "arn:aws:s3:::${self:service}-${self:provider.stage}-assets/*"
        - Effect: Allow
          Action:
            - sqs:SendMessage
            - sqs:ReceiveMessage
            - sqs:DeleteMessage
          Resource: !GetAtt EmailQueue.Arn
  
  logs:
    restApi:
      accessLogging: true
      format: '{ "requestId":"$context.requestId", "ip": "$context.identity.sourceIp", "httpMethod":"$context.httpMethod", "path":"$context.path", "status":"$context.status" }'

package:
  individually: true
  patterns:
    - '!node_modules/**'
    - '!tests/**'
    - '!*.test.ts'

plugins:
  - serverless-esbuild
  - serverless-offline
  - serverless-domain-manager

custom:
  esbuild:
    bundle: true
    minify: true
    sourcemap: true
    target: node20
    platform: node
    
  customDomain:
    domainName: api.myapp.com
    stage: ${self:provider.stage}
    createRoute53Record: true
    certificateName: '*.myapp.com'

functions:
  # Users API
  getUser:
    handler: src/functions/users.getUser
    events:
      - http:
          path: users/{userId}
          method: GET
          cors: true
          authorizer:
            name: jwtAuthorizer
            type: TOKEN
  
  createUser:
    handler: src/functions/users.createUser
    events:
      - http:
          path: users
          method: POST
          cors: true
  
  # Email Worker
  sendEmail:
    handler: src/functions/email.send
    reservedConcurrency: 10
    events:
      - sqs:
          arn: !GetAtt EmailQueue.Arn
          batchSize: 10
          functionResponseType: ReportBatchItemFailures
  
  # Scheduled Jobs
  dailyReport:
    handler: src/functions/reports.daily
    events:
      - schedule:
          rate: cron(0 8 * * ? *)  # ทุกวันเวลา 08:00 UTC
          enabled: true
  
  # Authorizer
  jwtAuthorizer:
    handler: src/functions/auth.authorize
  
  # Webhook
  stripeWebhook:
    handler: src/functions/stripe.webhook
    events:
      - http:
          path: webhooks/stripe
          method: POST

resources:
  Resources:
    # DynamoDB
    UsersTable:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:provider.environment.TABLE_NAME}
        BillingMode: PAY_PER_REQUEST
        PointInTimeRecoverySpecification:
          PointInTimeRecoveryEnabled: true
        AttributeDefinitions:
          - AttributeName: pk
            AttributeType: S
          - AttributeName: sk
            AttributeType: S
          - AttributeName: gsi1pk
            AttributeType: S
          - AttributeName: gsi1sk
            AttributeType: S
        KeySchema:
          - AttributeName: pk
            KeyType: HASH
          - AttributeName: sk
            KeyType: RANGE
        GlobalSecondaryIndexes:
          - IndexName: gsi1
            KeySchema:
              - AttributeName: gsi1pk
                KeyType: HASH
              - AttributeName: gsi1sk
                KeyType: RANGE
            Projection:
              ProjectionType: ALL
        TimeToLiveSpecification:
          AttributeName: ttl
          Enabled: true
    
    # SQS Queue
    EmailQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-${self:provider.stage}-email
        VisibilityTimeout: 120
        MessageRetentionPeriod: 86400
        RedrivePolicy:
          deadLetterTargetArn: !GetAtt EmailDLQ.Arn
          maxReceiveCount: 3
    
    EmailDLQ:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:service}-${self:provider.stage}-email-dlq
    
    # S3 Bucket
    AssetsBucket:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: ${self:service}-${self:provider.stage}-assets
        CorsConfiguration:
          CorsRules:
            - AllowedHeaders: ['*']
              AllowedMethods: [GET, PUT, POST]
              AllowedOrigins: ['*']
              MaxAge: 3000
```

---

## Step 1956: AWS SDK v3

### การใช้งาน AWS Services

```typescript
// src/lib/aws.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';
import { S3Client } from '@aws-sdk/client-s3';
import { SQSClient } from '@aws-sdk/client-sqs';
import { SNSClient } from '@aws-sdk/client-sns';
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';
import { SESClient, SendEmailCommand } from '@aws-sdk/client-ses';

const REGION = process.env.AWS_REGION || 'ap-southeast-1';

// DynamoDB
export const dynamoDB = DynamoDBDocumentClient.from(
  new DynamoDBClient({ region: REGION }),
  {
    marshallOptions: {
      removeUndefinedValues: true,
      convertClassInstanceToMap: true
    }
  }
);

// S3
export const s3 = new S3Client({ region: REGION });

// SQS
export const sqs = new SQSClient({ region: REGION });

// SNS
export const sns = new SNSClient({ region: REGION });

// SES
export const ses = new SESClient({ region: REGION });

// src/lib/s3.ts
import { 
  PutObjectCommand,
  GetObjectCommand,
  DeleteObjectCommand,
  ListObjectsV2Command,
  HeadObjectCommand
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { s3 } from './aws';

const BUCKET = process.env.S3_BUCKET!;

export const S3Service = {
  // Upload ไฟล์
  async upload(
    key: string,
    body: Buffer | ReadableStream | string,
    contentType: string,
    metadata?: Record<string, string>
  ) {
    await s3.send(new PutObjectCommand({
      Bucket: BUCKET,
      Key: key,
      Body: body,
      ContentType: contentType,
      Metadata: metadata,
      ServerSideEncryption: 'AES256'
    }));

    return `https://${BUCKET}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`;
  },

  // สร้าง pre-signed URL สำหรับ upload
  async getPresignedUploadUrl(key: string, contentType: string, expiresIn = 3600) {
    const command = new PutObjectCommand({
      Bucket: BUCKET,
      Key: key,
      ContentType: contentType
    });

    return getSignedUrl(s3, command, { expiresIn });
  },

  // สร้าง pre-signed URL สำหรับ download
  async getPresignedDownloadUrl(key: string, expiresIn = 3600) {
    const command = new GetObjectCommand({
      Bucket: BUCKET,
      Key: key
    });

    return getSignedUrl(s3, command, { expiresIn });
  },

  // ลบไฟล์
  async delete(key: string) {
    await s3.send(new DeleteObjectCommand({
      Bucket: BUCKET,
      Key: key
    }));
  },

  // List ไฟล์
  async list(prefix: string) {
    const result = await s3.send(new ListObjectsV2Command({
      Bucket: BUCKET,
      Prefix: prefix
    }));

    return result.Contents || [];
  },

  // ตรวจสอบว่าไฟล์มีอยู่หรือไม่
  async exists(key: string) {
    try {
      await s3.send(new HeadObjectCommand({
        Bucket: BUCKET,
        Key: key
      }));
      return true;
    } catch {
      return false;
    }
  }
};

// src/lib/sqs.ts
import {
  SendMessageCommand,
  ReceiveMessageCommand,
  DeleteMessageCommand,
  SendMessageBatchCommand
} from '@aws-sdk/client-sqs';
import { sqs } from './aws';

const QUEUE_URL = process.env.SQS_QUEUE_URL!;

export const SQSService = {
  async send(message: unknown, delaySeconds = 0, attributes?: Record<string, string>) {
    const result = await sqs.send(new SendMessageCommand({
      QueueUrl: QUEUE_URL,
      MessageBody: JSON.stringify(message),
      DelaySeconds: delaySeconds,
      MessageAttributes: attributes
        ? Object.fromEntries(
            Object.entries(attributes).map(([k, v]) => [
              k,
              { DataType: 'String', StringValue: v }
            ])
          )
        : undefined
    }));

    return result.MessageId;
  },

  async sendBatch(messages: unknown[]) {
    const entries = messages.map((msg, index) => ({
      Id: index.toString(),
      MessageBody: JSON.stringify(msg)
    }));

    const result = await sqs.send(new SendMessageBatchCommand({
      QueueUrl: QUEUE_URL,
      Entries: entries
    }));

    return {
      successful: result.Successful,
      failed: result.Failed
    };
  },

  async receive(maxMessages = 10, waitTimeSeconds = 20) {
    const result = await sqs.send(new ReceiveMessageCommand({
      QueueUrl: QUEUE_URL,
      MaxNumberOfMessages: maxMessages,
      WaitTimeSeconds: waitTimeSeconds,
      AttributeNames: ['All'],
      MessageAttributeNames: ['All']
    }));

    return result.Messages || [];
  },

  async delete(receiptHandle: string) {
    await sqs.send(new DeleteMessageCommand({
      QueueUrl: QUEUE_URL,
      ReceiptHandle: receiptHandle
    }));
  }
};

// Email ด้วย SES
export const EmailService = {
  async send(options: {
    to: string[];
    subject: string;
    html: string;
    text?: string;
    from?: string;
    replyTo?: string;
  }) {
    await ses.send(new SendEmailCommand({
      Source: options.from || process.env.EMAIL_FROM!,
      Destination: {
        ToAddresses: options.to
      },
      ReplyToAddresses: options.replyTo ? [options.replyTo] : undefined,
      Message: {
        Subject: {
          Data: options.subject,
          Charset: 'UTF-8'
        },
        Body: {
          Html: {
            Data: options.html,
            Charset: 'UTF-8'
          },
          Text: options.text ? {
            Data: options.text,
            Charset: 'UTF-8'
          } : undefined
        }
      }
    }));
  }
};
```

---

## Step 1957: Monitoring ด้วย Sentry

### Error Tracking และ Performance Monitoring

```typescript
// src/lib/monitoring.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

// Initialize Sentry
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.RELEASE_VERSION,
  
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Sentry.Integrations.Express({ app }),
    new Sentry.Integrations.Prisma({ client: prisma }),
    new ProfilingIntegration()
  ],
  
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  profilesSampleRate: 0.1,
  
  beforeSend(event, hint) {
    // ซ่อน sensitive data
    if (event.request?.data) {
      const data = event.request.data as Record<string, unknown>;
      if (data.password) data.password = '[FILTERED]';
      if (data.creditCard) data.creditCard = '[FILTERED]';
    }
    return event;
  }
});

// Express middleware
app.use(Sentry.Handlers.requestHandler());
app.use(Sentry.Handlers.tracingHandler());

// Error handler (ต้องใส่หลังสุด)
app.use(Sentry.Handlers.errorHandler({
  shouldHandleError(error) {
    return error.status !== undefined || error.statusCode !== undefined
      ? error.status >= 500 || error.statusCode >= 500
      : true;
  }
}));

// Custom error tracking
function captureError(error: Error, context?: Record<string, unknown>) {
  Sentry.withScope(scope => {
    if (context) {
      Object.entries(context).forEach(([key, value]) => {
        scope.setExtra(key, value);
      });
    }
    Sentry.captureException(error);
  });
}

// Performance monitoring
function measurePerformance<T>(
  name: string,
  operation: () => Promise<T>
): Promise<T> {
  const transaction = Sentry.startTransaction({ name, op: 'function' });
  Sentry.getCurrentHub().configureScope(scope => {
    scope.setSpan(transaction);
  });

  return operation()
    .then(result => {
      transaction.setStatus('ok');
      return result;
    })
    .catch(error => {
      transaction.setStatus('internal_error');
      throw error;
    })
    .finally(() => {
      transaction.finish();
    });
}

// DataDog metrics
import { StatsD } from 'hot-shots';

const dogstatsd = new StatsD({
  host: process.env.DD_AGENT_HOST || 'localhost',
  port: 8125,
  prefix: 'myapp.',
  globalTags: {
    env: process.env.NODE_ENV || 'development',
    service: 'api'
  }
});

export const metrics = {
  increment(metric: string, tags?: Record<string, string>) {
    const tagArray = tags
      ? Object.entries(tags).map(([k, v]) => `${k}:${v}`)
      : [];
    dogstatsd.increment(metric, 1, tagArray);
  },

  gauge(metric: string, value: number, tags?: Record<string, string>) {
    const tagArray = tags
      ? Object.entries(tags).map(([k, v]) => `${k}:${v}`)
      : [];
    dogstatsd.gauge(metric, value, tagArray);
  },

  timing(metric: string, time: number, tags?: Record<string, string>) {
    const tagArray = tags
      ? Object.entries(tags).map(([k, v]) => `${k}:${v}`)
      : [];
    dogstatsd.timing(metric, time, tagArray);
  },

  async measure<T>(
    metric: string,
    fn: () => Promise<T>,
    tags?: Record<string, string>
  ): Promise<T> {
    const start = Date.now();
    try {
      const result = await fn();
      this.timing(metric, Date.now() - start, { ...tags, status: 'success' });
      this.increment(`${metric}.success`, tags);
      return result;
    } catch (error) {
      this.timing(metric, Date.now() - start, { ...tags, status: 'error' });
      this.increment(`${metric}.error`, tags);
      throw error;
    }
  }
};
```

---

## Step 1958: Logging Strategies

### Structured Logging

```typescript
// src/lib/logger.ts
import pino from 'pino';
import { AsyncLocalStorage } from 'async_hooks';

const requestContext = new AsyncLocalStorage<Record<string, unknown>>();

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  
  // Structured logging
  formatters: {
    level(label) {
      return { level: label };
    },
    bindings(bindings) {
      return {
        pid: bindings.pid,
        hostname: bindings.hostname,
        service: process.env.SERVICE_NAME || 'api',
        version: process.env.APP_VERSION || '1.0.0',
        environment: process.env.NODE_ENV
      };
    }
  },
  
  // Redact sensitive fields
  redact: {
    paths: [
      'password',
      'token',
      'authorization',
      '*.password',
      '*.token',
      'req.headers.authorization',
      'req.body.password',
      'res.headers["set-cookie"]'
    ],
    censor: '[REDACTED]'
  },
  
  // Timestamp
  timestamp: pino.stdTimeFunctions.isoTime,
  
  // Serializers
  serializers: {
    req: pino.stdSerializers.req,
    res: pino.stdSerializers.res,
    err: pino.stdSerializers.err
  }
});

// Context-aware logger
function createContextLogger() {
  return new Proxy(logger, {
    get(target, prop) {
      if (typeof target[prop as keyof typeof target] === 'function') {
        return (...args: unknown[]) => {
          const context = requestContext.getStore();
          if (context) {
            return (target[prop as keyof typeof target] as Function)(
              { ...context, ...(typeof args[0] === 'object' ? args[0] : {}) },
              ...(typeof args[0] === 'object' ? args.slice(1) : args)
            );
          }
          return (target[prop as keyof typeof target] as Function)(...args);
        };
      }
      return target[prop as keyof typeof target];
    }
  });
}

export const log = createContextLogger();

// Express middleware
export function requestLogger() {
  return (req: Request, res: Response, next: NextFunction) => {
    const context = {
      requestId: req.headers['x-request-id'] || generateId(),
      userId: req.user?.id,
      sessionId: req.sessionID,
      ip: req.ip
    };

    requestContext.run(context, () => {
      const start = Date.now();

      res.on('finish', () => {
        log.info({
          req,
          res,
          responseTime: Date.now() - start
        }, 'Request completed');
      });

      next();
    });
  };
}

// Log aggregation สำหรับ CloudWatch
const cloudWatchTransport = pino.transport({
  target: 'pino-cloudwatch',
  options: {
    group: '/aws/lambda/myapp',
    stream: `api-${process.env.NODE_ENV}`,
    region: process.env.AWS_REGION
  }
});
```

---

## Step 1959: Docker

### Dockerfile สำหรับ Node.js

```dockerfile
# Dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

# ติดตั้ง dependencies ก่อน (cache layer)
COPY package*.json ./
RUN npm ci --only=production

# Copy source
COPY tsconfig.json ./
COPY src ./src

# Build
RUN npm run build

# Stage 2: Production
FROM node:20-alpine AS production

# Security: ไม่ใช้ root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy artifacts จาก builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./

# Set ownership
RUN chown -R appuser:appgroup /app

USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

EXPOSE 3000

ENV NODE_ENV=production

CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml สำหรับ development
version: '3.9'

services:
  api:
    build:
      context: .
      target: builder
    volumes:
      - ./src:/app/src
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://user:pass@postgres:5432/myapp
      REDIS_URL: redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: npm run dev

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass redispassword
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "redispassword", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "80:80"
    depends_on:
      - api

volumes:
  postgres_data:
  redis_data:
```

---

## Step 1960-1970: Security Best Practices บน AWS

### Security Implementation

```typescript
// src/middleware/security.ts
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import slowDown from 'express-slow-down';
import { createClient } from 'redis';
import RedisStore from 'rate-limit-redis';

const redis = createClient({ url: process.env.REDIS_URL });

// Helmet สำหรับ HTTP security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'nonce-{nonce}'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.myapp.com'],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 นาที
  max: 100, // 100 requests ต่อ window
  standardHeaders: true,
  legacyHeaders: false,
  store: new RedisStore({
    sendCommand: (...args: string[]) => redis.sendCommand(args)
  }),
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too many requests',
      retryAfter: Math.ceil(req.rateLimit.resetTime.getTime() / 1000)
    });
  }
});

// Slow down requests ก่อน rate limit
const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000,
  delayAfter: 50,
  delayMs: 500
});

app.use('/api/', limiter);
app.use('/api/', speedLimiter);

// Input sanitization
import { sanitize } from 'isomorphic-dompurify';
import validator from 'validator';

function sanitizeInput(data: Record<string, unknown>): Record<string, unknown> {
  const sanitized: Record<string, unknown> = {};

  for (const [key, value] of Object.entries(data)) {
    if (typeof value === 'string') {
      sanitized[key] = sanitize(validator.trim(value));
    } else if (typeof value === 'object' && value !== null) {
      sanitized[key] = sanitizeInput(value as Record<string, unknown>);
    } else {
      sanitized[key] = value;
    }
  }

  return sanitized;
}

// SQL Injection prevention (ใช้ parameterized queries เสมอ)
// ❌ NEVER DO THIS:
// const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ DO THIS:
// const result = await prisma.user.findUnique({ where: { id: userId } });

// AWS WAF Rules (via Terraform/CDK)
// - Rate limiting
// - SQL injection protection
// - XSS protection
// - Geo blocking
// - IP reputation lists

// Secrets Management
async function getSecret(secretId: string): Promise<Record<string, string>> {
  const client = new SecretsManagerClient({ region: process.env.AWS_REGION });
  const response = await client.send(
    new GetSecretValueCommand({ SecretId: secretId })
  );

  if (response.SecretString) {
    return JSON.parse(response.SecretString);
  }

  throw new Error('Secret not found');
}

// KMS Encryption
import { KMSClient, EncryptCommand, DecryptCommand } from '@aws-sdk/client-kms';

const kms = new KMSClient({ region: process.env.AWS_REGION });
const KEY_ID = process.env.KMS_KEY_ID!;

async function encrypt(plaintext: string): Promise<string> {
  const response = await kms.send(new EncryptCommand({
    KeyId: KEY_ID,
    Plaintext: Buffer.from(plaintext)
  }));

  return Buffer.from(response.CiphertextBlob!).toString('base64');
}

async function decrypt(ciphertext: string): Promise<string> {
  const response = await kms.send(new DecryptCommand({
    CiphertextBlob: Buffer.from(ciphertext, 'base64'),
    KeyId: KEY_ID
  }));

  return Buffer.from(response.Plaintext!).toString('utf-8');
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Serverless API
สร้าง Serverless REST API ด้วย AWS Lambda:
- CRUD endpoints
- DynamoDB
- API Gateway
- Authentication ด้วย JWT

### แบบฝึกหัดที่ 2: CI/CD Pipeline
ตั้งค่า CI/CD pipeline สมบูรณ์:
- GitHub Actions
- Unit + Integration tests
- Docker build
- Deploy ไปยัง AWS ECS

### แบบฝึกหัดที่ 3: Infrastructure as Code
สร้าง infrastructure ด้วย Pulumi:
- VPC + Subnets
- ECS + Fargate
- RDS + ElastiCache
- CloudFront + S3

---

## สรุป

DevOps และ Infrastructure เป็นส่วนสำคัญที่ทำให้ application สามารถ scale ได้และ reliable ในบทนี้เราเรียนรู้:

1. **CI/CD** ด้วย GitHub Actions
2. **Terraform** สำหรับ Infrastructure as Code
3. **Pulumi** สำหรับ TypeScript-based IaC
4. **AWS Lambda** และ Serverless patterns
5. **Serverless Framework** สำหรับ deployment ง่ายๆ
6. **AWS SDK v3** สำหรับ DynamoDB, S3, SQS, SES
7. **Monitoring** ด้วย Sentry และ DataDog
8. **Structured Logging** ด้วย Pino
9. **Docker** สำหรับ containerization
10. **Security Best Practices** บน AWS
