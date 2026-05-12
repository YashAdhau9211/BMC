# DevOps and Automation – Unit 2 Complete Exam Preparation Notes

## 1. Cover Page

**Subject:** DevOps and Automation  
**Unit:** Unit 2 – Version Control and Continuous Integration (CI)  
**Prepared For:** University Semester Exam Preparation  
**Target:** Full Marks (2, 5, and 10 marks questions)  
**Student Level:** Beginner–Intermediate (CS/IT)  

These notes combine: detailed theory, exam-style answers, question bank, revision points, and viva preparation for Unit 2: *Version Control and CI*.[file:3][file:6]

---

## 2. Unit Introduction

Unit 2 focuses on how teams **manage source code** using version control and how they **automatically build and test** their software using Continuous Integration (CI).[file:3][file:6] You will learn Git and GitHub basics, Git workflows and branching strategies, Maven snapshot builds, Git–Maven integration, and how to set up CI pipelines with Jenkins.[file:3][file:6]

**Syllabus Breakdown (Unit 2):**[file:3][file:6]
1. Introduction to Git and GitHub.  
2. Git workflow and branching strategies.  
3. Maven Snapshot Build.  
4. Git and Maven integration.  
5. Concepts and benefits of Continuous Integration (CI).  
6. Setting up CI pipelines with Jenkins.

**Exam Tip:** For each 10‑mark answer, always combine **definition + diagram + example + conclusion**.

---

## 3. Important Theory Notes

### 3.1 Introduction to Git

**One‑line Definition:**  
Git is a **free, open‑source distributed version control system** used to track changes in source code and enable collaboration among developers.[file:3]

**Detailed Definition (Exam Style):**  
Git is a distributed version control system (VCS) created by Linus Torvalds in 2005 that allows developers to **record, manage, and share changes** to files over time.[file:3] Each developer has a full copy of the repository, including history, which enables offline work, branching, merging, and safe experimentation.[file:3]

**Key Features of Git:**[file:3]
- Distributed repository (every clone has complete history).  
- Lightweight branching and merging.  
- Ability to revert to previous versions.  
- Supports offline work.  
- Integrates with platforms like GitHub, GitLab, Bitbucket.

**Why Git is Used:**[file:3]
- Track changes in code and text files over time.  
- Work in teams without overwriting each other’s work.  
- Experiment safely using branches.  
- Go back to a stable version if something breaks.

**Simple ASCII View of Commits:**

```
Commit History (Master/Main Branch)

[A] -- [B] -- [C] -- [D]
 ^      ^      ^      ^
 v1    v2     v3     v4
```

Each commit is a snapshot of the code at a point in time.

---

### 3.2 Introduction to GitHub

**Definition:**  
GitHub is a **web‑based hosting platform** for Git repositories that provides collaboration features such as pull requests, issues, project boards, code review, and CI/CD automation.[file:3]

**Key Points About GitHub:**[file:3]
- Hosts remote repositories in the cloud.  
- Provides web interfaces to browse code, commits, branches, and releases.  
- Pull Requests for code review and discussion.  
- Issues and Projects for task and bug tracking.  
- GitHub Actions for CI/CD automation.  
- Supports wikis, security alerts, releases, and package registries.

**Difference Between Git and GitHub (High‑level):**[file:3]
- Git = version control software (installed on your computer).  
- GitHub = hosting and collaboration platform on the internet.

**ASCII Relation:**

```
[Developer Laptop]
   | (Git)
   v
[Local Repo] <----push/pull----> [Remote Repo on GitHub]
```

---

### 3.3 Basic Git–GitHub Workflow

**Typical Steps for a New Project:**[file:3]
1. Clone the repository once:  
   `git clone https://github.com/username/repository.git`  
   `cd repository`  
2. Daily workflow:  
   - Create branch: `git checkout -b feature/new-feature`  
   - Stage changes: `git add .`  
   - Commit: `git commit -m "Add new feature"`  
   - Push: `git push origin feature/new-feature`  
   - Open Pull Request on GitHub and get code reviewed.  
   - After merge, update local main:  
     `git checkout main`  
     `git pull`

