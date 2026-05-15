# AWS Cloud Practitioner — Unit 4: Billing, Pricing, and Support
### Complete Exam Preparation Notes | 11 Hours

---

## Syllabus Coverage
- Compare AWS pricing models
- Understand resources for billing, budget, and cost management
- Identify AWS technical resources and AWS Support options

---

## Topic 1: AWS Pricing Model

### Definition
The AWS pricing model is a flexible, consumption-based framework where customers pay only for the services they use — no large upfront investment, no long-term contracts required for most services.

### Three Fundamental Cost Drivers
Every AWS bill is driven by three factors:

| Driver | Charged By | Notes |
|---|---|---|
| Compute | Per hour / per second | Varies by instance type |
| Storage | Per GB | Varies by service |
| Data Transfer | Per GB (outbound) | Inbound is always FREE |

> **Important Rule:** Inbound data transfer to AWS is free. Data transfer between services in the same region is also free. Only outbound data transfer is charged.

---

### Three Core Pricing Principles

#### 1. Pay for What You Use
- No upfront expenses.
- All services available on demand.
- No long-term contracts required.
- Billing is an **operational expense (OpEx)** instead of a **capital expense (CapEx)**.

#### 2. Pay Less When You Reserve
- Committing to 1-year or 3-year usage earns significant discounts.
- EC2 Reserved Instances offer up to **75% discount** vs On-Demand.
- Three payment options: All Upfront, Partial Upfront, No Upfront.

#### 3. Pay Less When You Use More (Volume Discounts)
- Tiered pricing for services like S3, EBS, EFS.
- The more you use, the lower the cost per GB.
- AWS also reduces prices as it grows — since 2006, AWS has lowered pricing **75+ times**.

---

### EC2 Pricing Models

| Model | Commitment | Discount | Interruption | Best For |
|---|---|---|---|---|
| On-Demand | None | None | None | Flexible, unpredictable workloads |
| Reserved Instance | 1–3 years | Up to 75% | None | Steady, predictable workloads |
| Spot Instance | None | Up to 90% | Possible (2-min notice) | Batch jobs, fault-tolerant tasks |
| Dedicated Host | None | None | None | Compliance, licensing requirements |

---

### AWS Free Tier
New customers get **12 months** of limited free usage:

| Service | Free Tier Offer |
|---|---|
| EC2 (t2.micro) | 750 hours/month |
| S3 | 5 GB storage |
| RDS | 750 hours/month |
| Lambda | 1 million requests/month |
| CloudFront | 50 GB data transfer |

---

### Services With No Additional Charge
These services are free — but resources they provision may incur charges:
- Amazon VPC
- AWS Elastic Beanstalk
- AWS Auto Scaling
- AWS CloudFormation
- AWS IAM

> **Example:** Auto Scaling is free, but the EC2 instances it launches are charged.

---

### Custom Pricing
For high-volume projects with unique requirements, AWS offers negotiated custom pricing models directly.

---

## Topic 2: AWS Billing and Cost Management

### Definition
AWS Billing and Cost Management is a service used to pay AWS bills, monitor expenses, forecast future costs, and get detailed visibility into resource usage.

---

### Key Billing Tools

#### 1. AWS Bills Page
- Lists costs incurred over the past month for each AWS service.
- Broken down by region and linked account.
- Most up-to-date view of charges.

#### 2. AWS Cost Explorer
- A **visual tool** to analyze AWS costs and usage over time.
- Default report shows monthly running costs for at least 3 months.
- **Forecasts** next month's costs with a confidence interval.
- Filter by service, region, account, or tag.

#### 3. AWS Budgets
- Create custom budget thresholds.
- Receive **email or SNS alerts** when costs approach or exceed the limit.
- Track at monthly, quarterly, or yearly level.
- Supports cost budgets, usage budgets, and reservation budgets.

#### 4. AWS Cost and Usage Report (CUR)
- Most **detailed** billing data available.
- Breaks usage into hourly or daily line items per service.
- Published to an **S3 bucket** and updated once per day.
- Used for audit, chargeback, and detailed cost analysis.

#### 5. AWS Pricing Calculator
- Free online tool to **estimate monthly AWS costs before deployment**.
- Model solutions before building them.
- Identify cost-saving opportunities.
- Explore available instance types and contract terms.

