# DevOps and Automation – Unit 3 Complete Exam Preparation Notes

## 1. Cover Page

**Subject:** DevOps and Automation  
**Unit:** Unit 3 – Continuous Delivery and Deployment  
**Prepared For:** University Semester Exam Preparation  
**Target:** Full Marks (2, 5, and 10 marks questions)  
**Student Level:** Beginner–Intermediate (CS/IT)  

These notes combine: detailed theory, exam-style answers, question bank, revision points, and viva preparation for Unit 3: *Continuous Delivery and Deployment*.[file:5]

---

## 2. Unit Introduction

This unit explains how software moves **beyond CI** (build and basic tests) into **automatic packaging, testing, staging, and releasing to production**.[file:5] It covers continuous delivery (CD), continuous deployment, build and test automation, deployment pipelines, and CI/CD pipelines with Jenkins.[file:5]

**Syllabus Points (Unit 3):**[file:5]
1. Understanding continuous delivery.  
2. Building and testing automation.  
3. Continuous deployment concepts.  
4. Implementing deployment pipelines.  
5. Creating CI/CD pipelines with Jenkins.

**Exam Tip:** Many students confuse **continuous delivery** and **continuous deployment** – always write a clear definition and a comparison table.

---

## 3. Important Theory Notes

### 3.1 Understanding Continuous Delivery (CD)

**One‑line Definition:**  
Continuous Delivery (CD) is a software development practice where **every code change is automatically built, tested, and prepared so it can be released to production at any time** with minimal risk.[file:5]

**Detailed Definition (Exam Style):**  
Continuous Delivery is the practice of keeping your software **always in a deployable state** by using automated builds, tests, and deployments to staging environments so that releases can be performed on demand whenever the business decides.[file:5]

**Key Characteristics of Continuous Delivery:**[file:5]
- Every commit goes through CI: automated build + tests.  
- Artifacts (packages, Docker images) are created and stored.  
- The system is automatically deployed to **staging** or pre‑production environments that mirror production.  
- Releasing to production usually needs a **manual approval or button click** – this is the main difference from continuous deployment.[file:5]

**Core Idea (from Jez Humble & David Farley):**  
“Your software is always in a deployable state.”[file:5]

**ASCII View – CD Concept:**

```
[Code Commit]
     |
  (CI Build & Tests)
     |
  [Artifact Repository]
     |
[Auto Deploy to Staging]
     |
[Manual Approval] ---> [Production]
```

**Example (Industry):**  
An online banking application runs a full CD pipeline: every merged change automatically goes to a staging environment, where business stakeholders can click “Deploy to Production” during a controlled release window.[file:5]

---

### 3.2 Building and Testing Automation

Automation of **build and test** is the heart of Continuous Delivery – without it, CD is impossible.[file:5]

#### 3.2.1 Build Automation

Every `git push` or pull‑request merge triggers a build through a CI server such as Jenkins, GitHub Actions, GitLab CI, or similar.[file:5]

**Typical Build Steps:**[file:5]
- Compile source code.  
- Run static code analysis (e.g., SonarQube, ESLint, Checkstyle).  
- Package the application (JAR, WAR, npm package, Docker image, etc.).  
- Store the artifact in a repository (Artifactory, Nexus, GitHub Packages, ECR).[file:5]

**Goal:** Reproducible, versioned, immutable artifacts created within minutes.[file:5]

#### 3.2.2 Testing Automation – The Testing Pyramid

Testing automation is the **most critical part** of CD – without strong automated tests, you cannot safely release often.[file:5]

**Testing Pyramid Layers (Summary):**[file:5]

| Layer                   | Speed          | Scope                    | Typical Tools                | Owners            | % of Tests (ideal) |
|-------------------------|----------------|--------------------------|------------------------------|-------------------|--------------------|
| Unit tests              | Very fast      | Single class/function    | JUnit, pytest, Jest, NUnit   | Developers        | 60–80%             |
| Component/Service tests | Fast–medium    | One service/module       | Testcontainers, WireMock     | Devs + Testers    | 15–25%             |
| Integration/Contract    | Medium         | Service‑service, DB      | Pact, Postman, Spring tests  | Service teams     | 5–15%              |
| UI/End‑to‑End           | Slow           | Full user journey        | Selenium, Cypress, Playwright| QA + Devs         | 1–10%              |
| Performance/Security    | Slow–very slow | System qualities         | Gatling, k6, OWASP ZAP, etc. | Specialized roles | Run less often     |

