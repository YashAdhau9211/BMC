# AWS Cloud Practitioner — Unit 1 Exam Preparation Question Bank + Study Notes

## Cover Page

**Subject:** AWS Cloud Practitioner  
**Unit:** Unit 1 — Cloud Concepts  
**Prepared For:** University Semester Examination  
**Material Type:** Detailed Study Notes + Question Bank + Exam Answers + Revision Notes  
**Level:** Beginner friendly, exam-oriented, full-mark preparation material  

---

## Subject Introduction

AWS Cloud Practitioner is the foundation-level study of cloud computing and Amazon Web Services. This subject helps students understand what cloud computing is, why organizations use AWS, how AWS helps businesses reduce cost and improve speed, what principles guide cloud design, how migration to cloud is planned, and how cloud economics changes the way companies think about IT spending. [file:1]

For university exams, Unit 1 is very important because it forms the base for later topics such as security, billing, pricing, and core AWS services. A student who understands Unit 1 clearly can write strong answers in simple language and score high marks. [file:1]

---

## Unit Overview

### Unit 1 Syllabus

**Cloud Concepts**  
- Define the benefits of the AWS Cloud. [file:1]
- Identify design principles of the AWS Cloud. [file:1]
- Understand the benefits of and strategies for migration to the AWS Cloud. [file:1]
- Understand concepts of cloud economics. [file:1]

### What this material covers

This material is designed so that a student can revise the entire Unit 1 from one place. It includes detailed theory, probable exam questions, 10-mark answers, short notes, one-word definitions, tables, diagrams, viva questions, probable 2-mark and 5-mark questions, and quick revision notes. [file:1]

### How to study this unit

1. Read the basic concepts first.
2. Learn definitions and keywords.
3. Practice 10-mark answers in exam format.
4. Revise differences and tables.
5. Use quick revision section before exam.

### Exam Tip

In AWS university papers, answers score better when they contain: definition, key headings, one table or diagram, example, and short conclusion.

---

## Important Theory Notes

## 1. What is Cloud Computing?

### Definition

Cloud computing is the on-demand delivery of IT resources over the internet with pay-as-you-go pricing. Instead of buying and maintaining physical data centers and servers, users can access services such as computing power, storage, and databases as needed from a cloud provider like AWS. [file:1]

### Simple Meaning

Cloud computing means using IT resources like servers, storage, databases, and software through the internet whenever required, and paying only for what is used.

### Main Features of Cloud Computing

- On-demand availability.
- Pay-as-you-go pricing.
- Elastic scaling up and down.
- Access through internet.
- Faster deployment of resources. [file:1]

### Real-world Example

A startup launching an e-commerce website during Diwali sale can increase server capacity quickly on AWS when traffic grows and reduce it after the sale ends. This avoids buying expensive servers for peak load only.

### Industry Connection

- In **DevOps**, cloud computing supports faster deployment and automation.
- In **ITIL**, cloud improves service delivery, service availability, and cost optimization.
- In service-based companies, cloud helps deliver applications to clients faster.

### ASCII Diagram

```text
User/Application
      |
      v
  Internet
      |
      v
+------------------------+
| Cloud Provider (AWS)   |
| Compute | Storage | DB |
+------------------------+
```

### Important Keywords

Cloud computing, on-demand, internet-based services, scalability, elasticity, pay-as-you-go, virtual resources.

---

## 2. What is AWS?

### Definition

AWS stands for Amazon Web Services. It is a secure cloud platform that provides a broad set of global cloud-based services such as compute, storage, networking, databases, analytics, monitoring, and more. [file:1]

### Simple Explanation

AWS provides IT services as building blocks. Organizations choose the required services and combine them to build applications quickly. [file:1]

### Ways to Access AWS

- AWS Management Console.
- AWS CLI.
- AWS SDKs. [file:1]

### Examples of AWS Services

- Compute: EC2, Lambda.
- Storage: S3, EBS, EFS.
- Databases: RDS, Aurora.
- Monitoring: CloudWatch, CloudTrail.
- Messaging: SNS, SQS. [file:1]

### Why Students Must Remember This

In exams, many answers can begin with a one-line AWS definition and then connect the service or concept to AWS infrastructure and business benefits.

---

## 3. Benefits of the AWS Cloud

This is one of the most important topics in Unit 1.

### Main Benefits

#### 1. Trade Capital Expense for Variable Expense

Organizations do not need to spend a large amount of money upfront on servers and data centers. They can pay only when they use resources. This changes IT spending from capital expenditure (CapEx) to operational expenditure (OpEx). [file:1]

#### 2. Benefit from Massive Economies of Scale

Because AWS serves many customers globally, it achieves lower cost per unit and passes savings to users through lower pricing. [file:1]

#### 3. Stop Guessing Capacity

In traditional IT, companies often overbuy or underbuy infrastructure. In AWS, resources can be scaled as needed, so capacity planning becomes easier and more accurate. [file:1]

#### 4. Increase Speed and Agility

Resources can be provisioned within minutes instead of weeks. This helps organizations innovate faster, test quickly, and release services earlier. [file:1]

#### 5. Stop Spending Money Running Data Centers