---

### Billing Tools — Quick Comparison

| Tool | Purpose | Output |
|---|---|---|
| Bills Page | View monthly charges | Itemized bill per service |
| Cost Explorer | Visual spending trends + forecast | Charts and reports |
| AWS Budgets | Alert when budget limit hit | Email / SNS notification |
| Cost and Usage Report | Detailed hourly/daily data | CSV file in S3 bucket |
| Pricing Calculator | Estimate cost before deployment | Monthly cost estimate |

---

### Billing Dashboard Flow

```
AWS Billing Dashboard
        |
        +-- Bills Page          → Current monthly charges (by service/region)
        |
        +-- Cost Explorer       → Visual trends + forecast
        |
        +-- AWS Budgets         → Alert when spending exceeds threshold
        |
        +-- Cost & Usage Report → Detailed hourly data → published to S3
```

---

### Exam Tips — Billing Tools
- **"Alert when budget exceeded"** → AWS Budgets
- **"Visualize costs over time"** → Cost Explorer
- **"Detailed hourly data in S3"** → Cost and Usage Report
- **"Estimate cost before building"** → Pricing Calculator

---

## Topic 3: Total Cost of Ownership (TCO)

### Definition
Total Cost of Ownership (TCO) is a financial estimate identifying **all direct and indirect costs** of owning and running IT infrastructure — whether on-premises or on AWS Cloud.

### Why TCO Matters
- Compare on-premises infrastructure costs vs AWS Cloud costs.
- Build a financial business case for cloud migration.
- Identify hidden costs of physical data centers.
- Help CFOs/IT managers justify cloud investment.

---

### On-Premises Cost Categories

| Category | What It Includes |
|---|---|
| Server costs | Hardware (servers, rack, switches), OS licenses, virtualization |
| Storage costs | Disks, SAN, fiber channel switches, storage admin |
| Network costs | LAN switches, load balancers, bandwidth, network admin |
| IT labor costs | Server admin, maintenance, helpdesk staff |

---

### On-Premises vs AWS Cloud

| Factor | On-Premises | AWS Cloud |
|---|---|---|
| Cost type | Fixed (always running) | Variable (usage-based) |
| Idle capacity | Always paid for | Never paid for when off |
| Maintenance | Company manages | AWS manages |
| Setup time | Weeks / months | Minutes |
| Upgrade cost | Manual hardware replacement | Automatic |
| Risk | Over-provision or under-provision | Scale exactly as needed |

---

### Hard Benefits (Quantifiable Savings)
- Reduced spending on compute, storage, networking.
- Fewer hardware and software purchases.
- Lower operational and maintenance costs.
- Reduced backup and disaster recovery costs.
- Fewer operations staff needed.

### Soft Benefits (Qualitative Improvements)
- Increased developer productivity.
- Improved customer satisfaction.
- Agile business processes.
- Reuse of services and applications.
- Increased global reach.

---

### CapEx vs OpEx

| Basis | CapEx (On-Premises) | OpEx (AWS Cloud) |
|---|---|---|
| Meaning | Large upfront investment | Ongoing pay-per-use spending |
| Example | Buying physical servers | AWS monthly bill |
| Flexibility | Low | High |
| Cash flow | Heavy upfront burden | Spread over time |
| Risk | High (may over-provision) | Low (usage-based) |

---

## Topic 4: AWS Organizations and Consolidated Billing

### Definition
AWS Organizations is a **free account management service** that consolidates multiple AWS accounts into a single organizational hierarchy with centralized billing and governance.

### Organizational Structure

```
Root (Master Account)
  |
  +-- OU: Finance Department
  |       +-- Account A (Finance Dev)
  |       +-- Account B (Finance Prod)
  |
  +-- OU: Engineering Department
          +-- Account C (Eng Dev)
          +-- Account D (Eng Prod)
```

---

### Key Features
- Group accounts into **Organizational Units (OUs)**.
- Apply **Service Control Policies (SCPs)** to OUs.
- API-based automated account management.
- **Consolidated billing** — single bill for all accounts.
- Volume discounts aggregated across all linked accounts.

