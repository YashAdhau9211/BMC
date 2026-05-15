# AWS Cloud Practitioner — Unit 3 Exam Preparation Question Bank + Study Notes

## Cover Page

**Subject:** AWS Cloud Practitioner  
**Unit:** Unit 3 — Cloud Technology and Services  
**Prepared For:** University Semester Examination  
**Material Type:** Detailed Study Notes + Question Bank + Exam Answers + Revision Notes  
**Level:** Beginner friendly, exam-oriented, full-mark preparation material  
**Hours:** 15 Hours (Largest Unit)

---

## Subject Introduction

Unit 3 is the **largest and most technical unit** in the AWS Cloud Practitioner syllabus. It covers the practical building blocks of the AWS ecosystem — the actual services students will encounter in internships, jobs, and certifications. [file:1]

This unit covers how to deploy and operate in AWS, the global infrastructure, compute, database, networking, storage, AI/ML, analytics, and miscellaneous services. Understanding every category of service and being able to explain each with definitions, diagrams, and examples is key to scoring full marks. [file:1]

---

## Unit Overview

### Unit 3 Syllabus — 15 Hours

- Define methods of deploying and operating in the AWS Cloud.
- Define the AWS global infrastructure.
- Identify AWS compute services.
- Identify AWS database services.
- Identify AWS network services.
- Identify AWS storage services.
- Identify AWS AI/ML and analytics services.
- Identify services from other in-scope AWS service categories.

---

## Important Theory Notes

---

## Topic 1: Methods of Deploying and Operating in the AWS Cloud

### Definition

Deploying in the AWS Cloud means creating and running applications and infrastructure using AWS services. Operating means managing, monitoring, and maintaining those resources. [file:1]

### Three Main Ways to Access and Deploy in AWS

#### 1. AWS Management Console

A graphical web-based interface to create, manage, and monitor AWS resources. Best for beginners, visual dashboards, and one-off tasks. [file:1]

#### 2. AWS CLI (Command Line Interface)

A unified tool that allows users to control multiple AWS services from the command line with scripts. Used for automation, batch operations, and DevOps workflows. [file:1]

#### 3. AWS SDKs (Software Development Kits)

Language-specific libraries (Python, Java, Node.js, etc.) that allow developers to integrate AWS services directly into applications programmatically. [file:1]

### Additional Deployment Methods

#### AWS CloudFormation

Infrastructure as Code (IaC) service. Allows users to define AWS infrastructure in template files (JSON or YAML) and provision it automatically and repeatedly. Closely aligned with DevOps practices. [file:1]

#### AWS Elastic Beanstalk

A fully managed Platform-as-a-Service that handles deployment, capacity provisioning, load balancing, auto-scaling, and monitoring. Developers only upload their application code. [file:1]

#### AWS OpsWorks

Configuration management service using Chef and Puppet to automate server configuration, deployment, and management.

### Comparison Table: Deployment Methods

| Method | Type | Best For |
|---|---|---|
| Management Console | GUI | Beginners, visual management |
| AWS CLI | Command line | Automation, scripting |
| AWS SDK | Code library | Application integration |
| CloudFormation | Infrastructure as Code | Repeatable infra provisioning |
| Elastic Beanstalk | PaaS | Simple app deployment |

### DevOps Connection

CloudFormation + CLI + SDK are the backbone of **CI/CD pipelines** in AWS. They support automated build, test, and deployment workflows. [file:1]

### Exam Tip

When asked "How can you interact with AWS?", always mention the three main methods: Console, CLI, and SDK. Then add CloudFormation and Elastic Beanstalk for higher marks.

---

## Topic 2: AWS Global Infrastructure

### Definition

The AWS Global Infrastructure is the worldwide network of physical data centers, regions, availability zones, and edge locations that power all AWS cloud services. [file:1]

### Why Global Infrastructure Matters

- Applications deployed closer to users respond faster.
- Multiple locations provide disaster recovery.
- Global reach supports business expansion.
- Fault isolation prevents one failure from affecting all services. [file:1]

### Key Components

#### 1. AWS Regions

A Region is a geographic area that contains at least two Availability Zones. Each region is independent and isolated from other regions for fault tolerance.

- Example Regions: us-east-1 (N. Virginia), ap-south-1 (Mumbai), eu-west-1 (Ireland).
- Customers choose which region to deploy resources in.
- Data stays within a region unless explicitly moved.

#### 2. Availability Zones (AZs)

An Availability Zone is one or more physically separate data centers within a region. AZs have redundant power, networking, and cooling. They are connected by high-speed, low-latency private networking.

- Each region has 2 or more AZs.
- Applications deployed across multiple AZs are highly available.

#### 3. Edge Locations

Edge locations are endpoints for AWS used for caching content. Used primarily by Amazon CloudFront (Content Delivery Network). There are more edge locations than regions.

- Reduce latency by serving content from the location nearest to the user.

### ASCII Diagram

```text
AWS Global Infrastructure
         |
    +----+----+
    |         |
 Region 1   Region 2
(Mumbai)  (Virginia)
    |
  +---+---+
  |       |
 AZ-1   AZ-2
  |       |
Data   Data
Center  Center

Edge Locations (CloudFront CDN caching)
```

### Table: Region vs AZ vs Edge Location

| Component | Description | Count |
|---|---|---|
| Region | Geographic area | 30+ worldwide |
| Availability Zone | Data center cluster in a region | 2–6 per region |
| Edge Location | CDN cache point | 400+ worldwide |

### Factors for Choosing a Region

- **Compliance:** Data residency laws (e.g., data must stay in India).
- **Proximity:** Closer region = lower latency.
- **Feature availability:** Not all services are available in all regions.
- **Pricing:** Prices can vary by region.

### Exam Tip

Many students confuse Regions with AZs. Remember: **Region = city; AZ = individual data center buildings in that city**.

---

## Topic 3: AWS Compute Services

### Definition

Compute services in AWS provide processing power to run applications. These services replace physical servers with on-demand virtual or serverless compute capacity.

### Major Compute Services

#### 1. Amazon EC2 — Elastic Compute Cloud

**Definition:** EC2 provides resizable virtual server instances in the cloud. [file:1]

**Key Features:**
- Wide variety of instance types (compute-optimized, memory-optimized, etc.).
- Full OS control — customer chooses OS and configures software.
- Billed per hour or per second depending on instance type.
- Supports multiple operating systems (Linux, Windows, etc.).
- Elastic IP addresses, Security Groups, Key Pairs for access.

**Pricing Models:**

| Pricing Type | Description | Use Case |
|---|---|---|
| On-Demand | Pay by hour/second | Flexible, unpredictable workloads |
| Reserved Instances | 1–3 year commitment, up to 75% discount | Steady, predictable workloads |
| Spot Instances | Bid for unused capacity, up to 90% discount | Batch jobs, fault-tolerant tasks |
| Dedicated Hosts | Physical server for one customer | Compliance, licensing |

