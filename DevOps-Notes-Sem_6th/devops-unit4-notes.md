# DevOps and Automation – Unit 4 Complete Exam Preparation Notes

## 1. Cover Page

**Subject:** DevOps and Automation  
**Unit:** Unit 4 – Configuration Management  
**Prepared For:** University Semester Exam Preparation  
**Target:** Full Marks (2, 5, and 10 marks questions)  
**Student Level:** Beginner–Intermediate (CS/IT)  

These notes combine: detailed theory, exam-style answers, question bank, revision points, and viva preparation for Unit 4: *Configuration Management*.[file:2][file:4]

---

## 2. Unit Introduction

Configuration Management (CM) in DevOps is about **keeping all servers, software, and settings consistent, organized, and automatically managed** across environments.[file:2] Instead of configuring machines manually, we define the desired state as code and let tools like **Ansible, Puppet, Chef, and SaltStack** enforce that state.[file:2]

This unit focuses on:  
- Importance of configuration management.  
- Tools for configuration management (Ansible, Puppet).  
- Installation and setup of Ansible.  
- Writing Ansible playbooks.  
- Automating tasks with Ansible.[file:2][file:4]

**Exam Tip:** In definition/short note questions, always use the idea: *"define how system should look, then automatically ensure it stays that way"* for configuration management.[file:2]

---

## 3. Important Theory Notes

### 3.1 What is Configuration Management in DevOps?

**One‑line Definition:**  
Configuration Management in DevOps is the practice of **automatically setting up and maintaining systems in a consistent and repeatable way**.[file:2]

**Detailed Definition (Exam Style):**  
Configuration Management is the **process, practice, and tooling** that ensures all environments, servers, applications, infrastructure components, settings, and runtime parameters remain **consistent, reproducible, and correctly configured** across the entire software lifecycle (development, testing, staging, production, and rollbacks).[file:2]

**Simple Explanation:**  
Instead of manually setting up servers again and again, we **write instructions once** (as code) and tools apply those instructions **automatically on many servers**.[file:2]

**Real‑life Example:**[file:2]
- Need to install Python and deploy web app version 1.2 on 10 servers with same settings.  
- Without CM: log into every server and install manually (slow, error‑prone).  
- With CM: write a script (e.g., Ansible playbook) to install Python, deploy app 1.2, set environment variables, and run it for all servers at once.

**ASCII Idea:**

```
[Config Code]  --->  [Config Tool]  --->  [10 Servers Configured Identically]
```

---

### 3.2 Why Configuration Management is Important

In DevOps, CM solves the classic **"it works on my machine"** problem.[file:2]

**Core Goals of Configuration Management:**[file:2]

| Goal             | What it ensures                                      | Without CM (Pain)                        |
|------------------|------------------------------------------------------|------------------------------------------|
| Consistency      | Same software & config behave same everywhere       | "It works on my machine" syndrome       |
| Reproducibility  | Any environment can be recreated from code          | Snowflake servers, slow onboarding       |
| Automation       | No manual server tweaking; everything as code       | Hours/days to provision/fix servers      |
| Version Control  | Config changes tracked and reversible via Git       | Unknown changes break production         |
| Drift Detection  | Detect & fix when servers drift from desired state  | Config drifts over time, random issues   |
| Speed & Safety   | Fast, low‑risk deployments                          | Fear of deploying, especially on Fridays |

**Relation to Infrastructure as Code (IaC):**  
- **IaC** creates infrastructure (servers, networks, databases).  
- **Configuration Management** configures what runs inside those servers.[file:2]  
Together, they fully automate environment creation and setup.

---

### 3.3 Infrastructure as Code (IaC) – Quick Recap

IaC is closely related to configuration management and often appears in theory questions.[file:2]

**Definition of IaC:**  
Infrastructure as Code means **managing and setting up infrastructure (servers, networks, databases) using code**, instead of manual configuration in a UI.[file:2]

**Simple Analogy:**[file:2]
- IaC code is like a **recipe**.  
- Recipe (code) → Dish (infrastructure).  
- Follow same recipe → get same dish every time.

**Why IaC is Important:**[file:2]
- Consistency – same setup each time.  
- Speed – infrastructure in minutes.  
- Version control – infra scripts stored in Git.  
- Easy recovery – recreate environment any time.

**Tools (for context):** Terraform, AWS CloudFormation, Azure ARM, Pulumi.[file:2]