### Consolidated Billing Benefits
- One bill for all accounts.
- Easy cost tracking per team or department.
- Combined usage qualifies for **tiered volume discounts**.
- No extra charge for using AWS Organizations.

### Ways to Access AWS Organizations
- AWS Management Console
- AWS CLI
- AWS SDKs
- HTTP Query API

> **Note:** AWS Organizations does NOT replace IAM. SCPs define maximum permissions at the account level. IAM still controls individual user/role permissions within each account.

---

## Topic 5: AWS Support Plans

### Definition
AWS Support Plans are tiered technical support subscriptions that give customers access to AWS support engineers, tools, and expert guidance.

---

### Four Support Plans

#### 1. Basic Support — FREE
- 24/7 customer service via email (billing/account only).
- AWS documentation, whitepapers, and support forums.
- AWS Trusted Advisor — **core checks only**.
- AWS Personal Health Dashboard.
- **No technical support cases.**

#### 2. Developer Support — Paid
- Everything in Basic, plus:
- Business hours **email access** to Cloud Support Associates.
- Unlimited technical support cases.
- Best practice guidance.
- Client-side diagnostic tools.
- Building block architectural support.

#### 3. Business Support — Paid
- Everything in Developer, plus:
- **24/7 phone, email, and chat** access to Cloud Support Engineers.
- **Full access to all AWS Trusted Advisor checks.**
- Use case guidance.
- Infrastructure event management.
- API for support center and Trusted Advisor.

#### 4. Enterprise Support — Paid (Highest Tier)
- Everything in Business, plus:
- **Technical Account Manager (TAM)** — dedicated AWS expert.
- Application architecture guidance.
- Proactive reviews, planning, and optimization.
- **AWS Support Concierge** — non-technical billing/account support.
- Fastest response times.

---

### Support Plans Comparison Table

| Feature | Basic | Developer | Business | Enterprise |
|---|---|---|---|---|
| Cost | Free | Paid | Paid | Paid (highest) |
| Technical support cases | ❌ None | ✅ Unlimited | ✅ Unlimited | ✅ Unlimited |
| Response — Critical | ❌ N/A | ❌ N/A | ✅ < 30 min | ✅ < 15 min |
| Response — Urgent | ❌ N/A | ❌ N/A | ✅ < 1 hour | ✅ < 1 hour |
| Response — Normal | ❌ N/A | ✅ < 12 hrs | ✅ < 12 hrs | ✅ < 12 hrs |
| Full Trusted Advisor | ❌ No | ❌ No | ✅ Yes | ✅ Yes |
| TAM (Technical Account Manager) | ❌ No | ❌ No | ❌ No | ✅ Yes |
| Support Concierge | ❌ No | ❌ No | ❌ No | ✅ Yes |
| Phone / Chat support | ❌ No | ❌ No | ✅ Yes | ✅ Yes |

> **Memory trick:** B-D-B-E = Basic, Developer, Business, Enterprise → **"Big Dogs Build Everything"**

---

## Topic 6: Support Case Severity Levels and Response Times

### Definition
Severity levels classify the business impact of a support case and determine how quickly AWS engineers respond.

### Five Severity Levels

| Severity | Situation | Enterprise SLA | Business SLA | Developer SLA |
|---|---|---|---|---|
| **Critical** | System completely down, data loss, all users affected | **< 15 minutes** | **< 30 minutes** | ❌ Not available |
| **Urgent** | Important production system unavailable | < 1 hour | < 1 hour | ❌ Not available |
| **High** | Important functionality impaired/degraded | < 4 hours | < 4 hours | ❌ Not available |
| **Normal** | Non-critical impairment or dev question | < 12 hours | < 12 hours | < 12 hours |
| **Low** | General guidance or feature request | < 24 hours | < 24 hours | < 24 hours |

> **Exam tip:** Critical and Urgent severity cases are NOT available on Developer or Basic plans. Always write exact response times — they are directly tested.

---

## Topic 7: AWS Trusted Advisor

### Definition
AWS Trusted Advisor is an automated online service that acts as a customized cloud expert, advising on AWS best practices across five key categories.

### Five Check Categories

