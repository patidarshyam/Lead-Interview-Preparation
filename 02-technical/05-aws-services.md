# AWS Services

> 📌 **Visual Reference:** [![AWS Services Cheatsheet](../assets/images/aws-services-cheatsheet.svg)](../assets/images/aws-services-cheatsheet.svg)
>
> 📌 **Flow Diagrams:**
>
> | Use Case | Diagram |
> |----------|---------|
> | UC-1: REST API with Caching | [![UC1](../assets/images/aws-use-case-1-rest-api-with-caching.png)](../assets/images/aws-use-case-1-rest-api-with-caching.png) |
> | UC-2: Async Export Pipeline | [![UC2](../assets/images/aws-use-case-2-async-export-pipline.png)](../assets/images/aws-use-case-2-async-export-pipline.png) |
> | UC-3: Event-Driven Fanout | [![UC3](../assets/images/aws-use-case-3-event-driven-fanout.png)](../assets/images/aws-use-case-3-event-driven-fanout.png) |
> | UC-4: VPC Architecture | [![UC4](../assets/images/aws-use-case-4-vpc-architecture.png)](../assets/images/aws-use-case-4-vpc-architecture.png) |
> | UC-5: Serverless — Lambda, API Gateway, DynamoDB | [![UC5](../assets/images/aws-use-case-5-serverless-lembda-api-gateway-dynamodb.png)](../assets/images/aws-use-case-5-serverless-lembda-api-gateway-dynamodb.png) |
> | UC-6: CI/CD Pipeline | [![UC6](../assets/images/aws-use-case-6-ci-cd-pipeline.png)](../assets/images/aws-use-case-6-ci-cd-pipeline.png) |

---

## Q1: Explain your experience with ECS Fargate. When would you choose Fargate vs EC2 launch type?

**Answer:**

**Fargate** — Serverless compute for containers. No EC2 instance management. You define CPU/memory per task.

**Choose Fargate when:**
- Workloads are bursty or unpredictable (e.g., export jobs that spin up on-demand)
- You want zero infrastructure management
- Tasks are short-lived (batch processing, event-driven)
- You need fine-grained per-task resource allocation

**Choose EC2 when:**
- You need GPU or specialized instance types
- Workloads are steady-state with predictable load (cheaper at scale)
- You need access to the host (privileged containers, custom AMIs)
- Large memory/CPU requirements that exceed Fargate limits

**Real-world context:** For export processing, Fargate is ideal — each export request spins up a dedicated container, processes the data, uploads to S3, and terminates. You pay only for execution time. For long-running API servers, EC2 with autoscaling can be more cost-effective.

---

## Q2: How does SQS FIFO differ from standard SQS? When do you use each?

**Answer:**

| Feature | Standard | FIFO |
|---------|----------|------|
| Ordering | Best-effort | Strict within message group |
| Deduplication | None (at-least-once) | Content-based or explicit dedup ID |
| Throughput | Virtually unlimited | 300 msg/s (batching: 3000) per API action, or high-throughput mode |
| Use case | Fanout, notifications | Ordered processing, financial, export pipelines |

**When I use FIFO:** Export pipelines where order matters per entity. Message group ID = entity ID (e.g., org ID), so requests for the same org are processed in order while different orgs are processed in parallel.

**When I use Standard:** Notifications, fanout patterns, or when throughput > 3000 msg/s and ordering isn't critical.

**Key design pattern:** FIFO queue + message group ID gives you **ordered parallelism** — strict ordering within a group, parallel processing across groups.

---

## Q3: How do you design for high availability on AWS?

**Answer:**

- **Multi-AZ deployments** — ECS tasks spread across AZs, RDS Multi-AZ, ElastiCache Multi-AZ with replicas
- **ALB health checks** — Configure proper health check paths and thresholds. Unhealthy targets get removed from rotation.
- **Auto-scaling** — ECS service auto-scaling based on CPU/memory/SQS queue depth. Scale-up Lambda triggered by queue depth for batch workloads.
- **Circuit breakers** — Prevent cascading failures to downstream services
- **DLQ (Dead Letter Queue)** — Messages that fail processing N times go to DLQ for manual investigation instead of blocking the queue
- **Retry with backoff** — Exponential backoff with jitter for transient failures
- **S3 for durability** — Store artifacts (exports, files) in S3 with versioning. Cross-region replication if needed.

