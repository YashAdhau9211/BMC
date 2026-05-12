# DevOps and Automation – Unit 5 Complete Exam Preparation Notes

## 1. Cover Page

**Subject:** DevOps and Automation  
**Unit:** Unit 5 – Infrastructure as Code (IaC)  
**Prepared For:** University Semester Exam Preparation  
**Target:** Full Marks (2, 5, and 10 marks questions)  
**Student Level:** Beginner–Intermediate (CS/IT)  

These notes combine: detailed theory, exam-style answers, question bank, revision points, and viva preparation for Unit 5: *Infrastructure as Code (IaC)*.[file:2][file:4]

---

## 2. Unit Introduction

Infrastructure as Code (IaC) is about **creating and managing infrastructure (servers, networks, databases, etc.) using code instead of manual clicks**.[file:2] In DevOps, IaC works together with configuration management to make environments **consistent, fast to create, and easy to reproduce**.[file:2] This unit focuses on IaC benefits, tools like Terraform and CloudFormation, and how to write and use Terraform configurations to automate infrastructure provisioning.[file:2][file:4]

**Syllabus Points (Unit 5):**[file:2][file:4]
1. Benefits of Infrastructure as Code.  
2. IaC tools (Terraform, CloudFormation, etc.).  
3. Writing Terraform configurations.  
4. Deploying infrastructure with Terraform.  
5. Automating infrastructure provisioning with Terraform.

**Exam Tip:** Almost every long question in this unit must include **definition of IaC + advantages + simple example/diagram + mention of Terraform**.

---

## 3. Important Theory Notes

### 3.1 What is Infrastructure as Code (IaC)?

**One‑line Definition:**  
Infrastructure as Code (IaC) means **managing and setting up your infrastructure using code files/scripts instead of manual configuration in a console or GUI**.[file:2]

**Detailed Definition (Exam Style):**  
IaC is a DevOps practice where infrastructure components like **servers, networks, storage, load balancers, and databases** are defined in machine‑readable configuration files, stored in version control, and automatically created/updated by tools rather than by humans clicking around in cloud dashboards.[file:2]

**Simple Analogy (from notes):**[file:2]
- Think of IaC as a **recipe**.  
- Recipe (code) → Dish (infrastructure).  
- If you follow the same recipe every time, you get the same dish; if you apply the same IaC code, you get the same infrastructure.

**Example:**[file:2]
- Instead of manually creating a server on AWS and installing Nginx, you write code that says:  
  - "Create 1 virtual machine"  
  - "Install Nginx"  
  - "Open port 80"  
- Running the script creates the full environment automatically.

**ASCII Idea:**

```
[ IaC Code Files ]  --->  [ IaC Tool ]  --->  [ Cloud Infrastructure ]
```

---

### 3.2 Why IaC is Important – Benefits

The notes list several strong reasons why IaC is essential in DevOps.[file:2][file:4]

**Key Benefits of IaC:**[file:2]

1. **Consistency:**  
   - The same setup every time – no manual variations between environments.  
   - Avoids "configuration drift" where dev, test, and prod slowly become different.

2. **Speed:**  
   - Infrastructure can be created and updated in **minutes** instead of days.  
   - Useful for spinning up temporary environments for testing.

3. **Version Control and Auditability:**  
   - IaC code is stored in Git, so you can track who changed what and when.[file:2]  
   - Easy rollback to a previous known‑good state.

4. **Reusability:**  
   - IaC scripts can be reused across projects or teams with minor changes.[file:2]  
   - Encourages standard patterns and best practices.

5. **Scalability:**  
   - Easily create **1 or 100 servers** by changing a parameter, not by manual repetition.[file:2]  

6. **Disaster Recovery & Easy Recovery:**  
   - If an environment breaks, you can recreate it quickly from the same IaC code.[file:2]

7. **Better Collaboration (Dev + Ops):**  
   - Infrastructure definitions are **code** that both Dev and Ops can review via pull requests.  
   - Enables GitOps‑style workflows.

**Exam Tip:** In a 10‑mark question “Explain benefits of IaC”, write at least **6–7 points** with headings like **Consistency, Speed, Versioning, Recovery**.

---

### 3.3 IaC Tools Overview (Terraform, CloudFormation, etc.)