**Key Principles:**[file:5]
- Fast feedback: most tests (unit + core integration) should finish in **5–10 minutes**.  
- Tests should be reliable (no flaky tests).  
- Tests run in parallel wherever possible.  
- Pipeline as code: entire sequence of build, test, deploy is defined in version control (often YAML).[file:5]

**Simplified CD Pipeline Stages (from notes):**[file:5]
1. Commit – CI server detects change.  
2. Build + unit tests.  
3. Package artifact.  
4. Deploy to test environment + run integration/API tests.  
5. Deploy to staging + run end‑to‑end/UI/performance tests.  
6. Manual approval gate (for Continuous Delivery).  
7. Deploy to production (canary, blue‑green, or feature flags).[file:5]

---

### 3.3 Continuous Deployment Concepts

**Definition:**  
Continuous Deployment is an advanced practice where **every change that passes all automated tests is automatically deployed to production** without manual approval.[file:5]

**Key Idea:**  
- CD (Delivery) = always deployable; release is a **business decision**, often triggered manually.  
- CD (Deployment) = always deployable **and actually deployed automatically** after pipeline success.[file:5]

**Requirements for Continuous Deployment:**[file:5]
- Very strong automated test coverage.  
- High confidence in staging environment mirroring production.  
- Outstanding observability and fast rollback mechanisms.  
- Business and organization culture comfortable with frequent automatic releases.

**Modern Release Techniques:**[file:5]
- **Blue‑Green Deployment:** Two identical production environments; switch traffic between them.  
- **Canary Releases:** Release to a small portion of users, monitor, then roll out gradually.  
- **Feature Flags:** Hide incomplete features behind configuration flags.

---

### 3.4 Difference: Continuous Integration vs Continuous Delivery vs Continuous Deployment

The notes provide a clear comparison.[file:5]

**Concept Summary:**[file:5]
- **Continuous Integration (CI):** Integrate code frequently, run automated build and tests.  
- **Continuous Delivery (CD):** Keep application always ready for deployment; automated deploy to staging; production deploy is manual.  
- **Continuous Deployment (CDp):** Automatically deploy every successful change to production.

**Comparison Table:**[file:5]

| Aspect                    | Continuous Integration (CI)     | Continuous Delivery (CD)                    | Continuous Deployment (CDp)                 |
|---------------------------|----------------------------------|---------------------------------------------|---------------------------------------------|
| Frequency of integration  | Multiple times per day          | Multiple times per day                      | Multiple times per day                      |
| Automated build & test    | Yes                             | Yes                                         | Yes                                         |
| Automated deploy to staging | Usually yes                    | Yes                                         | Yes                                         |
| Production deployment     | Not in CI scope                 | Manual approval / button                    | Fully automatic after tests                 |
| Software always…          | Integrated and tested           | Deployable to production                    | Deployed to production                      |
| Typical organizations     | Almost all modern teams         | Banks, enterprises, regulated industries    | High‑maturity SaaS (Netflix, GitHub, etc.)  |

**Exam Tip:** In a 10‑mark “compare CI, CD, and Continuous Deployment” question, **draw this table** and then explain at least one example scenario.

---

### 3.5 Implementing Deployment Pipelines

A **deployment pipeline** (also called release pipeline) is an automated sequence that moves code from commit to production through multiple quality gates.[file:5]

**Stages in a Typical Deployment Pipeline:**[file:5]
1. **Source/Commit Stage:** Detect new commits or PR merges in version control.  
2. **Build & Unit Test Stage:** Compile code, run unit tests, static analysis.  
3. **Package & Artifact Stage:** Generate deployable artifact (JAR/WAR, Docker image) and publish to repository.  
4. **Integration Test Stage:** Deploy to test environment; run integration and API tests.  
5. **Staging/UAT Stage:** Deploy to staging environment; run end‑to‑end and performance tests.  
6. **Approval Stage (for CD):** Business/operations approves production release.  
7. **Production Deploy Stage:** Deploy using safe strategies (blue‑green/canary).  
8. **Monitoring & Feedback Stage:** Monitor metrics, logs, and user feedback; roll back if necessary.

