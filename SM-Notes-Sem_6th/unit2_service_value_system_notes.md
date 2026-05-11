# SERVICE MANAGEMENT – UNIT 2 EXAM PREPARATION NOTES

## 1. Cover Page

**Subject:** Service Management  
**Unit:** Unit 2 – Service Value System (SVS) and Four Dimensions  
**Target:** University Semester Exam – Full Marks Preparation  
**Student Level:** Computer Science / IT (Beginner-Friendly, ITIL‑aligned)

Topics covered (as per syllabus):
- Four dimensions of service management.  
- Service Value System (SVS).  
- SVS guiding principles.  
- Service management automation.  
- SVS governance.  
- Service value chain.  
- ITIL 4 practices.  
- From processes to practices.  
- Process models.[file:2][file:4][file:7][file:11][file:8]

---

## 2. Subject Introduction – Role of Unit 2

Unit 1 introduced basic **service management concepts** such as services, value, cost, risk, utility, warranty, and service relationships.[file:6][file:5]  
Unit 2 now explains **how all these pieces fit together** inside the modern ITIL 4 **Service Value System (SVS)** and how work actually flows through the organization.[file:4]

In simple words, Unit 1 = "What is a service?" and Unit 2 = "How does an entire organization work together to deliver and improve services continuously?"[file:4]

Exam point: Unit 2 is heavily theory‑oriented but with diagrams and frameworks (four dimensions, SVS, value chain). If you draw neat diagrams and write structured explanations, it is easy to score full marks.

---

## 3. Unit 2 Overview (Syllabus Mapping)

**UNIT 2: Service Value System – 5 hours**

1. Four dimensions of service management.  
2. Service Value System (SVS).  
3. SVS guiding principles.  
4. Service management automation.  
5. SVS governance.  
6. Service value chain (six activities).  
7. ITIL practices and how they connect to the value chain.  
8. From processes to practices.  
9. Process models.[file:2][file:4][file:7][file:11][file:8]

You should be able to:
- Draw and explain the **Four Dimensions** diagram.
- Draw and explain the **SVS** diagram (guiding principles, governance, value chain, practices, continual improvement).[file:4]
- Describe the **Seven Guiding Principles** with examples.[file:11][file:4]
- Explain each **Service Value Chain** activity with industry examples.[file:7]
- Distinguish **processes vs practices** and explain why ITIL 4 moved to practices.[file:7][file:2][file:8]
- Explain **service management automation** using DevOps/AI examples.[file:7][file:11]

---

## 4. Important Theory Notes (Concept Explanation Guide)

### 4.1 Four Dimensions of Service Management

#### Definition

The **Four Dimensions of Service Management** provide a holistic framework to ensure that all aspects of service provision are considered:  
1. **Organizations and People**  
2. **Information and Technology**  
3. **Partners and Suppliers**  
4. **Value Streams and Processes**[file:2][file:4]

These dimensions must be balanced and integrated; focusing only on technology or only on processes leads to weak services.

#### ASCII diagram – Four dimensions

```text
          +-------------------------+
          |  Organizations & People |
          +-------------------------+
                     ^
                     |
+--------------------+--------------------+
|                                         |
v                                         v
+-------------------------+   +-------------------------+
|  Information &          |   |  Partners &             |
|  Technology             |   |  Suppliers              |
+-------------------------+   +-------------------------+
                     ^
                     |
                     v
          +-------------------------+
          | Value Streams &         |
          | Processes               |
          +-------------------------+
```

[file:2][file:4]

#### 4.1.1 Organizations and People

- Covers **structure, culture, roles, responsibilities, skills, and capabilities** of people involved in service management.[file:2][file:4]
- Includes leadership style, team organization, communication patterns, and HR aspects.

Example:
- For a college Wi‑Fi service: IT department head, network admins, helpdesk staff, and their skills and responsiveness belong to this dimension.[file:2]

Exam keywords: structure, culture, roles, responsibilities, skills.

#### 4.1.2 Information and Technology

- Covers **information, data, knowledge, applications, and infrastructure** needed to deliver and manage services.[file:2]
- Includes networks, servers, cloud platforms, monitoring tools, CMDB, knowledge bases, dashboards, etc.[file:2]

Example:
- For an e‑commerce site: web servers, databases, CDN, analytics tools, and logs all belong to this dimension.[file:7]

#### 4.1.3 Partners and Suppliers

- Covers **external organizations** that provide products or services which support the service provider’s value creation.[file:2][file:4]
- Involves contracts, SLAs, vendor management, strategic partnerships.

Example:
- Food delivery app using third‑party logistics partners and cloud hosting provider is relying on partners and suppliers.[file:4]

#### 4.1.4 Value Streams and Processes

- Describe **how work flows** through the organization to create and deliver value.
- **Value stream:** End‑to‑end series of steps that create value for a consumer.[file:2]
- **Process:** Set of interrelated activities transforming inputs into outputs.