**Example:** A web server running a university portal is hosted on EC2 instances. During exam season, more instances are launched to handle traffic. [file:1]

#### 2. AWS Lambda

**Definition:** Lambda is a serverless compute service that runs code in response to events without provisioning or managing servers.

**Key Features:**
- Run code without servers.
- Automatically scales based on requests.
- Charged only when code runs (per request and duration).
- Supports Node.js, Python, Java, Go, etc.
- Triggered by AWS events: S3 uploads, API calls, DynamoDB streams.

**Example:** When a photo is uploaded to S3, Lambda automatically triggers a function to resize the image.

#### 3. Amazon ECS — Elastic Container Service

**Definition:** A fully managed container orchestration service that runs, stops, and manages Docker containers on a cluster.

**Key Features:**
- Supports Docker containers.
- Integrates with other AWS services.
- Can run on EC2 or AWS Fargate (serverless containers).

#### 4. Amazon EKS — Elastic Kubernetes Service

**Definition:** A managed Kubernetes service that makes it easy to run Kubernetes on AWS.

**Key Features:**
- Managed Kubernetes control plane.
- Integrates with IAM, VPC, CloudTrail.
- Supports hybrid deployments.

#### 5. AWS Fargate

**Definition:** A serverless compute engine for containers. Works with both ECS and EKS.

**Key Features:**
- No need to provision or manage servers.
- Pay only for the CPU and memory resources containers use.
- Automatically scales.

#### 6. AWS Elastic Beanstalk

**Definition:** A PaaS service that simplifies deployment and management of web applications. Developers upload code and AWS handles everything else. [file:1]

### Compute Services Summary Table

| Service | Type | Managed By | Use Case |
|---|---|---|---|
| EC2 | Virtual servers | Customer manages OS | Full control server workloads |
| Lambda | Serverless | AWS managed | Event-driven functions |
| ECS | Container orchestration | AWS managed | Docker-based apps |
| EKS | Kubernetes | AWS managed | Kubernetes workloads |
| Fargate | Serverless containers | AWS managed | Container apps without servers |
| Elastic Beanstalk | PaaS | AWS managed | Simple web app deployment |

### ASCII Diagram

```text
Compute Services in AWS

EC2 (Virtual Machines)
  |
  +-- On-Demand, Reserved, Spot, Dedicated

Lambda (Serverless)
  |
  +-- Event triggered, no server management

ECS / EKS / Fargate (Containers)
  |
  +-- Docker / Kubernetes workloads

Elastic Beanstalk (PaaS)
  |
  +-- Upload code, AWS handles rest
```

---

## Topic 4: AWS Storage Services

### Definition

AWS storage services provide scalable, durable, and cost-effective ways to store, access, and protect data in the cloud.

### Major Storage Services

#### 1. Amazon S3 — Simple Storage Service

**Definition:** An object storage service offering industry-leading scalability, availability, and security for any amount of data.

**Key Features:**
- Objects stored in buckets.
- Unlimited storage capacity.
- 99.999999999% (11 nines) durability.
- Supports versioning, lifecycle policies, and replication.
- Private by default.
- Supports static website hosting.

**Storage Classes:**

| Class | Use Case | Cost |
|---|---|---|
| S3 Standard | Frequently accessed data | Higher |
| S3 Standard-IA | Infrequent access | Lower |
| S3 Glacier | Archive, long-term storage | Very low |
| S3 Glacier Deep Archive | Rarely accessed archives | Lowest |
| S3 Intelligent-Tiering | Auto-tiering based on access | Variable |

**Example:** A company stores application logs, media files, backups, and static website assets in S3.

#### 2. Amazon EBS — Elastic Block Store

**Definition:** A high-performance block storage service designed for use with EC2 instances.

**Key Features:**
- Persistent storage that survives EC2 instance stop/start.
- Single-digit millisecond latency.
- Snapshots for backup stored in S3.
- Can be detached from one EC2 and attached to another.

**Analogy:** EBS is like the hard drive of a virtual machine. It stays when the machine is turned off.

#### 3. Amazon EFS — Elastic File System

**Definition:** A scalable, fully managed elastic network file system for Linux workloads.

**Key Features:**
- Shared file system — multiple EC2 instances can access simultaneously.
- Automatically grows and shrinks with usage.
- No need to provision capacity.

**Analogy:** EFS is like a shared network drive that multiple computers can access at the same time.

#### 4. Amazon S3 Glacier

**Definition:** A secure, durable, and extremely low-cost cloud storage service for data archiving and long-term backup.

**Key Features:**
- Retrieval time ranges from minutes to hours.
- Much cheaper than S3 Standard.
- Used for compliance archives, historical records.

#### 5. AWS Storage Gateway

**Definition:** A hybrid cloud storage service that connects on-premises environments with cloud storage.

### Storage Services Comparison Table

| Service | Type | Access Pattern | Use Case |
|---|---|---|---|
| S3 | Object storage | Any | Files, backups, media |
| EBS | Block storage | EC2 only | OS disks, databases |
| EFS | File storage | Multiple EC2s | Shared file access |
| S3 Glacier | Object archive | Slow retrieval | Long-term archive |

---

## Topic 5: AWS Database Services

### Definition

AWS offers fully managed database services for relational, non-relational, in-memory, and other database workloads, removing the burden of manual setup, patching, and backups.

### Major Database Services

#### 1. Amazon RDS — Relational Database Service

**Definition:** A managed relational database service supporting multiple engines including MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora.

**Key Features:**
- Automated backups and patching.
- Multi-AZ deployment for high availability.
- Read replicas for read scaling.
- Encryption at rest and in transit.

**Example:** An e-commerce site stores product, customer, and order data in RDS MySQL.

#### 2. Amazon Aurora

**Definition:** A MySQL and PostgreSQL-compatible relational database with up to 5x better performance than standard MySQL.

**Key Features:**
- Fully managed and serverless option available.
- Automatically replicates across 3 AZs with 6 copies of data.
- Up to 15 read replicas.
- Highly available and fault tolerant.

#### 3. Amazon DynamoDB

**Definition:** A fully managed, serverless, key-value and document NoSQL database delivering single-digit millisecond performance.

**Key Features:**
- No server management.
- Scales automatically to any traffic level.
- Supports ACID transactions.
- Event-driven with DynamoDB Streams.

**Example:** A gaming leaderboard, user session store, or real-time IoT data.

#### 4. Amazon Redshift

**Definition:** A fully managed, petabyte-scale cloud data warehouse.

**Key Features:**
- Designed for analytics and reporting.
- Massively parallel processing (MPP).
- Integrates with Amazon S3 for data lakes.