---

## Q4: Explain VPC Peering and when it's needed. What pitfalls have you seen?

**Answer:**

**VPC Peering** connects two VPCs, enabling private IP communication without traversing the public internet.

**When needed:**
- Service A in VPC-1 needs to call Service B in VPC-2 (e.g., API service calling a downstream dependency in a different VPC)
- Cross-account access to shared services (databases, caches)

**Pitfalls I've encountered:**
- **Route table gaps** — Peering is established but route tables in one or both VPCs don't have entries for the peer CIDR. Traffic silently fails.
- **Security group misconfiguration** — Security groups reference the peer VPC's CIDR but miss specific subnets or ports.
- **AZ-specific routing** — Traffic may work from one AZ but fail from another if route tables are AZ-specific.
- **DNS resolution** — Private DNS names may not resolve across peered VPCs without enabling DNS resolution on the peering connection.

> I've debugged a production issue where intermittent API timeouts were caused by VPC peering route table entries missing for a specific AZ. The fix was adding the correct routes, but the diagnosis required going beyond application logs into network-level investigation.

---

## Q5: How do you approach cost optimization on AWS?

**Answer:**

- **Right-sizing** — Review ECS task CPU/memory. Over-provisioned tasks waste money. Use Container Insights metrics to right-size.
- **Fargate Spot** — For fault-tolerant batch workloads (export processing), Fargate Spot can save up to 70%.
- **Reserved capacity** — For steady-state services (API servers, Redis), Reserved Instances or Savings Plans.
- **S3 lifecycle policies** — Move infrequently accessed exports to S3 IA or Glacier.
- **Evaluate before building** — Before committing to a migration or new service, analyze whether it's needed (e.g., deferring an unnecessary DB migration saved thousands).
- **Caching** — Redis caching reduces calls to expensive downstream APIs, cutting compute and API costs.

---

<!-- Source: aws-developer-interview-prep.txt, aws-interview-qa.txt, tf-aws-update-policy.txt -->

## Q6: AWS Core Services : Compute, Storage, Database, Networking

**Answer:**

**AWS** : Amazon Web Services, a cloud platform providing on-demand computing resources with a pay-as-you-go model --> organized into service families: Compute, Storage, Database, Networking, Security, and Developer Tools --> eliminates need to own/maintain physical infrastructure.

**Core Services Summary:**

```
Compute:
  EC2         : Virtual servers (resizable, many instance types)
  Lambda      : Serverless functions (event-driven, no server management)
  ECS/EKS     : Container orchestration (Docker/Kubernetes)
  Elastic Beanstalk : PaaS: deploy apps without managing infra

Storage:
  S3          : Object storage (files, images, backups, static sites)
  EBS         : Block storage attached to EC2 (like a hard disk)
  EFS         : Managed NFS file system (shared across EC2 instances)
  Glacier     : Archival storage (low cost, slow retrieval)

Database:
  RDS         : Managed relational DB (MySQL, PostgreSQL, Oracle, SQL Server)
  Aurora      : AWS-optimized relational DB (MySQL/Postgres compatible, faster)
  DynamoDB    : Managed NoSQL (key-value, document, millisecond latency)
  ElastiCache : In-memory cache (Redis or Memcached)
  Redshift    : Data warehouse (analytics at petabyte scale)

Networking:
  VPC         : Virtual Private Cloud (isolated network)
  ELB/ALB/NLB : Load balancers (distribute traffic)
  CloudFront  : CDN (global edge caching)
  Route 53    : DNS service

Security:
  IAM         : Identity and Access Management (users, roles, policies)
  KMS         : Key Management Service (encryption keys)
  Cognito     : User authentication (OAuth2, SAML, social login)
  WAF         : Web Application Firewall

Developer Tools:
  CodeCommit/CodeBuild/CodeDeploy : CI/CD pipeline
  CloudFormation : Infrastructure as Code (IaC)
  CloudWatch  : Monitoring, logs, metrics, alarms
```