**Relation to CM:**[file:2]
- IaC: creates/updates servers and networks.  
- CM (e.g., Ansible, Puppet): installs packages, configures apps, services, and settings.

---

### 3.4 Types/Flavors of Configuration Management in Modern DevOps

The notes highlight two common styles.[file:2]

1. **Declarative / Desired‑State Configuration Management:**[file:2]
   - You **describe what the system should look like** (desired state).  
   - Tool ensures reality matches desired state.  
   - Tools: Ansible, Puppet, Chef, SaltStack.  

2. **GitOps‑style Configuration Management:**[file:2]
   - Desired state of environments lives in Git.  
   - An operator/agent continuously checks actual vs desired and reconciles differences.  
   - Tools: Argo CD, Flux, Jenkins X, Weave GitOps.  

**Real‑World Sentence:**[file:2]
> "The entire staging environment is defined in Git. If prod starts drifting, Argo CD shows the diff and auto‑corrects it in 30 seconds or we roll back to last week’s commit."

---

### 3.5 Tools for Configuration Management

**Popular Tools (Overview):**[file:2]
- **Ansible:** Simple, agentless, very popular; uses YAML playbooks.  
- **Puppet:** Powerful, model‑driven, used in large systems.  
- **Chef:** Ruby‑based configuration (code‑heavy).  
- **SaltStack:** Fast, scalable, good for large infrastructures.

#### 3.5.1 Why Ansible is Popular

From the attached notes, Unit 4 focuses deeply on **Ansible**.[file:4]

**Characteristics of Ansible:**[file:4]
- Automation tool that tells multiple computers what to do using simple instructions.  
- Used to configure servers, deploy applications, and automate repetitive tasks.  
- **Agentless** – no software required on target machines (only SSH and Python).  
- Uses **YAML** files called playbooks to describe tasks.

**Three Main Concepts in Ansible:**[file:4]
1. **Control Node** – your laptop/server where Ansible is installed.  
2. **Managed Nodes** – target servers you want to configure.  
3. **Agentless Architecture** – connects via SSH, runs modules, no agent required.[file:4]

**ASCII Overview:**

```
[Control Node with Ansible]
             |
          (SSH)
             |
   -----------------------
   |       |       |     |
[Server1][Server2][...][ServerN]
```

---

### 3.6 Installation and Setup of Ansible

The steps below follow the provided notes exactly.[file:4]

#### 3.6.1 System Requirements

- Operating System: Linux, macOS, or Windows via WSL.  
- Python 3.8+ installed on control node.  
- SSH access to target machines.[file:4]

#### 3.6.2 Step 1 – Install Ansible

**On Ubuntu/Debian:**[file:4]
```bash
sudo apt update
sudo apt install ansible -y
```

**On CentOS/RHEL:**[file:4]
```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

**Using pip (works on many OS):**[file:4]
```bash
pip install ansible
```

#### 3.6.3 Step 2 – Verify Installation

```bash
ansible --version
```
If Ansible is correctly installed, it shows version information.[file:4]

#### 3.6.4 Step 3 – Setup Inventory File

Ansible needs a list of machines (IP/hostnames) to manage, called an **inventory**.[file:4]

**Example inventory file (`hosts`):**[file:4]
```ini
[web]
192.168.1.10
192.168.1.11

[db]
192.168.1.20
```

- Groups like `[web]` and `[db]` help organize servers logically.[file:4]

#### 3.6.5 Step 4 – Setup SSH Access

Ansible uses SSH to connect to managed nodes.[file:4]

Commands (from notes):[file:4]
```bash
ssh-keygen
ssh-copy-id user@192.168.1.10
```

After this, the control node can SSH into target servers without a password.[file:4]

#### 3.6.6 Step 5 – Test Connection

Use the built‑in `ping` module to test connectivity.[file:4]

```bash
ansible -i hosts all -m ping
```

Expected output (per host): `SUCCESS => pong`.[file:4]

#### 3.6.7 Step 6 – Run a Simple Command

```bash
ansible -i hosts all -m shell -a "uptime"
```
This runs the `uptime` command on all servers simultaneously.[file:4]

---

### 3.7 Writing Ansible Playbooks

**What is a Playbook?**  
A playbook is a **YAML file** that describes one or more “plays”. Each play targets a group of hosts and runs a set of tasks (modules) on them.[file:4]

**Important Terms (from notes):**[file:4]
- **Inventory:** List of servers.  
- **Module:** Small reusable task (e.g., `ping`, `apt`, `copy`).  
- **Playbook:** YAML file containing plays and tasks.  
- **Task:** Single action like installing a package or copying a file.

#### 3.7.1 Example – First Ansible Playbook (Install Nginx)

Provided example from notes:[file:4]

```yaml
- name: Install nginx
  hosts: web
  become: yes
  tasks:
    - name: install nginx
      apt:
        name: nginx
        state: present