**Example:** A company uses Redshift to run complex analytical queries on years of sales data.

#### 5. Amazon ElastiCache

**Definition:** A fully managed in-memory data store and cache service supporting Redis and Memcached.

**Key Features:**
- Sub-millisecond latency.
- Reduces database load by caching frequent queries.
- Used for session management, caching, leaderboards.

#### 6. Amazon Neptune

**Definition:** A fully managed graph database service.

**Use Cases:** Social networks, fraud detection, recommendation engines.

### Database Services Comparison Table

| Service | Type | Use Case |
|---|---|---|
| RDS | Relational (SQL) | Traditional apps, structured data |
| Aurora | Relational (high performance) | High-traffic SQL workloads |
| DynamoDB | NoSQL (Key-Value/Document) | Real-time, serverless apps |
| Redshift | Data Warehouse | Analytics and reporting |
| ElastiCache | In-memory cache | Low-latency caching |
| Neptune | Graph database | Relationship-heavy queries |

---

## Topic 6: AWS Networking Services

### Definition

AWS networking services enable secure, reliable communication between AWS resources, the internet, and on-premises environments.

### Major Networking Services

#### 1. Amazon VPC — Virtual Private Cloud

**Definition:** A service that lets customers provision a logically isolated section of the AWS Cloud where they can launch resources in a virtual network they define. [file:1]

**Key Features:**
- Full control over IP address range, subnets, routing, and gateways.
- Public subnets (internet-accessible) and private subnets.
- Security Groups: virtual firewalls for EC2 instances.
- Network ACLs: stateless packet filters at subnet level.
- VPC Peering: connect two VPCs privately.
- VPN Gateway: connect on-premises network to VPC securely.

**ASCII Diagram:**

```text
VPC (10.0.0.0/16)
  |
  +-- Public Subnet (10.0.1.0/24)
  |       |
  |     EC2 (Web Server)
  |       |
  |   Internet Gateway
  |
  +-- Private Subnet (10.0.2.0/24)
          |
        RDS (Database) — no direct internet access
```

#### 2. Amazon Route 53

**Definition:** A scalable and highly available Domain Name System (DNS) web service.

**Key Features:**
- Domain registration.
- DNS routing policies (simple, weighted, failover, latency-based, geolocation).
- Health checks and routing.
- Integrates with other AWS services.

**Example:** Route 53 resolves www.myapp.com to the correct EC2 instance or load balancer.

#### 3. Amazon CloudFront

**Definition:** A fast content delivery network (CDN) that delivers data, videos, applications, and APIs globally with low latency.

**Key Features:**
- Caches content at Edge Locations.
- Reduces origin server load.
- Integrates with S3, EC2, Elastic Load Balancer, and Route 53.
- Supports HTTPS and custom SSL certificates.

**Example:** A news website uses CloudFront so readers in India get content from the nearest edge location rather than a server in the US.

#### 4. Elastic Load Balancing (ELB)

**Definition:** Automatically distributes incoming application traffic across multiple EC2 instances, containers, or IP addresses.

**Types:**

| Type | Description |
|---|---|
| Application Load Balancer (ALB) | HTTP/HTTPS traffic, layer 7 |
| Network Load Balancer (NLB) | TCP/UDP traffic, layer 4, ultra-high performance |
| Gateway Load Balancer | Third-party virtual network appliances |

**Example:** An application with 100,000 concurrent users uses ELB to distribute traffic across 20 EC2 instances.

#### 5. AWS Direct Connect

**Definition:** A dedicated private network connection between an organization's on-premises data center and AWS.

**Key Features:**
- Consistent network performance.
- Does not use the public internet.
- Ideal for high-volume data transfer or compliance requirements.

#### 6. AWS Transit Gateway

**Definition:** Connects multiple VPCs and on-premises networks through a central hub.

### Networking Services Summary Table

| Service | Purpose |
|---|---|
| VPC | Isolated virtual network |
| Route 53 | DNS and domain management |
| CloudFront | Content delivery network (CDN) |
| ELB | Traffic distribution across instances |
| Direct Connect | Private dedicated connection to AWS |
| Transit Gateway | Central hub for VPC and on-prem networks |

---

## Topic 7: AWS AI/ML and Analytics Services

### Definition

AWS provides a broad set of AI, ML, and analytics services that allow developers and data scientists to build intelligent applications without needing deep expertise in these fields.

### AI/ML Services

#### 1. Amazon SageMaker

**Definition:** A fully managed service that provides tools to build, train, and deploy machine learning models at scale.

**Key Features:**
- End-to-end ML workflow in one service.
- Built-in algorithms and support for custom code.
- Jupyter notebook integration.
- One-click model deployment.

**Example:** A bank builds a fraud detection model using SageMaker and deploys it as a real-time endpoint.

#### 2. Amazon Rekognition

**Definition:** A service that makes it easy to add image and video analysis to applications.

**Key Features:**
- Detect objects, scenes, and activities in images.
- Facial analysis and recognition.
- Text detection in images.
- Used for security surveillance, content moderation.

#### 3. Amazon Comprehend

**Definition:** A natural language processing (NLP) service that uses ML to find insights and relationships in text.

**Key Features:**
- Sentiment analysis.
- Entity recognition (people, places, organizations).
- Language detection.
- Key phrase extraction.

**Example:** An e-commerce company analyzes customer reviews using Comprehend to detect sentiment.

#### 4. Amazon Lex

**Definition:** A service for building conversational interfaces (chatbots) using voice and text.

**Key Features:**
- Same technology used in Amazon Alexa.
- Supports multi-turn conversations.
- Integrates with Lambda for custom logic.

#### 5. Amazon Polly

**Definition:** A text-to-speech (TTS) service that converts text into realistic speech.

**Key Features:**
- Supports multiple languages and voices.
- SSML support for speech customization.
- Used in accessibility apps, audiobooks, voice interfaces.

#### 6. Amazon Translate

**Definition:** A neural machine translation service for fast, high-quality, and customizable language translation.

#### 7. Amazon Transcribe

**Definition:** Automatic speech recognition (ASR) service that converts speech to text.

**Use Cases:** Meeting transcripts, call center analytics, captioning.

#### 8. Amazon Forecast

**Definition:** A time-series forecasting service using the same technology used by Amazon.com.

**Use Cases:** Demand forecasting, financial planning, resource planning.

#### 9. Amazon Personalize

**Definition:** A real-time recommendation service for personalized product and content recommendations.

**Example:** An OTT platform uses Personalize to recommend movies based on viewing history.

### AI/ML Services Summary Table