```
AWS Trusted Advisor
        |
        +-- 1. Cost Optimization   → Reduce wasteful or unused resources
        +-- 2. Performance         → Improve system response and throughput
        +-- 3. Security            → Fix vulnerabilities and insecure configs
        +-- 4. Fault Tolerance     → Improve availability and redundancy
        +-- 5. Service Limits      → Avoid hitting account/service quotas
```

#### 1. Cost Optimization
Identifies idle or underutilized resources.
- Idle EC2 instances.
- Unattached EBS volumes.
- Reserved Instance purchase recommendations.

#### 2. Performance
Highlights configuration improvements for better performance.
- High-utilization EC2 instances.
- CloudFront header forwarding optimization.

#### 3. Security
Flags insecure configurations.
- MFA not enabled on root account.
- Open (public) S3 buckets.
- Unused IAM credentials.
- Security group open to all (0.0.0.0/0).

#### 4. Fault Tolerance
Identifies gaps that increase risk of failure.
- EC2 instances not deployed across multiple AZs.
- RDS without Multi-AZ.
- S3 versioning not enabled.

#### 5. Service Limits
Warns when approaching maximum allowed limits.
- EC2 instance limit per region.
- S3 bucket limit.
- VPC limit.

---

### Trusted Advisor Access by Support Plan

| Category | Basic / Developer | Business / Enterprise |
|---|---|---|
| Core security checks | ✅ Yes | ✅ Yes |
| All 5 categories (full access) | ❌ No | ✅ Yes |

---

## Topic 8: Technical Account Manager (TAM) and Support Concierge

### Technical Account Manager (TAM)
A **dedicated AWS expert** assigned to Enterprise Support customers.

**Responsibilities:**
- Proactive architectural guidance and reviews.
- Ongoing planning and optimization communication.
- Infrastructure event management for large launches or migrations.
- Available **only with Enterprise Support Plan**.

### AWS Support Concierge
A team that handles **non-technical billing and account-level inquiries** for Enterprise customers.

**Handles:**
- Billing questions and account-level inquiries.
- Non-technical administrative issues.

### TAM vs Support Concierge

| Basis | TAM | Support Concierge |
|---|---|---|
| Focus | Technical | Non-technical (billing/account) |
| Role | Architect, planner, optimizer | Billing and account advisor |
| Available In | Enterprise only | Enterprise only |

---

## Topic 9: AWS Technical Resources

### Self-Service Learning and Support Resources

| Resource | Description |
|---|---|
| AWS Documentation | Official docs for every service (docs.aws.amazon.com) |
| AWS Knowledge Center | FAQs and common problem solutions |
| AWS re:Post | Community Q&A platform (formerly AWS Forums) |
| AWS Whitepapers | Deep-dive technical guides on architecture, security, pricing |
| AWS Personal Health Dashboard | Alerts about AWS issues affecting YOUR resources |
| AWS Service Health Dashboard | Global AWS service health status |
| AWS Marketplace | Catalog of third-party software on AWS |
| AWS Well-Architected Tool | Free architecture review tool |

---

### AWS Well-Architected Framework

A framework of best practices for designing reliable, secure, efficient, and cost-effective systems on AWS. Based on **five pillars**:

| Pillar | Focus | Key Practice |
|---|---|---|
| Operational Excellence | Run and monitor systems | Automate, review, improve |
| Security | Protect data and systems | IAM, encryption, auditing |
| Reliability | Recover from failures | Multi-AZ, backups, auto-scaling |
| Performance Efficiency | Use resources efficiently | Right-sizing, caching |
| Cost Optimization | Avoid unnecessary cost | Reserved instances, auto-scaling |

> **Memory trick:** O-S-R-P-C → **"Our Systems Really Perform Consistently"**

---

## Important Differences — Summary Tables

### On-Demand vs Reserved vs Spot vs Dedicated

| Basis | On-Demand | Reserved | Spot | Dedicated |
|---|---|---|---|---|
| Commitment | None | 1–3 years | None | None |
| Discount | None | Up to 75% | Up to 90% | None |
| Interruption risk | None | None | Yes (2-min notice) | None |
| Use case | Flexible workloads | Steady workloads | Batch/fault-tolerant | Compliance |

### AWS Support Plans — Key Differentiators