**ASCII Flow:**

```
Local:  edit -> git add -> git commit -> git push
                               |
                               v
                        GitHub: Pull Request -> Review -> Merge
```

**Exam Tip:** In “Explain GitHub workflow” questions, always mention **clone, branch, commit, push, pull request, merge**.

---

### 3.4 Git Workflow and Branching Strategies

A **Git workflow** defines how a team uses branches and merges to collaborate consistently.[file:3]

#### 3.4.1 GitFlow Workflow

GitFlow is a **branch‑based model** suitable for projects with well‑defined release cycles, such as desktop or on‑prem software.[file:3]

**Main Long‑Lived Branches:**[file:3]
- `main` – always production‑ready, each commit often tagged as a release version.  
- `develop` – integration branch where features are merged before a release.

**Supporting Short‑Lived Branches:**[file:3]
- `feature/*` – created from `develop` to build new features.  
- `release/*` – created from `develop` when preparing a release, only bug fixes and polishing.  
- `hotfix/*` – created from `main` to fix urgent production bugs.

**Pros / Cons:**[file:3]
- Pros: good for managing multiple versions and hotfixes; clear structure.  
- Cons: relatively complex; can slow down continuous delivery due to many merges.

**ASCII Overview:**

```
main    -----o---------o-----o------->
              \       ^\
               \      | \ (merge hotfix)
                \     |
 develop  -------o----o--o----------->
                / \  / \
feature/x -----   \/   ----->
```

---

#### 3.4.2 GitHub Flow

GitHub Flow is a **simple feature‑branch workflow** ideal for web apps and SaaS with continuous deployment.[file:3]

- Only one long‑lived branch: `main` (always deployable).  
- For each task: create short‑lived `feature` branch from `main`, implement changes, create Pull Request, review, merge, and deploy.[file:3]

**Pros / Cons:**[file:3]
- Pros: simple, fast, supports CI/CD.  
- Cons: not ideal for projects needing multiple maintained versions.

---

#### 3.4.3 Trunk‑Based Development (TBD)

Trunk‑Based Development emphasizes a single **trunk (main)** branch with **very short‑lived branches or direct commits**.[file:3]

**Key Principles:**[file:3]
- Small, frequent commits directly to main or via very short branches.  
- Use **feature flags** to hide incomplete features in production.  
- Requires strong automated tests and code review discipline.

**Pros / Cons:**[file:3]
- Pros: minimizes merge conflicts, supports very rapid CI/CD.  
- Cons: needs high test coverage and mature team practices.

---

#### 3.4.4 GitLab Flow (Hybrid)

GitLab Flow combines ideas from GitFlow and GitHub Flow by using **environment branches** like `staging` and `production`.[file:3]

- Code flows from `main` → `staging` → `production`.  
- Provides more control than pure GitHub Flow but less complexity than GitFlow.[file:3]

**Branching Strategy Comparison Table:**[file:3]

| Strategy               | Complexity | Long‑Lived Branches           | Best For                              | Release Cadence       |
|------------------------|-----------|-------------------------------|---------------------------------------|------------------------|
| GitHub Flow            | Low       | `main` only                   | Web/SaaS, CI/CD teams                 | Continuous/daily       |
| Trunk‑Based Development | Low–Med  | `main` (trunk)               | High‑velocity, microservices teams    | Many times per day    |
| GitFlow                | Medium    | `main`, `develop`            | Products with strict releases         | Weeks/months          |
| GitLab Flow            | Medium    | `main`, env branches         | Teams with staging/production envs    | Flexible              |

**Exam Tip:** Questions on “Git workflows” or “branching strategies” are very common – always draw a small diagram and mention **pros/cons**.

---

### 3.5 Maven Snapshot Build

**What is Maven (Quick Recall):**  
Maven is an **open‑source project management and build automation tool** for Java that uses a `pom.xml` file to manage dependencies, build lifecycle, tests, and packaging.[file:6]

#### 3.5.1 Snapshot vs Release Versions

**Release Version:**[file:6]
- A **stable, unchangeable** version (e.g., `1.0.0`).  
- Once released, it should never change; used in production environments.

