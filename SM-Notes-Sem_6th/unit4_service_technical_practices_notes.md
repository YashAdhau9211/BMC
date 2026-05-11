# SERVICE MANAGEMENT – UNIT 4 EXAM PREPARATION NOTES

## 1. Cover Page

**Subject:** Service Management  
**Unit:** Unit 4 – Service Management and Technical Management Practices  
**Target:** University Semester Exam – Full Marks Preparation  
**Student Level:** Computer Science / IT (Beginner-Friendly, ITIL‑aligned)

**Syllabus topics (Service & Technical management practices):**
- Availability management.  
- Business analysis.  
- Capacity and performance management.  
- Change enablement.  
- Incident management.  
- IT asset management.  
- Monitoring and event management.  
- Problem management.  
- Release management.  
- Service catalogue management.  
- Service configuration management.  
- Service continuity management.

(Plus linkage to deployment management and infrastructure/platform management as technical practices.)[file:8]

---

## 2. Subject Introduction – Role of Unit 4

Units 1–3 covered **concepts, SVS, and general practices**. Unit 4 focuses on **service management and technical practices** that run the day‑to‑day IT operations and projects ("on the ground" work).[file:8][file:7]

These practices largely support the **Deliver & Support, Design & Transition, Obtain & Build, and Improve** activities of the service value chain.[file:7][file:8]

Exam point: Questions from Unit 4 often combine **two or three practices in one scenario** (for example, incident + problem + change). Writing clear definitions plus flow diagrams is a high‑scoring strategy.

---

## 3. Unit 4 Overview (Syllabus Mapping)

**UNIT 4: Service and Technical Management Practices – 10 hours**

Key practices in your syllabus:[file:8]

1. Availability management.  
2. Business analysis.  
3. Capacity and performance management.  
4. Change enablement.  
5. Incident management.  
6. IT asset management.  
7. Monitoring and event management.  
8. Problem management.  
9. Release management.  
10. Service catalogue management.  
11. Service configuration management.  
12. Service continuity management.

You should be able to:
- Define each practice in **one exam line**.  
- Write purpose, 3–5 activities, and at least one realistic example.  
- Draw small process flows or relationship diagrams.

---

## 4. Core One‑Line Definitions (Memory Boost)

- **Availability management:** Ensures services meet agreed availability levels by analyzing performance and planning improvements.[file:8]
- **Business analysis:** Identifies business needs and recommends solutions that deliver value to stakeholders.[file:8]
- **Capacity and performance management:** Ensures services achieve agreed performance and capacity in a cost‑effective manner.[file:8]
- **Change enablement (change control):** Maximizes the number of successful changes by assessing and controlling risk.[file:8]
- **Incident management:** Minimizes negative impact of incidents by restoring normal service operation as quickly as possible.[file:8]
- **IT asset management:** Plans and manages the full lifecycle of IT assets to optimize value, control costs, and manage risks.[file:8]
- **Monitoring and event management:** Observes services and components to record events and determine appropriate actions.[file:8]
- **Problem management:** Reduces likelihood and impact of incidents by identifying and managing their root causes.[file:8]
- **Release management:** Makes new or changed services available for use with minimal disruption.[file:8]
- **Service catalogue management:** Provides a single source of consistent information on all operational services and those being prepared for operation.[file:8]
- **Service configuration management:** Ensures accurate information about configuration items (CIs) and their relationships is available when needed.[file:8]
- **Service continuity management:** Ensures the organization can continue service delivery at agreed levels after a disruption.[file:8]

---

## 5. Detailed Theory Notes – Each Practice

### 5.1 Availability Management

#### Definition

**Availability management** ensures that services deliver the agreed levels of availability to meet the needs of customers and users.[file:8]

#### Key points

- Focuses on **uptime, reliability, and maintainability**.  
- Analyzes historical incident data and performance metrics.  
- Works closely with capacity management, continuity management, and monitoring.[file:8]

#### Example