**Benefits / Trade-offs:** AWS breadth of services enables any architecture pattern. Trade-off: 200+ services create complexity : start with well-known services and expand; vendor lock-in risk for managed services.

---

## Q7: AWS Lambda : Serverless Computing and Event-Driven Architecture

**Answer:**

**AWS Lambda** : serverless compute service that runs code in response to events without managing servers --> scales automatically from zero to thousands of invocations --> pay only for execution time (billed per ms) --> stateless execution (each invocation is independent).

Lambda integrates with: API Gateway (REST/WebSocket), S3 (file events), DynamoDB Streams, SQS/SNS, EventBridge, CloudWatch Events.

```java
// Java Lambda Handler
public class OrderHandler implements RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent> {

    private final OrderService orderService = new OrderService();

    @Override
    public APIGatewayProxyResponseEvent handleRequest(
            APIGatewayProxyRequestEvent event, Context context) {

        context.getLogger().log("Processing order: " + event.getBody());

        try {
            Order order = objectMapper.readValue(event.getBody(), Order.class);
            OrderResult result = orderService.process(order);

            return new APIGatewayProxyResponseEvent()
                .withStatusCode(200)
                .withBody(objectMapper.writeValueAsString(result))
                .withHeaders(Map.of("Content-Type", "application/json"));

        } catch (Exception e) {
            return new APIGatewayProxyResponseEvent()
                .withStatusCode(500)
                .withBody("{\"error\": \"" + e.getMessage() + "\"}");
        }
    }
}

// SAM template (serverless deployment)
// Type: AWS::Serverless::Function
// Properties:
//   Runtime: java17
//   Handler: com.example.OrderHandler::handleRequest
//   Events:
//     ApiEvent: { Type: Api, Properties: { Path: /orders, Method: POST } }
```

**Lambda cold start mitigation:**
- Keep handler class lean (avoid heavy static initialization)
- Use SnapStart (Java 21 on Lambda) : snapshots JVM state, near-zero cold start
- Provisioned concurrency : pre-warms instances (higher cost)
- Use lighter runtimes (Node.js, Python) for latency-critical paths

**Benefits / Trade-offs:** Zero server management, auto-scaling, cost-effective for event-driven workloads. Trade-off: cold starts add latency (100ms-10s for Java); 15-minute max execution time; stateless : use S3/DynamoDB for state; debugging distributed lambda chains is complex.

---

## Q8: Amazon SQS vs SNS : Message Queue vs Pub/Sub

**Answer:**

**SQS (Simple Queue Service)** : managed message queue, point-to-point --> one producer puts messages, one consumer (or group) pulls and processes --> guarantees at-least-once delivery --> messages persist until consumed (up to 14 days).

**SNS (Simple Notification Service)** : managed pub/sub, one-to-many --> one publisher sends to a topic, all subscribed endpoints receive it simultaneously --> push model --> subscribers: SQS queues, Lambda, email, HTTP, SMS.

```
SQS Pattern (Decoupling):
OrderService --> [SQS Queue] --> ProcessingService (pulls messages)
                            --> one consumer processes each message

SNS Fan-out Pattern:
OrderPlaced --> [SNS Topic] --> [SQS Queue 1] --> EmailService (sends receipt)
                          --> [SQS Queue 2] --> InventoryService (deducts stock)
                          --> [SQS Queue 3] --> ShippingService (creates shipment)
                          All three happen simultaneously!
```

