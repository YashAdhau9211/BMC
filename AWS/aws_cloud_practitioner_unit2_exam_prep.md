# AWS Cloud Practitioner — Unit 2 Exam Preparation Question Bank + Study Notes

## Cover Page

**Subject:** AWS Cloud Practitioner  
**Unit:** Unit 2 — Security and Compliance  
**Prepared For:** University Semester Examination  
**Material Type:** Detailed Study Notes + Question Bank + Exam Answers + Revision Notes  
**Level:** Beginner friendly, exam-oriented, full-mark preparation material  

---

## Subject Introduction

Security and Compliance is one of the most critical units in AWS Cloud Practitioner. Security failures can destroy years of effort and cause significant financial loss within minutes. AWS takes security very seriously and provides a wide set of tools, services, and practices to help organizations protect their data and systems. [file:1]

This unit covers the AWS Shared Responsibility Model, Identity and Access Management, data encryption, compliance tools, and important security services like AWS Shield, KMS, Cognito, CloudTrail, and AWS Config. [file:1]

Understanding these concepts deeply is crucial for both your university exam and for real-world cloud work.

---

## Unit Overview

### Unit 2 Syllabus

**Security and Compliance — 10 Hours**

- Understand the AWS shared responsibility model.
- Understand AWS Cloud security, governance, and compliance concepts.
- Identify AWS access management capabilities.
- Identify components and resources for security. [file:1]

### How to Study This Unit

1. Start with the Shared Responsibility Model — it is the most asked concept.
2. Understand IAM components and how policies work.
3. Learn about encryption, CloudTrail, and compliance tools.
4. Memorize key security services and their purpose.
5. Revise tables and viva questions before the exam.

### Exam Tip

In exams, questions on this unit often begin with: *"Explain the AWS Shared Responsibility Model"* or *"Explain IAM and its components."* Always write full definitions, all components, tables, and one real example per answer.

---

## Important Theory Notes

---

## Topic 1: AWS Shared Responsibility Model

### Definition

The AWS Shared Responsibility Model defines the division of security responsibilities between AWS and its customers. Security is a shared concern — AWS is responsible for the **security OF the cloud**, while the customer is responsible for **security IN the cloud**. [file:1]

### Simple Explanation

Think of it like renting a flat. The building owner (AWS) is responsible for the building structure, locks on the main gate, and security of common areas. The tenant (customer) is responsible for the lock on their own flat door, what they keep inside, and who they let in.

### AWS Responsibilities — Security OF the Cloud

AWS manages and controls everything related to the physical infrastructure. [file:1]

- Physical security of data centers.
- Global infrastructure: Regions, Availability Zones, Edge Locations.
- Hardware, software, networking, and facilities that run AWS services.
- Virtualization infrastructure that provides isolation between customer workloads.
- Compute, storage, database, and networking layers. [file:1]

### Customer Responsibilities — Security IN the Cloud

The customer is responsible for everything they deploy and configure inside the cloud. [file:1]

- Customer data.
- Platform, applications, and identity and access management.
- Operating system, network configuration, and firewall configuration.
- Client-side data encryption and data authentication.
- Server-side encryption (file system or data).
- Network traffic protection (encryption, integrity, identity). [file:1]

### ASCII Diagram

```text
+----------------------------------------------------+
|         AWS Shared Responsibility Model            |
+----------------------------------------------------+
|  CUSTOMER (Security IN the Cloud)                  |
|  - Customer Data                                   |
|  - Applications & IAM                              |
|  - OS, Network, Firewall Config                    |
|  - Client-side & Server-side Encryption            |
|  - Network Traffic Protection                      |
+----------------------------------------------------+
|  AWS (Security OF the Cloud)                       |
|  - Compute, Storage, Database, Networking          |
|  - Hardware, Software, Facilities                  |
|  - Regions, Availability Zones, Edge Locations     |
+----------------------------------------------------+
```

### Responsibility by Service Type

| Service Model | Customer Manages | AWS Manages |
|---|---|---|
| IaaS (e.g., EC2) | OS, apps, config, security groups | Hardware, virtualization |
| PaaS (e.g., RDS) | App data, access, endpoints | OS, runtime, backups |
| SaaS | Usage settings only | Full platform |

### Real-world Example

When a company runs an EC2 instance, AWS handles the physical server and virtualization layer. But the company must configure the OS, install patches, configure firewalls, and manage who can access the instance. [file:1]

### Important Keywords

Shared responsibility, security of the cloud, security in the cloud, customer data, physical security, virtualization isolation, encryption, firewall configuration.

---

## Topic 2: Identity and Access Management (IAM)

### Definition

AWS Identity and Access Management (IAM) is a free, global service that allows organizations to define who can access AWS resources and what actions they are allowed to perform. [file:1]

### Simple Explanation

IAM is like an employee access card system in a large office. The admin decides which rooms each employee can enter, what they can do there, and for how long.

### Key Characteristics of IAM

- Free service — no cost for creating users, groups, roles, or policies.
- Global service — applies across all AWS regions.
- Handles both authentication (who are you?) and authorization (what can you do?).
- Follows the principle of **least privilege** — users get only the minimum permissions needed. [file:1]

### Components of IAM

#### 1. IAM User

A person or application that can authenticate with an AWS account. Users have permanent credentials (username/password or access keys). [file:1]

#### 2. IAM Group

A collection of IAM users that share the same permissions. Managing permissions through groups is easier than assigning them to individual users.

- A user can belong to multiple groups.
- Groups cannot be nested (no groups inside groups).
- There is no default group. [file:1]

#### 3. IAM Policy

A JSON document that lists the permissions that allow or deny access to AWS services and resources. Policies are attached to users, groups, or roles. [file:1]

#### 4. IAM Role

An IAM identity with specific permissions that is not permanently attached to one user. It is intended to be assumed by a person, application, or service temporarily. Roles provide temporary security credentials. [file:1]

### ASCII Diagram