| Service | Category | Use Case |
|---|---|---|
| SageMaker | Model build/train/deploy | Custom ML models |
| Rekognition | Computer vision | Image/video analysis |
| Comprehend | NLP | Text analysis, sentiment |
| Lex | Chatbot | Conversational AI |
| Polly | Text to speech | Voice applications |
| Translate | Machine translation | Language conversion |
| Transcribe | Speech to text | Audio transcription |
| Forecast | Time-series ML | Demand prediction |
| Personalize | Recommendations | Product suggestions |

---

### Analytics Services

#### 1. Amazon Athena

**Definition:** An interactive query service that makes it easy to analyze data in Amazon S3 using standard SQL.

**Key Features:**
- Serverless — no infrastructure to manage.
- Pay only per query.
- Integrates with AWS Glue for data cataloging.

#### 2. Amazon EMR — Elastic MapReduce

**Definition:** A cloud big data platform for processing large amounts of data using Apache Hadoop, Spark, HBase, and other tools.

**Use Cases:** Log analysis, machine learning data prep, large-scale ETL.

#### 3. Amazon Kinesis

**Definition:** A platform for real-time streaming data ingestion, processing, and analysis.

**Services within Kinesis:**
- **Kinesis Data Streams:** Real-time data streaming.
- **Kinesis Data Firehose:** Load streaming data to S3, Redshift, Elasticsearch.
- **Kinesis Data Analytics:** Analyze streaming data with SQL.

**Example:** A live-streaming platform uses Kinesis to process viewer events in real time.

#### 4. AWS Glue

**Definition:** A fully managed extract, transform, and load (ETL) service.

**Key Features:**
- Automatically discovers data and stores metadata in the Glue Data Catalog.
- Generates ETL code in Python or Scala.
- Serverless.

#### 5. Amazon QuickSight

**Definition:** A fast, cloud-powered business intelligence (BI) service for building visualizations and dashboards.

**Key Features:**
- Connects to many data sources.
- Serverless, auto-scaling.
- Pay-per-session pricing.

### Analytics Services Summary Table

| Service | Type | Use Case |
|---|---|---|
| Athena | SQL query on S3 | Ad-hoc data analysis |
| EMR | Big data processing | Hadoop/Spark workloads |
| Kinesis | Real-time streaming | Live data pipelines |
| Glue | ETL | Data preparation |
| Redshift | Data warehouse | Analytics queries |
| QuickSight | BI dashboards | Business reporting |

---

## Topic 8: Other In-Scope AWS Services

### Application Integration Services

#### 1. Amazon SNS — Simple Notification Service

**Definition:** A fully managed messaging service for application-to-application (A2A) and application-to-person (A2P) communication. [file:1]

**Key Features:**
- Pub/sub messaging model.
- Push notifications to mobile, email, HTTP.
- Integrates with Lambda, SQS, etc.

**Example:** Send an email or SMS when a server CPU exceeds 90%.

#### 2. Amazon SQS — Simple Queue Service

**Definition:** A fully managed message queuing service that enables decoupling and scaling of microservices and distributed systems. [file:1]

**Key Features:**
- Standard Queue: Best-effort ordering, at-least-once delivery.
- FIFO Queue: Exactly-once, first-in-first-out delivery.
- Enables asynchronous processing.

**Example:** An order processing system puts orders in a queue. Background workers process orders from the queue independently.

#### 3. Amazon EventBridge

**Definition:** A serverless event bus that connects applications using events from AWS services, SaaS apps, and custom sources.

### Developer and DevOps Services

#### 1. AWS CodeCommit

**Definition:** A fully managed source control service that hosts secure Git repositories.

#### 2. AWS CodeBuild

**Definition:** A fully managed continuous integration service that compiles source code, runs tests, and produces deployable software packages.

#### 3. AWS CodeDeploy

**Definition:** Automates software deployments to EC2, Lambda, or on-premises servers.

#### 4. AWS CodePipeline

**Definition:** A fully managed CI/CD service that automates the build, test, and deploy phases of release pipelines.

**DevOps Pipeline:**
```text
CodeCommit → CodeBuild → CodeDeploy = CodePipeline (Orchestrator)
```

### Monitoring Services

#### 1. Amazon CloudWatch

**Definition:** A monitoring and observability service that collects and tracks metrics, logs, and events from AWS resources.

**Key Features:**
- Dashboards, alarms, and automated actions.
- Collects custom metrics.
- Log Groups and Log Streams for centralized logging.
- Integrates with SNS for alert notifications. [file:1]

#### 2. AWS CloudTrail

**Definition:** Records API calls and user activity. Used for audit and compliance. (Covered in detail in Unit 2.) [file:1]

### Migration Services

#### 1. AWS Database Migration Service (DMS)

**Definition:** Helps migrate databases to AWS quickly and securely. The source database remains operational during migration.

#### 2. AWS Server Migration Service (SMS)

**Definition:** Automates migration of on-premises servers to AWS.

#### 3. AWS Snowball

**Definition:** A physical data transport device to move large amounts of data into and out of AWS when internet transfer is impractical.

**Variants:**
- **Snowball Edge:** Storage-optimized or compute-optimized.
- **Snowmobile:** An exabyte-scale migration service using a physical truck.

### Management and Governance Services

#### 1. AWS Trusted Advisor

**Definition:** An automated service that acts as a cloud expert recommending improvements in cost, performance, security, fault tolerance, and service limits. [file:1]

**Five Check Categories:**
- Cost optimization.
- Performance.
- Security.
- Fault tolerance.
- Service limits.

#### 2. AWS Auto Scaling

**Definition:** Automatically adjusts the number of EC2 instances or other resources in response to demand. [file:1]

**Key Features:**
- Maintains performance during demand spikes.
- Reduces cost during low-traffic periods.
- Works with EC2, ECS, DynamoDB, and more.

#### 3. AWS CloudFormation

**Definition:** Infrastructure as Code service. Provisions and manages infrastructure from template files. [file:1]

**Key Features:**
- Consistent, repeatable deployments.
- Template-based (JSON or YAML).
- Supports all major AWS services.

---

## One-Word / Very Short Definitions

- **EC2:** Virtual server on demand.
- **Lambda:** Serverless function runner.
- **S3:** Object storage service.
- **EBS:** Block storage for EC2.
- **EFS:** Shared file system.
- **RDS:** Managed relational database.
- **DynamoDB:** NoSQL serverless database.
- **Redshift:** Data warehouse.
- **VPC:** Isolated virtual network.
- **Route 53:** DNS service.
- **CloudFront:** Content delivery network.
- **ELB:** Load balancer.
- **SageMaker:** ML build/train/deploy platform.
- **Rekognition:** Image/video AI analysis.
- **Comprehend:** NLP text analysis.
- **Kinesis:** Real-time data streaming.
- **SNS:** Pub/sub notification service.
- **SQS:** Message queue service.
- **CloudWatch:** Metrics and monitoring service.
- **CloudFormation:** Infrastructure as Code.
- **Trusted Advisor:** Cloud best practice checker.
- **Auto Scaling:** Automatic resource scaling.
- **Snowball:** Physical data transfer device.
- **Direct Connect:** Dedicated private AWS connection.