**ASCII Deployment Pipeline:**

```
[Commit]
   |
   v
[Build + Unit Tests]
   |
   v
[Package Artifact]
   |
   v
[Deploy to Test] --> [Integration/API Tests]
   |
   v
[Deploy to Staging] --> [E2E + Performance Tests]
   |
   v
[Manual Approval]
   |
   v
[Deploy to Production] --> [Monitor & Rollback if needed]
```

**Key Principles:**[file:5]
- Pipeline as code – entire pipeline defined in config files.  
- Fast, automated feedback after every change.  
- Each stage acts as a **quality gate**.

---

### 3.6 Creating CI/CD Pipelines with Jenkins

Jenkins is a very common tool to implement **CI/CD pipelines** for building, testing, and deploying applications.[file:6][file:7]

**What is a CI/CD Pipeline?**  
A CI/CD pipeline is an **automated workflow** that takes code from commit to build, test, and deployment, reducing manual work and enabling faster and more reliable releases.[file:5]

#### 3.6.1 Jenkins Pipeline Basics

- Jenkins can use **Freestyle jobs** or **Pipeline as Code** (`Jenkinsfile`).[file:7]  
- Integrates with Git for SCM and with Maven, Docker, etc., for build and deployment.[file:6][file:7]  
- Stages commonly used: Build, Test, Package, Deploy to Test, Deploy to Staging, Deploy to Production.[file:5]

**Typical Jenkins CI/CD Flow (from notes):**[file:5]

1. **Developer git push / PR merge → Trigger pipeline** via webhook or polling.  
2. **Stage: Build + Unit Tests** (2–5 minutes).  
3. **Stage: Integration/API Tests** (5–15 minutes).  
4. **Stage: Deploy to Test/QA Environment**.  
5. **Stage: End‑to‑End UI Tests** (10–40 minutes).  
6. **Stage: Deploy to Staging**.  
7. **Manual approval (CD) or auto‑promote (Continuous Deployment)**.  
8. **Deploy to Production** using strategy (canary, blue‑green, feature flags).  
9. **Monitoring & Alerting; automatic rollback if metrics degrade**.[file:5]

**ASCII Jenkins CI/CD Pipeline:**

```
Git Push/PR ---> Jenkins Pipeline
       |
       +--> Stage 1: Build & Unit Tests
       +--> Stage 2: Integration/API Tests
       +--> Stage 3: Deploy to QA
       +--> Stage 4: UI/E2E Tests
       +--> Stage 5: Deploy to Staging
       +--> Stage 6: Manual or Auto Approval
       +--> Stage 7: Deploy to Production
       +--> Stage 8: Monitor & Rollback
```

**Relation to DevOps:**  
This Jenkins pipeline automates the movement from **CI → CD → (optional) continuous deployment**, directly addressing the goals of reliability and fast releases.[file:5][file:6]

---

## 4. Question Bank – Unit 3 (2, 5, and 10 Marks)

### 4.1 2‑Mark Questions

1. Define Continuous Delivery.  
2. Define Continuous Deployment.  
3. What is a deployment pipeline?  
4. What is meant by “software is always in a deployable state”?  
5. Name any two layers in the testing pyramid.  
6. What is the main difference between Continuous Delivery and Continuous Deployment?  
7. Why is automation important for Continuous Delivery?  
8. What is meant by “pipeline as code”?  
9. Name any two strategies for safe production deployment.  
10. Name any two CI/CD tools used in industry.

### 4.2 5‑Mark Questions (Short Notes)

1. Explain Continuous Delivery with key characteristics.  
2. Write a short note on build automation in a CD pipeline.  
3. Explain the testing pyramid in the context of Continuous Delivery.  
4. Write a short note on Continuous Deployment and its requirements.  
5. Explain the difference between CI, Continuous Delivery, and Continuous Deployment.  
6. Describe the main stages of a deployment pipeline.  
7. Explain CI/CD pipelines with a neat diagram.  
8. Write a short note on Jenkins pipelines for CI/CD.  
9. Explain blue‑green deployment and canary release in brief.  
10. Explain “pipeline as code” with example.

### 4.3 10‑Mark Questions (Long Answers)