AWS manages the physical infrastructure, so companies can focus on business goals instead of power, cooling, hardware maintenance, and server replacement. [file:1]

#### 6. Go Global in Minutes

Applications can be deployed in multiple regions quickly, reducing latency for users and improving customer experience. [file:1]

### Table: AWS Cloud Benefits

| Benefit | Meaning | Exam Writing Point |
|---|---|---|
| Variable expense | Pay only for usage | No large upfront cost |
| Economies of scale | AWS cost advantage | Lower pricing |
| Elastic capacity | Scale as needed | Avoid over/under provisioning |
| Agility | Faster deployment | Faster innovation |
| Reduced maintenance | AWS handles data centers | Focus on core business |
| Global reach | Use multiple regions | Better latency and expansion |

### Easy Mnemonic

**V-E-S-A-M-G** = Variable, Economies, Stop guessing, Agility, Maintenance reduction, Global reach.

### Real-world Example

Netflix-style streaming platforms, online gaming systems, and fintech applications benefit from elasticity and global deployment because user demand changes rapidly across time and geography.

### Exam Tip

When writing this answer, always connect each benefit with one business advantage such as cost saving, speed, customer satisfaction, or flexibility.

---

## 4. Design Principles of the AWS Cloud

AWS Cloud design principles guide how cloud-based systems should be built for efficiency, flexibility, and reliability.

### Important Design Principles

#### 1. Stop Guessing Capacity Needs

Do not build for fixed assumptions. Use scalable resources and adjust based on actual demand.

#### 2. Test Systems at Production Scale

Cloud makes it easier to test with realistic workloads because resources can be created on demand.

#### 3. Automate to Make Architectural Experimentation Easier

Automation reduces manual effort and improves consistency. This is closely connected with DevOps practices like Infrastructure as Code.

#### 4. Allow for Evolutionary Architectures

Systems should be designed to improve over time. In cloud, resources and services can be changed with less disruption.

#### 5. Drive Architectures Using Data

Use metrics, logs, and monitoring tools to make decisions. Cloud environments support better visibility.

#### 6. Improve Through Game Days

Organizations perform simulations and practice failure scenarios so teams are prepared for real incidents.

### Why These Principles Matter

These principles reduce risk, increase reliability, support innovation, and help companies build systems that can grow with business needs.

### DevOps Link

- Automation supports CI/CD.
- Monitoring supports observability.
- Evolutionary architecture supports continuous improvement.
- Game days support resilience engineering.

### ASCII Diagram

```text
Design Principles in AWS Cloud
        |
        +--> Scalability
        +--> Automation
        +--> Monitoring
        +--> Continuous Improvement
        +--> Resilience Testing
```

### Exam Tip

If the university asks for design principles, write at least 5 to 6 points with one-line explanation each and then add a short paragraph about why these principles are useful in modern IT organizations.

---

## 5. Cloud Deployment Models

### Definition

A cloud deployment model describes where cloud infrastructure is located and how it is managed.

### Types of Deployment Models

#### 1. Public Cloud

Infrastructure is owned and managed by a cloud provider and shared among multiple customers. Example: hosting a web app on AWS EC2 or storing files in Amazon S3. [file:1]

#### 2. Private Cloud

Infrastructure is dedicated to one organization only. Example: a bank running a private cloud in its own data center. [file:1]

#### 3. Hybrid Cloud

A combination of on-premises infrastructure and public cloud connected together. Example: a company stores sensitive data on-premises but uses AWS for analytics. [file:1]

#### 4. Community Cloud

Infrastructure is shared by several organizations with common requirements such as compliance or security. Example: universities sharing a research cloud. [file:1]

### Table: Deployment Models Comparison

| Model | Ownership | Cost | Control | Example |
|---|---|---|---|---|
| Public | Cloud provider | Lower upfront | Moderate | AWS-hosted web app |
| Private | Single organization | Higher | High | Internal enterprise cloud |
| Hybrid | Shared arrangement | Balanced | High + flexible | On-prem + AWS |
| Community | Shared organizations | Shared | Moderate | Government shared cloud |

### Exam Tip

Students often confuse hybrid and private cloud. Remember: **hybrid = on-prem + public cloud together**.

---

## 6. Service Models: IaaS, PaaS, SaaS

### Definition

Cloud service models explain how much responsibility remains with the customer and how much is handled by the cloud provider.

### IaaS — Infrastructure as a Service

Provides basic building blocks such as virtual servers, networking, and storage. Customers control the operating system, applications, and many configurations. Example: Amazon EC2. [file:1]

### PaaS — Platform as a Service

The cloud provider manages the underlying infrastructure and operating system, while the customer focuses on application development and deployment. Example: Amazon RDS as a platform-managed database environment. [file:1]

### SaaS — Software as a Service

A complete software solution is delivered to users, and the provider manages almost everything. Users simply access the software. [file:1]

### Table: IaaS vs PaaS vs SaaS

| Feature | IaaS | PaaS | SaaS |
|---|---|---|---|
| Control level | Highest | Medium | Lowest |
| Customer manages | OS, apps, config | Apps and data | Mostly only usage |
| Provider manages | Hardware, virtualization | Infra + OS | Entire platform and software |
| Example | EC2 | RDS-type managed platform | Ready-to-use cloud software |