Example:
- Online admission process from application to enrollment is a value stream; the steps and approvals form processes.[file:7]

#### Why four dimensions matter

- Provide a **holistic** view – ignoring any dimension leads to failure.
- Example: Great technology but poor people skills = bad service; great partners but weak processes = delays.[file:2][file:4]

---

### 4.2 Service Value System (SVS)

#### Definition

The **Service Value System (SVS)** is a model that describes how all components and activities of an organization work together as a system to enable value creation through IT‑enabled services.[file:4]

Main components of SVS:
1. **Guiding Principles.**  
2. **Governance.**  
3. **Service Value Chain.**  
4. **Practices.**  
5. **Continual Improvement.**[file:4]

#### ASCII diagram – SVS overview

```text
           +------------------------------+
           |        Guiding Principles    |
           +------------------------------+
                        |
                        v
+---------------------------------------------------------+
|                      Governance                         |
+---------------------------------------------------------+
                        |
                        v
+---------------------------------------------------------+
|                 Service Value Chain                     |
|   (Plan, Improve, Engage, Design&Transition,            |
|    Obtain&Build, Deliver&Support)                       |
+---------------------------------------------------------+
                        |
                        v
+----------------+   +--------------------+
|   Practices    |   | Continual          |
|  (34 ITIL 4)   |   | Improvement        |
+----------------+   +--------------------+
                        |
                        v
                    Value to Stakeholders
```

[file:4][file:7][file:8]

Key idea: SVS is **not linear** but a **dynamic, interconnected ecosystem** responding to changing needs and opportunities.[file:4]

---

### 4.3 Guiding Principles of SVS (Seven Principles)

Guiding principles are **universal recommendations** that guide decisions and actions in all circumstances, regardless of changes in goals, strategies, or management structures.[file:11][file:4]

#### List of seven guiding principles

1. **Focus on value.**  
2. **Start where you are.**  
3. **Progress iteratively with feedback.**  
4. **Collaborate and promote visibility.**  
5. **Think and work holistically.**  
6. **Keep it simple and practical.**  
7. **Optimize and automate.**[file:11][file:4]

##### 4.3.1 Focus on value

- Every activity must create value for customers and stakeholders.[file:11]
- Ask: who is the customer, what do they value, how is it measured?

Example: Streaming service focuses improvements on buffering reduction because customers value uninterrupted viewing.[file:11]

##### 4.3.2 Start where you are

- Do not ignore existing capabilities; assess current state first.[file:11]
- Reuse working processes, tools, and knowledge to reduce risk and cost.

Example: Bank launching a mobile app reuses existing fraud detection data and processes instead of rebuilding from scratch.[file:11]

##### 4.3.3 Progress iteratively with feedback

- Break work into small, manageable steps; get feedback after each.[file:11]
- Avoid big‑bang projects that fail late.

Example: Retail chain tests same‑day delivery in one city before nationwide rollout.[file:11]

##### 4.3.4 Collaborate and promote visibility

- Break silos, encourage cross‑functional teams, and share information openly.[file:11]

Example: SaaS company uses a shared dashboard during incidents to keep all teams aligned.[file:11]

##### 4.3.5 Think and work holistically

- View services as systems; changes in one part affect others.[file:11][file:4]

Example: Mobile app launch team considers IT, marketing, compliance, and operations together.[file:11]

##### 4.3.6 Keep it simple and practical

- Eliminate unnecessary complexity; simplest solution that works is preferred.[file:11]

Example: Instead of complex approval flow, introduce a single clear owner for each type of change.[file:11]

##### 4.3.7 Optimize and automate

- First simplify and optimize processes, then apply automation to increase efficiency.[file:11]

Example: After standardizing incident categorization, introduce AI chatbots for level‑1 support.[file:7][file:11]

Mnemonic for principles: **FSPCTKO** – Focus, Start, Progress, Collaborate, Think, Keep, Optimize.

---

### 4.4 Governance in the Service Value System

#### Definition

**Governance** in the SVS provides the **direction and control** that ensures service management activities align with organizational objectives and comply with policies and regulations.[file:4]

Key governance functions:
- Setting direction: Vision, strategy, priorities.
- Making decisions: Approving investments, resolving conflicts.
- Monitoring performance: Tracking KPIs and outcomes.
- Risk management and compliance.[file:4]

Example scenario: Food‑delivery platform governance deciding whether to accept a supplier’s demand for higher commission vs. risk to customer satisfaction and profits.[file:4]

ASCII view:

```text
Business Strategy
       |
       v
[ Governance ]  ---> Policies, decision rights, risk appetite
       |
       v
Service Value System Activities
```

Governance ensures that even when practices and teams change, decisions remain consistent and value‑driven.[file:4]

---

### 4.5 Service Value Chain (Six Activities)

The **Service Value Chain** is the **operational core** of the SVS – a set of six inter‑connected activities that transform inputs into value.[file:4][file:7]