---

## Short Notes

### Short Note 1: EC2

EC2 is a virtual server service. It provides resizable compute capacity in the cloud. Customers choose OS, instance type, and configuration. Supports On-Demand, Reserved, Spot, and Dedicated pricing. [file:1]

### Short Note 2: Lambda

Lambda is a serverless compute service. Code runs only when triggered by events. No server provisioning needed. Pay only when code executes. Ideal for event-driven architectures.

### Short Note 3: S3

S3 is an object storage service with unlimited scalability. Objects are stored in buckets. Private by default. Supports versioning, lifecycle policies, and multiple storage classes.

### Short Note 4: VPC

VPC is a virtual network in AWS. Customers define IP ranges, subnets, routing, and security. Public and private subnets isolate internet-facing and internal resources. [file:1]

### Short Note 5: CloudFront

CloudFront is a CDN that caches content at edge locations near users. Reduces latency and origin server load. Supports HTTPS, integrates with S3 and EC2.

### Short Note 6: SageMaker

SageMaker is a fully managed ML platform. Supports the end-to-end ML workflow: data labeling, model training, evaluation, and deployment. Used by data scientists and ML engineers.

---

## Important Differences in Table Format

### EC2 vs Lambda

| Basis | EC2 | Lambda |
|---|---|---|
| Server management | Customer manages | No server management |
| Billing | Per hour/second | Per request and duration |
| Scaling | Manual or Auto Scaling | Automatic |
| Use case | Long-running apps | Short, event-driven functions |
| OS control | Yes | No |

### S3 vs EBS vs EFS

| Basis | S3 | EBS | EFS |
|---|---|---|---|
| Type | Object storage | Block storage | File storage |
| Access | Any device/app via URL | One EC2 at a time | Multiple EC2s simultaneously |
| Durability | 11 nines | High | High |
| Scaling | Unlimited | Fixed, manually resized | Automatic |
| Use case | Files, media, backups | EC2 OS disk, database | Shared file access |

### RDS vs DynamoDB

| Basis | RDS | DynamoDB |
|---|---|---|
| Type | Relational (SQL) | NoSQL |
| Schema | Fixed, structured | Flexible |
| Query language | SQL | API / DynamoDB Query |
| Managed | Mostly managed | Fully managed, serverless |
| Use case | Traditional apps | Scalable modern apps |

### SNS vs SQS

| Basis | SNS | SQS |
|---|---|---|
| Model | Publish-Subscribe (push) | Queue (pull) |
| Delivery | Push to subscribers | Consumers poll for messages |
| Retention | No storage | Up to 14 days |
| Use case | Notifications, fan-out | Decoupling, async processing |

### CloudWatch vs CloudTrail

| Basis | CloudWatch | CloudTrail |
|---|---|---|
| Purpose | Monitor performance | Audit activity and API calls |
| Data | Metrics, logs, alarms | Who did what, when, from where |
| Use case | Operational monitoring | Security and compliance auditing |

### Region vs Availability Zone

| Basis | Region | Availability Zone |
|---|---|---|
| Definition | Geographic area | Physical data center cluster |
| Scope | Broad (e.g., Mumbai) | Specific (e.g., ap-south-1a) |
| Count | 30+ globally | 2–6 per region |
| Isolation | Fully isolated | Isolated within a region |

---

## 10-Mark Question Bank with Detailed Answers

---

## Q1. Define methods of deploying and operating in the AWS Cloud.

### Most Important Question

### Introduction

AWS provides multiple ways to deploy and manage cloud resources. Each method suits different skill levels and use cases.

### Definition

Deploying in AWS means creating and launching cloud resources. Operating means managing, monitoring, and maintaining them. [file:1]

### Methods

#### 1. AWS Management Console
A browser-based graphical interface. Best for beginners and visual management. [file:1]

#### 2. AWS CLI
Command-line tool for scripting and automation. [file:1]

#### 3. AWS SDKs
Language-specific APIs for integrating AWS into applications. [file:1]

#### 4. AWS CloudFormation
Infrastructure as Code. Define infra in templates (JSON/YAML). Automates provisioning. [file:1]

#### 5. AWS Elastic Beanstalk
PaaS — upload code, AWS handles the rest. [file:1]

### Comparison Table

| Method | Type | Best For |
|---|---|---|
| Console | GUI | Visual management |
| CLI | Command line | Automation |
| SDK | Library | Application code integration |
| CloudFormation | IaC | Repeatable infra |
| Elastic Beanstalk | PaaS | Quick app deployment |

### DevOps Connection

CLI, SDK, and CloudFormation form the core of AWS-based CI/CD pipelines — integrating automated build, test, and deployment workflows. [file:1]

### Conclusion

Different deployment methods serve different purposes. A complete DevOps team uses all five methods depending on the task.

---

## Q2. Explain the AWS Global Infrastructure in detail.

### Introduction

AWS's global network of physical facilities enables low-latency, high-availability, and globally distributed application deployment.

### Definition

The AWS Global Infrastructure is a worldwide system of data centers, regions, availability zones, and edge locations that provide the physical foundation for all AWS services. [file:1]

### Components

#### Regions
Geographically distinct areas. Each has at least two AZs. Data stays within a region unless moved. [file:1]

#### Availability Zones
Physically separate data centers within a region. Connected by low-latency private links. Deploying across AZs enables high availability. [file:1]

#### Edge Locations
Caching endpoints used by CloudFront CDN. 400+ worldwide. Reduce latency for end users. [file:1]

### Table

| Component | Description | Count |
|---|---|---|
| Regions | Geographic areas | 30+ |
| AZs | Data centers per region | 2–6 |
| Edge Locations | CDN cache points | 400+ |

### Choosing a Region

- Compliance requirements.
- Proximity to users.
- Service availability.
- Pricing.

### Conclusion

The global infrastructure enables organizations to build highly available, low-latency applications that serve users worldwide while meeting compliance requirements.

---

## Q3. Identify and explain AWS compute services.

### Most Important Question

### Introduction

Compute is the most fundamental service category in AWS. It provides the processing power to run applications.

### Definition

AWS compute services deliver virtual, serverless, and container-based processing capacity on demand. [file:1]

### Major Services

#### EC2
Virtual servers. Full OS control. Multiple pricing models. [file:1]

#### Lambda
Serverless. Event-driven. No server management. Pay per execution.

#### ECS / EKS
Container orchestration for Docker and Kubernetes workloads. [file:1]

#### Fargate
Serverless containers. No infrastructure management.

#### Elastic Beanstalk
PaaS. Upload code, AWS handles infra. [file:1]

### Pricing Models for EC2