### Memory Trick

**I-P-S = Infra, Platform, Software**  
Control decreases from IaaS to SaaS.

---

## 7. Strategies for Migration to AWS Cloud

Migration means moving applications, data, or workloads from on-premises systems or another environment to AWS Cloud.

### Benefits of Migration

- Better scalability.
- Lower infrastructure management effort.
- Faster deployment.
- Improved agility.
- Global reach.
- Better cost optimization through usage-based pricing. [file:1]

### Common Migration Strategies

A simple university-friendly way to remember migration is through practical strategies:

#### 1. Rehosting

Also called “lift and shift.” The application is moved to AWS with minimum changes.

#### 2. Replatforming

Some improvements are made during migration, but the application is not fully redesigned.

#### 3. Refactoring / Re-architecting

The application is redesigned to take advantage of cloud-native features such as scalability, microservices, or serverless services.

#### 4. Repurchasing

The organization moves to a different product, often a SaaS solution.

#### 5. Retiring

Unnecessary applications are removed.

#### 6. Retaining

Some systems remain on-premises for business, technical, or compliance reasons.

### Step-by-step Migration Process

1. Assess current applications and infrastructure.
2. Identify business goals.
3. Select suitable migration strategy.
4. Plan cost, security, and timeline.
5. Migrate data and workloads.
6. Test performance and security.
7. Optimize after migration.

### Real-world Example

A university may keep its student record database on-premises initially but move its website, backup, and learning portal to AWS. Later, analytics and application services can also be migrated.

### Industry Link

Service-based companies often begin with rehosting for quick client migration and later move toward refactoring for long-term optimization.

---

## 8. AWS Cloud Adoption Framework (AWS CAF)

This topic is extremely important for migration strategy questions.

### Definition

AWS Cloud Adoption Framework provides guidance and best practices to help organizations build a comprehensive approach to cloud adoption across the organization and throughout the IT lifecycle. It also helps identify gaps in skills and processes. [file:1]

### Purpose of AWS CAF

- Align business and IT goals.
- Identify organizational gaps.
- Improve cloud adoption planning.
- Support successful migration.
- Create prescriptive work streams for change. [file:1]

### Six Perspectives of AWS CAF

#### 1. Business Perspective

Focuses on business managers, finance managers, budget owners, and strategy stakeholders. It helps create a strong business case for cloud adoption and aligns IT strategy with business goals. [file:1]

#### 2. People Perspective

Focuses on HR, staffing, and training needs. It identifies skill gaps, role changes, and organizational changes required for cloud adoption. [file:1]

#### 3. Governance Perspective

Focuses on CIOs, program managers, enterprise architects, and analysts. It ensures IT investments provide maximum business value with minimum business risk. [file:1]

#### 4. Platform Perspective

Focuses on CTOs, IT managers, and solution architects. It deals with architectures, systems, and technical implementation. [file:1]

#### 5. Security Perspective

Focuses on security managers and analysts. It ensures visibility, control, auditability, and appropriate security controls. [file:1]

#### 6. Operations Perspective

Focuses on IT operations and support teams. It addresses day-to-day operating procedures, process changes, and training needs. [file:1]

### Table: AWS CAF Perspectives

| Perspective | Main Focus | Stakeholders |
|---|---|---|
| Business | Business value | Managers, finance, strategy teams |
| People | Skills and roles | HR, staffing, people managers |
| Governance | Risk and value alignment | CIO, architects, program managers |
| Platform | Technical architecture | CTO, IT managers, architects |
| Security | Protection and compliance | Security teams |
| Operations | Daily operations | IT operations and support |

### ASCII Diagram

```text
AWS CAF
 |
 +-- Business
 +-- People
 +-- Governance
 +-- Platform
 +-- Security
 +-- Operations
```

### Exam Tip

When a question asks “Explain AWS CAF,” always write: definition, purpose, six perspectives, and how it supports cloud migration success.

---

## 9. Cloud Economics

Cloud economics means understanding how cloud changes the financial model of IT.

### Definition

Cloud economics studies how organizations reduce cost and improve efficiency by using cloud services instead of traditional infrastructure.

### Key Concepts of Cloud Economics

#### 1. CapEx to OpEx

Traditional IT often requires large upfront investment in servers, storage, networking equipment, building space, power, and cooling. Cloud changes this to operational spending where payment happens based on consumption. [file:1]

#### 2. Pay-as-you-go Pricing

Users pay only for the resources they consume, not for unused idle capacity. [file:1]

#### 3. Elasticity Saves Cost

Resources can be increased or decreased as needed, which avoids paying for excess infrastructure. [file:1]

#### 4. Economies of Scale

AWS can offer lower costs because of global scale and large customer base. [file:1]

#### 5. Reduced Operational Overhead

AWS manages physical infrastructure, so organizations save on maintenance, staffing burden, and facility management. [file:1]

### Table: Traditional IT vs Cloud Economics