| Feature | Basic | Developer | Business | Enterprise |
|---|---|---|---|---|
| Cost | Free | Paid | Paid | Highest |
| Tech cases | ❌ | ✅ | ✅ | ✅ |
| TAM | ❌ | ❌ | ❌ | ✅ |
| Full Trusted Advisor | ❌ | ❌ | ✅ | ✅ |
| Critical SLA | ❌ | ❌ | 30 min | 15 min |

### Hard vs Soft Benefits of Cloud Migration

| Hard Benefits (Quantifiable) | Soft Benefits (Qualitative) |
|---|---|
| Reduced hardware spending | Increased developer productivity |
| Lower operational costs | Improved customer satisfaction |
| Fewer operations staff | Agile business processes |
| Lower DR and backup costs | Greater global reach |
| Reduced software licensing | Reuse of services |

---

## Short Notes

### Short Note 1: AWS Pricing Model
AWS uses a pay-as-you-go model. Three cost drivers: compute, storage, and data transfer. Inbound data transfer is free. Core principles: pay for what you use, pay less when you reserve, pay less as usage grows. AWS Free Tier provides 12 months of limited free services for new customers.

### Short Note 2: AWS Cost Explorer
A visual tool on the AWS Billing Dashboard that displays spending trends over past months and forecasts upcoming costs with confidence intervals. Filters can be applied by service, region, or tag.

### Short Note 3: AWS Budgets
Allows customers to define spending limits and receive email or SNS notifications when actual or forecasted costs approach or exceed the budget. Supports monthly, quarterly, and yearly tracking.

### Short Note 4: TCO (Total Cost of Ownership)
A financial model comparing all costs of running on-premises infrastructure vs AWS Cloud. Includes direct costs (hardware, power, storage) and indirect costs (network infrastructure, SAN). Hard benefits are quantifiable savings; soft benefits are qualitative improvements.

### Short Note 5: AWS Trusted Advisor
An automated service that checks AWS configurations across five categories: cost optimization, performance, security, fault tolerance, and service limits. Full access is available only on Business and Enterprise support plans.

### Short Note 6: Technical Account Manager (TAM)
A dedicated AWS expert exclusively available to Enterprise Support customers. Provides architectural guidance, proactive communication, and planning support for cloud optimization and major events.

---

## One-Word / Very Short Answer Definitions

| Term | Definition |
|---|---|
| TCO | Total Cost of Ownership |
| CUR | Cost and Usage Report |
| TAM | Technical Account Manager |
| RI | Reserved Instance |
| CapEx | Capital Expenditure (upfront investment) |
| OpEx | Operational Expenditure (usage-based cost) |
| Free Tier | 12 months free for new AWS customers |
| Cost Explorer | Visual AWS cost analysis tool |
| AWS Budgets | Cost alerting and threshold tool |
| Consolidated billing | Single bill for all AWS Organization accounts |
| Support Concierge | Non-technical billing/account support team |
| Hard benefits | Quantifiable financial savings |
| Soft benefits | Non-financial qualitative improvements |
| Trusted Advisor | Automated cloud best-practice checker |

---

## Question Bank

### 2-Mark Questions
1. What is pay-as-you-go pricing?
2. What are the three cost drivers in AWS?
3. What is the AWS Free Tier?
4. What is TCO?
5. What is AWS Budgets?
6. What is AWS Cost Explorer?
7. Differentiate On-Demand and Reserved Instances.
8. What is AWS Trusted Advisor?
9. What is a TAM?
10. List the four AWS Support Plans.
11. What are the five severity levels in AWS Support?
12. What is consolidated billing?
13. What is the AWS Pricing Calculator?
14. What is the AWS Cost and Usage Report?
15. What is the AWS Support Concierge?

### 5-Mark Questions
1. Explain the three core principles of AWS pricing.
2. Explain AWS Budgets and Cost Explorer.
3. Explain the four AWS Support Plans.
4. Explain TCO with direct and indirect costs.
5. Explain AWS Trusted Advisor and its five categories.
6. Explain consolidated billing in AWS Organizations.
7. Differentiate On-Demand, Reserved, and Spot Instances.
8. Explain severity levels and response times.