1. Define Continuous Delivery. Explain its principles, key characteristics, and typical pipeline stages with a neat diagram.  
2. Explain the role of build and test automation in enabling Continuous Delivery. Describe the testing pyramid in detail.  
3. Define Continuous Deployment. How is it different from Continuous Delivery? Explain requirements and deployment strategies.  
4. What is a deployment pipeline? Explain each stage of a typical deployment pipeline and how it reduces risk.  
5. Compare Continuous Integration, Continuous Delivery, and Continuous Deployment with examples and a comparison table.  
6. Explain how to implement a CI/CD pipeline using Jenkins for a web application.  
7. Describe the steps to create a Jenkins pipeline that builds, tests, and deploys an application to staging and production.  
8. Explain the term “pipeline as code” and its advantages in DevOps.  
9. Discuss the importance of monitoring and rollback in Continuous Deployment.  
10. Explain how automated tests at different levels support safe and frequent deployments.

**Most Expected Questions:**  
Q1, Q2, Q3, Q4, Q5, and Q6 are very likely as 10‑mark questions.

---

## 5. Detailed 10‑Mark Answers (Exam‑Oriented)

### Q1. Define Continuous Delivery. Explain its principles, key characteristics, and typical pipeline stages with a neat diagram.

**Introduction:**  
As organizations adopt DevOps, they want to release software frequently but safely. Continuous Delivery addresses this need by ensuring software is always ready for deployment.[file:5]

**Definition of Continuous Delivery:**  
Continuous Delivery (CD) is a software engineering practice where **every code change is automatically built, tested, and prepared so it can be released to production at any time**, with low risk and minimal manual effort.[file:5]

**Principles of Continuous Delivery:**[file:5]
1. **Always Deployable:** The main branch is always in a releasable state.  
2. **Automated Delivery Pipeline:** Build, test, and deployment to staging are automated end‑to‑end.  
3. **Fast Feedback:** Developers receive quick feedback about the quality and deployability of their changes.  
4. **Production‑like Environments:** Staging mirrors production as closely as possible.  
5. **Versioned, Immutable Artifacts:** Build once, then promote same artifact through environments.

**Key Characteristics:**[file:5]
- Every commit passes through CI.  
- Comprehensive automated tests (unit, integration, UI, performance) run automatically.  
- Artifacts (e.g., Docker images) are stored in a repository.  
- Automated deployment to staging/pre‑production environments.  
- Production deployment typically requires a **manual approval gate**.

**Typical CD Pipeline Stages:**[file:5]
1. **Commit Stage:** Detect code changes.  
2. **Build & Unit Test Stage:** Compile and test basic functionality.  
3. **Package Stage:** Create deployable artifact.  
4. **Test Environment Stage:** Deploy to test, run integration/API tests.  
5. **Staging Stage:** Deploy to staging, run end‑to‑end and performance tests.  
6. **Approval Stage:** Business or release manager approves production deployment.  
7. **Production Deploy Stage:** Deploy to live environment using safe rollout strategies.

**ASCII Diagram:**

```
[Commit]
   |
   v
[Build + Unit Tests]
   |
   v
[Package Artifact]
   |
   v
[Deploy to Test] -- Integration Tests
   |
   v
[Deploy to Staging] -- E2E + Performance
   |
   v
[Manual Approval]
   |
   v
[Deploy to Production]
```

**Example (Industry):**  
In an insurance company, every change to the policy management system is automatically built, tested, and pushed to staging. Releases to production are done twice a week after a manager presses a “Deploy” button, confident that the pipeline has validated the build.[file:5]

**Conclusion:**  
Continuous Delivery reduces release risk by turning deployment into a routine, low‑stress activity. It ensures software is always ready to release and aligns technical practices with business needs.[file:5]

---

### Q2. Explain the role of build and test automation in enabling Continuous Delivery. Describe the testing pyramid in detail.

**Introduction:**  
Without strong automation in building and testing, Continuous Delivery is impossible because manual steps are too slow and error‑prone.[file:5]

**Build Automation:**[file:5]
- Each git push or PR merge triggers an automated build in a CI server.  
- Build steps include compilation, static analysis, packaging, and storing artifacts.  
- Goal is fast, repeatable, and reliable creation of deployable artifacts.

**Testing Automation:**[file:5]
- Automated tests run at various levels to validate functionality, integration, and system qualities.  
- Fast feedback encourages frequent commits and safe refactoring.