```text
IAM Structure
    |
    +-- IAM User (person or app)
    |       |
    |       +--> belongs to IAM Group
    |
    +-- IAM Group (collection of users)
    |       |
    |       +--> attached with IAM Policy
    |
    +-- IAM Policy (JSON permission doc)
    |
    +-- IAM Role (temporary permissions)
```

### Authentication Types in IAM

| Access Type | Credentials Needed |
|---|---|
| Programmatic access (CLI/SDK) | Access Key ID + Secret Access Key |
| AWS Console access | Username + Password + 12-digit Account ID |
| With MFA enabled | Above credentials + MFA code |

### Multi-Factor Authentication (MFA)

MFA adds an extra layer of security. Even if a password is stolen, the attacker cannot access the account without the MFA code. MFA uses virtual apps like Google Authenticator or Microsoft Authenticator. [file:1]

### Authorization in IAM

After authentication, IAM checks what the user is allowed to do through policies.

- All permissions are **implicitly denied** by default.
- Explicit deny always wins over explicit allow.
- Principle of least privilege: grant only what is needed. [file:1]

### IAM Permission Logic Flow

```text
         Request to Access Resource
                   |
                   v
     Is the action EXPLICITLY DENIED?
           /           \
         YES             NO
          |               |
        DENY       Is the action EXPLICITLY ALLOWED?
                        /          \
                      YES            NO
                       |              |
                     ALLOW     DENY (Implicit Deny)
```

---

## Topic 3: IAM Policies in Detail

### Definition

An IAM policy is a JSON document that enables fine-grained access control over AWS resources. It specifies what actions are allowed or denied on which resources. [file:1]

### Types of IAM Policies

#### Identity-Based Policies

Attached to users, groups, or roles. Two subtypes:

- **Managed Policies:** Standalone policies that can be attached to multiple entities.
  - **AWS Managed Policies:** Created and managed by AWS.
  - **Customer Managed Policies:** Created and managed by the customer. More precise control.
- **Inline Policies:** Directly embedded in a single user, group, or role. Deleted when the identity is deleted. [file:1]

#### Resource-Based Policies

Attached to resources like S3, Lambda, EC2. Inline only (not managed). Specify who can access the resource and what they can do. Supported by only some services. [file:1]

### Policy Comparison Table

| Feature | Identity-Based Policy | Resource-Based Policy |
|---|---|---|
| Attached to | User, group, or role | AWS resource (S3, Lambda, etc.) |
| Managed or inline | Both options | Inline only |
| Scope | User action permissions | Who can access the resource |
| Example | User allowed to list S3 bucket | S3 triggers Lambda function |

### Sample Identity-Based Policy (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": ["arn:aws:s3:::bucket-name"]
    }
  ]
}
```

This policy allows the user to list objects in a specific S3 bucket. [file:1]

### Exam Tip

When writing about IAM policies in an exam, always mention: JSON format, Effect (Allow/Deny), Action (what to do), Resource (which service/resource), and Condition (optional, limiting the scope).

---

## Topic 4: IAM Roles

### Definition

An IAM role is an identity with defined permissions that is not tied to a specific user. It can be assumed temporarily by users, applications, or AWS services. [file:1]

### Key Features

- Provides temporary security credentials.
- No permanent password or access keys.
- Can be assumed by users in the same or different AWS account.
- Used when applications or services need access to other AWS resources. [file:1]

### Practical Example

An EC2 application needs to read files from an S3 bucket.

**Steps:**
1. Define an IAM policy that allows read access to the S3 bucket.
2. Attach the policy to a role.
3. Allow the EC2 instance to assume the role.
4. The application gets temporary credentials automatically. No manual key sharing needed. [file:1]

### ASCII Diagram

```text
+---------+      Assumes Role       +----------+
|  EC2    | ----------------------> | IAM Role |
| Instance|                         +----------+
+---------+                               |
                                    Policy attached
                                          |
                                    +----------+
                                    | S3 Bucket|
                                    +----------+
```

---

## Topic 5: Securing a New AWS Account

### Definition

When an AWS account is first created, a root user with full access to all services is created. Best practices must be followed immediately to secure this account. [file:1]

### Step-by-step Process to Secure an AWS Account

1. Log in as root user.
2. Create an IAM user for yourself and save access keys.
3. Create an IAM group with full administrator permissions.
4. Add the new IAM user to this group.
5. Disable and delete root account access keys.
6. Enable password policy for all users.
7. Sign in using the new IAM user credentials.
8. Store root credentials in a secure place.
9. Enable MFA for root and IAM users.
10. Enable AWS CloudTrail to track activity. [file:1]

### Why Root User Should Not Be Used Daily

Root user has unrestricted access to all services and billing. If compromised, the attacker gains complete control. IAM users with limited access reduce this risk significantly. [file:1]

---

## Topic 6: AWS CloudTrail

### Definition

AWS CloudTrail is a service that tracks user activity and API calls in an AWS account. It records what actions were taken, who took them, from where, and when. [file:1]

### Key Features

- Basic event history is enabled by default and is free.
- Records the last 90 days of management events.
- Logs can be stored in an S3 bucket for longer periods.
- Used for security investigations, forensics, and compliance documentation. [file:1]

### What CloudTrail Records

- Who made the request (user, role, or service).
- What action was requested.
- Which resource was affected.
- When and from where the request was made.

### Practical Use

When a security incident occurs in an AWS account, CloudTrail logs are examined to find out which user accessed which resource and when. [file:1]

### Exam Tip

CloudTrail is often confused with CloudWatch. Remember:
- **CloudTrail** = auditing user and API activity.
- **CloudWatch** = monitoring performance metrics.

---

## Topic 7: AWS Organizations

### Definition

AWS Organizations is a free account management service that allows consolidation of multiple AWS accounts into a centralized management structure called an organizational tree. [file:1]

### Structure

```text
Root
  |
  +-- Organizational Unit (OU) — e.g., Finance
  |         |
  |         +-- Account A
  |         +-- Account B
  |
  +-- Organizational Unit (OU) — e.g., Engineering
            |
            +-- Account C
            +-- Account D