The notes highlight several IaC tools and their roles.[file:2][file:4]

**Common IaC Tools:**[file:2]
- **Terraform (HashiCorp):** Most popular **multi‑cloud** IaC tool; can manage AWS, Azure, GCP, and many other providers.[file:4]  
- **AWS CloudFormation:** AWS‑specific IaC service using JSON/YAML templates.  
- **Azure Resource Manager (ARM):** Microsoft Azure‑specific templates.  
- **Pulumi:** IaC using general‑purpose languages like TypeScript, Python, etc.

The rest of this unit focuses mainly on **Terraform**, as given in your notes.[file:4]

---

### 3.4 What is Terraform?

**One‑line Definition:**  
Terraform by HashiCorp is an **open‑source Infrastructure as Code tool** that lets you define, provision, and manage cloud infrastructure using **declarative configuration files**.[file:4]

**Detailed Definition (Exam Style):**  
Terraform is an IaC tool that allows users to describe the desired state of resources (like virtual machines, storage, networks, databases) in **HashiCorp Configuration Language (HCL)**, and then automatically creates, updates, or destroys those resources by communicating with cloud providers like AWS, Azure, and GCP.[file:4]

**Key Characteristics of Terraform:**[file:4]
1. **Declarative Language:**  
   - You describe **what** infrastructure you want; Terraform figures out **how** to create it.  

2. **Infrastructure as Code:**  
   - Infrastructure is written in configuration files and version‑controlled in Git.[file:4]

3. **Cloud‑Agnostic:**  
   - Supports multiple providers such as AWS, Azure, and GCP (and many others).[file:4]

4. **Execution Plan:**  
   - Terraform generates a **plan** before applying changes, so you can see what it will do (create/modify/destroy) before actual execution.[file:4]

5. **State Management:**  
   - Maintains a **state file** (`terraform.tfstate`) to track real‑world resources and map them to configuration.

**Simple Meaning (from notes):**[file:4]
> Instead of manually creating servers, databases, and networks, you write code and Terraform creates everything for you.

---

### 3.5 Terraform Configuration Files – Basic Structure

A Terraform configuration is a set of `.tf` files, usually written in HCL.[file:4]

**Typical Files in a Terraform Project:**[file:4]
- `main.tf` – core configuration (providers, resources).  
- `variables.tf` – variable definitions.  
- `outputs.tf` – output values.  
- `terraform.tfvars` – variable values (e.g., per environment).

#### 3.5.1 Provider Block

A **provider** is responsible for interacting with the APIs of cloud platforms.[file:4]

**Example (AWS Provider):**[file:4]
```hcl
provider "aws" {
  region = "us-east-1"
}
```

This tells Terraform to use AWS in the `us-east-1` region.[file:4]

#### 3.5.2 Resource Block

A **resource** represents an infrastructure component (e.g., VM, database).[file:4]

**Example (AWS EC2 Instance):**[file:4]
```hcl
resource "aws_instance" "myserver" {
  ami           = "ami-123456"
  instance_type = "t2.micro"
}
```

This configuration creates an EC2 instance with the specified AMI and instance type.[file:4]

#### 3.5.3 Variables

Variables make the configuration reusable and parameterized.[file:4]

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

You can then use `var.instance_type` in resource blocks, and override this value via `terraform.tfvars` or command line.[file:4]

#### 3.5.4 Outputs

Outputs display useful information after execution.[file:4]

```hcl
output "instance_ip" {
  value = aws_instance.myserver.public_ip
}
```

This prints the created instance’s public IP at the end of `terraform apply`.[file:4]

#### 3.5.5 Data Sources

Data sources let Terraform **read existing resources** from a provider.[file:4]

```hcl
data "aws_ami" "latest" {
  most_recent = true
}
```

Data can then be used in resource definitions.

#### 3.5.6 Modules

Modules are **reusable blocks** of Terraform configuration.[file:4]

```hcl
module "webserver" {
  source = "./modules/web"
}
```

They help organize and reuse common patterns (e.g., web server stack, database setup).[file:4]

---

### 3.6 Terraform Workflow – Deploying Infrastructure

Terraform follows a clear workflow from configuration to running infrastructure.[file:4]