| Factor | Traditional IT | AWS Cloud |
|---|---|---|
| Spending model | Capital expense | Operational expense |
| Capacity planning | Guess in advance | Scale on demand |
| Resource utilization | Often low | More optimized |
| Setup time | Weeks/months | Minutes |
| Maintenance | Managed by company | Managed largely by AWS |
| Global expansion | Expensive and slow | Faster and easier |

### Example

A company expecting seasonal traffic usually buys extra servers in traditional data centers, even if those servers stay idle most of the year. In AWS, it can use extra resources only during peak season.

### Exam Tip

Whenever writing on cloud economics, use terms like **CapEx, OpEx, elasticity, economies of scale, utilization, variable cost**.

---

## 10. Why Organizations Move to AWS

### Main Reasons

- Faster innovation.
- Better customer experience.
- Lower cost.
- Global availability.
- Flexibility and scalability.
- Reduced infrastructure burden.
- Better support for modern software practices like DevOps. [file:1]

### Connection to ITIL and Service Management

From an ITIL point of view, AWS supports:

- Better service availability.
- Faster incident recovery.
- Improved capacity management.
- Better financial management of IT services.
- Stronger service continuity planning.

### Connection to DevOps

AWS helps DevOps teams by enabling:

- Rapid environment creation.
- Automation.
- Continuous integration and deployment.
- Monitoring and logs.
- Faster rollback and experimentation.

---

## One-Word / Very Short Definitions

- **Cloud:** Internet-based IT resource delivery.
- **AWS:** Amazon Web Services.
- **Agility:** Speed of change.
- **Elasticity:** Automatic scaling.
- **Scalability:** Ability to grow.
- **Migration:** Moving workloads.
- **CapEx:** Capital expense.
- **OpEx:** Operational expense.
- **Public cloud:** Provider-managed shared cloud.
- **Private cloud:** Single-organization cloud.
- **Hybrid cloud:** On-prem plus cloud.
- **IaaS:** Infrastructure service model.
- **PaaS:** Platform service model.
- **SaaS:** Software service model.
- **CAF:** Cloud Adoption Framework.
- **Economies of scale:** Lower unit cost at larger volume.

---

## Short Notes

### Short Note 1: Agility in AWS

Agility means the ability to create and modify IT resources quickly. In AWS, servers, storage, and services can be provisioned within minutes, allowing organizations to test ideas quickly and bring products to market faster. [file:1]

### Short Note 2: Elasticity

Elasticity means the ability to scale resources up or down depending on demand. This avoids over-provisioning and saves cost. [file:1]

### Short Note 3: Go Global in Minutes

AWS has infrastructure in multiple geographical regions. Organizations can deploy applications near users and reduce latency, improving user experience. [file:1]

### Short Note 4: Variable Expense Model

AWS allows organizations to pay only for actual usage, reducing the need for heavy upfront investment. This makes IT spending more flexible. [file:1]

### Short Note 5: AWS CAF

AWS CAF is a structured framework that helps organizations adopt cloud successfully by focusing on business, people, governance, platform, security, and operations. [file:1]

---

## Important Differences in Table Format

### Public Cloud vs Private Cloud vs Hybrid Cloud

| Basis | Public Cloud | Private Cloud | Hybrid Cloud |
|---|---|---|---|
| Ownership | Cloud provider | Single organization | Both |
| Cost | Lower upfront | Higher | Moderate |
| Control | Moderate | High | High + flexible |
| Scalability | Very high | Limited by internal setup | High |
| Example | AWS EC2 | Internal data center | On-prem + AWS |

### CapEx vs OpEx

| Basis | CapEx | OpEx |
|---|---|---|
| Meaning | Upfront capital spending | Ongoing operational spending |
| Example | Buying servers | Paying monthly cloud bill |
| Flexibility | Low | High |
| Risk | High if demand changes | Lower because of usage-based cost |

### Scalability vs Elasticity

| Basis | Scalability | Elasticity |
|---|---|---|
| Meaning | Ability to increase capacity | Ability to automatically increase or decrease capacity |
| Nature | Growth-focused | Dynamic adjustment |
| Example | Add bigger servers | Auto scale during traffic spike |

### IaaS vs PaaS vs SaaS

| Basis | IaaS | PaaS | SaaS |
|---|---|---|---|
| Control | High | Medium | Low |
| Managed by provider | Hardware | Hardware + OS/runtime | Full stack |
| Customer focus | Infrastructure config | App development | End use |

---

## Previous Exam Style Questions

1. Define cloud computing and explain the major benefits of AWS Cloud.
2. Explain the design principles of AWS Cloud in detail.
3. What is cloud economics? Explain CapEx and OpEx with examples.
4. Explain the different cloud deployment models with suitable examples.
5. Differentiate between IaaS, PaaS, and SaaS.
6. Explain the strategies for migration to AWS Cloud.
7. What is AWS Cloud Adoption Framework? Explain its perspectives.
8. Why do organizations migrate to AWS Cloud? Discuss business and technical reasons.
9. Explain elasticity and agility in cloud computing.
10. Write a note on economies of scale in AWS.
11. Compare traditional IT infrastructure with AWS Cloud model.
12. Explain how AWS supports global deployment and low-latency applications.