```yaml
# CloudFormation: SNS --> SQS fan-out
OrderTopic:
  Type: AWS::SNS::Topic

EmailQueue:
  Type: AWS::SQS::Queue

EmailSubscription:
  Type: AWS::SNS::Subscription
  Properties:
    TopicArn: !Ref OrderTopic
    Protocol: sqs
    Endpoint: !GetAtt EmailQueue.Arn
```

**Key differences:**

| Feature | SQS | SNS |
|---------|-----|-----|
| Pattern | Point-to-point queue | Pub/Sub broadcast |
| Delivery | Pull (consumer polls) | Push (immediate delivery) |
| Consumers | One consumer per message | All subscribers receive |
| Persistence | Up to 14 days | Not stored (best-effort delivery) |
| Use case | Task queue, decoupling | Notification fanout, event broadcast |

**Benefits / Trade-offs:** Combine both: SNS for fan-out + SQS as subscribers for buffering. SQS FIFO queue guarantees ordering and exactly-once processing (higher cost). DLQ (Dead Letter Queue) captures failed messages for analysis.

---

## Q9: AWS IAM : Roles, Policies, and Least Privilege

**Answer:**

**IAM (Identity and Access Management)** : controls WHO can access WHAT AWS resources --> three core concepts: **Users** (humans/apps), **Roles** (temporary credentials assumed by services/EC2/Lambda), **Policies** (JSON documents defining permissions) --> principle of least privilege: grant minimum permissions needed.

```json
// IAM Policy: allow Lambda to read from S3 and write to DynamoDB
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-orders-bucket",
        "arn:aws:s3:::my-orders-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:UpdateItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/Orders"
    }
  ]
}
```

```java
// Java SDK: use IAM Role (not hardcoded credentials!)
// When running on EC2/Lambda, SDK automatically uses instance role
AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
    .withRegion(Regions.US_EAST_1)
    .build(); // Picks up role credentials automatically from EC2 metadata

// NEVER do this:
// AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
//     .withCredentials(new AWSStaticCredentialsProvider(
//         new BasicAWSCredentials("AKIAIOSFODNN7EXAMPLE", "secret"))) // BAD!
//     .build();
```

**IAM Best Practices:**
- Use **roles** (temporary credentials) not users for services/EC2/Lambda
- **MFA** for human users, especially root account
- **Rotate** access keys regularly
- **Conditions** in policies (IP restrictions, MFA required, time-based)
- Use **AWS Organizations + SCPs** for account-wide guardrails

**Benefits / Trade-offs:** IAM provides fine-grained access control. Trade-off: complex policy debugging (use IAM Policy Simulator); cross-account access requires explicit trust relationships; over-permissive policies are a critical security risk.

---

## Q10: Terraform vs CloudFormation : IaC Comparison

**Answer:**

**CloudFormation** : AWS-native IaC service --> YAML/JSON templates --> tight AWS integration --> free (pay for resources created) --> state managed by AWS --> update via changesets.

**Terraform** : HashiCorp multi-cloud IaC --> HCL syntax --> works across AWS, Azure, GCP, on-prem --> state stored in S3+DynamoDB (remote backend) --> plan/apply workflow --> larger ecosystem.

```hcl
# Terraform: update an S3 bucket policy
resource "aws_s3_bucket" "orders" {
  bucket = "my-orders-bucket"
  tags = {
    Environment = "production"
    Team        = "backend"
  }
}

resource "aws_s3_bucket_policy" "orders" {
  bucket = aws_s3_bucket.orders.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { AWS = aws_iam_role.lambda_role.arn }
      Action    = ["s3:GetObject"]
      Resource  = "${aws_s3_bucket.orders.arn}/*"
    }]
  })
}
```

```yaml
# CloudFormation equivalent
OrdersBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: my-orders-bucket
    Tags:
      - Key: Environment
        Value: production

OrdersBucketPolicy:
  Type: AWS::S3::BucketPolicy
  Properties:
    Bucket: !Ref OrdersBucket
    PolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            AWS: !GetAtt LambdaRole.Arn
          Action: s3:GetObject
          Resource: !Sub "${OrdersBucket.Arn}/*"
```