**Testing Pyramid Explained:**[file:5]

1. **Unit Tests (Base of Pyramid):**  
   - Very fast tests on individual methods/classes.  
   - Written and maintained by developers.  
   - Should form around 60–80% of tests.  

2. **Component/Service Tests:**  
   - Test one microservice or module in isolation, sometimes using mocks or test containers.  
   - Identify issues at service boundaries.  

3. **Integration/Contract Tests:**  
   - Check interaction between services and databases or external APIs.  
   - Contract tests (e.g., Pact) ensure that services respect each other’s APIs.

4. **UI/End‑to‑End Tests:**  
   - Simulate real user flows across the full system via browser or mobile automation.  
   - Slow and flaky if overused; keep their number small.

5. **Performance/Security/Chaos Tests (Top):**  
   - Run less frequently due to resource cost, but critical for production‑like behavior.[file:5]

**ASCII Testing Pyramid:**

```
      [Performance / Security]
         [ UI / E2E Tests ]
      [Integration / Contract]
     [Component / Service Tests]
      [   Unit Tests (Base)   ]
```

**How Automation Enables CD:**[file:5]
- Automated builds and tests reduce human error.  
- Fast, reliable tests make it safe to deploy frequently.  
- Failures are caught early in the pipeline instead of in production.

**Conclusion:**  
Build and test automation form the backbone of Continuous Delivery. The testing pyramid ensures the right balance between speed and coverage, making frequent releases both safe and efficient.[file:5]

---

### Q3. Define Continuous Deployment. How is it different from Continuous Delivery? Explain requirements and deployment strategies.

**Introduction:**  
Continuous Deployment is the next step after Continuous Delivery, where even the final production deployment becomes fully automated.[file:5]

**Definition of Continuous Deployment:**  
Continuous Deployment is a DevOps practice where **every change that passes all automated tests is automatically deployed to production** without any human approval gate.[file:5]

**Difference from Continuous Delivery:**[file:5]
- Continuous Delivery: pipeline prepares software for release; **production deployment is manual**.  
- Continuous Deployment: same pipeline automatically **deploys every successful change to production**.

**Requirements for Continuous Deployment:**[file:5]
- Excellent automated test coverage (unit, integration, E2E).  
- Reliable staging environments matching production.  
- Strong monitoring and alerting (observability).  
- Fast rollback or roll‑forward mechanisms.  
- Organizational buy‑in to frequent, automatic releases.

**Deployment Strategies:**

1. **Blue‑Green Deployment:**  
   - Two identical environments: Blue (current production) and Green (new version).  
   - Traffic is switched from Blue to Green after verification; roll back by switching back.

2. **Canary Releases:**  
   - New version deployed to a small set of users or servers first.  
   - If metrics look good, rollout is gradually increased.

3. **Feature Flags:**  
   - Release code to production but hide features behind toggles.  
   - Allows enabling/disabling features instantly without redeploying.

**Conclusion:**  
Continuous Deployment maximizes automation and release speed but requires very high technical maturity. It builds on Continuous Delivery and adds fully automated production deployment with safe rollout strategies.[file:5]

---

### Q4. What is a deployment pipeline? Explain each stage of a typical deployment pipeline and how it reduces risk.

**Introduction:**  
A deployment pipeline is a key concept in Continuous Delivery – it is the automated path from code commit to production.[file:5]

**Definition:**  
A deployment pipeline is an **automated sequence of stages** (build, test, deploy) that code changes pass through, with each stage increasing confidence and reducing risk before reaching production.[file:5]

**Stages and Risk Reduction:**[file:5]

1. **Source/Commit Stage:**  
   - Detects new commits in version control.  
   - Risk reduction: ensures only tracked changes enter the pipeline.

2. **Build & Unit Test Stage:**  
   - Compiles code and runs fast unit tests.  
   - Risk reduction: catches syntax and basic logic errors quickly.

3. **Package & Artifact Stage:**  
   - Packages code into versioned artifacts and stores them.  
   - Risk reduction: ensures reproducible builds and consistent artifacts.

4. **Integration Test Stage:**  
   - Deploys artifact to a test environment and runs integration/API tests.  
   - Risk reduction: verifies service interactions and database operations.