```

**Explanation:**[file:4]
- `name`: description of play.  
- `hosts: web`: target group in inventory.  
- `become: yes`: run tasks with sudo/root privileges.  
- `tasks`: list of actions.  
- `apt` module: ensures `nginx` package is installed.

**Run the playbook:**[file:4]
```bash
ansible-playbook -i hosts playbook.yml
```

**Real‑World Example (Nginx):**  
Nginx is a web server (pronounced "engine‑x") that can host websites, act as reverse proxy, and do load balancing.[file:4]
When you open `www.example.com`, Nginx receives your request and returns website files like HTML, CSS, and images.[file:4]

---

### 3.8 Automating Tasks with Ansible

Ansible helps automate many repetitive DevOps tasks.[file:4]

**Typical Use Cases:**[file:4]
- Installing packages (e.g., Python, Java, Nginx).  
- Configuring services (e.g., starting/stopping daemons).  
- Deploying web applications.  
- Managing configuration files and templates.  
- Running shell commands/scripts across many servers.

**Simple Automation Flow:**[file:2][file:4]

1. Developer pushes code.  
2. CI/CD pipeline triggers.  
3. Configuration tool (Ansible) sets up server, installs dependencies, deploys app.  
4. Everything happens automatically.

**ASCII Flow with CI/CD:**

```
[Developer Pushes Code]
          |
        (CI/CD)
          |
[Ansible Playbooks Applied]
          |
    [Servers Configured + App Deployed]