```

### Key Features

- Group accounts into Organizational Units (OUs).
- Attach different access policies to each OU.
- Integrates with IAM for fine-grained access control.
- Supports consolidated billing.
- API for automating account management.
- Policy-based and group-based account management. [file:1]

### Service Control Policies (SCPs)

SCPs are the policies used in AWS Organizations. They specify the **maximum permissions** for all accounts in an OU. [file:1]

| Feature | SCP | IAM Policy |
|---|---|---|
| Applied to | AWS accounts/OUs | Users, groups, roles |
| Syntax | JSON | JSON |
| Grants permissions? | No — only limits | Yes — grants and limits |
| Purpose | Organization-level boundary | User-level control |

### Important Point

SCPs do **not** grant permissions. They set the maximum boundary of what actions are possible. Actual permissions must still be granted through IAM policies. [file:1]

---

## Topic 8: AWS Key Management Service (KMS)

### Definition

AWS Key Management Service (KMS) is a service that enables creation and management of encryption keys to control the use of encryption across AWS services and applications. [file:1]

### Key Features

- Uses hardware security modules (HSMs) to protect keys.
- Integrates with AWS CloudTrail for logging key usage.
- Customer master keys (CMKs) control access to data encryption keys.
- Keys can be created, rotated, and disabled as needed.
- Keys can be imported from an organization's own key management infrastructure.
- Integrates with most other AWS services. [file:1]

### Use Cases

- Encrypting data in S3, EBS, RDS, and other services.
- Managing who can use which keys.
- Meeting compliance requirements for key management.

### ASCII Diagram

```text
Application
    |
    v
AWS KMS (Create & Manage Keys)
    |
    +-- Customer Master Key (CMK)
    |       |
    |       +--> Data Encryption Key
    |               |
    |               +--> Encrypt/Decrypt Data
    |
    +-- CloudTrail Logs all key usage