| Feature | Terraform | CloudFormation |
|---------|-----------|---------------|
| Cloud support | Multi-cloud | AWS only |
| State management | S3 + DynamoDB remote | AWS-managed |
| Syntax | HCL (expressive) | YAML/JSON (verbose) |
| Modules | Terraform Registry | AWS CDK / SAR |
| Plan preview | `terraform plan` | Changeset preview |

**Benefits / Trade-offs:** Terraform preferred for multi-cloud or when you want a consistent IaC tool across clouds. CloudFormation preferred for AWS-only shops needing deep service integration and no extra tooling. CDK (AWS Cloud Development Kit) is a modern alternative : write infrastructure in Java/Python/TypeScript, compiles to CloudFormation.

---

## Q11: RDS vs DynamoDB vs ElastiCache : Choosing the Right Data Store

**Answer:**

**RDS (Relational)** : structured data, ACID transactions, complex joins/queries --> best for financial records, user data, order systems --> scales vertically + Read Replicas --> up to 64TB (Aurora).

**DynamoDB (NoSQL)** : key-value/document, single-digit millisecond latency at any scale, auto-sharding --> best for high-throughput, variable-schema data (sessions, events, gaming) --> infinite horizontal scale.

**ElastiCache (In-memory)** : sub-millisecond cache layer --> Redis (rich data types, persistence, Pub/Sub) or Memcached (simple caching) --> reduces DB load for hot data.

```java
// DynamoDB: high-throughput access
@DynamoDbTable(tableName = "UserSessions")
class UserSession {
    @DynamoDbPartitionKey String userId;
    @DynamoDbSortKey String sessionId;
    String data;
    long ttl; // auto-expiry
}

// ElastiCache (Redis): cache DB results
@Cacheable(value = "orders", key = "#userId")
public List<Order> getOrders(String userId) {
    return orderRepository.findByUserId(userId); // DB hit only on cache miss
}

// Use all three together:
// User request --> ElastiCache (hit: return cache) --> DynamoDB (session) --> RDS (order details)
```

**Decision matrix:**

| Need | Use |
|------|-----|
| ACID transactions, complex queries | RDS (PostgreSQL/Aurora) |
| Millisecond NoSQL, auto-scaling | DynamoDB |
| Sub-ms reads, offload DB | ElastiCache (Redis) |
| Time-series, IoT data | DynamoDB (TTL) |
| Data warehouse, analytics | Redshift |

**Benefits / Trade-offs:** RDS provides ACID guarantees but limits horizontal write scaling. DynamoDB scales infinitely but no joins (denormalize data). ElastiCache is pure performance : but cache invalidation adds complexity.

---

## Q12: CloudWatch : Monitoring, Logs, Alarms, and Dashboards

**Answer:**

**CloudWatch** : AWS observability service for metrics, logs, alarms, and dashboards --> collects metrics from EC2, Lambda, RDS, ECS automatically --> log groups for application/service logs --> alarms trigger SNS, Auto Scaling, or Lambda actions.

```java
// Custom CloudWatch metrics from Java
AmazonCloudWatch cw = AmazonCloudWatchClientBuilder.standard()
    .withRegion(Regions.US_EAST_1).build();

// Publish business metric
MetricDatum datum = new MetricDatum()
    .withMetricName("OrdersProcessed")
    .withUnit(StandardUnit.Count)
    .withValue(1.0)
    .withDimensions(new Dimension()
        .withName("Environment").withValue("production"));

cw.putMetricData(new PutMetricDataRequest()
    .withNamespace("OrderService")
    .withMetricData(datum));

// Application logging (auto-sent to CloudWatch Logs via CloudWatch Agent)
import org.slf4j.Logger;
Logger log = LoggerFactory.getLogger(OrderService.class);
log.info("Order created: orderId={}, userId={}", orderId, userId);
// Appears in CloudWatch Log Group: /aws/lambda/order-service
```