**Terraform Execution Workflow:**[file:4]
1. **Write Configuration Files:**  
   - Define providers, resources, variables, and outputs in `.tf` files.

2. **Initialize Working Directory – `terraform init`:**  
   - Downloads provider plugins.  
   - Prepares the directory for use.

3. **Generate Execution Plan – `terraform plan`:**  
   - Shows proposed changes (e.g., `+ create`, `~ modify`, `- destroy`).  
   - Lets you review what Terraform will do.

4. **Apply the Configuration – `terraform apply`:**  
   - Provisions infrastructure based on the plan.  
   - Prompts for confirmation unless `-auto-approve` is used.

**ASCII Workflow:**

```
Write .tf files
   |
   v
terraform init
   |
   v
terraform plan
   |
   v
terraform apply
   |
   v
[Infrastructure Created]
```

**State File (`terraform.tfstate`):**[file:4]
- Tracks current infrastructure state.  
- Maps configuration to actual cloud resources.  
- Enables updates and deletions without recreating everything.

**Important Concepts:**[file:4]
- **Idempotency:** Running the same configuration multiple times results in the same infrastructure without duplicating resources.  
- **Dependency Management:** Terraform automatically determines the correct order to create resources based on references between them.

---

### 3.7 Automating Infrastructure Provisioning with Terraform

Terraform is designed to **automate infrastructure provisioning** end‑to‑end.[file:4]

**Key Automation Capabilities:**[file:4]
- Create networks, subnets, and security groups.  
- Provision VMs/instances and attach storage.  
- Create managed databases and load balancers.  
- Integrate with DNS and other services.

**Integration with CI/CD (Conceptual):**[file:2][file:4]
- You can integrate Terraform into CI/CD pipelines:  
  - On config change (commit to Git), pipeline runs `terraform plan` and `terraform apply` (often with approvals).  
  - This enables true GitOps‑style infrastructure management.

**Advantages of Using Terraform for Provisioning:**[file:4]
- Automation of infrastructure provisioning.  
- Reduced manual errors.  
- Version‑controlled infrastructure.  
- Scalability and consistency across multiple environments.  
- Multi‑cloud support.

**Limitations Mentioned in Notes:**[file:4]
- Requires learning HCL.  
- State file management can be complex.  
- Debugging may be challenging for large configs.

---

## 4. Question Bank – Unit 5 (2, 5, and 10 Marks)

### 4.1 2‑Mark Questions

1. Define Infrastructure as Code (IaC).  
2. Mention any two benefits of IaC.  
3. Name any two IaC tools.  
4. What is Terraform?  
5. What is a provider in Terraform?  
6. What is a Terraform resource?  
7. What does `terraform init` do?  
8. What is the purpose of `terraform.tfstate`?  
9. What is a Terraform module?  
10. What is meant by “declarative language” in IaC?

### 4.2 5‑Mark Questions (Short Notes)

1. Explain Infrastructure as Code with a simple example.  
2. Write a short note on benefits of IaC in DevOps.  
3. Write a short note on Terraform as an IaC tool.  
4. Describe the main components of a Terraform configuration file.  
5. Explain the Terraform workflow (`init`, `plan`, `apply`).  
6. Explain the role of the Terraform state file.  
7. Write short notes on: (a) Provider, (b) Resource, (c) Output.  
8. Explain how IaC supports CI/CD and GitOps.  
9. Compare Terraform and CloudFormation in brief.  
10. Explain the concept of idempotency and dependency management in Terraform.

### 4.3 10‑Mark Questions (Long Answers)

1. Define Infrastructure as Code. Explain its benefits and relation to DevOps with examples.  
2. What are the main IaC tools? Explain Terraform in detail with its key characteristics.  
3. Describe the structure of Terraform configuration files (`provider`, `resource`, `variables`, `outputs`, `modules`) with examples.  
4. Explain the Terraform workflow from writing configuration to applying it. How does Terraform manage state and ensure idempotency?  
5. Discuss how Terraform can be used to automate infrastructure provisioning in a cloud environment. Give at least two concrete examples.  
6. Explain the advantages and limitations of Terraform as an IaC tool.  
7. Explain how IaC and Terraform fit into CI/CD pipelines and GitOps practices.  
8. Describe a complete scenario of deploying a web application infrastructure using Terraform (network, VM, and outputs).  
9. Explain the role of modules in Terraform and how they help in reusability and standardization.  
10. Compare manual provisioning of infrastructure with Terraform‑based provisioning in terms of speed, reliability, and maintainability.