```

---

## Topic 9: Amazon Cognito

### Definition

Amazon Cognito is a service that provides solutions to control access to AWS resources from applications. It allows defining user roles and mapping users to roles so applications can access only authorized resources. [file:1]

### Key Features

- Supports SAML 2.0 (Security Assertion Markup Language).
- Enables single sign-on (SSO) using corporate credentials.
- Works with identity providers like Microsoft Active Directory.
- Meets security and compliance requirements for regulated industries such as healthcare and retail. [file:1]

### What is SAML?

SAML is an open standard for exchanging identity and security information between applications and identity service providers. With SAML, users can sign in with existing corporate credentials across multiple applications. [file:1]

### Simple Explanation

Cognito lets users log in to your cloud app using their company credentials (like email and password from Microsoft Active Directory), without needing a separate account.

---

## Topic 10: AWS Shield

### Definition

AWS Shield is a managed Distributed Denial of Service (DDoS) attack protection service that safeguards applications running on AWS. [file:1]

### What is a DDoS Attack?

A DDoS attack occurs when attackers flood a server with massive amounts of fake traffic so that real users cannot access the application.

### Types of DDoS Attacks AWS Shield Protects Against

- **Infrastructure layer attacks:** UDP floods.
- **State exhaustion attacks:** TCP SYN floods.
- **Application layer attacks:** HTTP GET or POST floods. [file:1]

### AWS Shield Plans

| Feature | AWS Shield Standard | AWS Shield Advanced |
|---|---|---|
| Cost | Free (automatic) | Paid service |
| Protection level | Basic DDoS | Advanced and larger attacks |
| Applicable services | All customers | EC2, ELB, CloudFront, Route 53 |
| Support team access | Not included | DDoS Response Team (DRMS) |

### Important Point

AWS Shield Standard is automatically enabled for all AWS customers at no cost. No configuration is needed to get basic DDoS protection. [file:1]

---

## Topic 11: Data Encryption on AWS

### Definition

Encryption is the process of converting data into an unreadable form so that only authorized parties with the correct key can read it. [file:1]

### Types of Data

| Type | Description | How Encrypted |
|---|---|---|
| Data at rest | Data stored on disk and not moving | AES-256 encryption via AWS KMS |
| Data in transit | Data moving across a network | TLS 1.2 (formerly SSL), AES-256 cipher |

### AES-256

Advanced Encryption Standard with 256-bit key length. It is an open standard used globally for secure data encryption. [file:1]

### TLS / SSL

Transport Layer Security (TLS) secures data as it travels across networks. TLS was formerly called SSL. [file:1]

### AWS Certificate Manager

A service that helps provision and manage SSL and TLS certificates for AWS services. It secures network communications and establishes the identity of websites. [file:1]

### Key Points for Exam

- Data at rest → AES-256 → AWS KMS manages keys.
- Data in transit → TLS 1.2 → AWS Certificate Manager manages certificates.
- Encryption and decryption handled automatically when using KMS.

---

## Topic 12: Securing S3 Buckets and Objects

### Definition

Amazon S3 is AWS's object storage service. All S3 buckets are private by default and can only be accessed by authorized users. [file:1]

### Security Methods for S3

#### 1. Block Public Access
Enabled by default when a bucket is created. Overrides all other policies or object permissions to prevent public access. [file:1]

#### 2. IAM Policies / Resource Policies
Write specific policies to allow access to particular users, roles, or applications. Used when the user or system cannot authenticate using IAM directly. [file:1]

#### 3. Access Control Lists (ACLs)
An older mechanism to control access at bucket or object level. Less commonly used since IAM provides more powerful access control. [file:1]

### Exam Tip

S3 is private by default. Any question about S3 security must mention: Block Public Access, IAM Resource Policy, and ACLs.

---

## Topic 13: AWS Config

### Definition

AWS Config is a service used to assess, audit, and evaluate the configurations of AWS resources. It maintains a history of resource configurations and enables compliance monitoring. [file:1]

### Key Features

- Continuously monitors and records resource configurations.
- Detects and flags non-compliant resources.
- Dashboard shows inventory of all resources in the account.
- Regional service — must be enabled in each region.
- Supports aggregator features for multi-region and multi-account visibility. [file:1]

### Use Cases

- Identifying who changed a security group setting.
- Checking whether encryption is enabled on all storage resources.
- Monitoring compliance with organizational security rules. [file:1]

---

## Topic 14: AWS Artifact

### Definition

AWS Artifact is a service that provides access to AWS security and compliance reports and select online agreements. [file:1]

### What it Provides

- Security and compliance reports and certifications.
- Access to review and accept online agreements such as the Business Associate Agreement (BAA).
- Useful for audit purposes and regulatory compliance. [file:1]

### Important Compliance Framework: HIPAA

Health Insurance Portability and Accountability Act. Used by healthcare companies to ensure patient data privacy. AWS supports HIPAA compliance. [file:1]

---

## Topic 15: Governance and Compliance Concepts

### Definition

Compliance in cloud computing means meeting the regulatory, legal, and organizational requirements for information security management.

### How AWS Supports Compliance

- Engages with external certifying bodies and independent auditors.
- Provides documentation on security policies, processes, and controls.
- Offers compliance reports through AWS Artifact.
- Uses AWS Config for continuous compliance monitoring.
- Maintains certifications such as HIPAA, ISO, SOC, and PCI DSS. [file:1]

### Key Terms

- **ISMS:** Information Security Management System — defines how AWS manages security holistically.
- **HIPAA:** Healthcare compliance framework.
- **PCI DSS:** Payment card industry data security standard.
- **ISO 27001:** International security management standard.

---

## One-Word / Very Short Definitions

- **IAM:** Identity and Access Management.
- **MFA:** Multi-Factor Authentication.
- **SCP:** Service Control Policy.
- **KMS:** Key Management Service.
- **ACL:** Access Control List.
- **TLS:** Transport Layer Security.
- **AES-256:** Encryption standard.
- **SAML:** Security Assertion Markup Language.
- **SSO:** Single Sign-On.
- **DDoS:** Distributed Denial of Service.
- **CloudTrail:** Activity tracking service.
- **AWS Config:** Configuration compliance service.
- **AWS Artifact:** Compliance report portal.
- **CapEx:** Capital Expenditure.
- **Least privilege:** Minimum required permissions.
- **HIPAA:** Healthcare compliance standard.
- **Inline Policy:** Policy directly attached to one identity.
- **Managed Policy:** Reusable, standalone policy.

---

## Short Notes

### Short Note 1: AWS Shared Responsibility Model

AWS is responsible for security of the cloud — physical data centers, hardware, software, and global infrastructure. The customer is responsible for security in the cloud — applications, data, IAM, operating systems, encryption, and firewall configuration. [file:1]

### Short Note 2: IAM

IAM is a free global service for managing users, groups, roles, and permissions. It controls authentication and authorization and supports principle of least privilege. [file:1]

### Short Note 3: AWS Shield

AWS Shield protects against DDoS attacks. Standard version is free and automatic. Advanced version is paid and provides protection for EC2, ELB, CloudFront, and Route 53. [file:1]

### Short Note 4: AWS KMS

KMS helps create, manage, and control encryption keys. Uses hardware security modules. Integrates with CloudTrail for audit logs of key usage. [file:1]

### Short Note 5: AWS CloudTrail

CloudTrail records all API calls and user activity for the past 90 days by default. Logs are stored in S3. Used for security investigations and compliance. [file:1]

### Short Note 6: Amazon Cognito

Cognito controls application-level access to AWS resources. Supports SAML and SSO for enterprise identity federation. [file:1]

---

## Important Differences in Table Format

### AWS Responsibility vs Customer Responsibility

| Aspect | AWS Responsible | Customer Responsible |
|---|---|---|
| Physical security | Yes | No |
| Global infrastructure | Yes | No |
| Virtualization layer | Yes | No |
| OS on EC2 | No | Yes |
| Application security | No | Yes |
| Data encryption | Tools provided by AWS | Implementation by customer |
| IAM configuration | No | Yes |
| Firewall rules | No | Yes |

### IAM User vs IAM Role

| Basis | IAM User | IAM Role |
|---|---|---|
| Credential type | Permanent (password or keys) | Temporary |
| Association | Specific person or app | Assumable by users, apps, services |
| Use case | Regular human access | Service-to-service or cross-account |
| Example | Developer login | EC2 reading from S3 |

### IAM Policy Types

| Type | Attached To | Managed? | Reusable? |
|---|---|---|---|
| AWS Managed | User/group/role | By AWS | Yes |
| Customer Managed | User/group/role | By customer | Yes |
| Inline | Single identity | No | No (1:1 only) |
| Resource-based | Resource | No | No |

### AWS Shield Standard vs Advanced

| Feature | Standard | Advanced |
|---|---|---|
| Cost | Free | Paid |
| Activation | Automatic | Manual opt-in |
| Protection | Basic DDoS | Advanced/large-scale DDoS |
| Support team | No | Yes (DRMS) |

### CloudTrail vs AWS Config vs CloudWatch

| Service | Purpose |
|---|---|
| CloudTrail | Who did what and when (API/user activity audit) |
| AWS Config | What is the current and historical configuration of resources |
| CloudWatch | Real-time performance metrics and monitoring |

### Data at Rest vs Data in Transit

| Basis | Data at Rest | Data in Transit |
|---|---|---|
| Location | Stored on disk | Moving across network |
| Encryption method | AES-256 via KMS | TLS 1.2 (AES-256 cipher) |
| AWS Tool | AWS KMS | AWS Certificate Manager |

---

## 10-Mark Question Bank with Detailed Answers

---

## Q1. Explain the AWS Shared Responsibility Model in detail.

### Most Important Question

### Introduction

Security in cloud computing is not solely AWS's job or the customer's job. It is a shared obligation. AWS defines this clearly through the Shared Responsibility Model.

### Definition

The AWS Shared Responsibility Model is a framework that divides cloud security duties between AWS and the customer. AWS handles security of the cloud infrastructure while the customer handles security within their own deployments. [file:1]

### AWS Responsibilities

AWS is responsible for all the infrastructure that runs cloud services. [file:1]

- Physical data center security.
- Hardware and software that supports AWS services.
- Networking facilities.
- Regions, Availability Zones, Edge Locations.
- Virtualization layer isolation between customers.

### Customer Responsibilities

The customer must secure everything they build and configure on top of AWS. [file:1]

- Customer data.
- Applications and IAM configuration.
- Operating systems, network settings, and firewall rules.
- Encryption of data at rest and in transit.
- Network traffic protection.

### Diagram

```text
+-----------------------------------+
| CUSTOMER                          |
| Security IN the Cloud             |
| - Customer Data                   |
| - Applications & IAM              |
| - OS, Firewall, Network Config    |
| - Encryption                      |
+-----------------------------------+
| AWS                               |
| Security OF the Cloud             |
| - Hardware, Software, Networking  |
| - Regions, AZs, Edge Locations    |
| - Physical Data Center Security   |
+-----------------------------------+
```

### Responsibility by Model

- In IaaS (EC2), customers have most control and most responsibility.
- In PaaS (RDS), AWS handles more.
- In SaaS, AWS handles almost everything except data and access. [file:1]

### Example

A customer running an EC2 instance is responsible for choosing the OS, installing patches, configuring firewalls, and setting up encryption. AWS handles the physical server, virtualization, and networking hardware. [file:1]

### Conclusion

The Shared Responsibility Model ensures both parties do their part. Understanding this model helps organizations properly design their cloud security strategy.

---

## Q2. Explain IAM and its main components in detail.

### Most Important Question

### Introduction

Every cloud environment must control who can access what. IAM is AWS's core solution for this problem.

### Definition

IAM stands for Identity and Access Management. It is a free, global AWS service that allows definition of users, groups, roles, and permissions to control access to AWS resources. [file:1]

### Components

#### IAM User

A person or application authenticated with AWS credentials. Can have programmatic access (access keys) or console access (username, password). [file:1]

#### IAM Group

A logical collection of users. Policies attached to the group apply to all users in it. Users can belong to multiple groups. Groups cannot be nested. [file:1]

#### IAM Policy

A JSON document that specifies what actions are allowed or denied on which resources. Two types: identity-based and resource-based. [file:1]

#### IAM Role

An identity with temporary permissions. Assumed by applications, users, or AWS services. Provides temporary credentials without sharing permanent passwords or keys. [file:1]

### Authentication Types

| Access Type | Credentials |
|---|---|
| Console | Username + Password + MFA code |
| Programmatic | Access Key ID + Secret Access Key |

### Authorization

After authentication, IAM checks policies to decide what the user can do. Default is implicit deny. Explicit deny always wins. [file:1]

### Principle of Least Privilege

Users and applications should be granted only the permissions they specifically need — nothing more. [file:1]

### Example

An organization creates three IAM groups: Developers, Admins, and Analysts. Each group has policies that match their role. Developers cannot access billing. Admins can access everything. Analysts have only read access to databases.

### Conclusion

IAM is the foundation of AWS security. Without proper IAM setup, any cloud environment is vulnerable to unauthorized access.

---

## Q3. Explain IAM policies in detail with types and examples.

### Introduction

Policies determine what an authenticated user is actually allowed to do. They are the authorization layer of IAM.

### Definition

An IAM policy is a JSON document that defines which AWS resources can be accessed and what actions can be performed on them. [file:1]

### Types of IAM Policies

#### Identity-Based Policies
Attached to users, groups, or roles.

- **AWS Managed Policies:** Ready-made policies from AWS.
- **Customer Managed Policies:** Custom-built policies by the customer.
- **Inline Policies:** Embedded in a single entity. Deleted when the entity is deleted. [file:1]

#### Resource-Based Policies
Attached directly to resources like S3 or Lambda. Inline only, not reusable. [file:1]

### Sample JSON Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": ["arn:aws:s3:::my-bucket"]
    }
  ]
}
```