### 10-Mark Questions
1. Explain the AWS pricing model in detail with a comparison table.
2. Explain AWS Billing and Cost Management tools.
3. Explain Total Cost of Ownership (TCO) in cloud computing.
4. Explain AWS Support Plans in detail with comparison table.
5. Explain AWS Trusted Advisor and its five categories.
6. Explain AWS Organizations and consolidated billing.
7. Explain the AWS Well-Architected Framework and its five pillars.
8. Compare EC2 pricing models: On-Demand, Reserved, Spot, and Dedicated.
9. Explain severity levels in AWS Support with response time table.
10. Explain TCO considerations and the AWS Pricing Calculator.

---

## 10-Mark Model Answers

### Q1. Explain the AWS Pricing Model in Detail

**Introduction:**
Understanding how AWS charges customers is fundamental to effective cloud financial management. AWS pricing is designed to be flexible, transparent, and cost-efficient.

**Definition:**
The AWS pricing model is a consumption-based financial model where customers pay only for the services they use, without large upfront commitments for most services.

**Three Fundamental Cost Drivers:**

| Driver | Charged By | Notes |
|---|---|---|
| Compute | Per hour/second | Varies by instance type |
| Storage | Per GB | Varies by service |
| Data Transfer | Per GB (outbound) | Inbound is free |

**Three Core Principles:**

1. **Pay for What You Use** — No upfront, pay per consumption, available on demand.
2. **Pay Less When You Reserve** — Up to 75% discount for 1–3 year Reserved Instances.
3. **Pay Less as AWS Grows** — Volume-based tiered discounts; AWS has reduced pricing 75+ times since 2006.

**AWS Free Tier:** New customers get 12 months of limited free usage including EC2, S3, and RDS.

**Services With No Charge:** VPC, Elastic Beanstalk, Auto Scaling, CloudFormation, IAM.

**Conclusion:**
AWS pricing gives organizations of all sizes the ability to start small and scale efficiently, always paying in proportion to actual use.

---

### Q2. Explain AWS Billing and Cost Management Tools

**Introduction:**
Managing cloud costs is as important as deploying cloud resources. AWS provides a complete set of tools for cost visibility, tracking, and control.

**Definition:**
AWS Billing and Cost Management is a service and dashboard that helps customers pay bills, monitor usage, forecast future costs, and optimize cloud spending.

**Key Tools:**

**1. AWS Bills Page** — Month-to-date spending breakdown by service and region.

**2. AWS Cost Explorer** — Visual cost analysis tool. Shows spending trends over past months. Forecasts next month's costs. Filter by service, region, or tag.

**3. AWS Budgets** — Create custom thresholds. Receive email/SNS alerts when spending approaches or exceeds budget. Track monthly, quarterly, or yearly.

**4. AWS Cost and Usage Report (CUR)** — Most detailed billing data. Hourly or daily line items per service. Published to S3. Updated once per day.

**5. AWS Pricing Calculator** — Free tool to estimate monthly AWS costs before deployment. Identifies savings opportunities.

**Comparison Table:**

| Tool | Purpose | Output |
|---|---|---|
| Bills Page | Current monthly charges | Itemized bill |
| Cost Explorer | Visual trends + forecast | Charts and reports |
| AWS Budgets | Alert on limit breach | Email/SNS |
| CUR | Detailed hourly data | CSV in S3 |
| Pricing Calculator | Pre-deployment estimate | Monthly estimate |

**Conclusion:**
AWS billing tools transform cost management from reactive to proactive, providing full visibility and control over cloud spending.

---

### Q3. Explain Total Cost of Ownership (TCO)

**Introduction:**
Before migrating to the cloud, organizations need to evaluate whether the move is financially justified. TCO analysis provides this justification.

**Definition:**
TCO is a financial estimate that identifies all direct and indirect costs of owning and running an IT system — whether on-premises or on AWS Cloud.

**On-Premises Cost Categories:**

| Category | What It Includes |
|---|---|
| Server costs | Hardware, OS licenses, virtualization |
| Storage costs | Disks, SAN, fiber channel, admin |
| Network costs | LAN switches, load balancers, bandwidth |
| IT labor costs | Server admin, maintenance, helpdesk |

**On-Premises vs AWS Cloud:**

| Factor | On-Premises | AWS Cloud |
|---|---|---|
| Cost type | Fixed, always running | Variable, usage-based |
| Idle capacity | Always paid for | Zero cost when off |
| Maintenance | Company managed | AWS managed |
| Setup time | Weeks/months | Minutes |