**Most Expected Questions:**  
Q1, Q2, Q3, Q4, and Q5 are highly likely as 10‑mark questions.

---

## 5. Detailed 10‑Mark Answers (Exam‑Oriented)

### Q1. Define Infrastructure as Code. Explain its benefits and relation to DevOps with examples.

**Introduction:**  
As applications move to the cloud, infrastructure becomes more complex and dynamic. Manually creating servers and networks is slow and error‑prone. Infrastructure as Code addresses this problem.[file:2][file:4]

**Definition of IaC:**  
Infrastructure as Code is a practice where infrastructure (servers, networks, databases, etc.) is **defined using code or configuration files** and automatically created and managed by tools, instead of performing the setup manually via GUIs or commands.[file:2]

**Benefits of IaC:**[file:2]
1. **Consistency:**  
   - Same script = same infrastructure in dev, test, and prod.  
   - Prevents configuration drift.

2. **Speed:**  
   - Entire environments can be set up in minutes using scripts instead of days of manual work.

3. **Version Control & Audit:**  
   - IaC code is stored in Git; you can see who changed what and when.  
   - Easy rollback to earlier versions if a change causes issues.

4. **Reusability:**  
   - Templates and modules can be reused across multiple projects.

5. **Scalability:**  
   - You can easily scale from 1 to many servers by adjusting configuration.

6. **Disaster Recovery:**  
   - If infrastructure fails, it can be recreated quickly from IaC definitions.

**Relation to DevOps:**[file:2]
- DevOps aims at frequent, reliable releases and close collaboration between Dev and Ops.  
- IaC supports this by **turning infrastructure into code** that can be reviewed, tested, and deployed just like application code.  
- It integrates with CI/CD pipelines: infrastructure changes go through pull requests, automated checks, and controlled rollout.

**Example:**[file:2]
- For an e‑commerce site, instead of manually creating EC2 instances, load balancers, and RDS databases in AWS console, the team uses IaC code (e.g., Terraform) to define everything.  
- When a new region is needed, they run the same code with different parameters to bring up identical infrastructure.

**Conclusion:**  
IaC is a key DevOps enabler, bringing infrastructure under the same discipline as software development: versioned, testable, repeatable, and automated.[file:2][file:4]

---

### Q2. What are the main IaC tools? Explain Terraform in detail with its key characteristics.

**Introduction:**  
Several tools implement IaC, but Terraform is one of the most widely used for multi‑cloud environments.[file:2][file:4]

**Main IaC Tools (Overview):**[file:2]
- Terraform – multi‑cloud, open‑source, declarative.  
- AWS CloudFormation – AWS‑specific template‑based IaC.  
- Azure Resource Manager (ARM) – Azure‑specific template‑based IaC.  
- Pulumi – uses general‑purpose languages like TypeScript, Python.

**Terraform – Detailed Explanation:**[file:4]

1. **Declarative Language (HCL):**  
   - User describes desired infrastructure state (e.g., "1 EC2 instance", "1 VPC").  
   - Terraform decides the steps to reach that state.

2. **Providers:**  
   - Integrate with cloud APIs (AWS, Azure, GCP, etc.).  
   - Example:
     ```hcl
     provider "aws" {
       region = "us-east-1"
     }
     ```[file:4]

3. **Resources:**  
   - Represent actual infrastructure components, such as servers and databases.  
   - Example:
     ```hcl
     resource "aws_instance" "myserver" {
       ami           = "ami-123456"
       instance_type = "t2.micro"
     }
     ```[file:4]

4. **Execution Plan:**  
   - `terraform plan` shows changes to be made before running `terraform apply`.[file:4]

5. **State Management:**  
   - Terraform maintains a state file mapping configuration to real resources, supporting updates and deletions without full recreation.[file:4]

6. **Modules and Reusability:**  
   - Modules let teams package common patterns (e.g., a web server stack) and reuse them across projects.[file:4]