**Snapshot Version:**[file:6]
- A **development version** marked with `-SNAPSHOT` suffix (e.g., `1.0.0-SNAPSHOT`).  
- Represents work‑in‑progress; Maven treats it as mutable and checks for updates frequently.  
- Used only during development, not recommended for production.

**Key Characteristics of Snapshot Builds:**[file:6]
- Mutable: the same version (`1.0.0-SNAPSHOT`) can be replaced by newer builds.  
- Maven checks remote repositories for newer snapshot timestamps (by default, daily).  
- Supports **team collaboration** by sharing latest development artifacts.

**ASCII View:**

```
Stable:   1.0.0      (fixed, immutable)
Dev:      1.0.0-SNAPSHOT -> keeps changing as dev progresses
```

**Exam Tip:** For “Maven Snapshot Build” questions, clearly write: **-SNAPSHOT**, **mutable**, **dev phase only**, **frequent updates**.

---

### 3.6 Git and Maven Integration

Git and Maven usually work together in **Java projects** to manage source code (Git) and builds (Maven).[file:6]

#### 3.6.1 Basic Setup

1. Initialize Git in the Maven project: `git init`.[file:6]  
2. Add remote repository: `git remote add origin <repo-url>`.[file:6]  
3. Track `pom.xml` and source files in Git for version control.[file:6]

#### 3.6.2 Maven Plugins for Git Integration

**Maven Release Plugin:**[file:6]
- Automates release process by:  
  - Updating version from `x.y.z-SNAPSHOT` to `x.y.z`.  
  - Committing version changes.  
  - Tagging the release in Git.  
  - Incrementing version to next `x.y.(z+1)-SNAPSHOT` for further development.

**Git‑Flow Maven Plugin:**[file:6]
- Supports GitFlow branching model – can start/finish feature, release, and hotfix branches via Maven commands.

**Git Commit ID Plugin:**[file:6]
- Exposes current Git commit hash/branch as properties in the build, useful for embedding build info into artifacts.

#### 3.6.3 CI/CD Integration (Git + Maven + Jenkins)

- Jenkins polls or receives webhooks from a Git repository.  
- On each commit, Jenkins clones the repo, runs Maven goals like `mvn clean install`, runs tests, and packages artifacts.[file:6]  
- Artifacts (JAR/WAR) can be published to repository managers or used to build Docker images.[file:6]

**ASCII Pipeline:**

```
[Git Repo] --webhook--> [Jenkins]
                        |
                        +--> mvn clean test
                        +--> mvn package (JAR/WAR)
                        +--> deploy artifact
```

---

### 3.7 Concepts and Benefits of Continuous Integration (CI)

**Definition of CI:**  
Continuous Integration (CI) is a **development practice where developers frequently integrate their code changes into a shared repository**, and each integration triggers an automated build and test process.[file:6]

**Core Concepts of CI:**[file:6]
- **Frequent Commits:** Developers commit small changes several times a day instead of big batches.  
- **Automated Build:** On each commit, a CI server compiles code and creates build artifacts.  
- **Automated Testing:** Unit tests, integration tests, static analysis, and sometimes smoke tests run automatically.  
- **Shared Repository:** A central Git repository (GitHub/GitLab/Bitbucket).  
- **Fast Feedback Loop:** Developers get quick feedback (within minutes) if their change breaks the build.  
- **Build Once, Promote Many:** The same built artifact moves through dev, test, and production environments.

#### 3.7.1 Benefits of Continuous Integration

According to DevOps best practices, CI provides many advantages:[file:6]

- **Early Bug Detection:** Problems are found minutes after commit, not weeks later.  
- **Reduced Integration Risk:** Avoids “integration hell” caused by long‑lived unmerged code.  
- **Faster Feedback and Higher Productivity:** Developers know quickly whether their code works.  
- **Improved Code Quality:** Encourages better tests, modular design, and cleaner code.  
- **Better Team Collaboration:** Everyone works on the latest codebase, reducing “works on my machine” issues.  
- **Faster Time‑to‑Market:** Enables small, frequent releases and continuous delivery.  
- **Scalability for Large Teams:** Supports dozens or hundreds of developers working on the same codebase.[file:6]