**Hard Benefits:** Reduced hardware, lower ops costs, fewer staff, lower DR costs.

**Soft Benefits:** Developer productivity, customer satisfaction, global reach, agility.

**AWS Pricing Calculator:** Free tool to estimate costs, model solutions, identify savings before building.

**Conclusion:**
TCO analysis is the starting point for every cloud migration decision — it gives organizations the financial clarity to justify the move to AWS.

---

### Q4. Explain AWS Support Plans in Detail

**Introduction:**
Different organizations have different needs for technical support. AWS offers four tiered plans to match workload criticality.

**Definition:**
AWS Support Plans are subscription-based packages that give customers access to AWS support engineers, resources, and expertise.

**Four Plans:**

1. **Basic (Free)** — Documentation, core Trusted Advisor, billing support. No tech cases.
2. **Developer (Paid)** — Email access during business hours. Unlimited tech cases. Best practice guidance.
3. **Business (Paid)** — 24/7 phone/email/chat. Full Trusted Advisor access. Use case guidance.
4. **Enterprise (Paid)** — TAM, Support Concierge, fastest response times, architectural reviews.

**Comparison Table:**

| Feature | Basic | Developer | Business | Enterprise |
|---|---|---|---|---|
| Cost | Free | Paid | Paid | Paid |
| Tech cases | ❌ | ✅ | ✅ | ✅ |
| TAM | ❌ | ❌ | ❌ | ✅ |
| Full Trusted Advisor | ❌ | ❌ | ✅ | ✅ |
| Critical SLA | ❌ | ❌ | 30 min | 15 min |

**Response Times:**

| Severity | Enterprise | Business | Developer |
|---|---|---|---|
| Critical | < 15 min | < 30 min | ❌ |
| Urgent | < 1 hour | < 1 hour | ❌ |
| High | < 4 hours | < 4 hours | ❌ |
| Normal | < 12 hours | < 12 hours | < 12 hours |
| Low | < 24 hours | < 24 hours | < 24 hours |

**Conclusion:**
Choosing the right support plan depends on workload criticality. Enterprise support is essential for mission-critical production systems.

---

### Q5. Explain AWS Trusted Advisor and Its Five Categories

**Introduction:**
AWS Trusted Advisor acts as an automated cloud expert, continuously reviewing account configurations against AWS best practices.

**Definition:**
AWS Trusted Advisor is an automated online service that provides real-time guidance to reduce cost, improve performance, enhance security, increase fault tolerance, and stay within service limits.

**Five Categories:**

| Category | Purpose | Example Check |
|---|---|---|
| Cost Optimization | Find waste | Idle EC2, unattached EBS |
| Performance | Improve speed | High-CPU EC2 instances |
| Security | Fix vulnerabilities | Open S3 buckets, no MFA on root |
| Fault Tolerance | Improve availability | No Multi-AZ on RDS |
| Service Limits | Avoid quota breach | EC2 limit usage |

**Access by Support Plan:**

| Plan | Access Level |
|---|---|
| Basic / Developer | Core security checks only |
| Business / Enterprise | Full access to all 5 categories |

**Conclusion:**
Trusted Advisor is one of the most valuable automated tools in AWS — it proactively identifies improvements across cost, security, performance, availability, and compliance.

---

## Viva Questions with Answers

| Question | Answer |
|---|---|
| What are the 3 pricing principles in AWS? | Pay for what you use, pay less when you reserve, pay less as AWS grows |
| What is the AWS Free Tier? | 12 months of limited free usage for new customers |
| Is inbound data transfer free in AWS? | Yes, inbound data transfer is always free |
| What is TCO? | Financial model comparing all on-premises costs vs AWS Cloud costs |
| What does Cost Explorer do? | Visualizes AWS spending over time and forecasts next month's costs |
| What does AWS Budgets do? | Alerts via email/SNS when spending approaches or exceeds a set limit |
| What is the CUR? | Detailed hourly/daily usage report published to S3 |
| What is the cheapest support plan? | Basic Support — it is free |
| Which plan includes a TAM? | Enterprise Support only |
| Critical response time — Enterprise? | Less than 15 minutes |
| Critical response time — Business? | Less than 30 minutes |
| What is Trusted Advisor? | Automated checker for 5 best-practice categories |
| Which plans get full Trusted Advisor? | Business and Enterprise |
| What is consolidated billing? | Single bill for all accounts under one AWS Organization |
| What is a hard benefit of cloud? | Quantifiable savings like reduced hardware or lower ops costs |
| What is a soft benefit of cloud? | Qualitative improvement like developer productivity or global reach |
| What is CapEx? | Capital expenditure — large upfront infrastructure investment |
| What is OpEx? | Operational expenditure — ongoing usage-based spending |