This policy allows listing objects in a specific S3 bucket. [file:1]

### Key Elements of a Policy

- **Version:** Policy language version.
- **Effect:** Allow or Deny.
- **Action:** Which AWS service action is targeted.
- **Resource:** Which specific resource the policy applies to.
- **Condition:** Optional constraint limiting when the policy applies.

### Policy Evaluation Logic

1. Is the action explicitly denied? → Deny.
2. Is the action explicitly allowed? → Allow.
3. Otherwise → Implicit deny. [file:1]

### Conclusion

Understanding IAM policies is essential for designing secure AWS environments. The correct use of managed, inline, and resource-based policies reduces risk.

---

## Q4. Explain IAM Roles and their use case with an example.

### Introduction

Sometimes applications and services need to access AWS resources without hardcoding credentials. IAM roles solve this problem.

### Definition

An IAM role is an identity with specific permissions that can be assumed temporarily by a user, application, or AWS service. Roles use temporary credentials, not permanent keys. [file:1]

### Why Roles Are Better Than Embedding Credentials

- No need to store or share access keys in application code.
- Credentials are automatically rotated.
- Reduces risk of key leakage.
- Supports cross-account access.

### Practical Example

A web application running on EC2 needs to read files from S3.

Steps:
1. Create an IAM policy allowing S3 read access.
2. Create an IAM role and attach the policy.
3. Attach the role to the EC2 instance.
4. The application automatically gets temporary credentials.

The developer never needs to embed keys in code. [file:1]

### Diagram

```text
Developer --> Creates IAM Role --> Attaches S3 Read Policy
                     |
EC2 Instance assumes the Role
                     |
Application accesses S3 using temporary credentials
```