**Exam Tip:** When asked “Explain concepts and benefits of CI”, always separate **concepts** and **benefits** into two clearly titled sub‑sections.

---

### 3.8 Setting up CI Pipelines with Jenkins

**What is Jenkins (Quick Recall):**  
Jenkins is an **open‑source automation server** used to implement CI and CD pipelines.[file:7]

#### 3.8.1 Basic Components of a Jenkins CI Pipeline

- **Jenkins Server:** Core service where jobs run.  
- **Jobs/Pipelines:** Configured to build projects (Freestyle or Pipeline jobs).  
- **Source Code Management (SCM):** Git repository configuration.  
- **Build Steps:** Commands (e.g., `mvn clean install`).  
- **Post‑Build Actions:** Publish artifacts, run tests, send notifications, or trigger deployments.

#### 3.8.2 Steps to Set Up a Simple CI Pipeline

1. **Install Jenkins** on a server (or use containerized Jenkins).  
2. **Install Plugins** for Git, Maven, and any required tools.  
3. **Create a New Job** (e.g., Freestyle or Pipeline).  
4. **Configure Git Repository URL** and credentials.  
5. **Set Build Triggers:** Poll SCM or use webhooks so each commit starts a build.  
6. **Add Build Step:** Run Maven goals (`mvn clean test`, `mvn package`).  
7. **Configure Post‑Build Actions:** Archive artifacts, publish JUnit test reports, or deploy to servers.

**ASCII CI Pipeline (Git + Jenkins + Maven):**

```
Developer Commit --> GitHub Repo --> Jenkins Job Triggered
                                      |
                                      v
                              [mvn clean test]
                                      |
                                      v
                              [mvn package]
                                      |
                                      v
                           [Archive/Deploy Artifact]
```

**Relation to DevOps:**  
CI with Jenkins is a core DevOps practice – it automates integration and testing, ensuring that the main branch is always in a buildable state.[file:6][file:7]

---

## 4. Question Bank – Unit 2 (2, 5, and 10 Marks)

### 4.1 2‑Mark Questions

1. What is Git?  
2. What is GitHub?  
3. Define version control system.  
4. List any two Git branching strategies.  
5. What is a Git snapshot? (Trick: Git doesn’t use this term in the same way as Maven; answer: state of repository at a commit.)  
6. What is a Maven snapshot version?  
7. Differentiate between snapshot and release versions in Maven (any two points).  
8. What is Continuous Integration?  
9. Name any two benefits of CI.  
10. Name any two CI tools other than Jenkins.  
11. What is a Jenkins pipeline?  
12. What is the role of Git in a CI pipeline?

### 4.2 5‑Mark Questions (Short Notes)

1. Explain Git and GitHub with suitable examples.  
2. Write a short note on Git workflow and feature branching.  
3. Explain any two Git branching strategies.  
4. Explain Maven snapshot and release builds.  
5. Describe Git and Maven integration in a Java project.  
6. Explain the core concepts of Continuous Integration (CI).  
7. Explain the benefits of CI for software development teams.  
8. Write a short note on Jenkins as a CI server.  
9. Describe the steps to configure a simple CI pipeline using Jenkins.  
10. Write short notes on: (a) GitFlow, (b) Trunk‑Based Development.

### 4.3 10‑Mark Questions (Long Answers)

1. Explain Git and GitHub in detail. Describe a typical Git–GitHub workflow used in DevOps projects.  
2. What are Git branching strategies? Explain GitFlow, GitHub Flow, Trunk‑Based Development, and GitLab Flow with diagrams and pros/cons.  
3. Explain the concept of Maven snapshot build. Differentiate between snapshot and release builds and describe when each is used.  
4. How are Git and Maven integrated in a CI/CD environment? Explain with an example pipeline.  
5. Define Continuous Integration. Explain its core concepts and discuss the benefits of implementing CI.  
6. Describe the steps involved in setting up a CI pipeline using Jenkins, Git, and Maven.  
7. Explain how CI supports DevOps practices and improves software delivery.  
8. Compare different Git workflows and suggest a suitable workflow for a web‑based SaaS project.  
9. Explain the role of Jenkins in DevOps. How does Jenkins integrate with Git and Maven to provide CI?  
10. Discuss the importance of automated testing in CI and its impact on software quality.