```

**Advantages of Using Ansible for Automation:**[file:2][file:4]
- No agent on target machines (simpler operations).  
- Human‑readable YAML syntax.  
- Idempotent tasks (running same playbook again does not break servers).  
- Integrates with CI/CD tools.

---

## 4. Question Bank – Unit 4 (2, 5, and 10 Marks)

### 4.1 2‑Mark Questions

1. Define configuration management in DevOps.  
2. What is Infrastructure as Code (IaC)?  
3. Name any two configuration management tools.  
4. What is Ansible?  
5. What is meant by “agentless” in Ansible?  
6. What is an Ansible inventory?  
7. What is a playbook in Ansible?  
8. Name any two common Ansible modules.  
9. Why is configuration management important? (write any two reasons).  
10. What are managed nodes in Ansible?

### 4.2 5‑Mark Questions (Short Notes)

1. Explain configuration management with a simple example.  
2. Write a short note on the importance of configuration management in DevOps.  
3. Explain Infrastructure as Code (IaC) with advantages.  
4. Write a short note on Ansible and its architecture.  
5. Describe the steps for installation and setup of Ansible.  
6. Explain the structure of an Ansible playbook with example.  
7. Explain how Ansible is used to automate tasks in DevOps.  
8. Compare Ansible with any other configuration management tool (e.g., Puppet).  
9. Explain the role of configuration management in CI/CD pipelines.  
10. Explain declarative vs GitOps‑style configuration management.

### 4.3 10‑Mark Questions (Long Answers)

1. Define configuration management. Explain its goals and importance in DevOps with examples.  
2. What is Infrastructure as Code? Explain its principles, benefits, and relation to configuration management.  
3. Explain the architecture, features, and working of Ansible as a configuration management tool.  
4. Describe step‑by‑step installation and setup of Ansible, including inventory and SSH configuration.  
5. Explain how to write Ansible playbooks with a detailed example for installing and configuring a web server.  
6. Discuss how Ansible can be used to automate common DevOps tasks. Provide at least two use‑case examples.  
7. Explain different flavors of configuration management (declarative, GitOps) with examples.  
8. Compare Ansible and Puppet as configuration management tools in terms of features, architecture, and use cases.  
9. Explain the role of configuration management in ensuring consistency across Dev, Test, and Production.  
10. Describe a real‑life DevOps scenario where configuration management and Ansible are used end‑to‑end.

**Most Expected Questions:**  
Q1, Q3, Q4, Q5, and Q6 are highly likely as 10‑mark questions.

---

## 5. Detailed 10‑Mark Answers (Exam‑Oriented)

### Q1. Define configuration management. Explain its goals and importance in DevOps with examples.

**Introduction:**  
Modern applications run on many servers and environments (dev, test, staging, production). Configuring all of them manually is slow and error‑prone. Configuration Management solves this problem.[file:2]

**Definition of Configuration Management:**  
Configuration Management in DevOps is the practice and tooling that **ensures all environments, servers, applications, and configurations remain consistent, reproducible, and correct**, by defining desired state as code and applying it automatically.[file:2]

**Goals of Configuration Management:**[file:2]
1. **Consistency:** Same software and configuration behave identically across environments.  
2. **Reproducibility:** Any environment (e.g., test, staging) can be recreated from code.  
3. **Automation:** Minimize manual setup; treat configuration as code.  
4. **Version Control:** Track who changed what configuration and when.  
5. **Drift Detection and Correction:** Detect when real systems diverge from desired state and automatically fix them.  
6. **Speed and Safety of Releases:** Enable fast, low‑risk deployments.

**Importance in DevOps:**[file:2]
- DevOps aims at **frequent, reliable releases**; this is impossible if each environment is configured manually.  
- CM ensures **"it works on my machine"** issues are minimized by standardizing configurations.  
- It supports **CI/CD pipelines** by ensuring that build, test, and production environments are consistent.

**Example Scenario:**[file:2]
- Company has web application running on 20 servers.  
- With CM (e.g., Ansible playbook), they define:  
  - Install specific version of Python.  
  - Deploy web app version 1.2.  
  - Set environment variables and configuration files.  
- The same configuration is applied to **all 20 servers** in minutes. If one server breaks, they can recreate it using the same code.

**Conclusion:**  
Configuration Management is a key DevOps practice that guarantees consistent, reliable, and repeatable environments, which is essential for fast and safe software delivery.[file:2]

---

### Q2. What is Infrastructure as Code? Explain its principles, benefits, and relation to configuration management.

**Introduction:**  
Infrastructure as Code (IaC) extends the ideas of configuration management to the level of infrastructure – servers, networks, and cloud resources are all defined in code.[file:2]

**Definition of IaC:**  
IaC means **managing and provisioning infrastructure (servers, networks, databases) using machine‑readable configuration files or code**, rather than manual steps.[file:2]

**Principles of IaC:**[file:2]
1. **Infrastructure as Files:** Configurations stored in text files (e.g., Terraform `.tf`, CloudFormation JSON/YAML).  
2. **Version Control:** Infrastructure definitions tracked in Git like application code.  
3. **Idempotence:** Running the same script multiple times leads to the same result.  
4. **Declarative or Imperative Styles:**  
   - Declarative: define **what** you want (e.g., 2 servers).  
   - Imperative: define **how** to create them (step‑by‑step).  

**Benefits of IaC:**[file:2]
- **Consistency:** Same infra across environments, no manual configuration mistakes.  
- **Speed:** Provision infrastructure in minutes instead of days.  
- **Versioning and Audit:** Roll back infra changes, see history.  
- **Scalability:** Easily create 1 or 100 servers from the same code.  
- **Disaster Recovery:** Recreate entire environment quickly after failures.

**Relation to Configuration Management:**[file:2]
- **IaC:** Creates the **infrastructure** (machines, networks).  
- **Configuration Management:** Configures **what runs on those machines** (apps, packages, services).  
- Example: Terraform creates EC2 instances; Ansible installs Nginx and configures app.

**Conclusion:**  
IaC and configuration management together form the backbone of automated environments in DevOps, enabling consistent, repeatable, and scalable infrastructure.[file:2]

---

### Q3. Explain the architecture, features, and working of Ansible as a configuration management tool.

**Introduction:**  
Ansible is a widely used configuration management and automation tool in DevOps due to its simplicity and agentless design.[file:4]

**Ansible Architecture:**[file:4]
- **Control Node:** Machine where Ansible is installed.  
- **Managed Nodes:** Target servers controlled by Ansible.  
- **Inventory:** File listing managed nodes and groups.  
- **Modules:** Units of work (e.g., `ping`, `apt`, `service`).  
- **Playbooks:** YAML files that define plays and tasks.  
- **Connection:** Uses SSH to connect to managed nodes (no agent).

**Key Features of Ansible:**[file:4]
- Agentless – no need to install agents on target machines.  
- Uses simple YAML syntax – easy to read/write.  
- Idempotent tasks – running same playbook multiple times is safe.  
- Extensible modules – many built‑in modules plus custom modules.  
- Integration with CI/CD tools.

**How Ansible Works (Step‑by‑Step):**[file:4]
1. Administrator writes a **playbook** in YAML.  
2. Ansible reads the playbook.  
3. Connects to managed nodes using SSH.  
4. Executes appropriate **modules** on each node.  
5. Returns results (changed/ok/failed) to control node.

**ASCII Flow:**

```
[Playbook YAML] --> [Ansible Engine] --SSH--> [Modules Run on Servers]
                                              |
                                           [Results]