Activities:
1. **Plan.**  
2. **Improve.**  
3. **Engage.**  
4. **Design & Transition.**  
5. **Obtain & Build.**  
6. **Deliver & Support.**[file:7]

#### ASCII diagram – simplified value chain

```text
              +-------+
              | Plan  |
              +-------+
                  |
      +-----------+------------+
      v                        v
  +--------+   +--------+   +--------+
  | Engage |-->| Design |-->| Obtain |
  +--------+   | & Tran |   | & Build|
      ^        +--------+   +--------+
      |             \         /
      |              v       /
      |            +-----------+
      |            | Deliver & |
      +------------| Support   |
                   +-----------+
                         |
                         v
                      +------+
                      |Improve|
                      +------+
```

[file:7][file:4]

##### 4.5.1 Plan

- Defines **vision, strategy, policies, and standardization** for all value chain activities.[file:7][file:4]
- Aligns service portfolio and value streams with business objectives.

Example: University IT defines digital strategy, standards for new apps, and KPIs.

##### 4.5.2 Improve

- Ensures **continuous improvement** of products, services, and practices.
- Uses feedback, metrics, and improvement opportunities.[file:7]

Example: After outage, problem management and continual improvement teams implement changes to prevent recurrence.[file:8]

##### 4.5.3 Engage

- Builds and maintains **relationships** with stakeholders.
- Collects requirements, expectations, feedback, and handles communication.[file:7]

Example: Customer support, relationship managers, service desk interacting with users.[file:8]

##### 4.5.4 Design & Transition

- Designs new or changed services and moves them into live environments.[file:7]
- Ensures services meet quality, risk, and performance requirements.

Example: Designing a new online admission module, testing it, and rolling it out.

##### 4.5.5 Obtain & Build

- Acquires or develops **service components**: hardware, software, skills, suppliers.[file:7]

Example: Building a microservice, procuring cloud resources, signing contract with payment gateway.

##### 4.5.6 Deliver & Support

- Day‑to‑day operation and support of services to ensure agreed performance.[file:7]

Example: Incident management, service desk, monitoring, routine maintenance.[file:8]

Important: The value chain is **flexible** – activities can be combined in different sequences depending on the value stream (for example, bug fix vs. new project).[file:7]

---

### 4.6 ITIL 4 Practices – From Processes to Practices

#### 4.6.1 Why move from processes to practices?

Earlier ITIL versions were **process‑centric**, often rigid and heavily documented. ITIL 4 moves to **practices** which are more flexible and holistic.[file:2][file:7][file:8]

**Process approach (old):**
- Linear, sequential workflows.  
- Focus on activity steps only.  
- Siloed processes (Incident vs Change vs Problem).

**Practice approach (ITIL 4):**
- Broader: includes **processes, roles, tools, skills, and information**.[file:8]
- Outcome‑focused and adaptable.
- Supports Agile, DevOps, cloud‑native ways of working.[file:2]

Short definition:  
A **practice** is a set of organizational resources designed for performing work or accomplishing an objective.[file:8]

There are **34 ITIL 4 practices**, grouped into:
- 14 General Management Practices.  
- 17 Service Management Practices.  
- 3 Technical Management Practices.[file:8]

#### 4.6.2 Examples of key practices and their link to value chain

- **Incident management:** Mainly supports **Deliver & Support** and sometimes **Engage**.[file:8]
- **Change enablement:** Strongly used in **Design & Transition** and **Obtain & Build**.[file:8]
- **Service desk:** Central to **Engage** and **Deliver & Support**.[file:8]
- **Service level management:** Supports **Plan, Engage, Deliver & Support**.[file:8]
- **Continual improvement:** Supports **Improve** and all other activities.[file:8]

Exam tip: In questions about SVS and value chain, always mention that **practices feed and support the value chain activities**.

---

### 4.7 Service Management Automation

Automation uses technology to perform tasks with **minimal human intervention**, reducing errors and speeding up service delivery.[file:7][file:11]

Types of automation in service management:
- **Intelligent automation:** Chatbots, virtual agents handling common requests.[file:7][file:11]
- **Process orchestration:** Automated workflows across tools (for example, CI/CD pipelines).[file:7]
- **Predictive analytics:** Machine learning predicting incidents or capacity issues in advance.[file:7]

Benefits:
- Faster response times and 24x7 support.
- Reduced manual errors.
- Frees people to focus on complex tasks.

Example:
- AI‑based virtual agent integrated with service desk to resolve password resets and FAQs.
- Automated incident routing and change deployment using DevOps pipelines.

Important: ITIL emphasizes **"Optimize and then Automate"** – do not automate broken processes.[file:11]

---

### 4.8 Process Models

A **process model** is a visual or textual representation showing steps, inputs, outputs, and roles of a process.

In ITIL and SVS context, process models help to:
- Understand how **value streams and processes** flow through the value chain.[file:2][file:7]
- Identify where practices are applied.
- Find automation opportunities and improvement areas.