**Most Expected Questions:**  
Q1, Q2, Q3, Q4, Q5, and Q6 are highly likely as 10‑mark questions.

---

## 5. Detailed 10‑Mark Answers (Exam‑Oriented)

### Q1. Explain Git and GitHub in detail. Describe a typical Git–GitHub workflow used in DevOps projects.

**Introduction:**  
Version control is essential in modern software development to track code changes, collaborate in teams, and safely experiment. Git and GitHub together form a powerful ecosystem for code management in DevOps.[file:3]

**Definition of Git:**  
Git is a **distributed version control system (DVCS)** that stores the entire history of a project on every developer’s machine and allows branching, merging, and rollback of changes.[file:3]

**Definition of GitHub:**  
GitHub is a **cloud‑based hosting platform for Git repositories** that adds collaboration tools like pull requests, issue tracking, project boards, CI/CD, and code review.[file:3]

**Key Features of Git:**[file:3]
- **Distributed Architecture:** Each clone contains a full copy of the repository history.  
- **Branching and Merging:** Easy creation of branches to develop new features without affecting main code.  
- **History and Revert:** View previous versions and revert to stable commits.  
- **Offline Work:** Most Git operations work without internet.

**Key Features of GitHub:**[file:3]
- Hosting of remotes for collaboration.  
- Pull Requests and reviews for quality control.  
- Issues and Projects for bug and task management.  
- GitHub Actions for automated CI/CD.

**Typical Git–GitHub Workflow:**[file:3]
1. **Clone Repository:** `git clone <url>` to get local copy.  
2. **Create Feature Branch:** `git checkout -b feature/login-page`.  
3. **Do Work Locally:** Edit files, run local tests.  
4. **Stage and Commit:**  
   `git add .`  
   `git commit -m "Add login page"`  
5. **Push Branch:** `git push origin feature/login-page`.  
6. **Open Pull Request (PR):** On GitHub, create PR from feature branch to `main` for code review.  
7. **Code Review & Merge:** After review and approvals, merge PR into `main`.  
8. **Update Local Main:**  
   `git checkout main`  
   `git pull`

**ASCII Diagram of Workflow:**

```
Local repo:   main  ----o-----------o----->
                       \
                        \--o--o--->  feature/login-page
                               |
                           git push
                               |
Remote (GitHub): PR -> Review -> Merge -> main updated
```

**Example (Industry):**  
In a web application team, each new feature (e.g., search bar) is developed in its own feature branch, then merged via GitHub pull requests. Jenkins is integrated to automatically build and test each PR before merge.[file:3][file:6]

**Conclusion:**  
Git provides the core version control capabilities, while GitHub adds collaboration and automation. Together, they support efficient and safe teamwork, which is a foundation of DevOps workflows.[file:3]

---

### Q2. What are Git branching strategies? Explain GitFlow, GitHub Flow, Trunk‑Based Development, and GitLab Flow with diagrams and pros/cons.

**Introduction:**  
Branching strategies define **how branches are created, used, and merged** in a Git‑based project. The right strategy depends on project size, release cycle, and team maturity.[file:3]

#### (a) GitFlow Workflow

- Main branches: `main` (production) and `develop` (integration).  
- Supporting branches: `feature/*`, `release/*`, `hotfix/*`.[file:3]

**Diagram (Simplified):**

```
main:    ---------o---------o-----------o------>
            \
             \   hotfix
              o---------o--/

develop: -----o----o-----------o--------------->
             / \  / \
feature:   o   oo   o
```

**Pros:** Handles multiple releases, clear structure for hotfixes.  
**Cons:** Many branches and merges; slower CI/CD.[file:3]

#### (b) GitHub Flow