| Model | Discount | Use Case |
|---|---|---|
| On-Demand | None | Flexible usage |
| Reserved | Up to 75% | Steady workloads |
| Spot | Up to 90% | Batch/fault-tolerant |
| Dedicated | None | Compliance |

### Diagram

```text
EC2 → Virtual Machines (full control)
Lambda → Functions (event-driven, no servers)
ECS/EKS → Containers (managed orchestration)
Fargate → Serverless containers
Beanstalk → PaaS (code-only deployment)
```

### Conclusion

AWS compute services offer options for every type of workload — from full virtual machines with OS control to completely serverless code execution.

---

## Q4. Explain AWS storage services with examples.

### Introduction

Every application needs storage. AWS provides multiple types to match different access patterns and cost requirements.

### Definition

AWS storage services provide scalable, durable, and cost-effective storage for objects, blocks, and files.

### Services

#### Amazon S3
Object storage. Unlimited capacity. 11 nines durability. Multiple storage classes. Private by default. [file:1]

#### Amazon EBS
Block storage attached to EC2. Persistent. Snapshots for backup. Low latency.

#### Amazon EFS
Shared file system. Multiple EC2s can access simultaneously. Auto-scaling capacity.

#### Amazon Glacier
Archive storage. Very low cost. Slow retrieval. Used for compliance and long-term backup.

### Table

| Service | Type | Use Case |
|---|---|---|
| S3 | Object | Files, media, websites |
| EBS | Block | EC2 disk, databases |
| EFS | File | Shared access workloads |
| Glacier | Archive | Long-term cold storage |

### Example

A healthcare company stores active patient records in S3 Standard, less-accessed records in S3-IA, and compliance archives in S3 Glacier.

### Conclusion

Selecting the right storage service depends on the access pattern, performance need, and cost target. AWS provides a storage class for every use case.

---

## Q5. Explain AWS database services in detail.

### Introduction

Databases power almost every application. AWS offers managed services for relational, NoSQL, in-memory, graph, and data warehouse workloads.

### Definition

AWS database services are fully managed database environments that remove the burden of provisioning, patching, backup, and replication.

### Services

#### RDS
Managed relational databases. Supports MySQL, PostgreSQL, SQL Server, Oracle, Aurora. Automated backups. Multi-AZ support.

#### Aurora
High-performance MySQL/PostgreSQL compatible. 5x faster than standard MySQL. 6 copies across 3 AZs.

#### DynamoDB
Serverless NoSQL. Single-digit millisecond performance. Auto-scaling. Good for real-time apps.

#### Redshift
Petabyte-scale data warehouse. MPP architecture. Used for analytics.

#### ElastiCache
In-memory cache (Redis/Memcached). Sub-millisecond latency. Reduces DB load.

### Table

| Service | Type | Use Case |
|---|---|---|
| RDS | Relational | Traditional apps |
| Aurora | Relational | High performance SQL |
| DynamoDB | NoSQL | Scalable real-time |
| Redshift | Data Warehouse | Analytics |
| ElastiCache | In-memory | Caching |

### Conclusion

The right database depends on data structure, query pattern, and scale. AWS's managed services eliminate operational overhead and provide enterprise-grade reliability.

---

## Q6. Explain Amazon VPC and its components.

### Introduction

Networking is the foundation of any cloud architecture. VPC gives organizations full control over their virtual network environment.

### Definition

Amazon Virtual Private Cloud (VPC) lets users provision a logically isolated section of the AWS Cloud where they can launch resources in a virtual network they define. [file:1]

### Key Components

#### Subnets
Segments within a VPC.
- **Public subnet:** Resources accessible from the internet (via Internet Gateway).
- **Private subnet:** Resources not directly accessible from the internet.

#### Internet Gateway
Connects a VPC to the public internet.

#### Security Groups
Virtual firewalls for EC2 instances. Stateful — allow rules applied in both directions.

#### Network ACLs
Stateless firewalls at the subnet level. Must define inbound and outbound rules separately.

#### Route Tables
Define where network traffic is directed.

#### VPC Peering
Connect two VPCs privately, even across accounts or regions.

#### VPN Gateway
Secure encrypted connection from on-premises to VPC.

### Diagram

```text
VPC (192.168.0.0/16)
  |
  +-- Internet Gateway
  |
  +-- Public Subnet
  |       +-- EC2 Web Server (public IP)
  |
  +-- Private Subnet
          +-- RDS Database (no public IP)
          +-- App Server
```

### Conclusion

VPC provides the networking backbone for AWS deployments. Understanding VPC components is essential for designing secure, scalable cloud architectures.

---

## Q7. Explain Amazon CloudFront, Route 53, and Elastic Load Balancing.

### Introduction

These three networking services work together to deliver fast, available, and globally distributed applications.

### CloudFront

Amazon CloudFront is a CDN that caches content at edge locations worldwide, reducing latency for end users. It serves static and dynamic content, supports HTTPS, and integrates with S3, EC2, and ELB.

**Example:** A video streaming platform uses CloudFront so viewers get video segments from the nearest edge location.

### Route 53

Amazon Route 53 is AWS's DNS service. It translates domain names to IP addresses and supports routing policies like weighted, latency-based, failover, and geolocation routing.

**Example:** Route 53 directs users in Asia to a server in the Mumbai region and users in Europe to Frankfurt for lower latency.

### Elastic Load Balancing

ELB distributes incoming traffic across multiple EC2 instances or targets. Three types:
- ALB: HTTP/HTTPS, content-based routing.
- NLB: TCP/UDP, ultra-high performance.
- GLB: Third-party virtual appliances.

### Together They Form a Complete Delivery Architecture

```text
User Request
    |
Route 53 (DNS: resolve domain)
    |
CloudFront (CDN: serve cached content)
    |
ELB (Load Balancer: distribute to instances)
    |
EC2 Instances (Application Servers)
```

### Conclusion

Route 53, CloudFront, and ELB together form the delivery layer of a highly available AWS architecture.

---

## Q8. Explain AWS AI/ML services in detail.

### Introduction

AWS provides a wide range of AI and ML services that allow developers to add intelligence to applications without building models from scratch.

### Key Services

#### SageMaker
End-to-end ML platform. Build, train, and deploy models at scale.

#### Rekognition
Computer vision — analyze images and videos. Face detection, object recognition.

#### Comprehend
NLP — sentiment analysis, entity recognition, language detection.

#### Lex
Chatbot service. Conversational AI using voice and text.

#### Polly
Text-to-speech. Converts written text into spoken audio.

#### Translate
Neural machine translation.

#### Transcribe
Speech-to-text — convert audio to written transcripts.

#### Forecast
Time-series machine learning for demand prediction.

#### Personalize
Real-time recommendations.

### Summary Table