### Conclusion

IAM roles are the best practice for giving AWS services and applications access to other AWS resources securely without credentials sharing.

---

## Q5. Explain data encryption on AWS. Differentiate between data at rest and data in transit.

### Introduction

Data protection through encryption is a fundamental security requirement. AWS provides tools to encrypt both stored and moving data.

### Definition

Encryption is the process of converting data into unreadable ciphertext that can only be decoded using the correct cryptographic key. [file:1]

### Data at Rest

Data stored on physical or virtual storage and not actively moving.

- Encrypted using **AES-256** algorithm.
- AWS KMS manages the encryption keys.
- Encryption/decryption happens automatically and transparently. [file:1]

### Data in Transit

Data moving across a network from one location to another.

- Encrypted using **TLS 1.2** (formerly called SSL).
- Uses AES-256 cipher.
- AWS Certificate Manager manages SSL/TLS certificates. [file:1]

### Comparison Table

| Basis | Data at Rest | Data in Transit |
|---|---|---|
| State | Stored on disk | Moving across network |
| Encryption method | AES-256 | TLS 1.2 (AES-256 cipher) |
| Tool | AWS KMS | AWS Certificate Manager |
| Example | S3 files, EBS volumes | API calls, web traffic |

### Example

A hospital stores patient records in S3 (data at rest, encrypted using AES-256 via KMS). When a doctor accesses records through a web browser, the traffic is protected by TLS.

### Conclusion

AWS provides robust tools for encrypting both types of data. Organizations must ensure encryption is enabled and configured correctly.

---

## Q6. Explain AWS KMS and AWS Shield.

### Introduction

AWS provides dedicated security services to manage encryption keys and protect against network attacks.

### AWS KMS

**Definition:** AWS Key Management Service enables creation, management, and control of encryption keys used across AWS services. [file:1]

**Features:**
- Uses hardware security modules (HSMs).
- Integrates with CloudTrail for key usage audit logs.
- Supports Customer Master Keys (CMKs).
- Keys can be imported from external key management systems.
- Integrates with most AWS services. [file:1]

### AWS Shield

**Definition:** AWS Shield is a managed DDoS protection service for applications on AWS. [file:1]

**What is DDoS?**
A Distributed Denial of Service attack floods a server with massive traffic so that real users cannot access it.

**Shield Standard:**
- Free, automatically enabled.
- Protects against common DDoS attacks.

**Shield Advanced:**
- Paid service.
- Protects against sophisticated large-scale attacks.
- Available for EC2, ELB, CloudFront, Route 53.
- Includes access to DDoS Response Team. [file:1]

### Comparison Table

| Service | Purpose | Cost |
|---|---|---|
| AWS KMS | Encryption key management | Pay per key and request |
| AWS Shield Standard | Basic DDoS protection | Free |
| AWS Shield Advanced | Advanced DDoS protection | Paid |

### Conclusion

KMS protects data confidentiality, while Shield protects application availability. Both are critical components of a complete AWS security architecture.

---

## Q7. Explain Amazon Cognito and its role in AWS security.

### Introduction

Applications need a way to securely authenticate users without building complex identity management from scratch. Amazon Cognito solves this.

### Definition

Amazon Cognito is a service that controls access to AWS resources from applications by managing user authentication and authorization. [file:1]

### Key Features

- Supports SAML 2.0 for federated identity.
- Works with corporate identity providers like Microsoft Active Directory.
- Enables single sign-on (SSO) — one login, multiple applications.
- Meets compliance requirements for healthcare and financial industries. [file:1]

### What is SAML?

Security Assertion Markup Language is an open standard for exchanging authentication information between an identity provider and a service provider. [file:1]

### Use Cases

- A company employee uses their corporate email login to access an internal AWS application.
- A healthcare application uses Cognito to authenticate doctors and patients with role-based access.
- A fintech company enables SSO so users log in once and access all internal tools.

### Conclusion

Cognito simplifies identity management for cloud applications and improves security by centralizing authentication through established standards.

---

## Q8. Explain AWS Organizations and Service Control Policies.

### Introduction

Large enterprises may have many AWS accounts for different teams. AWS Organizations provides a way to manage them all centrally.

### Definition

AWS Organizations is a free service that enables consolidation and centralized management of multiple AWS accounts into an organizational hierarchy. [file:1]

### Structure

```text
Root
 |
 +-- OU (Finance)
 |     +-- Account 1
 |     +-- Account 2
 |
 +-- OU (Engineering)
       +-- Account 3
       +-- Account 4
```

### Key Features

- Group accounts into Organizational Units (OUs).
- Attach access policies to each OU.
- Centralized billing.
- Automate account management using API.
- Apply security controls across accounts. [file:1]

### Service Control Policies (SCPs)

SCPs set the maximum permissions for all accounts in an OU. Written in JSON, like IAM policies, but they never grant permissions — they only define boundaries. [file:1]

### Important Difference: SCPs vs IAM Policies

| Feature | SCP | IAM Policy |
|---|---|---|
| Applied to | Accounts/OUs | Users/groups/roles |
| Can grant permissions? | No | Yes |
| Purpose | Limit maximum access | Grant specific access |

### Real-world Example

A financial services company uses one OU for production and another for development. An SCP restricts the development OU from accessing production databases.

### Conclusion

AWS Organizations improves governance and security at scale. SCPs work alongside IAM to enforce strict access control.

---

## Q9. Explain CloudTrail, AWS Config, and AWS Artifact — compliance and governance tools.

### Introduction

AWS provides dedicated services to help organizations meet compliance requirements and track security events.

### AWS CloudTrail

Tracks who did what in the AWS account. Records API calls, user actions, and management events. [file:1]

- Default: last 90 days of events.
- Logs stored in S3 for extended periods.
- Useful for forensic investigations. [file:1]

### AWS Config

Assesses, audits, and monitors the configuration of AWS resources. [file:1]