For a digital banking app with a 99.9% availability SLA, availability management plans redundancy (active‑active data centers), uses monitoring tools to detect outages quickly, and works with problem management to prevent recurring failures.[file:8]

#### ASCII view – high‑level

```text
[Monitoring data] --> [Availability analysis] --> [Improvement plan]
         ^                     |                        |
         |                     v                        v
   Incident data        Design changes          Updated SLAs
```

#### Exam‑style conclusion

Availability management ensures that customers can actually use services when needed, directly influencing customer satisfaction and trust.

**Keywords:** availability, uptime, SLA, reliability, redundancy.

---

### 5.2 Business Analysis

#### Definition

**Business analysis** identifies business needs and recommends value‑adding solutions that facilitate outcomes for stakeholders.[file:8]

#### Key points

- Elicits and documents **requirements** for new or changed services.  
- Conducts **process analysis** and **impact assessment**.  
- Bridges the gap between business and IT.

#### Example

Before building an online admission system, business analysts interview departments, map existing manual processes, and produce use‑cases and user stories, ensuring the solution actually meets user needs.[file:7]

#### Exam‑style conclusion

Business analysis ensures that technical solutions are aligned with real business problems and do not become "IT for IT’s sake".

**Keywords:** requirements, stakeholder needs, solution options, value.

---

### 5.3 Capacity and Performance Management

#### Definition

**Capacity and performance management** ensures that services achieve agreed and expected performance, handling current and future demand cost‑effectively.[file:8]

#### Key points

- Monitors **resource utilization and response times**.  
- Forecasts future demand and plans scaling.  
- Balances **cost vs performance**.[file:8]

#### Example

E‑commerce site monitors CPU, memory, and response time over festive seasons, then increases capacity (auto‑scaling) during peak hours to avoid slowdowns.[file:8]

#### Exam‑style conclusion

This practice prevents both **over‑utilization** (slow services) and **over‑provisioning** (wasted cost), maintaining good user experience.

**Keywords:** performance, capacity planning, forecasting, scalability.

---

### 5.4 Change Enablement (Change Control)

#### Definition

**Change enablement** ensures that risks are properly assessed, authorizes changes to proceed, and manages the change schedule to maximize successful outcomes and minimize disruption.[file:8]

#### Key points

- Evaluates **risk, impact, and benefit** of each change.  
- Uses change advisory structures (e.g., CAB for high‑risk changes).  
- Coordinates with release management, configuration management, and deployment management.[file:8]

#### ASCII – simplified flow

```text
[Change request]
      |
      v
[Assess risk & impact]
      |
      v
[Approve?] --No--> [Reject/Defer]
      |
     Yes
      v
[Plan & schedule] --> [Implement] --> [Review & close]
```

[file:8]

#### Example

Before migrating a database to a new version, a change record is raised, impact analysis performed, CAB approves, and implementation is scheduled during low‑usage window with rollback plan.

#### Exam‑style conclusion

Change enablement balances **innovation and stability** by controlling changes without stopping progress.

**Keywords:** risk assessment, authorization, CAB, change schedule.

---

### 5.5 Incident Management

#### Definition

**Incident management** minimizes the negative impact of incidents (unplanned interruptions or reductions in quality of services) by restoring normal service operation as quickly as possible.[file:8]

#### Key points

- Focus is on **restoration, not root cause** (that is handled by problem management).  
- Uses priority (impact × urgency) to allocate resources.[file:8]  
- Often driven through **service desk** and monitoring alerts.[file:8]

#### ASCII – basic incident flow

```text
[Detect incident] --> [Log] --> [Classify & prioritize]
        |                          |
        v                          v
   [Initial diagnosis] ----> [Escalate if needed]
        |                          |
        v                          v
    [Resolve] ----------------> [Close & document]
```

[file:8]

#### Example

A UPI payment outage is detected through alerts and user calls; incident team quickly applies workaround by routing traffic to backup systems, restoring service while root cause is investigated later.