| Service | Function |
|---|---|
| SageMaker | Build/train/deploy ML models |
| Rekognition | Image and video AI analysis |
| Comprehend | NLP text analysis |
| Lex | Chatbot / conversational AI |
| Polly | Text to speech |
| Translate | Language translation |
| Transcribe | Speech to text |
| Forecast | Demand prediction |
| Personalize | Product recommendations |

### Real-world Example

An OTT platform uses Rekognition to tag video content, Comprehend to analyze viewer comments, Personalize to suggest shows, and SageMaker to predict churn.

### Conclusion

AWS AI/ML services democratize machine learning by providing managed APIs that any developer can integrate, regardless of ML background.

---

## Q9. Explain AWS analytics services.

### Introduction

Organizations generate massive volumes of data. AWS analytics services help collect, store, process, and visualize this data.

### Key Services

#### Amazon Athena
SQL queries on S3 data. Serverless. Pay per query. No ETL needed.

#### Amazon EMR
Big data processing using Hadoop, Spark, HBase. Useful for large-scale ETL and ML data preparation.

#### Amazon Kinesis
Real-time data streaming. Process live events, logs, and transactions.
- Kinesis Streams: raw stream ingestion.
- Kinesis Firehose: deliver to S3, Redshift.
- Kinesis Analytics: real-time SQL on streams.

#### AWS Glue
Serverless ETL service. Automatically discovers data and generates ETL code.

#### Amazon Redshift
Petabyte-scale cloud data warehouse. MPP for fast analytical queries.

#### Amazon QuickSight
BI dashboards and visualizations. Serverless, pay-per-session.

### Diagram

```text
Raw Data Sources
      |
   Kinesis (stream) → Firehose → S3
      |                              |
   Lambda (transform)            Glue (ETL)
                                     |
                               Redshift (warehouse)
                                     |
                              QuickSight (dashboard)
```

### Conclusion

AWS analytics services form a full data pipeline — from ingestion and processing to visualization and business insight.

---

## Q10. Explain Amazon SNS and SQS — application integration services.

### Introduction

Modern applications are built as microservices and distributed systems. SNS and SQS enable these services to communicate reliably without tight coupling.

### Amazon SNS

Simple Notification Service. A pub/sub messaging service. [file:1]

**How it works:**
- A publisher sends a message to an SNS topic.
- SNS delivers the message to all subscribed endpoints.

**Subscribers:** Lambda, SQS, HTTP/HTTPS, email, SMS, mobile push.

**Example:** When a server CPU goes above 90%, CloudWatch triggers SNS, which sends email alerts and invokes a Lambda function.

### Amazon SQS

Simple Queue Service. A message queue service. [file:1]

**How it works:**
- A producer puts messages into a queue.
- A consumer pulls messages and processes them.
- Decouples producers and consumers.

**Queue Types:**
- Standard Queue: At-least-once delivery, best-effort ordering.
- FIFO Queue: Exactly-once delivery, strict ordering.

**Example:** An e-commerce order processing system places orders in an SQS queue. Backend processors independently process orders without overloading the database.

### Comparison

| Basis | SNS | SQS |
|---|---|---|
| Model | Push (pub/sub) | Pull (queue) |
| Fanout | Yes | No |
| Retention | No storage | Up to 14 days |
| Use case | Fan-out notifications | Async decoupling |

### Conclusion

SNS and SQS are fundamental to building resilient, decoupled cloud architectures. They are frequently used together in event-driven systems.

---

## Q11. Explain AWS monitoring and management services: CloudWatch, Trusted Advisor, and Auto Scaling.

### Introduction

Operating cloud infrastructure requires continuous monitoring, expert guidance, and automatic resource adjustment.

### Amazon CloudWatch

Monitoring and observability service that collects and tracks metrics, logs, and events. [file:1]

**Key Features:**
- Dashboards for visual monitoring.
- Alarms trigger actions when thresholds are breached.
- Logs — centralized log management.
- Custom metrics from applications.
- Integrates with SNS for alerts.

**Example:** Set an alarm when EC2 CPU > 80%. CloudWatch triggers SNS to notify the team and Auto Scaling to add more instances.

### AWS Trusted Advisor

An automated expert that advises on five categories: [file:1]

1. Cost optimization.
2. Performance.
3. Security.
4. Fault tolerance.
5. Service limits.

**Example:** Trusted Advisor flags an S3 bucket with public access or an EC2 instance that is underutilized.

### AWS Auto Scaling

Automatically adjusts the number of instances based on demand. [file:1]

**Key Features:**
- Scale out (add instances) during high traffic.
- Scale in (remove instances) during low traffic.
- Maintains performance and reduces cost.
- Works with EC2, DynamoDB, ECS, and more.

### Conclusion

CloudWatch, Trusted Advisor, and Auto Scaling together form the operational excellence layer of AWS — monitor, advise, and adapt automatically.

---

## Q12. Explain AWS developer and DevOps services.

### Introduction

AWS provides a full suite of developer tools that support CI/CD pipelines and agile software delivery.

### Services

#### CodeCommit
Managed Git repository. Source code storage.

#### CodeBuild
Continuous integration. Compiles code, runs tests, produces build artifacts.

#### CodeDeploy
Automates deployment to EC2, Lambda, or on-premises.

#### CodePipeline
Orchestrates the full CI/CD pipeline — from commit to production. [file:1]

### Complete Pipeline Flow

```text
Developer commits code
        |
    CodeCommit (Source control)
        |
    CodeBuild (Build + Test)
        |
    CodeDeploy (Deploy to environment)
        |
   Running Application
(Entire flow managed by CodePipeline)
```

### DevOps Connection

These services are the AWS implementation of a full DevOps pipeline. Combined with CloudFormation (IaC), CloudWatch (monitoring), and Lambda (automation), they enable continuous delivery of software with minimal human intervention.

### Conclusion

AWS developer tools reduce manual deployment effort, improve deployment frequency, and increase software quality through automation.

---

## 2-Mark Probable Questions

1. What is EC2?
2. What is Lambda?
3. What is S3?
4. What is EBS?
5. What is VPC?
6. What is Route 53?
7. What is CloudFront?
8. What is ELB?
9. What is RDS?
10. What is DynamoDB?
11. What is CloudWatch?
12. What is Auto Scaling?
13. What is SageMaker?
14. What is SNS?
15. What is SQS?
16. What is an Availability Zone?
17. What is an AWS Region?
18. What is CloudFormation?
19. What is AWS Trusted Advisor?
20. What is Kinesis?

---

## 5-Mark Probable Questions

1. Explain any five AWS compute services.
2. Differentiate between EC2 and Lambda.
3. Differentiate between S3, EBS, and EFS.
4. Explain Amazon VPC and its subnets.
5. Explain the AWS Global Infrastructure.
6. Explain any five AWS AI/ML services.
7. Differentiate between SNS and SQS.
8. Differentiate between RDS and DynamoDB.
9. Explain CloudWatch and its features.
10. Explain CloudFront and its working.