- Maintains history of configuration changes.
- Flags non-compliant resources.
- Regional service with aggregator support for multi-account. [file:1]

### AWS Artifact

Provides access to compliance reports and online agreements. [file:1]

- Download security and compliance certifications.
- Accept Business Associate Agreements (BAA) for HIPAA.
- Used during audits to provide evidence of compliance. [file:1]

### Comparison Table

| Service | What it Does |
|---|---|
| CloudTrail | Activity and API logging |
| AWS Config | Resource configuration monitoring |
| AWS Artifact | Compliance reports and agreements |

### Conclusion

Together, CloudTrail, AWS Config, and AWS Artifact form the three-pillar compliance framework in AWS.

---

## Q10. Explain best practices for securing an AWS account.

### Introduction

A new AWS account is most vulnerable if best practices are not immediately implemented. The following steps ensure a secure environment. [file:1]

### Best Practices

1. **Do not use root user daily.** Create an IAM user for everyday work.
2. **Enable MFA** on root and all IAM users.
3. **Apply least privilege.** Give only minimum permissions needed.
4. **Use IAM groups** to manage permissions instead of individual users.
5. **Enable AWS CloudTrail** to track all user activity.
6. **Enable AWS Config** to monitor configuration compliance.
7. **Use strong password policies** for all IAM users.
8. **Encrypt data** at rest and in transit using KMS and Certificate Manager.
9. **Keep S3 buckets private** by default. Review and restrict public access.
10. **Enable billing alerts** to detect unexpected usage. [file:1]

### Example

After creating an AWS account, an administrator creates a dedicated IAM admin user, enables MFA, disables root user access keys, enables CloudTrail logging to an S3 bucket, and enables AWS Config with compliance rules.

### Conclusion

Proactive security setup minimizes risk. These best practices protect AWS accounts from unauthorized access and data breaches.

---

## Q11. Explain the process of how IAM determines permissions.

### Introduction

IAM uses a clear evaluation logic when any user or service requests access to an AWS resource.

### Evaluation Steps

1. Check: Is the action **explicitly denied** in any policy? → If yes: **DENY**.
2. Check: Is the action **explicitly allowed** in any applicable policy? → If yes: **ALLOW**.
3. If neither: → **IMPLICIT DENY** (default deny). [file:1]

### Diagram

```text
         Access Request
               |
        Explicit DENY?
        /           \
      YES             NO
      |               |
    DENY        Explicit ALLOW?
                /           \
              YES             NO
              |               |
            ALLOW      IMPLICIT DENY
```

### Key Rules

- An explicit DENY always overrides any ALLOW.
- All permissions start as implicitly denied.
- Users must be explicitly granted permissions to do anything.

### Example

A user is in the Developers group, which allows S3 access. But a separate policy explicitly denies access to a specific bucket. The user cannot access that bucket even though the group policy allows S3. [file:1]

### Conclusion

Understanding IAM evaluation logic is important for designing correct and secure permission structures.

---

## Q12. Explain securing S3 buckets on AWS.

### Introduction

Amazon S3 is a widely used storage service. Misconfigured S3 buckets have historically caused major data breaches. AWS provides multiple layers of protection. [file:1]

### Default Behavior

All S3 buckets are **private** by default. Public access is blocked by default when a bucket is created. [file:1]

### Security Mechanisms

#### Block Public Access
Enabled by default. Overrides all other access policies to prevent accidental public exposure. [file:1]

#### IAM Resource Policies
Define which users or services can access specific S3 resources. [file:1]

#### Bucket ACLs
Access Control Lists provide object-level access control. Less commonly used today since IAM policies are more powerful. [file:1]

#### Encryption
S3 supports server-side encryption automatically using AES-256 via AWS KMS. [file:1]

### Best Practices

- Never make a bucket public unless absolutely necessary.
- Use bucket policies to limit access to specific IAM roles.
- Enable versioning to prevent accidental deletion.
- Enable logging to track who accessed the bucket.
- Encrypt all bucket contents at rest.

### Conclusion

S3 security requires multiple layers of defense. Default settings are secure but must be maintained through access control policies and encryption.

---

## 2-Mark Probable Questions

1. Define IAM.
2. What is MFA?
3. What is a DDoS attack?
4. What is the AWS Shared Responsibility Model?
5. What is an IAM Group?
6. What is an IAM Role?
7. What is AWS Shield?
8. What is AWS KMS?
9. What is CloudTrail?
10. What is data encryption?
11. What is AES-256?
12. What is TLS?
13. What is SAML?
14. What is an SCP?
15. What is AWS Artifact?

---

## 5-Mark Probable Questions

1. Explain the AWS Shared Responsibility Model.
2. Explain the four components of IAM.
3. Differentiate between IAM User and IAM Role.
4. Explain data at rest and data in transit encryption.
5. Explain AWS Shield Standard and Advanced.
6. Explain the principle of least privilege.
7. Explain AWS Config and its use case.
8. Differentiate between SCP and IAM Policy.

---

## 10-Mark Probable Questions

1. Explain the AWS Shared Responsibility Model in detail.
2. Explain IAM and all its components with examples.
3. Explain IAM policies with types and JSON example.
4. Explain AWS KMS and AWS Shield.
5. Explain data encryption in AWS in detail.
6. Explain AWS Organizations and SCPs.
7. Explain CloudTrail, AWS Config, and AWS Artifact.
8. Explain Amazon Cognito and SAML.
9. Explain S3 security best practices.
10. Explain best practices for securing a new AWS account.

---

## Viva Questions with Answers

**1. What is the AWS Shared Responsibility Model?**  
AWS manages security of the cloud; customers manage security in the cloud. [file:1]

**2. What is the difference between authentication and authorization?**  
Authentication verifies identity (who are you?); authorization determines permissions (what can you do?).

**3. What is IAM?**  
Free, global AWS service for managing users, groups, roles, and policies to control access. [file:1]