**Key CloudWatch concepts:**
- **Metrics** : time-series data (CPU%, request count, error rate)
- **Log Groups** : container for log streams from a service
- **Alarms** : notify when metric crosses threshold (`CPUUtilization > 80%`)
- **Dashboards** : unified view of multiple metrics
- **Insights** : query logs with SQL-like syntax

```
# CloudWatch Insights query: find errors in last 1 hour
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100
```

**Benefits / Trade-offs:** Tight AWS integration means metrics/logs flow automatically for most services. Trade-off: complex multi-service distributed tracing needs AWS X-Ray; CloudWatch Logs can be expensive for high-volume logging : use log retention policies and sampling.

---

## Q13: Kinesis vs SQS vs SNS : When to Use Each and Fan-Out Pattern

**Answer:**

| Service | Model | Delivery | Consumers | Use Case |
|---------|-------|----------|-----------|----------|
| **SQS** | Queue (task buffer) | Each message to ONE consumer | Competing consumers | Async job processing, one-consumer-per-message |
| **Kinesis** | Event stream | ALL consumers see ALL events | Multiple independent readers | High-throughput telemetry, replay, ordered events |
| **SNS** | Pub/Sub | Broadcast to ALL subscribers | Fan-out | Notifications, trigger multiple targets |

**SQS : Work Queue (competing consumers):**
```
Producer --> [SQS Queue] --> Consumer A (gets message, other consumers miss it)
```
- Message consumed once then deleted
- Multiple consumers = competing for messages (load balancing)
- Ordered guarantee: SQS FIFO only

**Kinesis : Event Stream (fan-out by design):**
```
Sensor Telemetry --> [Kinesis Stream]
                        â†“         â†“
               Anomaly Service  Analytics Service  (both see every event)
```
- Ordered per partition key
- Replay-capable (24hrs to 7 days)
- Parallel consumers via Enhanced Fan-Out or shard iterator

**SNS --> Multiple SQS Fan-Out (recommended pattern for multi-consumer with SQS):**
```java
// Architecture
Producer --> SNS Topic
                â†“          â†“           â†“
          SQS-Anomaly  SQS-Analytics  SQS-Audit
```
Each subscriber gets an independent copy of every message.

```yaml
# CloudFormation / SAM snippet
OrderEventsTopic:
  Type: AWS::SNS::Topic

AnomalyQueue:
  Type: AWS::SQS::Queue
  Properties:
    RedrivePolicy: { deadLetterTargetArn: !GetAtt AnomalyDLQ.Arn, maxReceiveCount: 3 }

AnomalySubscription:
  Type: AWS::SNS::Subscription
  Properties:
    TopicArn: !Ref OrderEventsTopic
    Protocol: sqs
    Endpoint: !GetAtt AnomalyQueue.Arn
```

**Decision guide:**

| Requirement | Best Choice |
|-------------|-------------|
| Task processing (one consumer) | SQS |
| Multiple services consume same event | Kinesis or SNS-->SQS fan-out |
| Need replay + ordering per device | Kinesis |
| Push notifications / email / Lambda trigger | SNS |
| High throughput (millions/sec) | Kinesis |
| Simple job queue | SQS Standard |
| Enterprise event routing (rule-based) | EventBridge |

**Interview one-liner:** "SQS is a work queue (one consumer per message), Kinesis is an event stream (many consumers + replay), SNS is pub/sub broadcast."

---

## Q14: RDS PostgreSQL vs Aurora Serverless

**Answer:**

| Aspect | RDS PostgreSQL | Aurora Serverless v2 |
|--------|----------------|----------------------|
| **Compute** | Fixed instance (manual scaling) | Auto-scales based on load (ACUs) |
| **Cost model** | Pay per instance-hour (running 24/7) | Pay per ACU-hour (billed per use) |
| **Availability** | Multi-AZ failover | Built-in HA, multi-AZ by design |
| **Cold start** | None (always running) | Near real-time scaling (seconds) |
| **Connection handling** | Standard pooling | Needs RDS Proxy for connection stability |
| **Performance** | Predictable, tunable | Excellent at scale but slight latency on burst |
| **Use case** | Steady, predictable, latency-sensitive workloads | Spiky/intermittent, dev/test, variable traffic |