**Conclusion:**  
Among IaC tools, Terraform stands out for its declarative syntax, multi‑cloud support, and strong ecosystem. It is a core tool in many DevOps pipelines for provisioning infrastructure.[file:4]

---

### Q3. Describe the structure of Terraform configuration files with examples.

**Introduction:**  
Terraform configuration files are written in HCL and usually split across multiple `.tf` files for clarity.[file:4]

**Key Components:**[file:4]

1. **Provider Block:**  
   - Specifies which cloud/service provider Terraform should use.  
   - Example:
     ```hcl
     provider "aws" {
       region = "us-east-1"
     }
     ```

2. **Resource Block:**  
   - Defines an infrastructure object.  
   - Example:
     ```hcl
     resource "aws_instance" "myserver" {
       ami           = "ami-123456"
       instance_type = "t2.micro"
     }
     ```

3. **Variables:**  
   - Parameterize configurations; make them easier to reuse.  
   - Example:
     ```hcl
     variable "instance_type" {
       default = "t2.micro"
     }
     ```

4. **Outputs:**  
   - Return useful information after apply.  
   - Example:
     ```hcl
     output "instance_ip" {
       value = aws_instance.myserver.public_ip
     }
     ```

5. **Modules:**  
   - Group resources into reusable units.  
   - Example:
     ```hcl
     module "webserver" {
       source = "./modules/web"
     }
     ```[file:4]

6. **Data Sources:**  
   - Import data about existing resources (e.g., latest AMI) for use in configuration.

**Conclusion:**  
A well‑structured Terraform configuration separates providers, resources, variables, outputs, and modules, making infrastructure definitions modular, maintainable, and reusable.[file:4]

---

### Q4. Explain the Terraform workflow from writing configuration to applying it. How does Terraform manage state and ensure idempotency?

**Introduction:**  
Terraform follows a consistent workflow for creating and managing infrastructure, using a state file to keep track of resources.[file:4]

**Terraform Workflow:**[file:4]
1. **Write Configuration:**  
   - Create `.tf` files with providers, resources, variables, etc.

2. **Initialize – `terraform init`:**  
   - Downloads required plugins for providers.  
   - Prepares backend and project directory.

3. **Plan – `terraform plan`:**  
   - Compares current state (from `terraform.tfstate`) with desired state (from `.tf` files).  
   - Shows an execution plan indicating resources to be created, modified, or destroyed.

4. **Apply – `terraform apply`:**  
   - Executes changes described in the plan.  
   - Creates/updates/deletes resources in the cloud.

5. **Update Configuration:**  
   - When you change `.tf` files and re‑run `plan` and `apply`, Terraform updates only necessary parts of the infrastructure.

**State Management:**[file:4]
- Terraform keeps a **state file (`terraform.tfstate`)** that:  
  - Tracks current real‑world resources.  
  - Maps configuration to resource IDs.  
  - Allows incremental changes instead of recreating everything.  

**Idempotency:**[file:4]
- If you run `terraform apply` multiple times with the same configuration, Terraform detects no changes are required and **does not duplicate resources**.  
- This property is called idempotency – repeated execution leads to the same result.

**Dependency Management:**[file:4]
- Terraform automatically determines the order in which to create resources based on references (e.g., EC2 instance depends on VPC).  
- Ensures that required resources (like networks) are created before dependents (like VMs).

**Conclusion:**  
Through its clear `init → plan → apply` workflow, state tracking, and idempotent operations, Terraform provides a robust way to safely evolve infrastructure over time.[file:4]

---

### Q5. Discuss how Terraform can be used to automate infrastructure provisioning in a cloud environment. Give at least two concrete examples.

**Introduction:**  
Terraform’s main purpose is to automate infrastructure provisioning across various cloud platforms.[file:4]

**General Automation Approach:**[file:4]
- Write HCL configurations describing desired infrastructure.  
- Use Terraform commands to create and update resources automatically.  
- Store configurations in Git to integrate with CI/CD.

**Example 1 – Simple Web Server Stack (AWS):**[file:4]
- Resources defined:  
  - VPC, subnets, and security groups.  
  - EC2 instance with a public IP.  
  - Outputs exposing public IP.