#### Exam‑style conclusion

Incident management directly protects business operations by focusing on **speedy recovery** and user communication.

**Keywords:** incident, restore, priority, service desk, workaround.

---

### 5.6 IT Asset Management

#### Definition

**IT asset management** plans and manages the full lifecycle of IT assets (hardware, software, cloud subscriptions, etc.) to maximize value, control costs, manage risks, and support decision‑making.[file:8]

#### Key points

- Covers **acquisition, deployment, maintenance, and disposal** of assets.  
- Tracks compliance with licenses and contracts.  
- Integrates with configuration management (CIs may reference assets).[file:8]

#### Example

IT asset management tracks laptops, servers, and SaaS licenses; unused licenses are reclaimed, and end‑of‑life hardware is securely disposed.

#### Exam‑style conclusion

Effective IT asset management reduces waste, supports budgeting, and avoids legal risk from license violations.

**Keywords:** lifecycle, inventory, license compliance, optimization.

---

### 5.7 Monitoring and Event Management

#### Definition

**Monitoring and event management** systematically observes services and service components, records and evaluates events, and determines the appropriate response.[file:8]

#### Key points

- **Event:** any detectable change of state that has significance (information, warning, or exception).  
- Monitoring tools generate events; event management decides which events become **incidents or alerts**.[file:8]

#### ASCII – simple view

```text
[Monitoring tools] --> [Events]
                         |
                         v
                 [Event filtering]
                         |
       +-----------------+----------------+
       v                                  v
  [Informational]                 [Alert/exception]
       |                                  |
   Logged only                     -> Incident/problem
```

[file:8]

#### Example

CPU usage above 90% for 10 minutes triggers a warning event; if it continues, it creates an incident for capacity/performance team.

#### Exam‑style conclusion

Monitoring and event management are the **eyes and ears** of service management, enabling proactive detection before users complain.

**Keywords:** events, monitoring, thresholds, alerts, correlation.

---

### 5.8 Problem Management

#### Definition

**Problem management** reduces the likelihood and impact of incidents by identifying actual and potential causes and managing solutions.[file:8]

#### Key points

- A **problem** is the cause or potential cause of one or more incidents.  
- Includes **reactive** (after incidents) and **proactive** (before incidents) analysis.[file:8]  
- Produces **workarounds** and **known error records**.

#### Relationship with incident management

- Incident management: "fix it fast" (restore service).  
- Problem management: "fix it right" (find and remove root cause).

#### Example

Recurring outages of an app lead to a problem record; analysis reveals memory leaks in code; permanent fix is deployed through change and release management.

#### Exam‑style conclusion

Problem management turns repeated firefighting into long‑term stability and reliability.

**Keywords:** root cause, known error, workaround, proactive/reactive.

---

### 5.9 Release Management

#### Definition

**Release management** makes new and changed services and features available for use in a controlled way that protects existing services.[file:8]

#### Key points

- Plans **release calendar** and coordinates with change and deployment management.  
- Ensures testing, documentation, and rollback options exist.[file:8]

#### ASCII – release lifecycle

```text
[Approved change] --> [Build & integrate]
        |
        v
     [Test] --> [Deploy to production] --> [Review]
```

[file:8]

#### Example

A new version of a mobile app is bundled with backend changes; release management coordinates development, testing, production deployment, and post‑release monitoring.

#### Exam‑style conclusion

Release management reduces risk and chaos when introducing new functionality into live environments.

**Keywords:** release plan, calendar, testing, deployment, rollback.

---

### 5.10 Service Catalogue Management

#### Definition

**Service catalogue management** provides a single, consistent source of information on all services and service offerings in operation and those being prepared for operation.[file:8]

#### Key points

- Maintains **service catalogue** accessible to customers and support teams.  
- Each entry includes description, target users, SLAs, costs, and request options.[file:8]

#### Example

An IT self‑service portal lists services like "email account", "VPN access", "new laptop", each with details and request forms.