Example – simple incident management process model:

```text
[User reports issue]
        |
        v
[Log ticket at Service Desk]
        |
        v
[Classify & prioritize]
        |
        v
[Resolve at L1?] --Yes--> [Close ticket]
        |
       No
        v
[Escalate to L2/L3]
        |
        v
[Implement fix]
        |
        v
[User confirmation & close]
```

This model fits inside the **Deliver & Support** value chain activity and uses the **incident management** practice.[file:7][file:8]

---

## 5. Question Bank with Detailed 10‑Mark Answers (Unit 2)

Below are **12 important 10‑mark questions** for Unit 2, with structured answers.

### Q1. Explain the four dimensions of service management with suitable examples. Why is a holistic view important?

#### Answer Outline

1. Introduction.  
2. List and define four dimensions.  
3. Example for each dimension.  
4. Diagram.  
5. Importance of integration.  
6. Conclusion.

#### Model Answer

**Introduction**  
ITIL 4 defines four dimensions of service management to ensure that services are designed and managed from a holistic perspective, not from a purely technical or process view.[file:2][file:4]

**Four dimensions – definitions**  
1. **Organizations and People:** Structure, culture, roles, responsibilities, and competencies of people involved in service management.[file:2]
2. **Information and Technology:** Data, information, knowledge, applications, and infrastructure required to deliver and support services.[file:2]
3. **Partners and Suppliers:** External organizations that provide products or services supporting service value creation.[file:2][file:4]
4. **Value Streams and Processes:** Activities, workflows, and procedures that create and deliver value to stakeholders.[file:2]

**ASCII diagram**  
(Refer to diagram in section 4.1.)

**Examples for each dimension**  
- **Organizations and People:**  
  - In a college Wi‑Fi service, the IT head, network team, and helpdesk staff with their skills and communication practices belong to this dimension.[file:2]

- **Information and Technology:**  
  - Routers, switches, Wi‑Fi controllers, authentication servers, and the monitoring dashboard are part of this dimension.[file:2]

- **Partners and Suppliers:**  
  - Internet service provider, hardware vendors, and outsourced support company are partners and suppliers.[file:2][file:4]

- **Value Streams and Processes:**  
  - Workflow for "new student Wi‑Fi access" – request, approval, account creation, and configuration – forms a value stream supported by specific processes.[file:2]

**Importance of holistic view**  
- If **organizations and people** are weak (poor skills, no ownership), services fail even with strong technology.
- If **information and technology** are outdated, performance and security suffer.
- If **partners and suppliers** are unmanaged, outages and delays happen.
- If **value streams and processes** are inefficient, users face delays and confusion.[file:4]

**Conclusion**  
The four dimensions ensure that service management considers people, technology, external partners, and workflows together. A holistic view reduces blind spots and improves reliability and value.

**Exam keywords:** four dimensions, holistic view, organizations & people, information & technology, partners & suppliers, value streams & processes.

---

### Q2. Define the Service Value System (SVS). Describe its main components with a neat diagram.

#### Answer Outline

1. Introduction and definition.  
2. List components.  
3. Diagram.  
4. Brief explanation of each component.  
5. Example.  
6. Conclusion.

#### Model Answer

**Introduction**  
ITIL 4 introduces the **Service Value System (SVS)** to describe how all organizational components work together to create value through services.[file:4]

**Definition**  
The **Service Value System** is a model that shows how the activities and components of an organization interact to enable value creation through IT‑enabled services.[file:4]

**Main components of SVS**[file:4]
1. **Guiding Principles.**  
2. **Governance.**  
3. **Service Value Chain.**  
4. **Practices.**  
5. **Continual Improvement.**

**ASCII diagram**  
(Use the diagram from section 4.2.)

**Component explanations**  
- **Guiding Principles:**  
  - Seven high‑level recommendations such as focus on value and start where you are, guiding decisions and actions across the organization.[file:4][file:11]

- **Governance:**  
  - Direction and control function that sets policies, makes key decisions, and ensures alignment with strategy and regulations.[file:4]

- **Service Value Chain:**  
  - Six core activities (Plan, Improve, Engage, Design & Transition, Obtain & Build, Deliver & Support) that operate together to transform demand into value.[file:7]

- **Practices:**  
  - 34 ITIL practices providing guidance and resources (roles, processes, tools) to perform specific types of work like incident management or change enablement.[file:8]

- **Continual Improvement:**  
  - Ongoing activity across the SVS that identifies and implements enhancements to services, practices, and components.[file:4][file:8]

**Example – food delivery platform**  
- Guiding principles: focus on customer value, progress iteratively when launching new features.[file:11]
- Governance: leadership decides on commission structures and quality policies.[file:4]
- Value chain: plan menu strategy, engage restaurants, design new app features, obtain & build microservices, deliver & support operations, improve using feedback.[file:7]
- Practices: incident management for outages, relationship management for partners, service desk for users.[file:8]