```

**Example Use Cases:**[file:4]
- Install Nginx on all web servers.  
- Deploy a new version of a web application.  
- Configure system users and permissions.

**Conclusion:**  
Ansible provides a simple yet powerful way to perform configuration management at scale, making it a preferred tool in many DevOps setups.[file:4]

---

### Q4. Describe step‑by‑step installation and setup of Ansible, including inventory and SSH configuration.

**Introduction:**  
Before using Ansible to automate tasks, we must install it on a control node and prepare inventory and SSH access to managed nodes.[file:4]

**Step 1 – Check System Requirements:**[file:4]
- Control node: Linux/macOS/Windows with WSL.  
- Python 3.8 installed.  
- SSH connectivity to managed nodes.

**Step 2 – Install Ansible:**[file:4]
- On Ubuntu/Debian:
  ```bash
  sudo apt update
  sudo apt install ansible -y
  ```  
- On CentOS/RHEL:
  ```bash
  sudo yum install epel-release -y
  sudo yum install ansible -y
  ```  
- Or via pip: `pip install ansible`.

**Step 3 – Verify Installation:**[file:4]
```bash
ansible --version
```
If version info is printed, Ansible is installed correctly.

**Step 4 – Create Inventory File:**[file:4]
- Create `hosts` file:
  ```ini
  [web]
  192.168.1.10
  192.168.1.11

  [db]
  192.168.1.20
  ```
- Use group names like `web`, `db` to target specific sets of servers.

**Step 5 – Configure SSH Access:**[file:4]
- Generate SSH keys:
  ```bash
  ssh-keygen
  ```  
- Copy key to remote server:
  ```bash
  ssh-copy-id user@192.168.1.10
  ```
- Now Ansible can connect without prompting for password.

**Step 6 – Test Connectivity:**[file:4]
```bash
ansible -i hosts all -m ping
```
If successful, you will see `SUCCESS` and `pong` from each host.

**Step 7 – Run a Simple Command:**[file:4]
```bash
ansible -i hosts all -m shell -a "uptime"
```
This verifies that Ansible can execute commands on all managed nodes.

**Conclusion:**  
Following these steps prepares Ansible for configuration management tasks, with inventory and SSH enabling secure communication with all target machines.[file:4]

---

### Q5. Explain how to write Ansible playbooks with a detailed example for installing and configuring a web server.

**Introduction:**  
Playbooks are at the heart of Ansible. They describe **what to do** on which hosts, using readable YAML syntax.[file:4]

**Structure of an Ansible Playbook:**[file:4]
- A playbook contains one or more **plays**.  
- Each play targets a group of hosts and runs a list of **tasks**.  
- Each task uses a **module** with parameters.

**Example Playbook – Install and Start Nginx:**[file:4]

```yaml
- name: Configure web servers
  hosts: web
  become: yes

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

**Explanation of Key Sections:**[file:4]
- `name`: human‑readable description.  
- `hosts: web`: target the `web` group from inventory.  
- `become: yes`: use sudo/root.  
- First task uses `apt` module to ensure nginx package is installed.  
- Second task uses `service` module to start nginx and enable it at boot.

**Running the Playbook:**[file:4]
```bash
ansible-playbook -i hosts webserver.yml
```

**Real‑World Behavior:**  
After running this playbook, all servers in `[web]` group will have Nginx installed and running, hosting configured websites.[file:4]

**Conclusion:**  
Ansible playbooks provide a straightforward and readable way to define complex server configuration in code, supporting robust and repeatable setups across many machines.[file:4]

---

### Q6. Discuss how Ansible can be used to automate common DevOps tasks. Provide at least two use‑case examples.