#### Exam‑style conclusion

Service catalogue management increases transparency and enables users to understand what services are available and how to request them.

**Keywords:** catalogue, service offerings, portal, single source of information.

---

### 5.11 Service Configuration Management

#### Definition

**Service configuration management** ensures that accurate and reliable information about configuration items (CIs) and their relationships is available when and where needed.[file:8]

#### Key points

- Maintains **Configuration Management Database (CMDB)** or configuration model.  
- CIs include servers, applications, network devices, docs, and relationships.[file:8]
- Vital for impact analysis in change and incident management.

#### ASCII – relationship example

```text
[Web App] --> [App Server] --> [Database] --> [Storage]
      |              |
   depends on     runs on
```

[file:8]

#### Example

Before approving a change on a database server, change enablement checks CMDB to see which applications and services depend on it.

#### Exam‑style conclusion

Service configuration management gives visibility into the technical landscape, enabling better decisions and faster troubleshooting.

**Keywords:** CI, CMDB, relationships, configuration model.

---

### 5.12 Service Continuity Management

#### Definition

**Service continuity management** ensures that the organization can continue to deliver services at agreed levels following a disruption.[file:8]

#### Key points

- Aligns with **business continuity management**.  
- Activities: business impact analysis (BIA), risk assessment, continuity plans, testing and exercises.[file:8][file:3]

#### Example

For a payment platform, continuity management designs active‑active architecture with failover to secondary region and conducts regular DR drills simulating major outages.[file:3][file:8]

#### Exam‑style conclusion

Service continuity management prepares organizations for worst‑case scenarios, minimizing downtime and data loss.

**Keywords:** BIA, DR (disaster recovery), RTO/RPO, continuity plans.

---

## 6. 10‑Mark Question Bank (Unit 4 – Model Structures)

Below are **10 high‑probability 10‑mark questions** with structured points.

### Q1. Explain incident management and problem management. How do they differ and how do they work together?

**Model points:**
- Definitions and purpose of each.[file:8]  
- Incident = restore quickly; Problem = root cause and prevention.  
- Simple comparison table: focus, time horizon, metrics.  
- Example chain: recurring outage → incidents handled quickly, then problem management finds root cause and change enables permanent fix.  
- Conclusion: both are complementary and essential.

---

### Q2. Describe change enablement with a neat flow diagram. Why is it critical for stable IT services?

**Model points:**
- Definition and objectives.[file:8]  
- Steps: raise → assess → authorize → plan → implement → review.  
- Mention CAB for major changes.  
- Example: database upgrade.  
- Explain link to risk, availability, and continuity.

---

### Q3. Explain availability management and capacity & performance management. How do they support each other?

**Model points:**
- Definitions.[file:8]  
- Availability: uptime & reliability; Capacity/performance: resources to achieve performance.  
- Examples from e‑commerce/banking.  
- Explain how insufficient capacity reduces availability; well‑planned capacity supports availability SLAs.  
- Conclude on joint planning and monitoring.

---

### Q4. What is service configuration management? Explain its role in change and incident management.

**Model points:**
- Definition and CMDB.[file:8]  
- CIs and relationships.  
- Use in change impact analysis.  
- Use in incident root cause identification.  
- Example diagram with app → server → DB.

---

### Q5. Explain monitoring and event management with an example from a cloud application.

**Model points:**
- Definition and event types (informational, warning, exception).[file:8]  
- Integration with monitoring tools and logging.  
- Example: CPU, memory, HTTP error rate; automatic incident when thresholds breached.  
- Benefits: early detection, automation opportunities.

---

### Q6. Describe service catalogue management and IT asset management. How do they differ?

**Model points:**
- Service catalogue = list and details of services.[file:8]  
- IT asset management = lifecycle of hardware/software assets.[file:8]  
- Table contrasting "service" vs "asset" focus.  
- Example: portal listing VPN vs inventory of laptops and licenses.

---