**Conclusion**  
SVS provides a complete picture of how an organization’s guidance, governance, operations, and improvement mechanisms interact to deliver continual value to stakeholders.

**Exam keywords:** SVS, guiding principles, governance, service value chain, practices, continual improvement.

---

### Q3. Describe the seven guiding principles of ITIL 4 with suitable industry examples.

#### Answer Outline

1. Introduction – role of guiding principles.  
2. List all seven.  
3. Explain each with 1–2 lines + example.  
4. Conclusion.

#### Model Answer

**Introduction**  
ITIL 4 defines seven **guiding principles** which are universal recommendations that help organizations adopt and adapt service management practices in any situation.[file:11][file:4]

**1. Focus on value**  
- Ensure every activity contributes directly or indirectly to value recognized by stakeholders.[file:11]
- Example: Streaming platform prioritizes low buffering and content quality because those drive subscriber retention.

**2. Start where you are**  
- Assess current state and reuse existing processes, tools, and data instead of building everything from scratch.[file:11]
- Example: Bank launching new mobile app analyzes existing transaction data to identify pain points before designing new flows.[file:11]

**3. Progress iteratively with feedback**  
- Deliver work in small increments, using feedback to adapt and reduce risk.[file:11]
- Example: Retail chain pilots same‑day delivery in one city, measures NPS, then expands to more cities.[file:11]

**4. Collaborate and promote visibility**  
- Break down silos, share information openly, and ensure transparency.[file:11]
- Example: During incident response, SaaS provider uses common war‑room chat and dashboards for all teams to see status.[file:11]

**5. Think and work holistically**  
- View services and organizations as integrated systems; consider all dimensions and how they interact.[file:11][file:4]
- Example: Bank mobile app launch involves IT, operations, marketing, legal, and compliance working together.[file:11]

**6. Keep it simple and practical**  
- Use the simplest solution that achieves objectives; avoid over‑engineering.[file:11]
- Example: Instead of creating multi‑level approval workflow, assign a single change owner with clear criteria.

**7. Optimize and automate**  
- First streamline and simplify processes, then apply automation to reduce manual work and errors.[file:11][file:7]
- Example: After standardizing ticket categories, organization introduces AI chatbot for password resets.

**Conclusion**  
These guiding principles are applied across all practices and value chain activities, helping organizations make better decisions in dynamic real‑world environments.

**Exam keywords:** focus on value, start where you are, progress iteratively, collaborate, holistic, simple, optimize & automate.

---

### Q4. Explain governance in the Service Value System. How does it influence decision making in service management? Give an example.

#### Answer Outline

1. Introduction and definition.  
2. Key functions of governance.  
3. Relation to SVS and value chain.  
4. Example scenario.  
5. Conclusion.

#### Model Answer

**Introduction**  
Governance is a critical component of the Service Value System that ensures alignment between service management activities and organizational objectives.[file:4]

**Definition**  
In ITIL 4, **governance** refers to the means by which an organization is directed and controlled. It includes policies, decision‑making mechanisms, and monitoring frameworks.[file:4]

**Key functions of governance**[file:4]
- **Setting direction:** Establishing vision, strategy, and priorities.
- **Decision rights:** Clarifying who can approve which types of changes and investments.
- **Performance monitoring:** Tracking KPIs, SLAs, and outcomes to ensure objectives are met.
- **Risk management and compliance:** Overseeing major risks and ensuring adherence to regulations and standards.

**Influence on SVS and value chain**  
- Governance decides which **value streams** and projects get funded and prioritized.
- It shapes policies used by practices (for example, change enablement, risk management).[file:4][file:8]
- It defines escalation paths and tolerances for issues.

**Example – Restaurant partner dispute in food delivery platform**  
- Scenario: Key restaurant partner demands higher commission, affecting profitability but also customer satisfaction.[file:4]
- Governance board must decide whether to accept increase, negotiate, or shift to new partners.
- They analyze impact on stakeholders, financial metrics, and long‑term strategy before taking a decision.[file:4]

**Conclusion**  
Governance acts as the steering wheel of the SVS, guiding all value chain activities and practices to stay aligned with strategy, manage risk, and deliver consistent value.

**Exam keywords:** direction and control, decision rights, strategy alignment, risk and compliance, governance scenario.

---

### Q5. Describe the six activities of the Service Value Chain. Explain each with a suitable example.

#### Answer Outline

1. Introduction – service value chain as operating model.  
2. List six activities.  
3. Explain each with example (e‑commerce or admission system).  
4. Mention flexibility.  
5. Conclusion.

#### Model Answer

**Introduction**  
The **Service Value Chain** is the central element of the SVS, representing how an organization converts demand into value through six interconnected activities.[file:4][file:7]

**Six value chain activities**[file:7]
1. Plan.  
2. Improve.  
3. Engage.  
4. Design & Transition.  
5. Obtain & Build.  
6. Deliver & Support.