**When to choose RDS PostgreSQL:**
- Continuous high-throughput writes (telemetry, orders)
- Requires fine-grained performance tuning
- Predictable load patterns
- Latency-sensitive transactional systems

**When to choose Aurora Serverless v2:**
- Bursty or unpredictable traffic (dealer portals, batch analytics)
- Dev/test environments (pause when idle = zero cost)
- Cost optimization over predictability

**RDS Proxy (critical for serverless):**

Aurora Serverless scales compute dynamically --> connection count can spike --> DB overwhelmed.
```yaml
# RDS Proxy sits between app and Aurora, pools and manages connections
rds_proxy_endpoint: my-proxy.proxy-xxx.us-east-1.rds.amazonaws.com
max_connections_percent: 100
connection_borrow_timeout: 120
```
- Reduces new connection creation overhead
- Handles connection spikes from Lambda scale-out (1000 Lambdas --> pooled to ~50 DB connections)

**Interview answer:**
> "RDS PostgreSQL is ideal for steady, latency-sensitive production workloads requiring predictable performance. Aurora Serverless v2 suits variable or bursty traffic where auto-scaling and cost efficiency matter more than tuning control. For serverless, always use RDS Proxy to manage connection spikes."

**Interview trap:** "Is Aurora Serverless always better?" --> No. Cold start latency, connection management complexity, and less predictable performance make it unsuitable for always-on, latency-critical systems.


---

## Q15: ECS Task Definition, Scaling Types, and Fargate vs EC2 Cost

**Answer:**

**Task Definition (ECS blueprint):**

A Task Definition is a versioned, immutable configuration spec for running containers in ECS : equivalent to a Docker Compose/Pod spec.

```json
{
  "family": "tractor-alert-service",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "networkMode": "awsvpc",
  "containerDefinitions": [{
    "name": "alert-processor",
    "image": "123456.dkr.ecr.us-east-1.amazonaws.com/alert-service:latest",
    "portMappings": [{ "containerPort": 8080 }],
    "environment": [{ "name": "SPRING_PROFILES_ACTIVE", "value": "prod" }],
    "secrets": [{ "name": "DB_PASSWORD", "valueFrom": "arn:aws:ssm:..." }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": { "awslogs-group": "/ecs/alert-service", "awslogs-region": "us-east-1" }
    }
  }],
  "taskRoleArn": "arn:aws:iam::...:role/ecs-task-role"
}
```

Key fields: container image, CPU/memory, port mappings, env vars, secrets (from SSM/Secrets Manager), IAM task role, logging.

**ECS Auto Scaling : two independent layers:**

| Layer | What scales | Applies to |
|-------|------------|------------|
| Service (task) scaling | Number of running containers | EC2 + Fargate |
| Cluster (compute) scaling | Number of EC2 instances in ASG | EC2 only (Fargate is managed) |

**Scaling policy types:**

*Target Tracking (simple, self-adjusting):*
```bash
aws application-autoscaling put-scaling-policy \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 60.0,
    "PredefinedMetricSpecification": {"PredefinedMetricType": "ECSServiceAverageCPUUtilization"}
  }'
```
Works like a thermostat : continuously adjusts task count to keep CPU near 60%. Best for stable, predictable workloads.

*Step Scaling (rule-based, aggressive):*
```json
{
  "StepAdjustments": [
    { "MetricIntervalLowerBound": 0, "MetricIntervalUpperBound": 20, "ScalingAdjustment": 2 },
    { "MetricIntervalLowerBound": 20, "ScalingAdjustment": 5 }
  ]
}
```
CPU > 70% --> +2 tasks; CPU > 90% --> +5 tasks. Best for spiky workloads (flash sales, telemetry bursts).

**Fargate vs ECS on EC2 : cost model:**