- Once defined, `terraform apply` provisions complete infrastructure for a basic web application.

**Example 2 – Multi‑Environment Setup:**[file:2][file:4]
- Same Terraform module is used for **dev**, **staging**, and **prod**; only variable values differ (e.g., number of instances).  
- Dev environment uses smaller instance sizes; prod uses larger sizes and more instances.  
- This automation ensures that all environments are structurally similar, reducing surprises in production.

**Integration with DevOps Workflows:**[file:2][file:4]
- A CI/CD pipeline can run `terraform plan` on each pull request to show proposed infra changes.  
- On approval, `terraform apply` is run to update infrastructure.  
- This is a practical form of GitOps for infrastructure.

**Conclusion:**  
Using Terraform, teams can automate complex, multi‑resource infrastructure setups reliably and repeatably, which is a fundamental requirement for modern cloud‑native DevOps environments.[file:4]

---

## 6. Viva Questions and Answers (Unit 5)

1. **Q:** What is Infrastructure as Code?  
   **A:** Managing and provisioning infrastructure using code/configuration files instead of manual setup.[file:2]

2. **Q:** Give any two benefits of IaC.  
   **A:** Consistency and speed (plus reusability, recovery, etc.).[file:2]

3. **Q:** Name any two IaC tools.  
   **A:** Terraform, AWS CloudFormation, Azure ARM, Pulumi (any two).[file:2][file:4]

4. **Q:** What is Terraform?  
   **A:** An open‑source IaC tool by HashiCorp that defines and manages infrastructure using declarative configuration files.[file:4]

5. **Q:** What does `terraform init` do?  
   **A:** Initializes the Terraform working directory and downloads provider plugins.[file:4]

6. **Q:** What is the purpose of `terraform plan`?  
   **A:** Shows the proposed changes to infrastructure before applying them.[file:4]

7. **Q:** What is the Terraform state file?  
   **A:** A file that tracks the current infrastructure state and maps configuration to real resources.[file:4]

8. **Q:** What is idempotency in Terraform?  
   **A:** The property that applying the same configuration repeatedly results in the same infrastructure without duplicates.[file:4]

9. **Q:** What is a provider in Terraform?  
   **A:** A plugin that interacts with a specific cloud or service API (e.g., AWS, Azure).[file:4]

10. **Q:** What language is typically used for Terraform configurations?  
    **A:** HashiCorp Configuration Language (HCL).[file:4]

---

## 7. Quick Revision Summary (Unit 5)

- IaC = **infrastructure managed using code**, not manual clicks.[file:2]  
- Benefits: **consistency, speed, versioning, recovery, scalability, reusability**.[file:2]  
- IaC tools: **Terraform (multi‑cloud), CloudFormation (AWS), ARM (Azure), Pulumi**.[file:2][file:4]  
- Terraform: **declarative, provider‑based, uses HCL, keeps state, generates plans**.[file:4]  
- Terraform workflow: **write configs → `init` → `plan` → `apply` → infrastructure created**.[file:4]  
- State file: tracks infrastructure, enables safe updates and deletions.[file:4]  
- Key concepts: **provider, resource, variable, output, module, data source, idempotency, dependency management**.[file:4]

---

## 8. Important Exam Tips for Unit 5

- Always start answers with **clear definitions** (IaC, Terraform, state, provider).  
- Use **small IaC examples** (e.g., 1 VM + Nginx) to make answers concrete.  
- Draw simple **workflow diagrams** for Terraform (`init`, `plan`, `apply`).  
- Use **tables** to compare manual vs IaC provisioning, or Terraform vs other tools.  
- Highlight keywords: **declarative, state, idempotent, execution plan, provider, resource, HCL**.  
- Connect IaC tools back to **DevOps, CI/CD, and GitOps** whenever possible.

---

## 9. Most Expected Questions – Unit 5

1. Define Infrastructure as Code. Explain its benefits with examples.  
2. Explain Terraform as an IaC tool and its key characteristics.  
3. Describe the structure and components of Terraform configuration files.  
4. Explain Terraform workflow (`init`, `plan`, `apply`) and the role of the state file.  
5. Discuss how Terraform automates infrastructure provisioning in a cloud environment.