**1. Plan**  
- Purpose: Ensure a shared understanding of vision, current status, and improvement direction for all four dimensions and products/services.[file:7]
- Example: University defines its three‑year plan for digital learning, Wi‑Fi upgrades, and online exam systems.

**2. Improve**  
- Purpose: Ensure continual improvement of services, practices, and all SVS components.[file:7]
- Example: After analyzing feedback, college IT improves Wi‑Fi login process and expands coverage.

**3. Engage**  
- Purpose: Provide good understanding of stakeholder needs and ensure continual engagement and transparency.[file:7]
- Example: Service desk and relationship managers regularly interact with departments to gather requirements and feedback.

**4. Design & Transition**  
- Purpose: Ensure that products and services meet stakeholder expectations for quality, cost, and time‑to‑market.[file:7]
- Example: Designing a new online admission portal and transitioning it from test to production with proper change management.[file:7]

**5. Obtain & Build**  
- Purpose: Ensure that service components are available when and where needed and meet agreed specifications.[file:7]
- Example: Developing the admission portal, procuring servers or cloud subscriptions, and integrating payment systems.

**6. Deliver & Support**  
- Purpose: Ensure that services are delivered and supported according to agreed specifications and user expectations.[file:7]
- Example: Day‑to‑day running of the admission portal, handling incidents, answering user queries through helpdesk.[file:7][file:8]

**Conclusion**  
These six activities can be combined in various sequences to form value streams. Together, they create a flexible operating model for modern service management.

**Exam keywords:** Plan, Improve, Engage, Design & Transition, Obtain & Build, Deliver & Support.

---

### Q6. What are ITIL 4 practices? Explain the shift from processes to practices with examples.

#### Answer Outline

1. Introduction – ITIL evolution.  
2. Definition of practice.  
3. Difference between process and practice.  
4. Types of practices.  
5. Examples mapping to value chain.  
6. Conclusion.

#### Model Answer

**Introduction**  
Earlier ITIL versions focused mainly on **processes**. ITIL 4 introduces the broader concept of **practices** to better support modern, Agile and DevOps organizations.[file:2][file:8]

**Definition of practice**  
A **practice** is a set of organizational resources designed for performing work or accomplishing an objective.[file:8]

**Processes vs practices**  
- **Process:** Sequence of activities transforming inputs into outputs.
- **Practice:** Includes processes plus **roles, skills, tools, information, and culture** required to achieve outcomes.[file:8]

**Types of practices (34 total)**[file:8]
- **General management practices:** e.g., strategy management, risk management, continual improvement, project management.
- **Service management practices:** e.g., incident management, change enablement, service desk, service level management.
- **Technical management practices:** e.g., deployment management, infrastructure and platform management, software development and management.

**Examples of practices supporting value chain**  
- **Incident management** (service management practice): Supports **Deliver & Support** by restoring normal service operation quickly.[file:8]
- **Change enablement:** Supports **Design & Transition** and **Obtain & Build** by authorizing and scheduling changes while managing risk.[file:8]
- **Service desk:** Supports **Engage** by serving as single point of contact for users.[file:8]

**Why the shift matters**  
- Practices are more flexible and can adapt to Agile, cloud, and DevOps models.
- They encourage thinking about capabilities and culture, not just documented procedures.[file:2][file:7]

**Conclusion**  
By moving from processes to practices, ITIL 4 enables organizations to combine people, tools, and workflows into cohesive capabilities that support the service value chain effectively.

**Exam keywords:** practice definition, 34 practices, general/service/technical, processes vs practices.

---

### Q7. Explain service management automation with reference to the Service Value System and Service Value Chain.

#### Answer Outline

1. Introduction – automation definition.  
2. Types of automation.  
3. Where automation fits in SVS and value chain.  
4. Examples (chatbots, CI/CD, predictive analytics).  
5. Benefits and cautions.  
6. Conclusion.

#### Model Answer

**Introduction**  
Service management automation uses technology to execute tasks in service delivery and support with minimal human intervention, increasing speed and reducing errors.[file:7][file:11]

**Types of automation**  
- **Intelligent automation:** AI chatbots, virtual agents for routine requests.[file:7][file:11]
- **Process orchestration:** Workflow engines coordinating multiple tools across the value chain.[file:7]
- **Predictive analytics:** Machine learning predicting incidents or capacity shortages.[file:7]

**Automation in the SVS**  
- Guided by **Optimize and automate** principle – optimize processes first, then automate.[file:11]
- Applies within practices like incident management, request management, deployment management, monitoring and event management.[file:8]

**Automation across value chain activities**  
- **Plan:** Automated dashboards and reporting for KPIs.
- **Engage:** Chatbots answering user queries and raising tickets.[file:7]
- **Design & Transition:** CI/CD pipelines automating build, test, and deployment.
- **Obtain & Build:** Infrastructure as code provisioning environments.
- **Deliver & Support:** Event correlation tools triggering incident creation and self‑healing scripts.[file:7][file:8]