- Single long‑lived `main` branch.  
- Short‑lived feature branches created from `main`, merged back via PRs.[file:3]

**Diagram:**

```
main:  ---o------o-------o------------->
            \
             \--o--o---> feature
```

**Pros:** Simple, fast, great for continuous deployment.  
**Cons:** Not ideal when multiple versions must be maintained.[file:3]

#### (c) Trunk‑Based Development (TBD)

- Focuses on a single trunk (`main`); branches are extremely short‑lived or developers commit directly to trunk.[file:3]

**Diagram:**

```
main (trunk): o-o-o-o-o-o-o-o-o-o-o-o-o-o-o
```

**Pros:** Minimizes merge conflicts, supports rapid CI/CD.  
**Cons:** Requires strong automated testing and discipline.[file:3]

#### (d) GitLab Flow

- Combines ideas from GitFlow and GitHub Flow.  
- Uses environment branches like `staging` and `production` in addition to `main` and feature branches.[file:3]

**Diagram:**

```
main  -----> staging -----> production
   \             ^             ^
    \-- feature  |             |
```

**Pros:** Better control with multiple environments, simpler than full GitFlow.  
**Cons:** Slightly more complex than pure GitHub Flow.[file:3]

**Conclusion:**  
Git branching strategies provide a structured way to manage parallel work. Modern DevOps organizations often choose simpler models like GitHub Flow or Trunk‑Based Development to support continuous integration and delivery.[file:3]

---

### Q3. Explain the concept of Maven snapshot build. Differentiate between snapshot and release builds and describe when each is used.

**Introduction:**  
In large Java projects, teams need a consistent way to share build artifacts across environments and team members. Maven supports this through **snapshot** and **release** builds.[file:6]

**Definition of Maven Snapshot Build:**  
A Maven snapshot build is an **in‑development version** of a project, marked with the suffix `-SNAPSHOT` in its version number (e.g., `1.0.0-SNAPSHOT`), intended for use during the development phase and allowed to change frequently.[file:6]

**Snapshot vs Release – Key Differences:**[file:6]

| Aspect             | Snapshot Version (`x.y.z-SNAPSHOT`)               | Release Version (`x.y.z`)                    |
|--------------------|----------------------------------------------------|----------------------------------------------|
| Stability          | Unstable, work‑in‑progress                         | Stable, tested, production‑ready             |
| Mutability         | Mutable – same version can be updated             | Immutable – must not change once released    |
| Usage              | Development, internal testing                     | Production deployments, public distribution   |
| Repository Checks  | Maven checks for new snapshots more frequently    | Downloaded once and cached                   |

**How Snapshot Builds Work:**[file:6]
- The project version in `pom.xml` is set to `x.y.z-SNAPSHOT`.  
- When you run `mvn deploy`, Maven uploads artifacts labeled as snapshot to the snapshot repository.  
- Each snapshot build is stored with a unique timestamp internally, but the dependency version remains `x.y.z-SNAPSHOT`.  
- Dependent projects with snapshot dependencies get newer builds automatically (subject to update policy).

**When to Use Which:**[file:6]
- **Use Snapshot Builds** when teams are actively developing and need to share the latest changes frequently for integration testing.  
- **Use Release Builds** when the version is stable, fully tested, and ready for production or external customers.

**Conclusion:**  
Maven snapshot builds enable teams to collaborate quickly during development, while release builds provide fixed, reliable versions for deployment. Correct use of both is essential for a structured DevOps pipeline.[file:6]

---

### Q4. How are Git and Maven integrated in a CI/CD environment? Explain with an example pipeline.

**Introduction:**  
Git manages the source code, while Maven manages the build lifecycle. In a CI/CD environment, tools like Jenkins bring them together into an automated pipeline.[file:6]

**Role of Git:**[file:3][file:6]
- Central repository of source code and `pom.xml`.  
- Every commit or merge event can trigger a CI build.  
- Branches are used to separate development, testing, and production changes.

**Role of Maven:**[file:6]
- Handles compiling Java code (`mvn compile`).  
- Runs tests (`mvn test`).  
- Packages the application (`mvn package` for JAR/WAR).  
- Deploys artifacts to repositories (`mvn deploy`).