---

## 10-Mark Question Bank with Detailed Answers

## Q1. Define cloud computing and explain its major characteristics and benefits.

### Most Important Question

### Introduction

Cloud computing has transformed the way organizations use IT resources. Instead of owning physical servers and infrastructure, businesses now obtain computing services through the internet whenever needed.

### Definition

Cloud computing is the on-demand delivery of IT resources over the internet with pay-as-you-go pricing. It allows users to access compute power, storage, databases, and other services as needed from a cloud provider like AWS. [file:1]

### Main Characteristics

#### 1. On-demand self-service

Users can provision resources whenever needed without waiting for long procurement cycles.

#### 2. Broad network access

Services are available over the internet and can be accessed from many devices.

#### 3. Resource pooling

Cloud providers share large pools of infrastructure to serve multiple customers efficiently.

#### 4. Rapid elasticity

Capacity can be increased or reduced quickly based on demand.

#### 5. Measured service

Users are billed based on actual resource consumption.

### Benefits

- Faster deployment.
- Better flexibility.
- Reduced cost.
- Improved scalability.
- Easier global access.
- Better support for innovation. [file:1]

### Example

An online food delivery app can use more servers during dinner hours and reduce them late at night.

### Conclusion

Cloud computing provides speed, flexibility, and cost efficiency. It is preferred by modern organizations because it allows them to focus on innovation instead of infrastructure management.

---

## Q2. Explain the major benefits of AWS Cloud.

### Introduction

AWS provides several benefits over traditional IT infrastructure. These benefits improve both technical performance and business outcomes.

### Definition

AWS Cloud refers to the on-demand global cloud infrastructure and services offered by Amazon Web Services. [file:1]

### Major Benefits

#### 1. Trade CapEx for OpEx

Users avoid heavy initial investment and instead pay based on actual usage. [file:1]

#### 2. Massive economies of scale

AWS serves many customers globally, which lowers cost. [file:1]

#### 3. Stop guessing capacity

Capacity can be scaled based on demand. [file:1]

#### 4. Increase speed and agility

Resources can be provisioned in minutes. [file:1]

#### 5. Stop running data centers

AWS handles physical infrastructure, cooling, power, and maintenance. [file:1]

#### 6. Go global in minutes

Applications can be deployed in many regions quickly. [file:1]

### Industry Example

A startup can launch its app in India first and later expand to Europe or North America without building new data centers.

### Conclusion

AWS Cloud benefits organizations by reducing cost, increasing speed, supporting growth, and enabling international expansion.

---

## Q3. Explain the design principles of AWS Cloud.

### Introduction

Cloud design principles are best practices used to build effective and reliable systems on the cloud.

### Definition

Design principles of AWS Cloud are the architectural ideas that help organizations create scalable, efficient, and resilient cloud-based solutions.

### Principles

#### 1. Stop guessing capacity needs
Use scalable infrastructure.

#### 2. Test systems at production scale
Use on-demand resources for realistic testing.

#### 3. Automate experimentation
Automation supports consistency and faster deployment.

#### 4. Allow evolutionary architectures
Systems can change gradually with business needs.

#### 5. Drive architecture using data
Use monitoring and metrics for decision making.

#### 6. Improve through game days
Practice failure handling and recovery drills.

### Benefits of These Principles

- Better availability.
- Faster improvement.
- Lower risk.
- Easier DevOps integration.
- More reliable systems.

### Conclusion

AWS design principles help organizations build modern applications that are scalable, flexible, and efficient.

---

## Q4. Explain cloud deployment models with suitable examples.

### Introduction

Cloud deployment models describe how cloud environments are organized and who controls them.

### Definition

A deployment model indicates whether infrastructure is public, private, hybrid, or community based. [file:1]

### Types

#### Public Cloud
Owned by cloud provider and shared among customers. Example: Web app hosted on AWS EC2. [file:1]

#### Private Cloud
Dedicated to one organization only. Example: Internal banking infrastructure. [file:1]

#### Hybrid Cloud
Combination of on-premises and public cloud. Example: Company stores sensitive data on-prem and uses AWS analytics. [file:1]

#### Community Cloud
Shared by organizations with similar needs. Example: Government agencies sharing a secure cloud. [file:1]

### Comparison Points

- Cost.
- Control.
- Security.
- Flexibility.
- Use case.

### Conclusion

Each deployment model serves different business needs. Hybrid is common when organizations want both control and cloud flexibility.

---

## Q5. Differentiate between IaaS, PaaS, and SaaS.

### Introduction

Cloud service models define the level of management responsibility between customer and provider.

### Definitions

- IaaS provides infrastructure resources. [file:1]
- PaaS provides a managed platform for application development. [file:1]
- SaaS provides ready-to-use software over the internet. [file:1]

### Detailed Difference

| Basis | IaaS | PaaS | SaaS |
|---|---|---|---|
| Control | Highest | Medium | Lowest |
| Customer manages | OS, app, config | App and data | Only usage/settings |
| Examples | EC2 | Managed app/database platforms | Cloud software |
| Best for | Admin control | Developers | End users |