### Q7. Explain release management with suitable example. How is it related to change enablement and deployment management?

**Model points:**
- Definition and goals.[file:8]  
- Release = package of changes delivered together.  
- Relationship: change approves, release bundles, deployment installs.  
- Example: new version rollout for mobile app and backend.

---

### Q8. What is service continuity management? Describe key steps in planning for continuity.

**Model points:**
- Definition and link to business continuity.[file:8][file:3]  
- Steps: BIA, risk assessment, define RTO/RPO, design DR solutions, test regularly.  
- Example: multi‑region deployment, regular DR drills.

---

### Q9. Short notes (any two): (a) Business analysis, (b) Capacity and performance management, (c) Problem management.

**Model points:** definitions, 3–4 activities each, and examples from digital services.[file:8]

---

### Q10. Discuss how incident management, problem management, and change enablement together prevent large‑scale outages.

**Model points:**
- Explain each practice briefly.[file:8]  
- Describe lifecycle during an outage: detect & restore (incident), analyze root cause (problem), implement fix under control (change).  
- Example: global cloud outage mapped to ITIL practices.

---

## 7. Viva / Short‑Answer Question Ideas

### 7.1 Sample 2‑mark questions

1. Define incident and give one example.  
2. What is a configuration item (CI)?  
3. What is the main purpose of release management?  
4. Define service catalogue.  
5. State what is meant by "availability" in service management.  
6. What is a known error?  
7. Define service continuity management.  
8. What is an event in monitoring and event management?  
9. Differentiate between incident and problem (one line each).  
10. What is capacity planning?

### 7.2 Sample 5‑mark questions

1. Explain the steps in incident management.  
2. Short note on IT asset management.  
3. Describe the role of service catalogue management.  
4. Explain the relationship between monitoring, incidents, and problems.  
5. Short note on business analysis and release management.

---

## 8. Quick Revision Notes (Last‑Minute)

- **Incident vs problem:** incident = restore quickly; problem = investigate and fix root cause.[file:8]  
- **Change enablement:** control and approve changes, manage risk, use CAB for major changes.[file:8]  
- **Availability & capacity:** availability = uptime; capacity/performance = resources; they are inter‑dependent.[file:8]  
- **Monitoring & events:** monitoring tools → events → some events → incidents/problems.[file:8]  
- **Service catalogue vs configuration:** catalogue = list of services; configuration = details of components (CIs) and relationships.[file:8]  
- **Continuity:** plan for disasters so that services continue at minimum agreed level.[file:8][file:3]  
- **IT asset management:** track hardware/software/cloud assets across lifecycle.[file:8]  
- **Release management:** controlled introduction of new/changed services.[file:8]

Mnemonics: for main practices **AB C2I2 M2 P2 R2 S3**  
A – Availability, B – Business analysis,  
C2 – Capacity & performance, Change enablement,  
I2 – Incident, IT asset,  
M2 – Monitoring & event, (Service) catalogue Management,  
P2 – Problem, Performance (capacity),  
R2 – Release, (Service) configuration,  
S3 – Service continuity, Service catalogue, Service configuration.

---

## 9. Exam Tips and Most Expected Questions – Unit 4

- Always **pair practices with examples**: UPI outage, e‑commerce peak sale, cloud migration.  
- For 10‑marks, use structure: **Introduction → Definition → Main points (with bullets) → Diagram/flow → Example → Conclusion.**  
- Draw small ASCII or box diagrams for incident/problem flow, change flow, CMDB relationships, and continuity architecture.  
- Underline ITIL terms: incident, problem, known error, CI, CMDB, SLA, CAB, DR.

**Most expected 10‑mark questions:**
1. Incident vs problem management with diagram and example.  
2. Explain change enablement and release management.  
3. Describe availability and capacity management with examples.  
4. Explain service configuration and service catalogue management.  
5. Discuss service continuity management steps.

If you thoroughly prepare these notes and practise full‑length answers, you will be exam‑ready for all Unit 4 questions.