5. **Staging/UAT Stage:**  
   - Deploys artifact to a production‑like environment.  
   - Runs end‑to‑end and performance tests; sometimes user acceptance tests.  
   - Risk reduction: simulates real production usage.

6. **Approval Stage:**  
   - Business/product owner reviews results and approves production release.  
   - Risk reduction: adds human judgment for high‑impact changes.

7. **Production Deploy Stage:**  
   - Uses strategies like blue‑green or canary to deploy gradually.  
   - Risk reduction: allows quick rollback and limits blast radius.

8. **Monitoring & Feedback Stage:**  
   - Monitors logs, metrics, and user feedback to detect issues quickly.  
   - Risk reduction: enables fast detection and response to production problems.

**ASCII Diagram:**

```
Commit -> Build+Unit -> Package -> Test Env -> Staging -> Approve -> Prod
                         ^                                   |
                         |                                   v
                      Feedback <------------------------- Monitor
```

**Conclusion:**  
By breaking the journey to production into automated, well‑defined stages, deployment pipelines turn risky "big‑bang" releases into a sequence of safe, incremental checks.[file:5]

---

### Q5. Compare Continuous Integration, Continuous Delivery, and Continuous Deployment with examples and a comparison table.

**Introduction:**  
CI, CD, and Continuous Deployment are closely related but distinct DevOps practices. Understanding their differences is vital for exams and real‑world projects.[file:5][file:6]

**Definitions (Short):**[file:5][file:6]
- **CI:** Frequently merging code to a shared repo with automated build and tests.  
- **CD (Delivery):** Ensuring code is always ready to deploy to production.  
- **CDp (Deployment):** Automatically deploying every successful change to production.

**Comparison Table:**[file:5]

| Feature                     | Continuous Integration (CI)              | Continuous Delivery (CD)                            | Continuous Deployment (CDp)                           |
|----------------------------|------------------------------------------|-----------------------------------------------------|-------------------------------------------------------|
| Main Goal                  | Integrate & test changes frequently      | Always keep app deployable                          | Deploy every change automatically                     |
| Scope                      | Build + automated tests                  | CI + automated deploy to staging                    | CD + auto deploy to production                         |
| Production Deploy Trigger  | Out of scope                             | Manual approval                                     | Fully automated                                       |
| Risk Level                 | Reduces integration issues               | Reduces release risk                                | Requires very high automation & confidence           |
| Typical Users              | Almost all teams                         | Most enterprises & regulated industries             | High‑maturity SaaS (e.g., Netflix, GitHub)            |

**Examples:**[file:5]
- CI only: Team runs tests on each commit but deploys manually once a month.  
- CI + CD: Bank keeps app always ready for deploy, but releases only after approvals and change windows.  
- CI + CD + CDp: Streaming service deploys new code to production dozens of times per day automatically.

**Conclusion:**  
CI is the foundation, Continuous Delivery adds automated readiness for release, and Continuous Deployment adds fully automated production releases. Together, they form the backbone of modern DevOps practices.[file:5]

---

### Q6. Explain how to implement a CI/CD pipeline using Jenkins for a web application.

**Introduction:**  
Jenkins is a popular automation server used to implement CI/CD for web applications using tools like Git and Maven.[file:6][file:7]

**Steps to Implement CI/CD Pipeline in Jenkins:**

1. **Set Up Jenkins:**  
   - Install Jenkins and required plugins (Git, Maven, pipeline).  
   - Configure JDK and Maven in Jenkins global settings.[file:7]

2. **Create Jenkins Pipeline Job:**  
   - Use “Pipeline” job type.  
   - Define pipeline using `Jenkinsfile` (pipeline as code).[file:7]

3. **Integrate with Git Repository:**  
   - Configure SCM section with Git URL and branch (e.g., `main`).  
   - Set up webhook from GitHub to trigger pipeline on push/PR.[file:6]

4. **Define Pipeline Stages in Jenkinsfile:**[file:5]
   - `stage('Build')` – run `mvn clean compile`.  
   - `stage('Unit Tests')` – run `mvn test`.  
   - `stage('Package')` – run `mvn package` to create JAR/WAR.  
   - `stage('Deploy to QA')` – deploy artifact to QA server or container.  
   - `stage('Integration Tests')` – run API tests.  
   - `stage('Deploy to Staging')`.  
   - `stage('E2E Tests')`.  
   - `stage('Deploy to Production')` with manual input (for CD) or automatic (for CDp).