### Example

A developer using EC2 chooses the operating system and software manually, while on a managed platform much of that work is handled by the provider.

### Conclusion

As we move from IaaS to SaaS, customer control decreases and provider responsibility increases. [file:1]

---

## Q6. Explain the benefits of migration to AWS Cloud.

### Introduction

Migration to AWS means moving systems, workloads, or data from traditional environments to AWS Cloud.

### Main Benefits

- Improved agility.
- Faster deployment.
- Better scalability.
- Lower operational burden.
- Better cost efficiency.
- Access to global infrastructure. [file:1]

### Explanation

Migration helps businesses modernize their IT environment. Instead of maintaining old hardware and manual systems, they can use scalable services and focus on core business innovation.

### Real-world Example

An educational institution can migrate website hosting, backup storage, and student portals to AWS for better uptime and simpler scaling during admission periods.

### Conclusion

Migration is not only a technical change; it is also a business improvement strategy that supports speed, flexibility, and digital growth.

---

## Q7. Explain strategies for migration to AWS Cloud.

### Introduction

There is no single migration method for all applications. Different systems require different strategies.

### Migration Strategies

#### 1. Rehosting
Move application with minimal changes.

#### 2. Replatforming
Make small improvements while migrating.

#### 3. Refactoring
Redesign application for cloud-native benefits.

#### 4. Repurchasing
Move to another product or SaaS service.

#### 5. Retiring
Remove unused applications.

#### 6. Retaining
Keep some applications in current environment.

### Step-by-step Approach

1. Assess workload.
2. Identify business value.
3. Choose migration strategy.
4. Plan security and cost.
5. Execute migration.
6. Test and optimize.

### Conclusion

Choosing the right migration strategy reduces risk and improves the chances of successful cloud adoption.

---

## Q8. Explain AWS Cloud Adoption Framework in detail.

### Most Important Question

### Introduction

Cloud adoption affects the entire organization, not just the IT department. Therefore, a structured framework is needed.

### Definition

AWS Cloud Adoption Framework is a guidance model that helps organizations build a comprehensive approach to cloud adoption and identify gaps in skills and processes. [file:1]

### Need for AWS CAF

- Align business and IT goals.
- Identify organizational readiness.
- Improve migration planning.
- Manage people, security, and operations effectively. [file:1]

### Six Perspectives

#### 1. Business
Builds business case and prioritizes cloud initiatives. [file:1]

#### 2. People
Identifies training, staffing, and skill requirements. [file:1]

#### 3. Governance
Ensures risk control and value alignment. [file:1]

#### 4. Platform
Handles technical architecture and infrastructure planning. [file:1]

#### 5. Security
Supports auditability, visibility, and control. [file:1]

#### 6. Operations
Improves operational processes and service delivery. [file:1]

### Diagram

```text
Successful Cloud Adoption
        |
        v
+-------------------------+
| AWS CAF                 |
+-------------------------+
| Business                |
| People                  |
| Governance              |
| Platform                |
| Security                |
| Operations              |
+-------------------------+
```

### Conclusion

AWS CAF is important because successful migration depends on people, process, business value, governance, and technical readiness—not only technology.

---

## Q9. Explain cloud economics with suitable examples.

### Introduction

Cloud economics is the financial logic behind using cloud services instead of traditional infrastructure.

### Definition

Cloud economics refers to the cost and value principles that make cloud computing financially beneficial for organizations.

### Main Concepts

#### 1. CapEx to OpEx
Traditional infrastructure requires upfront capital spending, but cloud uses ongoing operational spending. [file:1]

#### 2. Pay-as-you-go
Users pay only for what they consume. [file:1]

#### 3. Elasticity
Resources expand and shrink with demand, preventing waste. [file:1]

#### 4. Economies of scale
Large providers reduce cost per unit through global scale. [file:1]

#### 5. Reduced maintenance cost
Less need to run data centers and manage hardware. [file:1]

### Example

A retail company may need heavy server usage only during festive season sales. Cloud lets it pay more only during the peak period and less during normal business periods.

### Conclusion

Cloud economics improves cost efficiency, financial flexibility, and infrastructure utilization.

---

## Q10. Differentiate between traditional IT infrastructure and AWS Cloud model.

### Introduction

Traditional data center models and cloud models differ in ownership, cost, management, and flexibility.

### Difference Table

| Basis | Traditional IT | AWS Cloud |
|---|---|---|
| Ownership | Company-owned servers | AWS-managed infrastructure |
| Cost type | CapEx heavy | OpEx based |
| Scaling | Slow | Fast |
| Deployment time | Weeks to months | Minutes |
| Maintenance | Company responsibility | Mostly AWS responsibility |
| Global expansion | Difficult | Easy |

### Explanation

Traditional systems require planning, purchasing, installation, and maintenance. AWS allows organizations to launch and manage services quickly without owning the physical infrastructure. [file:1]

### Conclusion

AWS Cloud provides better agility, flexibility, and cost efficiency than traditional infrastructure for many modern workloads.

---

## Q11. Explain agility and elasticity in the context of AWS Cloud.

### Introduction