**4. What is a root user in AWS?**  
The first user created when an AWS account is made. Has complete access to all services. Should not be used daily. [file:1]

**5. What is MFA?**  
Multi-Factor Authentication. Adds extra security by requiring a code from a device in addition to a password. [file:1]

**6. What is least privilege in IAM?**  
Grant only the minimum permissions required for a user to perform their job. [file:1]

**7. What is an IAM inline policy?**  
A policy directly embedded in a single user, group, or role. Deleted when the identity is deleted. [file:1]

**8. What is AWS CloudTrail used for?**  
Tracking and logging all API calls and user activity in an AWS account. [file:1]

**9. What is AWS Config?**  
A service that monitors and evaluates resource configurations for compliance. [file:1]

**10. What does AWS KMS do?**  
Creates and manages encryption keys used to encrypt and decrypt data in AWS. [file:1]

**11. What is AWS Shield?**  
A managed DDoS protection service. Standard is free and auto-enabled. Advanced is paid. [file:1]

**12. What is Amazon Cognito?**  
A service for managing user identity and controlling access to AWS applications. Supports SAML and SSO. [file:1]

**13. What is SAML?**  
Security Assertion Markup Language — open standard for exchanging identity information between apps and identity providers. [file:1]

**14. What is an SCP?**  
Service Control Policy. Used in AWS Organizations to set maximum permissions for accounts in an OU. [file:1]

**15. What is AWS Artifact?**  
A portal providing access to AWS compliance reports and legal agreements. [file:1]

---

## Common Mistakes Students Make in Exams

- Confusing **CloudTrail** with **CloudWatch** — CloudTrail = audit logging; CloudWatch = performance monitoring.
- Writing the Shared Responsibility Model incorrectly — AWS = security OF cloud; Customer = security IN cloud.
- Forgetting that IAM is a **global** service.
- Not mentioning that SCPs **never grant permissions** — they only limit maximum permissions.
- Confusing **IAM roles** with **IAM users** — roles use temporary credentials.
- Forgetting that S3 is **private by default**.
- Confusing **AES-256** (data at rest) with **TLS 1.2** (data in transit).
- Writing incomplete policy explanation — always mention Effect, Action, Resource.
- Missing the real-world example in 10-mark answers.
- Forgetting to mention that inline policies are deleted with the identity.

---

## Important Keywords Likely to Fetch Marks

Shared responsibility model, security of the cloud, security in the cloud, IAM, MFA, least privilege, IAM user, IAM group, IAM policy, IAM role, authentication, authorization, managed policy, inline policy, resource-based policy, CloudTrail, AWS Config, AWS Artifact, compliance, KMS, encryption, AES-256, TLS 1.2, data at rest, data in transit, AWS Shield, DDoS, SAML, SSO, Amazon Cognito, SCP, AWS Organizations, S3 private by default, Block Public Access.

---

## Important Exam Tips

### Writing 10-Mark Security Answers

1. Start with a definition of the concept.
2. Add importance or need.
3. List components or types with sub-headings.
4. Include one diagram or table.
5. Provide one real-world example.
6. End with a conclusion.

### Common Patterns in Security Questions

- "Explain the Shared Responsibility Model." → Always draw the two-tier diagram.
- "Explain IAM." → Always list all four components.
- "Explain CloudTrail." → Always mention 90 days, S3 storage, forensic use.
- "Explain encryption." → Always mention AES-256, TLS 1.2, KMS, and Certificate Manager.

### Memory Tricks

- **IAM Components:** U-G-P-R = User, Group, Policy, Role
- **AWS Responsible for:** Physical things = data center, hardware, regions, AZs, edge locations
- **Customer Responsible for:** Logical things = apps, OS, IAM config, encryption, firewall
- **Shield types:** Standard = free/auto; Advanced = paid/premium
- **CloudTrail vs Config:** Trail = WHO did WHAT; Config = WHAT is the CONFIGURATION

---

## Quick Revision Notes

### 1. Shared Responsibility

- AWS = security OF cloud = physical infra, regions, AZs, virtualization. [file:1]
- Customer = security IN cloud = apps, data, IAM, OS, firewall, encryption. [file:1]

### 2. IAM Summary

- 4 components: User, Group, Policy, Role.
- Free global service.
- Default = implicit deny.
- Principle of least privilege. [file:1]

### 3. IAM Policies

- JSON documents.
- Types: AWS Managed, Customer Managed, Inline, Resource-based.
- Explicit deny always wins. [file:1]

### 4. Security Services

| Service | Purpose |
|---|---|
| KMS | Encryption key management |
| Shield | DDoS protection |
| Cognito | App identity management, SSO |
| CloudTrail | Activity logging |
| AWS Config | Configuration compliance |
| AWS Artifact | Compliance reports |

### 5. Encryption Quick Recall

- At rest → AES-256 → KMS. [file:1]
- In transit → TLS 1.2 → Certificate Manager. [file:1]

### 6. AWS Organizations

- OUs = groups of accounts.
- SCPs = max permission boundaries.
- SCPs never grant permissions. [file:1]

### 7. S3 Security

- Private by default.
- Block public access enabled by default.
- Secure using IAM policies, ACLs, and encryption. [file:1]

---

## Most Expected Questions

### Very High Probability

1. Explain the AWS Shared Responsibility Model.
2. Explain IAM and its components.
3. Explain IAM Policies with types.
4. Explain data encryption in AWS.
5. Explain AWS Shield.

### High Probability

6. Explain IAM Roles with example.
7. Explain CloudTrail.
8. Explain AWS Organizations and SCPs.
9. Explain best practices for securing an AWS account.
10. Explain Amazon Cognito and SAML.

---

## Final Revision Formula

**Unit 2 = Shared Responsibility + IAM + Policies + Encryption + Shield + KMS + Cognito + CloudTrail + Config + Artifact + Organizations + S3 Security**

> A strong student knows WHO is responsible, HOW access is controlled, HOW data is protected, and WHICH services enforce compliance.