| Scenario | Best Choice | Why |
|----------|------------|-----|
| 24/7 stable microservices | ECS on EC2 (Reserved) | Fixed cost, cheaper for constant load |
| Spiky / event-driven tasks | Fargate | Pay per second; zero cost when idle |
| Dev/test environments | Fargate | No idle EC2 cost |
| CPU-heavy, long-running jobs | EC2 Spot | Fargate per-second billing makes it expensive |

**Fargate becomes expensive when:** steady-state utilization is high (always-on) : per-second billing adds up faster than reserved EC2. Also watch NAT Gateway egress costs with Fargate in private subnets.

**Interview one-liner:** "ECS orchestrates containers; Fargate eliminates infra management by handling compute automatically; Task Definition is the immutable blueprint; Target Tracking for steady load, Step Scaling for spiky bursts."

---

## Q16: SQS : Standard vs FIFO, DLQ, Long Polling, Cost Model

**Answer:**

**SQS Standard vs FIFO:**

| Feature | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| Ordering | Best-effort (not guaranteed) | Strict FIFO |
| Duplicates | Possible (at-least-once) | Exactly-once |
| Throughput | Unlimited | 300 msg/s (3000 with batching) |
| Cost | Lower | Higher |
| Use case | Background jobs, logging, analytics | Payments, order processing, financial txns |

```java
// FIFO requires MessageGroupId (ordering) + optional deduplication
SendMessageRequest req = SendMessageRequest.builder()
    .queueUrl(fifoQueueUrl)
    .messageBody(body)
    .messageGroupId("order-group-123")    // ordering within group
    .messageDeduplicationId(UUID.randomUUID().toString()) // exactly-once
    .build();
```

**Dead Letter Queue (DLQ) : failure handling:**
```
Consumer picks message --> processing fails --> visibility timeout expires --> message reappears
                                                                        --> retry
                                                                        --> maxReceiveCount (3) reached
                                                                        --> moved to DLQ ✅
```

```yaml
# CloudFormation snippet
MainQueue:
  RedrivePolicy:
    deadLetterTargetArn: !GetAtt DLQ.Arn
    maxReceiveCount: 3

DLQ:
  Type: AWS::SQS::Queue
  Properties:
    MessageRetentionPeriod: 1209600  # 14 days
```

**Visibility timeout:** Time a message is hidden after being read (default 30s). If not deleted within this window, message reappears for another consumer. Set â‰¥ max processing time.

**Long polling (cost optimization):**
```java
// Without long polling : empty responses waste money
ReceiveMessageRequest shortPoll = ReceiveMessageRequest.builder()
    .queueUrl(url)
    .maxNumberOfMessages(10)
    .build(); // returns immediately, even if empty

// With long polling : waits up to 20s for messages
ReceiveMessageRequest longPoll = ReceiveMessageRequest.builder()
    .queueUrl(url)
    .maxNumberOfMessages(10)
    .waitTimeSeconds(20)  // reduces empty polls, lowers cost
    .build();
```

**Cost model:**
- Billed per request unit (64 KB per unit)
- 1–64 KB message = 1 unit; 65–128 KB = 2 units; max message = 256 KB
- For payloads > 256 KB: use SQS Extended Client --> stores payload in S3, message contains S3 reference
- Use long polling + batching (`SendMessageBatch`, `ReceiveMessage(max=10)`) to minimize request count

**SQS failure handling flow:**
```
message read --> Visibility Timeout starts
   â†“ success --> DeleteMessage (removed permanently)
   â†“ fail    --> timeout expires --> message reappears
   â†“ fail N times --> DLQ (dead letter queue for debugging)
```

**Idempotency is mandatory:** SQS provides at-least-once delivery (duplicates possible in Standard). Design consumers to handle duplicate messages safely (check DB for existing records before writing).

**Interview one-liner:** "Use Standard for throughput, FIFO for ordering/exactly-once; always configure DLQ for failure visibility; use long polling to reduce cost; design consumers to be idempotent."