**Example – online shopping platform**  
- Customer chatbots handle order status and FAQs.  
- Automated workflows update inventory, trigger delivery partner APIs, and send notifications.[file:7]

**Benefits and cautions**  
- Benefits: Faster service, lower manual effort, improved consistency.  
- Caution: Automating a broken process increases damage; must follow "keep it simple" and "optimize and automate" principles.[file:11]

**Conclusion**  
Automation, when aligned with SVS principles and practices, significantly enhances efficiency and quality of service management.

**Exam keywords:** automation, chatbots, orchestration, predictive analytics, optimize then automate.

---

### Q8. With a neat diagram, explain how the Service Value System integrates the four dimensions and ITIL 4 practices to co‑create value.

#### Answer Outline

1. Introduction.  
2. Mention four dimensions + SVS components.  
3. Integrated diagram.  
4. Explanation using example.  
5. Conclusion.

#### Model Answer

**Introduction**  
The Service Value System does not operate in isolation from the four dimensions; together, they provide a complete, integrated view of how value is co‑created.[file:4][file:2]

**Integrated ASCII diagram**

```text
           Four Dimensions
+-------------------------------------------+
| Organizations & People                    |
| Information & Technology                  |
| Partners & Suppliers                      |
| Value Streams & Processes                 |
+-------------------------------------------+
                 |
                 v
       +-------------------------+
       |   Service Value System  |
       |  (Guiding Principles,   |
       |   Governance, Value     |
       |   Chain, Practices, CI) |
       +-------------------------+
                 |
                 v
            Value Co-Creation
```

[file:4][file:2]

**Explanation**  
- The **four dimensions** provide the environment and resources (people, technology, partners, workflows).[file:2][file:4]
- The **SVS** provides structure and flow (principles, governance, value chain, practices, continual improvement).[file:4]
- **Practices** use resources from all dimensions to perform work during value chain activities.[file:8]

**Example – university online learning platform**  
- Organizations & people: faculty, IT staff, students.[file:2]
- Information & technology: LMS platform, video servers, networks.[file:2]
- Partners & suppliers: cloud provider, content vendors.[file:4]
- Value streams & processes: course creation, enrollment, exam conduction.[file:2]
- SVS: guiding principles used to design simple, value‑focused workflows; governance sets digital strategy; value chain executes Plan–Design–Obtain–Deliver for new LMS features; practices like incident management and service desk support daily use.[file:4][file:7][file:8]

**Conclusion**  
By aligning four dimensions with SVS components and practices, organizations build a powerful, adaptable system for continuous value co‑creation.

**Exam keywords:** integration, four dimensions, SVS, practices, value co‑creation.

---

### Q9. Write short notes on: (a) From processes to practices, (b) Value streams and process models in the SVS.

#### Model Points

**(a) From processes to practices**  
- ITIL V2/V3 were **process‑centric**, with defined steps for incident, change, problem, etc.[file:2]
- ITIL 4 expands to **practices** which include processes, roles, tools, skills, data, and culture.[file:8]
- Practices are flexible and support Agile/DevOps and digital transformation.[file:2][file:7]
- Focus is on **outcomes**, not just following procedures.

**(b) Value streams and process models in the SVS**  
- **Value stream:** End‑to‑end sequence of activities that create value for a stakeholder (for example, customer order to delivery).[file:2][file:7]
- **Process model:** Diagram or description showing steps, decisions, inputs/outputs, and roles of a process.
- In SVS, value streams flow through **value chain activities**, supported by various **practices**.[file:7][file:8]
- Process models help identify bottlenecks and automation/improvement opportunities.

---

### Q10. Explain with example how the guiding principles "Keep it simple and practical" and "Optimize and automate" apply to service management automation.

#### Model Answer (10 marks style)

**Introduction**  
Automation is powerful but can fail if applied to overly complex or poorly designed processes. Two ITIL guiding principles are especially relevant: **Keep it simple and practical** and **Optimize and automate**.[file:11]

**Keep it simple and practical**  
- Avoid unnecessary steps, approvals, and technical complexity.[file:11]
- Use clear, easy‑to‑follow workflows that users and staff can understand.

**Optimize and automate**  
- First, **optimize** processes by removing waste and improving flow.  
- Second, **automate** stable and predictable tasks using technology.[file:11][file:7]

**Application to service management automation**  
- Before adding a chatbot, standardize incident categories and resolution scripts (simplify and optimize).
- Automate only those scenarios that are well‑understood, such as password reset or status queries.[file:7][file:8]

**Example – Password reset process**  
1. Original process: User emails IT, ticket manually created, technician resets password, user notified – multiple delays.  
2. Simplification: Standard form with required fields, defined policy for verification (Keep it simple).[file:11]  
3. Optimization: Remove unnecessary approvals, set SLAs, create standard response templates.  
4. Automation: Implement self‑service portal where users answer security questions and reset passwords automatically (Optimize and automate).[file:7]