Agility and elasticity are two major benefits of cloud computing and are commonly asked in short and long answers.

### Agility

Agility means the ability to obtain and configure IT resources quickly, enabling faster innovation and experimentation. [file:1]

### Elasticity

Elasticity means the ability to scale resources up or down instantly based on demand. [file:1]

### Difference

Agility focuses on speed of provisioning and business response, while elasticity focuses on dynamic resource adjustment.

### Example

A startup launching a new app demonstrates agility when it deploys its environment in minutes. The same app demonstrates elasticity when it handles a sudden increase in users without service failure.

### Conclusion

Both agility and elasticity help organizations become faster, smarter, and more cost efficient.

---

## Q12. Why do organizations choose AWS for global deployment?

### Introduction

Modern businesses often serve users in different countries, so global deployment is important.

### Explanation

AWS has infrastructure across the world, allowing applications to be deployed in multiple physical locations quickly. This reduces latency because applications are closer to end users. [file:1]

### Benefits

- Better customer experience.
- Lower response time.
- Faster business expansion.
- Greater availability.
- Regional presence for international growth. [file:1]

### Example

An edtech platform can host services closer to students in different regions to improve performance during online exams or live classes.

### Conclusion

Global deployment through AWS helps organizations reach users faster and support international growth with lower complexity.

---

## Q13. Explain the role of AWS Cloud in modern IT industry practices.

### Introduction

AWS is not only a technology platform; it also supports modern IT operating models.

### Role in Industry Practices

#### In DevOps
- Rapid environment creation.
- Automation support.
- Faster testing and deployment.
- Better scaling.

#### In ITIL / Service Management
- Better capacity management.
- Improved service continuity.
- Flexible financial management.
- Faster service provisioning.

#### In Software Companies
- Faster product launches.
- Reduced infrastructure overhead.
- Support for global customer base.

#### In Service-based Organizations
- Quick client onboarding.
- Easy project environment creation.
- Better cost control.

### Conclusion

AWS supports the practical needs of modern software engineering, service management, and digital transformation.

---

## Q14. Explain the statement “Stop guessing capacity” in AWS Cloud.

### Introduction

In traditional data centers, capacity planning is difficult because future demand is uncertain.

### Meaning

“Stop guessing capacity” means organizations do not need to predict exact future infrastructure requirements long in advance. AWS allows them to scale resources according to real-time needs. [file:1]

### Why This Matters

- Avoids over-provisioning.
- Avoids under-provisioning.
- Saves cost.
- Improves performance during demand spikes.
- Supports business growth. [file:1]

### Example

An online ticket booking site may see heavy traffic when a major event opens for booking. AWS allows temporary scaling for that period.

### Conclusion

This principle improves both technical efficiency and financial efficiency.

---

## Q15. Explain why AWS Cloud is suitable for beginners, startups, and growing enterprises.

### Introduction

AWS is useful for organizations of different sizes because it offers flexibility, low entry cost, and wide service choice.

### Reasons

- No need for large upfront investment. [file:1]
- Services can be started quickly. [file:1]
- Scaling is easy. [file:1]
- Many service options are available. [file:1]
- Global reach supports business growth. [file:1]
- Maintenance burden is reduced. [file:1]

### Example

A startup can begin with a small application environment and expand gradually as customers increase.

### Conclusion

AWS is suitable for both small and large organizations because it supports gradual growth, financial flexibility, and rapid innovation.

---

## 2-Mark Probable Questions

1. Define cloud computing.
2. What is AWS?
3. What is elasticity?
4. What is agility?
5. Define CapEx.
6. Define OpEx.
7. What is public cloud?
8. What is hybrid cloud?
9. Expand AWS CAF.
10. What is migration?
11. What is IaaS?
12. What is PaaS?
13. What is SaaS?
14. What is pay-as-you-go pricing?
15. What is economies of scale?

---

## 5-Mark Probable Questions

1. Explain any five benefits of AWS Cloud.
2. Differentiate between public, private, and hybrid cloud.
3. Write a short note on cloud economics.
4. Explain agility and elasticity.
5. Explain AWS CAF briefly.
6. Differentiate between IaaS, PaaS, and SaaS.
7. Explain migration benefits.
8. Explain CapEx and OpEx with examples.

---

## 10-Mark Probable Questions

1. Explain the major benefits of AWS Cloud in detail.
2. Describe the design principles of AWS Cloud.
3. Explain AWS Cloud Adoption Framework with all perspectives.
4. Explain cloud economics in detail with suitable examples.
5. Explain the strategies for migration to AWS Cloud.
6. Compare traditional IT with AWS Cloud.
7. Explain deployment models with examples.
8. Differentiate IaaS, PaaS, and SaaS in detail.
9. Explain agility, elasticity, and scalability in cloud computing.
10. Why do organizations adopt AWS Cloud? Explain business and technical reasons.

---

## Viva Questions with Answers

### 1. What is AWS?
AWS is Amazon Web Services, a cloud platform that offers on-demand IT services over the internet. [file:1]

### 2. What is cloud computing?
It is the on-demand delivery of IT resources over the internet with pay-as-you-go pricing. [file:1]