5. **Configure Post‑Build Actions:**  
   - Archive artifacts.  
   - Publish JUnit reports.  
   - Notify team via email/Slack on success/failure.[file:6]

**ASCII Jenkins CI/CD Flow:**

```
Git Push --> Jenkins Pipeline
              |
           [Build]
              |
          [Unit Tests]
              |
          [Package]
              |
        [Deploy to QA]
              |
        [Integration]
              |
        [Deploy to Staging]
              |
        [E2E Tests]
              |
        [Deploy to Prod]
```

**Conclusion:**  
By integrating Jenkins with Git and build tools, teams can implement a powerful CI/CD pipeline that automates build, test, and deployment steps, delivering web applications quickly and reliably.[file:5][file:6][file:7]

---

## 6. Viva Questions and Answers (Unit 3)

1. **Q:** What is Continuous Delivery?  
   **A:** It is a practice where every change is automatically built, tested, and kept in a deployable state so it can be released at any time.[file:5]

2. **Q:** What is Continuous Deployment?  
   **A:** It is a practice where every change that passes all automated tests is automatically deployed to production.[file:5]

3. **Q:** What is a deployment pipeline?  
   **A:** An automated sequence of stages (build, test, deploy) that changes go through before reaching production.[file:5]

4. **Q:** Why is testing automation important for CD?  
   **A:** It provides fast, reliable feedback and makes frequent releases safe.[file:5]

5. **Q:** What is meant by “pipeline as code”?  
   **A:** Defining the pipeline (stages, steps) using code or configuration files stored in version control.[file:5]

6. **Q:** Name two tools used to implement CI/CD pipelines.  
   **A:** Jenkins, GitHub Actions, GitLab CI, CircleCI (any two).[file:5][file:6]

7. **Q:** What is blue‑green deployment?  
   **A:** A deployment strategy using two identical environments where traffic is switched from the old to the new version.[file:5]

8. **Q:** What is a canary release?  
   **A:** Releasing a new version to a small subset of users first, then rolling out gradually.[file:5]

9. **Q:** What is meant by “build once, deploy many”?  
   **A:** The same built artifact is promoted across test, staging, and production environments.[file:5]

10. **Q:** How does Jenkins help in Continuous Delivery?  
    **A:** It automates builds, tests, and deployments through pipelines triggered by code changes.[file:5][file:6]

---

## 7. Quick Revision Summary (Unit 3)

- Continuous Delivery (CD) = **always deployable**; production release is a business decision.[file:5]  
- Continuous Deployment = CD + **automatic production deployments** after tests pass.[file:5]  
- Build automation: compile, analyze, package, and store artifacts on each commit.[file:5]  
- Testing pyramid: unit (base) → service → integration → E2E → performance/security.[file:5]  
- Deployment pipeline = automated stages from commit to production with quality gates.[file:5]  
- CI vs CD vs Continuous Deployment: table and scenarios are important for exams.[file:5][file:6]  
- Jenkins is a key tool for implementing CI/CD pipelines using stages and `Jenkinsfile`.[file:6][file:7]

---

## 8. Important Exam Tips for Unit 3

- For 10‑mark questions, write: **definition + key characteristics + diagram + example + conclusion**.  
- Always differentiate clearly between **Continuous Integration, Continuous Delivery, and Continuous Deployment**.  
- Draw the **testing pyramid** and **deployment pipeline** diagrams – easy scoring content.  
- Connect CI/CD pipeline concepts back to DevOps goals: **faster, safer, more reliable releases**.  
- Use keywords such as **deployable state, pipeline, automation, staging, blue‑green, canary, feature flags**.

---

## 9. Most Expected Questions – Unit 3

1. Define Continuous Delivery. Explain its principles and pipeline stages with a neat diagram.  
2. Explain the role of build and test automation in Continuous Delivery.  
3. Define Continuous Deployment and distinguish it from Continuous Delivery.  
4. Explain the concept of a deployment pipeline and its stages.  
5. Compare CI, Continuous Delivery, and Continuous Deployment.  
6. Explain the implementation of a CI/CD pipeline using Jenkins.