**Example CI Pipeline (Git + Maven + Jenkins):**[file:6]

1. **Developer pushes code** to GitHub main or feature branch.  
2. **Jenkins receives webhook** from GitHub and triggers a job.  
3. **Jenkins checks out code** from Git repository.  
4. **Jenkins runs Maven commands:**  
   - `mvn clean test` – compile and run tests.  
   - `mvn package` – create JAR/WAR file.  
5. **Post‑build actions:**  
   - Archive artifacts.  
   - Publish test reports.  
   - Optionally deploy artifacts to staging or production.  

**ASCII Pipeline:**

```
[Developer] -> git push -> [GitHub]
                             |
                        webhook
                             |
                          [Jenkins]
                             |
                      mvn clean test
                             |
                      mvn package
                             |
                       Deploy/Archive
```

**Conclusion:**  
Git, Maven, and Jenkins together form a typical DevOps toolchain. Git detects changes, Maven builds and tests them, and Jenkins orchestrates the entire CI/CD process.[file:6]

---

### Q5. Define Continuous Integration. Explain its core concepts and discuss the benefits of implementing CI.

**Introduction:**  
Continuous Integration (CI) is a central DevOps practice that keeps the codebase always in a buildable and testable state, significantly reducing integration problems.[file:6]

**Definition of CI:**  
CI is a practice in which **developers frequently merge their code changes into a shared repository**, and each merge triggers an automated build and test sequence to detect issues early.[file:6]

**Core Concepts of CI:**[file:6]
1. **Frequent Commits:** Small, incremental commits multiple times a day.  
2. **Automated Builds:** Every commit triggers automatic compilation and packaging.  
3. **Automated Tests:** A comprehensive test suite runs automatically.  
4. **Shared Mainline:** All developers integrate into a shared main branch, avoiding long‑lived divergent branches.  
5. **Fast Feedback:** Build and test results are available quickly to the developer.  
6. **Build Once, Deploy Many:** Same artifacts used across environments.

**Benefits of CI:**[file:6]
- **Early Detection of Defects:** Issues are caught minutes after changes, reducing fix cost.  
- **Reduced Integration Risk:** Avoids last‑minute “integration hell”.  
- **Improved Code Quality:** Encourages good modular design and testing.  
- **Higher Developer Productivity:** Less time debugging, more time building features.  
- **Better Collaboration:** Everyone shares a single source of truth.  
- **Foundation for CD/DevOps:** CI is the first step towards continuous delivery and deployment.

**Conclusion:**  
CI transforms software development by making integration a continuous, low‑risk activity instead of a rare, high‑risk event. It directly supports DevOps goals of speed, quality, and reliability.[file:6]

---

### Q6. Describe the steps involved in setting up a CI pipeline using Jenkins, Git, and Maven.

**Introduction:**  
Jenkins is widely used to create CI pipelines that automate build and test flows based on changes in a Git repository, using tools like Maven.[file:6][file:7]

**Steps to Set Up CI Pipeline:**

1. **Install Jenkins:**  
   - Install Jenkins on a server or container.  
   - Configure Java and necessary environment variables.[file:7]

2. **Install Required Plugins:**  
   - Git plugin for SCM integration.  
   - Maven Integration plugin.  
   - Any test report plugins (e.g., JUnit).[file:7]

3. **Configure Global Tools:**  
   - Set up JDK and Maven installations in Jenkins global configuration.  

4. **Create a New Job:**  
   - Choose Freestyle or Pipeline job.  
   - Provide job name and description.[file:7]

5. **Configure Source Code Management:**  
   - Add Git repository URL and credentials if required.  
   - Optionally specify branch (e.g., `main`).[file:6]

6. **Set Build Triggers:**  
   - Poll SCM at intervals or configure webhooks from GitHub to trigger builds on each push.[file:6]

7. **Add Build Steps:**  
   - Add a build step to run Maven goals such as `clean test` or `clean install`.[file:6]