---

## Common Exam Mistakes

- ❌ Forgetting that **inbound data transfer is FREE** — only outbound is charged
- ❌ Confusing **Cost Explorer** (visualization) with **AWS Budgets** (alerts)
- ❌ Writing that **Basic Support includes technical cases** — it does NOT
- ❌ Forgetting **TAM is only in Enterprise Support**
- ❌ Mixing up Critical SLAs: Enterprise = **15 min**, Business = **30 min**
- ❌ Not mentioning **all four support plans** in answers
- ❌ Confusing **hard benefits** (financial/quantifiable) with **soft benefits** (qualitative)
- ❌ Forgetting that **VPC, IAM, CloudFormation, Auto Scaling, Beanstalk are free services**
- ❌ Writing Trusted Advisor has 3 or 4 categories — it has exactly **5**
- ❌ Not mentioning both **direct AND indirect costs** in TCO answers

---

## Memory Tricks

| Topic | Mnemonic |
|---|---|
| EC2 Pricing Models | **O-R-S-D** → On-Demand, Reserved, Spot, Dedicated |
| Support Plans | **B-D-B-E** → Basic, Developer, Business, Enterprise → *"Big Dogs Build Everything"* |
| Trusted Advisor Categories | **C-P-S-F-S** → Cost, Performance, Security, Fault Tolerance, Service Limits → *"Can People Stay Fault-Safe?"* |
| TCO Cost Areas | **S-S-N-L** → Server, Storage, Network, Labor |
| Well-Architected Pillars | **O-S-R-P-C** → Operational Excellence, Security, Reliability, Performance, Cost → *"Our Systems Really Perform Consistently"* |

---

## Quick Revision Notes

### Pricing Model
- 3 cost drivers: Compute, Storage, Data Transfer
- Inbound = FREE | Outbound = CHARGED
- Free Tier: 12 months for new customers
- Free services: VPC, IAM, Auto Scaling, CloudFormation, Beanstalk

### Billing Tools
- Bills Page → monthly breakdown
- Cost Explorer → visual trends + forecast
- Budgets → alert on limit breach
- CUR → detailed hourly data in S3

### TCO
- Direct: power, hardware, storage, IT labor
- Indirect: network infrastructure, SAN
- Hard benefits: financial savings | Soft benefits: productivity, agility

### Support Plans
| Plan | Key Feature |
|---|---|
| Basic | Free, no tech cases |
| Developer | Email support |
| Business | 24/7 phone + Full Trusted Advisor |
| Enterprise | TAM + Concierge + fastest SLAs |

### Critical Response Times
- Enterprise: **< 15 minutes**
- Business: **< 30 minutes**

### Trusted Advisor — 5 Categories
Cost → Performance → Security → Fault Tolerance → Service Limits

### EC2 Pricing
- On-Demand: No discount | Reserved: 75% | Spot: 90% | Dedicated: No discount

---

## Most Expected Exam Questions

### Very High Probability ⭐⭐⭐
1. Explain the AWS pricing model in detail.
2. Explain AWS Support Plans with comparison table.
3. Explain TCO (Total Cost of Ownership).
4. Explain AWS Trusted Advisor and its five categories.
5. Explain AWS Billing and Cost Management tools.

### High Probability ⭐⭐
6. Explain consolidated billing in AWS Organizations.
7. Explain severity levels and response times.
8. Explain the AWS Well-Architected Framework.
9. Compare EC2 pricing models.
10. Explain hard and soft benefits of cloud migration.

---

> **Golden Rule for Unit 4:** Know the **exact numbers** — response times, discount percentages, number of pillars, and number of Trusted Advisor categories. These are directly tested and guarantee marks.