### 3. What is elasticity?
Elasticity is the ability to scale resources up or down based on demand. [file:1]

### 4. What is agility?
Agility is the ability to quickly provision and modify IT resources. [file:1]

### 5. What is the difference between CapEx and OpEx?
CapEx is upfront spending on infrastructure, while OpEx is ongoing spending based on usage.

### 6. What is public cloud?
It is a cloud model where infrastructure is owned by the provider and shared by multiple customers. [file:1]

### 7. What is hybrid cloud?
It is a combination of on-premises infrastructure and public cloud working together. [file:1]

### 8. What is AWS CAF?
AWS CAF is a framework that helps organizations adopt cloud successfully by addressing business and technical perspectives. [file:1]

### 9. What is IaaS?
Infrastructure as a Service gives virtualized infrastructure resources like compute, storage, and networking. [file:1]

### 10. Why do companies migrate to AWS?
They migrate for scalability, speed, reduced maintenance, global reach, and cost efficiency. [file:1]

### 11. What is pay-as-you-go?
It means customers pay only for the resources they actually use. [file:1]

### 12. What is economies of scale?
It means large-scale operation reduces cost per unit, allowing AWS to offer lower prices. [file:1]

---

## Common Mistakes Students Make in Exams

- Writing cloud computing definition without mentioning **on-demand** and **pay-as-you-go**.
- Confusing **scalability** with **elasticity**.
- Mixing up **public cloud** and **hybrid cloud**.
- Forgetting the six AWS CAF perspectives.
- Writing only points without explanation in 10-mark answers.
- Forgetting to add examples.
- Not differentiating **CapEx** and **OpEx** clearly.
- Writing IaaS, PaaS, SaaS in random order without explaining control level.
- Missing conclusion in long answers.
- Giving overly technical answers instead of simple exam language.

---

## Important Keywords Likely to Fetch Marks

Cloud computing, AWS, on-demand, pay-as-you-go, elasticity, scalability, agility, migration, cloud adoption, AWS CAF, business perspective, people perspective, governance, platform, security, operations, CapEx, OpEx, economies of scale, public cloud, private cloud, hybrid cloud, IaaS, PaaS, SaaS, global deployment, reduced latency, variable expense, cost optimization.

---

## Important Exam Tips

### How to write a 10-mark answer

1. Start with a definition.
2. Write 5 to 8 proper headings.
3. Add one table, list, or diagram.
4. Give one real-world example.
5. End with a short conclusion.

### Presentation Tips

- Underline keywords.
- Use headings and subheadings.
- Write in points for clarity.
- Draw simple diagrams wherever possible.
- Keep examples practical.

### Memory Tips

- Learn CAF as: **Business, People, Governance, Platform, Security, Operations**.
- Learn AWS Cloud benefits as: **Cost, Scale, Speed, Capacity, Maintenance, Global**.
- Learn service models in order: **IaaS → PaaS → SaaS**.

---

## Most Expected Questions

### Very High Probability

1. Explain the benefits of AWS Cloud.
2. Explain AWS Cloud Adoption Framework.
3. Explain cloud economics.
4. Explain cloud deployment models.
5. Differentiate IaaS, PaaS, and SaaS.

### High Probability

6. Explain migration strategies to AWS Cloud.
7. Explain design principles of AWS Cloud.
8. Differentiate traditional IT and AWS Cloud.
9. Explain agility and elasticity.
10. Why do organizations move to AWS?

---

## Quick Revision Notes

### 1. Cloud Computing in One View

- IT resources delivered over internet.
- Use when needed.
- Pay only for usage.
- Fast, flexible, scalable. [file:1]

### 2. AWS Cloud Benefits

- No heavy upfront cost.
- Pay-as-you-go.
- Elastic scaling.
- Fast provisioning.
- No need to run data centers.
- Global deployment. [file:1]

### 3. Deployment Models

- Public = provider-managed shared cloud.
- Private = single organization cloud.
- Hybrid = on-prem + public cloud.
- Community = shared by similar organizations. [file:1]

### 4. Service Models

- IaaS = infrastructure.
- PaaS = platform.
- SaaS = software. [file:1]

### 5. AWS CAF

- Business.
- People.
- Governance.
- Platform.
- Security.
- Operations. [file:1]

### 6. Cloud Economics

- CapEx becomes OpEx.
- Pay only for use.
- Economies of scale reduce cost.
- Elasticity avoids waste. [file:1]

### 7. Migration Strategy Recall

- Rehost.
- Replatform.
- Refactor.
- Repurchase.
- Retire.
- Retain.

### Final 1-Minute Revision Formula

**AWS Cloud = On-demand + Pay-as-you-go + Elastic + Agile + Global + Lower Cost**

---

## Final Revision Conclusion

Unit 1 is the foundation of AWS Cloud Practitioner. If a student understands cloud computing basics, AWS benefits, deployment models, service models, migration, AWS CAF, and cloud economics, then the student can confidently answer most conceptual questions in the exam. [file:1]

This material is designed to help in understanding, memorizing, and writing answers in university format. Regular revision of definitions, tables, keywords, and the 10-mark answers will strongly improve exam performance.