**Introduction:**  
In DevOps, Ansible is commonly integrated with CI/CD pipelines to automate tasks beyond simple package installation.[file:2][file:4]

**General Automation Capabilities:**[file:4]
- Configure servers.  
- Deploy applications.  
- Manage users and permissions.  
- Run database migrations.  
- Orchestrate multi‑tier deployments.

**Use Case 1 – Web Application Deployment:**[file:2][file:4]
1. CI server builds artifact (JAR/Docker image).  
2. Ansible playbook:  
   - Pulls new artifact or image.  
   - Stops old version of app.  
   - Deploys new version and restarts service.  
3. Playbook can also update Nginx configuration and reload it.

**Use Case 2 – Environment Bootstrapping:**[file:2][file:4]
- For new developers or new test environments, Ansible playbook:  
  - Installs required packages (Git, Java, Python).  
  - Sets environment variables.  
  - Clones repositories.  
  - Configures databases and sample data.  
This greatly speeds up onboarding and avoids manual errors.

**Conclusion:**  
Ansible’s flexibility allows DevOps teams to automate almost any repetitive server‑side task, boosting speed, consistency, and reliability across the software lifecycle.[file:2][file:4]

---

## 6. Viva Questions and Answers (Unit 4)

1. **Q:** What is configuration management?  
   **A:** It is the practice of automatically setting up and maintaining systems in a consistent and repeatable way.[file:2]

2. **Q:** Why is configuration management needed in DevOps?  
   **A:** To ensure consistent environments, reduce manual work, and avoid "it works on my machine" issues.[file:2]

3. **Q:** What is Ansible?  
   **A:** Ansible is an agentless automation and configuration management tool that uses SSH and YAML playbooks.[file:4]

4. **Q:** What is an inventory in Ansible?  
   **A:** A file that lists the managed nodes (servers) and groups.[file:4]

5. **Q:** What is a playbook?  
   **A:** A YAML file in Ansible that defines plays and tasks to run on target hosts.[file:4]

6. **Q:** What does agentless mean?  
   **A:** No dedicated software agent is required on target machines; only SSH is used.[file:4]

7. **Q:** Name any two configuration management tools.  
   **A:** Ansible, Puppet, Chef, SaltStack (any two).[file:2]

8. **Q:** What is IaC?  
   **A:** Managing infrastructure using code files instead of manual steps.[file:2]

9. **Q:** Give one advantage of using Ansible.  
   **A:** Simple YAML syntax and no agents on target servers.[file:4]

10. **Q:** How does Ansible connect to managed nodes?  
    **A:** Using SSH.[file:4]

---

## 7. Quick Revision Summary (Unit 4)

- Configuration Management = **define desired state + enforce it automatically**.[file:2]  
- Goals: consistency, reproducibility, automation, version control, drift detection, speed & safety.[file:2]  
- IaC creates infrastructure; configuration management configures what’s inside servers.[file:2]  
- Tools: Ansible (agentless), Puppet, Chef, SaltStack.[file:2][file:4]  
- Ansible architecture: control node, managed nodes, inventory, modules, playbooks, SSH.[file:4]  
- Ansible installation: install Ansible → create inventory → set up SSH → test with `ping` → run playbooks.[file:4]  
- Playbooks = YAML, contain plays and tasks; use modules like `apt`, `service`, `copy`.[file:4]  
- Ansible automates deployments, server provisioning, environment setup, and more.[file:2][file:4]

---

## 8. Important Exam Tips for Unit 4

- In long answers, always combine theory with 1–2 **practical examples** (e.g., installing Nginx via Ansible).  
- Use **tables** to compare goals of configuration management or to compare Ansible with other tools.  
- Draw small **architecture diagrams** (control node → managed nodes via SSH).  
- Emphasize keywords: **desired state, consistency, agentless, inventory, playbook, YAML, SSH**.  
- Connect configuration management answers back to **DevOps goals** like reliability, speed, and reproducibility.

---

## 9. Most Expected Questions – Unit 4

1. Define configuration management. Explain its goals and importance in DevOps.  
2. Explain Infrastructure as Code (IaC) and its relation to configuration management.  
3. Explain the architecture and working of Ansible.  
4. Describe the installation and setup steps for Ansible.  
5. Explain Ansible playbooks with example for installing a web server.  
6. Discuss how Ansible automates common DevOps tasks.