**Conclusion**  
By following these two principles, organizations avoid automating chaos and instead create efficient, user‑friendly automated services.

**Exam keywords:** keep it simple, optimize and automate, automation example, password reset.

---

### Q11. Discuss how the Service Value Chain activities would map to an online admission system for a university.

#### Model Points

- **Plan:** Define digitalization goals, timelines, and policies for admissions.[file:7]
- **Engage:** Collect requirements from departments and students; communicate schedules and eligibility.[file:7]
- **Design & Transition:** Design application forms, workflows, and integrate payment gateways; test and roll out.[file:7]
- **Obtain & Build:** Develop portal, procure cloud hosting, configure databases.[file:7]
- **Deliver & Support:** Run the system during admission cycle; handle incidents and queries via helpdesk.[file:7][file:8]
- **Improve:** Review issues and feedback to enhance next year’s admission process.

---

### Q12. Common student mistakes and how to avoid them (Unit 2)

**Common mistakes**
- Mixing up **four dimensions** with **SVS components**.  
- Forgetting to mention **all seven guiding principles** or skipping examples.[file:11]  
- Drawing incomplete **value chain diagrams** (missing some activities).[file:7]  
- Writing very generic answers without connecting to ITIL 4 terms like practices, value streams, governance.[file:4][file:8]

**How to avoid**
- Learn one simple mnemonic for the four dimensions and seven principles.  
- Always draw at least one **diagram** in 10‑mark answers (dimensions, SVS, or value chain).  
- Underline ITIL keywords: SVS, guiding principles, value chain, practices, four dimensions.  
- Link theory to **real examples** (Wi‑Fi, food delivery, online shopping, online admission, streaming apps).

---

## 6. Viva Questions (2‑Mark and 5‑Mark Style)

### 6.1 Sample 2‑mark questions

1. List the four dimensions of service management.  
2. Define Service Value System.  
3. Name any four ITIL 4 practices.  
4. What is meant by "value stream"?  
5. State any two guiding principles of ITIL 4.  
6. What is the purpose of the Plan value chain activity?  
7. What do you mean by "governance" in SVS?  
8. Define "practice" in ITIL 4.  
9. Give one example of service management automation.  
10. What is the difference between process and practice (one line each)?

### 6.2 Sample 5‑mark questions

1. Explain the four dimensions of service management.  
2. Describe the components of the Service Value System.  
3. Write short notes on any three ITIL 4 guiding principles.  
4. Explain any three activities of the Service Value Chain.  
5. Short note on "From processes to practices".  
6. Discuss the role of automation in service management.  
7. Explain governance in the context of SVS.

---

## 7. Quick Revision Notes (Last‑Minute)

- **Four dimensions:** Organizations & People, Information & Technology, Partners & Suppliers, Value Streams & Processes.[file:2][file:4]  
- **SVS components:** Guiding Principles, Governance, Service Value Chain, Practices, Continual Improvement.[file:4]  
- **Guiding principles:** Focus on value; Start where you are; Progress iteratively; Collaborate and promote visibility; Think and work holistically; Keep it simple and practical; Optimize and automate.[file:11]  
- **Service value chain activities:** Plan, Improve, Engage, Design & Transition, Obtain & Build, Deliver & Support.[file:7]  
- **Practice:** Organizational resources (people, process, tools) for achieving an objective.[file:8]  
- **Automation:** Use of technology to perform tasks with minimal human intervention – chatbots, CI/CD, self‑service.[file:7][file:11]

Mnemonic suggestions:
- Four dimensions: **OP‑IT‑PS‑VP** = Organizations & People, Information & Technology, Partners & Suppliers, Value Streams & Processes.
- SVS components: **G2P‑CV** = Guiding principles, Governance, Practices, Continual improvement, Value chain.

---

## 8. Important Exam Tips and Most Expected Questions – Unit 2

### 8.1 Exam tips

- Always draw at least one **diagram**: four dimensions, SVS, or value chain.  
- Clearly separate **four dimensions** vs **SVS components** vs **value chain activities**.  
- Use **examples** from cloud, e‑commerce, UPI, food delivery, or university systems.  
- For 10‑mark questions, write: **Introduction → Definition → Main points with headings → Diagram/table → Example → Conclusion.**  
- Underline key terms: SVS, guiding principles, value chain, practices, four dimensions.

### 8.2 Most expected 10‑mark questions for Unit 2

1. Explain the four dimensions of service management with examples.  
2. Define the Service Value System. Explain its components with a neat diagram.  
3. Describe the seven guiding principles of ITIL 4 with suitable examples.  
4. Explain the Service Value Chain and its activities.  
5. Explain the shift from processes to practices in ITIL 4.  
6. Discuss service management automation with examples.  
7. Explain governance in the SVS and its role in decision making.

If you master the above questions using these notes, you can confidently answer any Unit 2 question in the university exam.