---

## 10-Mark Probable Questions

1. Explain AWS compute services in detail with comparison.
2. Explain the AWS Global Infrastructure with diagram.
3. Explain AWS storage services with examples.
4. Explain AWS database services with comparison table.
5. Explain AWS networking services in detail.
6. Explain AWS AI/ML services.
7. Explain AWS analytics services.
8. Explain SNS and SQS with use cases and comparison.
9. Explain AWS deployment methods.
10. Explain CloudWatch, Auto Scaling, and Trusted Advisor.

---

## Viva Questions with Answers

**1. What is EC2?**  
EC2 is a virtual server service that provides resizable compute capacity in the cloud. [file:1]

**2. What is Lambda?**  
Serverless compute service. Runs code in response to events without provisioning servers.

**3. What is S3?**  
Object storage with unlimited capacity, 11 nines durability, and multiple storage classes.

**4. What is the difference between S3 and EBS?**  
S3 is object storage accessible by any device. EBS is block storage attached to one EC2 instance at a time.

**5. What is a VPC?**  
Virtual Private Cloud — a logically isolated network in AWS. [file:1]

**6. What is an Availability Zone?**  
A physically separate data center within an AWS Region.

**7. What is Route 53?**  
AWS's DNS service that routes user traffic to AWS endpoints.

**8. What is CloudFront?**  
A CDN that caches content at edge locations to reduce latency for end users.

**9. What is Elastic Load Balancing?**  
Distributes incoming traffic across multiple EC2 instances.

**10. What is RDS?**  
A managed relational database service supporting multiple SQL engines.

**11. What is DynamoDB?**  
A serverless NoSQL database with single-digit millisecond latency.

**12. What is SageMaker?**  
A fully managed service for building, training, and deploying machine learning models.

**13. What is Kinesis?**  
A real-time streaming data ingestion and processing service.

**14. What is Auto Scaling?**  
Automatically adjusts the number of resources in response to demand. [file:1]

**15. What is the difference between SNS and SQS?**  
SNS is a push-based pub/sub notification service. SQS is a pull-based message queue for decoupling services. [file:1]

---

## Common Mistakes Students Make in Exams

- Confusing **S3** (object storage) with **EBS** (block storage).
- Writing about EC2 without mentioning pricing models.
- Not explaining the **difference between Region and AZ**.
- Confusing **CloudWatch** (monitoring) with **CloudTrail** (auditing).
- Forgetting that **Lambda is serverless** — common mistake is saying it "runs on EC2."
- Confusing **SNS** (push) with **SQS** (pull/queue).
- Not including **EFS** when listing storage services.
- Mixing up **RDS** (relational) and **DynamoDB** (NoSQL).
- Calling Route 53 a "server" — it is a DNS service.
- Forgetting to mention **edge locations** when explaining CloudFront.

---

## Important Keywords Likely to Fetch Marks

EC2, Lambda, serverless, ECS, EKS, Fargate, Elastic Beanstalk, S3, EBS, EFS, Glacier, RDS, Aurora, DynamoDB, Redshift, ElastiCache, VPC, subnet, security group, Internet Gateway, Route 53, DNS, CloudFront, CDN, edge location, ELB, ALB, NLB, Direct Connect, SageMaker, Rekognition, Comprehend, Lex, Polly, Translate, Transcribe, Kinesis, Athena, Glue, QuickSight, SNS, SQS, CloudWatch, Auto Scaling, CloudFormation, Trusted Advisor, Region, Availability Zone, On-Demand, Reserved, Spot, Dedicated, IaC.

---

## Important Exam Tips

### Writing 10-Mark Service Answers

1. Start with a definition.
2. List key features in bullets.
3. Add a table if comparing with another service.
4. Draw a simple ASCII diagram if applicable.
5. Give a real-world example.
6. End with a short conclusion.

### Memory Tricks

- **Compute services:** E-L-E-F-B = EC2, Lambda, ECS/EKS, Fargate, Beanstalk
- **Storage services:** S-E-E-G = S3, EBS, EFS, Glacier
- **Database services:** R-A-D-R-E-N = RDS, Aurora, DynamoDB, Redshift, ElastiCache, Neptune
- **AI services:** S-R-C-L-P-T-T = SageMaker, Rekognition, Comprehend, Lex, Polly, Transcribe, Translate
- **Networking:** V-R-C-E-D = VPC, Route53, CloudFront, ELB, Direct Connect

---

## Quick Revision Notes

### 1. Deployment Methods
Console → GUI | CLI → Scripts | SDK → Code | CloudFormation → IaC | Beanstalk → PaaS [file:1]

### 2. Global Infrastructure
Region (geo area) → AZ (data center) → Edge Locations (CDN cache) [file:1]

### 3. Compute Services
EC2 = VMs | Lambda = Serverless | ECS/EKS = Containers | Fargate = Serverless Containers | Beanstalk = PaaS [file:1]

### 4. Storage Services
S3 = Object | EBS = Block (EC2 disk) | EFS = Shared File | Glacier = Archive

### 5. Database Services
RDS = SQL | DynamoDB = NoSQL | Redshift = Warehouse | ElastiCache = Cache | Aurora = High perf SQL

### 6. Networking
VPC = Virtual Network | Route 53 = DNS | CloudFront = CDN | ELB = Load Balancer | Direct Connect = Private link

### 7. AI/ML
SageMaker = ML platform | Rekognition = Image AI | Comprehend = NLP | Lex = Chatbot | Polly = TTS

### 8. Analytics
Kinesis = Real-time streams | Athena = SQL on S3 | Glue = ETL | Redshift = Warehouse | QuickSight = BI

### 9. Integration
SNS = Push/Pub-Sub | SQS = Queue/Pull [file:1]

### 10. Management
CloudWatch = Monitoring | CloudTrail = Audit | Trusted Advisor = Best practices | Auto Scaling = Scale automatically [file:1]

---

## Most Expected Questions

### Very High Probability

1. Explain AWS compute services.
2. Explain AWS storage services with comparison.
3. Explain Amazon VPC.
4. Explain AWS Global Infrastructure.
5. Explain AI/ML services.

### High Probability

6. Explain AWS database services.
7. Explain SNS and SQS.
8. Explain networking services: Route 53, CloudFront, ELB.
9. Explain deployment methods.
10. Explain CloudWatch and Auto Scaling.

---

## Final Revision Formula

**Unit 3 = Deploy + Global Infra + Compute + Storage + Database + Network + AI/ML + Analytics + Integration + Management**

> The golden rule: Know the **name, definition, key features, use case, and one example** for every AWS service. That alone can get you full marks.