8. **Post‑Build Actions:**  
   - Archive the generated artifacts.  
   - Publish JUnit test results.  
   - Optionally trigger deployment jobs.[file:6]

**ASCII Diagram of Jenkins CI:**

```
Git Push --> Jenkins Trigger --> Checkout Code
                                    |
                                mvn clean test
                                    |
                                mvn package
                                    |
                               Archive/Deploy
```

**Conclusion:**  
By connecting Jenkins with Git and Maven, organizations can automatically build and test their applications on every change, ensuring continuous integration and laying the foundation for a full DevOps pipeline.[file:6][file:7]

---

## 6. Viva Questions and Answers (Unit 2)

1. **Q:** What is Git?  
   **A:** Git is a distributed version control system used to track changes in files and support collaborative development.[file:3]

2. **Q:** What is GitHub?  
   **A:** GitHub is a cloud‑based platform that hosts Git repositories and provides collaboration features like pull requests and issue tracking.[file:3]

3. **Q:** What is a branch in Git?  
   **A:** A branch is a movable pointer to a series of commits, used to develop features independently from the main line.[file:3]

4. **Q:** What is a snapshot version in Maven?  
   **A:** A snapshot version (e.g., `1.0.0-SNAPSHOT`) represents an in‑development, mutable build used during active development.[file:6]

5. **Q:** What is a release version in Maven?  
   **A:** A release version (e.g., `1.0.0`) is a stable, immutable build intended for production use.[file:6]

6. **Q:** Define Continuous Integration.  
   **A:** CI is the practice of frequently integrating code into a shared repository and automatically building and testing each integration.[file:6]

7. **Q:** Name any two benefits of CI.  
   **A:** Early bug detection and reduced integration risk (plus many others).[file:6]

8. **Q:** What is Jenkins?  
   **A:** Jenkins is an open‑source automation server used to implement CI/CD pipelines.[file:7]

9. **Q:** How does Jenkins know when to start a build?  
   **A:** Through triggers such as SCM polling or webhooks sent when commits are pushed to Git.[file:6]

10. **Q:** What is the role of `pom.xml` in a Maven project?  
    **A:** It defines project metadata, dependencies, and build configuration.[file:6]

---

## 7. Quick Revision Summary (Unit 2)

- Git = **distributed version control system** for tracking code changes.[file:3]  
- GitHub = **cloud platform** for hosting Git repos and enabling collaboration.[file:3]  
- Common Git workflows: **GitFlow, GitHub Flow, Trunk‑Based Development, GitLab Flow**.[file:3]  
- Maven = **build automation and dependency management tool** for Java.[file:6]  
- Snapshot builds (`-SNAPSHOT`) are **mutable, development‑phase** versions; release builds are **stable and immutable**.[file:6]  
- Git + Maven + Jenkins form a standard **CI pipeline**: commit → build → test → package → deploy.[file:6][file:7]  
- CI = frequent integration + automated builds/tests + fast feedback.[file:6]  
- Benefits of CI: early bug detection, reduced integration risk, faster delivery, higher code quality.[file:6]

---

## 8. Important Exam Tips for Unit 2

- For 10‑mark answers, always start with **definition + context**, then explain **concepts with headings**, include a **diagram/ASCII flow**, add a **real‑world example**, and finish with a **conclusion**.  
- Use comparison **tables** for snapshot vs release, Git vs GitHub, and various Git workflows.  
- Highlight keywords such as **“distributed”, “snapshot”, “immutable”, “CI”, “pipeline”, “webhook”, “branching strategy”**.  
- Draw small but clear diagrams of branches and pipelines – they fetch easy marks.  
- Connect Unit 2 concepts back to **DevOps goals** like fast, reliable delivery and collaboration.

---

## 9. Most Expected Questions – Unit 2

1. Explain Git and GitHub with workflow.  
2. Explain different Git branching strategies with diagrams.  
3. Explain Maven snapshot build and compare snapshot vs release.  
4. Explain integration of Git and Maven in a CI/CD pipeline.  
5. Define Continuous Integration and explain its concepts and benefits.  
6. Describe steps to configure a CI pipeline using Jenkins, Git, and Maven.
