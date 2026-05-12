# DevOps and Automation – Unit 6 Complete Exam Preparation Notes

## 1. Cover Page

**Subject:** DevOps and Automation  
**Unit:** Unit 6 – Docker and Kubernetes  
**Prepared For:** University Semester Exam Preparation  
**Target:** Full Marks (2, 5, and 10 marks questions)  
**Student Level:** Beginner–Intermediate (CS/IT)  

These notes combine: detailed theory, exam-style answers, question bank, revision points, and viva preparation for Unit 6: *Docker and Kubernetes*.[file:1]

---

## 2. Unit Introduction

This unit introduces **Docker** for containerization and **Kubernetes** for container orchestration.[file:1] You will learn what containers are, how they differ from virtual machines, how to create and run Docker images and containers, and how Kubernetes deploys and manages containerized applications at scale.[file:1]

**Syllabus Points (Unit 6):**[file:1]
1. Docker Introduction.  
2. Containers in Docker.  
3. Difference between Virtual Machine and Container.  
4. How to create and run Docker image.  
5. Introduction to Kubernetes.  
6. Deploying and managing applications on Kubernetes.

**Exam Tip:** For this unit, diagrams (Docker architecture, VM vs container, Kubernetes cluster) and simple YAML examples fetch easy marks.

---

## 3. Important Theory Notes – Docker

### 3.1 Docker Introduction

**One‑line Definition:**  
Docker is an **open platform for developing, shipping, and running applications using containerization**.[file:1]

**Detailed Definition (Exam Style):**  
Docker is a containerization platform that lets you package an application and all its dependencies (code, runtime, libraries, system tools, and settings) into a single, portable unit called a **container**.[file:1] This container can run consistently in any environment (development, testing, staging, production), eliminating the classic “it works on my machine” problem.[file:1]

**Why Docker is Needed:**[file:1]
- Before Docker, applications depended on:  
  - OS version.  
  - Libraries and runtimes.  
  - Other system dependencies.  
- When deployed to a different machine, **missing or different versions** caused errors.  
- Docker solves this by **bundling everything** the app needs into an image that runs anywhere Docker is available.[file:1]

**Key Idea:**  
Docker **separates applications from the underlying infrastructure**, so you can deliver software faster and treat infrastructure more like code.[file:1]

---

### 3.2 Docker Architecture

The main components in Docker architecture are the **Docker Client**, **Docker Host (Daemon)**, and **Docker Registry**.[file:1]

**1. Docker Client:**[file:1]
- Command-line interface (`docker` command).  
- Sends REST API requests to Docker Daemon when you run commands like `docker run`, `docker build`.  
- Human entry point into Docker.

**2. Docker Host and Daemon (`dockerd`):**[file:1]
- Machine where Docker Engine runs.  
- Docker Daemon listens for API requests and manages Docker **objects**: images, containers, networks, volumes.

**3. Docker Registry:**[file:1]
- Remote repository for storing and distributing Docker images.  
- Public registry: **Docker Hub** (default).  
- Private registries: Harbor, AWS ECR, Google Artifact Registry, etc.

**High‑Level Interaction Loop:**[file:1]

```
[User / Docker Client]  --->  [Docker Daemon on Host]
          |                          |
          |                    (pull/push)
          |                          v
          +------------------> [Docker Registry]
```

**Image Lifecycle Commands:**[file:1]
- `docker pull <image>` – download image from registry.  
- `docker push <image>` – upload local image to registry.

---

### 3.3 Containers in Docker

**Definition:**  
A Docker container is a **runnable instance of a Docker image** – a lightweight, standalone, executable package that includes application code, runtime environment, system libraries, tools, configuration, and dependencies.[file:1]

**Key Characteristics of Containers:**[file:1]
1. **Self‑contained:** Everything required to run the app is inside the container; minimal reliance on host software.  
2. **Isolated:** Uses Linux kernel features like **namespaces** (for isolation of processes, networks, filesystems) and **cgroups** (for CPU, memory, I/O limits).  
3. **Immutable and Ephemeral:** Containers are usually created from images, run, and then destroyed. Changes inside running containers are not kept unless committed to a new image or stored in volumes.[file:1]
4. **Portable:** Run the same container on laptops, servers, or cloud platforms (AWS, Azure, GCP).  
5. **Scalable:** Easy to start multiple identical containers from a single image for load balancing.[file:1]

**How Containers Work Internally:**[file:1]
- `docker run` takes an image, adds a read‑write layer on top (using a union filesystem like Overlay2), and starts the containerized process.  
- Multiple containers can share the same image layers, saving disk space and improving start up time.

---

### 3.4 Difference Between Virtual Machine and Container

The notes clearly compare VMs and containers.[file:1]

**Virtual Machine (VM) – Definition:**  
A virtual machine is a **software‑based emulation of a physical computer** that runs its own operating system and applications independently, using part of a physical host’s resources.[file:1]

**Container – Short Description:**  
A container is a **lightweight box that carries only your app and its dependencies**, sharing the host OS kernel instead of running a full OS inside.[file:1]

**Comparison Table:**[file:1]

| Feature        | Virtual Machine (VM)                        | Container                                  |
|---------------|----------------------------------------------|--------------------------------------------|
| OS            | Full OS per VM                               | Shares host OS kernel                      |
| Size          | Heavy (GBs)                                  | Lightweight (MBs)                          |
| Startup Time  | Slow (minutes)                               | Fast (seconds)                             |
| Performance   | Slower (more overhead)                       | Faster (near native)                       |
| Isolation     | Strong (hardware + OS level)                 | Moderate (process/kernel level)            |
| Examples      | VMware, VirtualBox, Hyper‑V                  | Docker containers                          |

**ASCII Illustration:**

```
Traditional VMs:
[HW] -> [Host OS] -> [Hypervisor] -> [Guest OS] -> [App]

Containers:
[HW] -> [Host OS + Docker Engine] -> [Container: App+Deps]
```

**Exam Tip:** For a 5 or 10‑mark question, always draw this comparison table and a small architecture sketch.

---

### 3.5 How to Create and Run a Docker Image

The notes give a clear step‑by‑step example for a Node.js app.[file:1]

#### Step 1 – Create a Dockerfile

**What is a Dockerfile?**  
A Dockerfile is a **text file containing instructions** to build a Docker image (base image, copied files, commands to run, etc.).[file:1]

**Example Dockerfile for a Node.js App:**[file:1]

```dockerfile
FROM node:18          # Base image – Node.js environment
WORKDIR /app          # Working directory inside container
COPY . .              # Copy all files into container
RUN npm install       # Install dependencies
CMD ["node", "app.js"] # Command to start app
```

#### Step 2 – Build Docker Image

Command:[file:1]
```bash
docker build -t my-app .
```

- `docker build` – builds an image from Dockerfile.  
- `-t my-app` – tags the image with name `my-app`.  
- `.` – current directory containing Dockerfile.

**What Happens Internally:**[file:1]
1. Docker reads the Dockerfile.  
2. Executes instructions one by one.  
3. Creates layers for each instruction.  
4. Stores the final built image.

Check images:[file:1]
```bash
docker images
```

#### Step 3 – Run Docker Container

Command:[file:1]
```bash
docker run -d -p 3000:3000 my-app
```

- `docker run` – creates and starts a container.  
- `-d` – detached mode (background).  
- `-p 3000:3000` – maps host port 3000 to container port 3000.  
- `my-app` – image name.

**Result:**  
Container is created from `my-app` image; your app runs and is accessible at `http://localhost:3000`.[file:1]

#### Step 4 – Manage Running Containers

Commands:[file:1]
- List running containers: `docker ps`.  
- Stop container: `docker stop <container_id>`.  
- Remove container: `docker rm <container_id>`.

#### Step 5 – Push Image to Docker Hub

Commands:[file:1]
```bash
docker tag my-app username/my-app
docker push username/my-app
```

Others can now pull and run your image.

---

## 4. Important Theory Notes – Kubernetes

### 4.1 Introduction to Kubernetes

**One‑line Definition:**  
Kubernetes (K8s) is an **open‑source container orchestration platform** that automates deployment, scaling, and management of containerized applications.[file:1]

**Detailed Definition (Exam Style):**  
Kubernetes, originally developed by Google and now maintained by CNCF, manages large numbers of containers across many servers by providing features like automatic scaling, self‑healing, load balancing, rolling updates, and configuration management.[file:1]

**Why We Need Kubernetes:**[file:1]
- Running a few containers with Docker is easy; running **hundreds or thousands** across many servers is hard.  
- We need to handle:  
  - Automatic scaling up/down.  
  - Self‑healing (restart failed containers).  
  - Load balancing and networking.  
  - Rolling updates and rollbacks.  
  - Resource allocation and multi‑tenancy.  
  - Secrets and configuration management.  
Kubernetes acts as the **conductor** that coordinates all these container “musicians”.[file:1]

---

### 4.2 High‑Level Kubernetes Architecture

A Kubernetes cluster consists of the **Control Plane** (master) and **Worker Nodes** (data plane).[file:1]

#### 4.2.1 Control Plane Components

1. **API Server (`kube-apiserver`):**[file:1]
   - Central hub exposing Kubernetes API.  
   - All operations (kubectl, controllers, nodes) go through it.

2. **etcd:**[file:1]
   - Distributed key‑value store holding entire cluster state (config, metadata, desired state).  
   - Single source of truth for the cluster.

3. **Scheduler (`kube-scheduler`):**[file:1]
   - Decides on which worker node a new Pod should run, based on resource requirements, constraints, and policies.

4. **Controller Manager (`kube-controller-manager`):**[file:1]
   - Runs controllers (e.g., ReplicaSet, Deployment) that ensure desired state is maintained (e.g., right number of Pods).

5. **Cloud Controller Manager (optional):**[file:1]
   - Handles cloud‑specific logic (AWS, Azure, GCP).

#### 4.2.2 Worker Node Components

1. **Kubelet:**[file:1]
   - Agent running on each node.  
   - Communicates with API server and ensures containers are running as specified.

2. **Kube‑proxy:**[file:1]
   - Manages networking rules for services on the node.  
   - Implements service discovery and load balancing using iptables or IPVS.

3. **Container Runtime:**[file:1]
   - Software that actually runs containers (e.g., containerd, CRI‑O; historically Docker).  
   - Follows Container Runtime Interface (CRI).

4. **Pods:**[file:1]
   - Smallest deployable unit in Kubernetes.  
   - Group of one or more containers that share network namespace and storage.  
   - Usually, one Pod = one application container.

**How It Works Together (Summary):**[file:1]
1. User runs `kubectl apply` to create a Deployment or Pod.  
2. Request goes to API server.  
3. Scheduler picks a node.  
4. Controller Manager ensures desired number of Pods exist.  
5. Kubelet on chosen node pulls container image and starts Pod.  
6. Kube‑proxy sets network rules so traffic reaches the Pod.

---

### 4.3 Deploying and Managing Applications on Kubernetes

The notes use example of deploying an **Nginx web server**.[file:1]

#### Step 1 – Create Deployment YAML (`deployment.yaml`)

**Example:**[file:1]

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 3              # Run 3 copies
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

**Explanation:**[file:1]
- `replicas: 3` – Kubernetes keeps 3 Pods running.  
- If any Pod dies, Deployment controller automatically creates a new one.

#### Step 2 – Create Service YAML (`service.yaml`)

**Example:**[file:1]

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer   # or ClusterIP, NodePort
```

**Explanation:**[file:1]
- `selector: app: nginx` – Service routes traffic to Pods with `app=nginx` label.  
- Service provides a **stable IP and DNS name** even if Pods change.

**Relationship Between Deployment and Service:**[file:1]
- `Deployment.yaml` – creates and manages Pods (actual running app).  
- `Service.yaml` – provides fixed entry point for those Pods and load balances traffic.

#### Step 3 – Apply Manifests with `kubectl`

Commands:[file:1]
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Kubernetes then creates the Deployment, Pods, and Service.

#### Common Management Commands

From the notes:[file:1]
- See Pods: `kubectl get pods`.  
- See Deployments: `kubectl get deployments`.  
- See Services: `kubectl get services`.  
- Detailed info: `kubectl describe pod <pod-name>`.  
- View logs: `kubectl logs <pod-name>`.  
- Scale up/down:  
  `kubectl scale deployment nginx-app --replicas=5`.  
- Delete all resources created by a file:  
  `kubectl delete -f deployment.yaml`.

**Real‑Life Management Flow (from notes):**[file:1]
1. Deploy: `kubectl apply -f ...`.  
2. Check health: `kubectl get pods`.  
3. Scale: increase `replicas` or use `kubectl scale`.  
4. Update: change image tag in YAML and re‑apply; Kubernetes does rolling update.  
5. Rollback: revert to previous version if something breaks.  
6. Monitor: Kubernetes automatically restarts crashed Pods (self‑healing).

---

## 5. Question Bank – Unit 6 (2, 5, and 10 Marks)

### 5.1 2‑Mark Questions

1. What is Docker?  
2. What is a Docker container?  
3. What is a Docker image?  
4. Name any two Docker commands used to manage containers.  
5. Define Kubernetes.  
6. What is a Pod in Kubernetes?  
7. Name any two components of Kubernetes control plane.  
8. What is the difference between `Deployment` and `Service` in Kubernetes?  
9. What is Docker Hub?  
10. Write any two differences between VM and container.

### 5.2 5‑Mark Questions (Short Notes)

1. Explain Docker architecture with a neat diagram.  
2. Write a short note on Docker containers and their key characteristics.  
3. Compare Virtual Machines and Containers.  
4. Explain steps to create and run a Docker image.  
5. Write a short note on Kubernetes cluster architecture.  
6. Explain the role of Pods, Deployments, and Services in Kubernetes.  
7. Explain why Kubernetes is needed when we already have Docker.  
8. Describe common `kubectl` commands used to manage applications.  
9. Explain the image lifecycle commands in Docker (`pull`, `push`, etc.).  
10. Short note: Dockerfile and its purpose.

### 5.3 10‑Mark Questions (Long Answers)

1. Explain Docker and its architecture in detail. Include components like Docker client, host, daemon, and registry.  
2. What is a Docker container? Explain its characteristics and how it works internally.  
3. Compare Virtual Machines and Docker containers with diagrams and a comparison table.  
4. Explain step‑by‑step how to create a Dockerfile, build an image, run a container, and push image to Docker Hub.  
5. Explain Kubernetes architecture. Describe control plane components and worker node components in detail.  
6. What is Kubernetes? Explain why it is used and how it deploys and manages containerized applications.  
7. Describe the steps for deploying an Nginx web server on Kubernetes using Deployment and Service YAML files.  
8. Explain how scaling and rolling updates are handled in Kubernetes.  
9. Discuss how Docker and Kubernetes together support DevOps practices.  
10. Explain with examples how `kubectl` is used to monitor and manage applications in a Kubernetes cluster.

**Most Expected Questions:**  
Q1, Q3, Q4, Q5, Q6, and Q7 are very likely as 10‑mark questions.

---

## 6. Detailed 10‑Mark Answers (Exam‑Oriented)

### Q1. Explain Docker and its architecture in detail.

**Introduction:**  
Docker has become the industry standard for containerization because it simplifies building, shipping, and running applications across different environments.[file:1]

**Definition of Docker:**  
Docker is an open platform that uses containerization to package applications with their dependencies into portable containers that run consistently anywhere.[file:1]

**Key Components of Docker Architecture:**[file:1]

1. **Docker Client:**  
   - CLI used by developers/admins to interact with Docker (`docker run`, `docker build`, etc.).  
   - Translates user commands into REST API calls to the Docker Daemon.

2. **Docker Daemon (`dockerd`):**  
   - Background process running on Docker Host.  
   - Listens for Docker API requests and manages images, containers, networks, and volumes.

3. **Docker Host:**  
   - Physical or virtual machine running the OS, Docker Engine (daemon), local images, and containers.

4. **Docker Registry:**  
   - Storage for Docker images.  
   - Public (Docker Hub) or private registries.

**Interaction Flow (ASCII Diagram):**[file:1]

```
[User] -- docker CLI --> [Docker Client] --REST--> [Docker Daemon]
                                               |
                                               v
                                          [Images & Containers]
                                               |
                                               v
                                         [Docker Registry]
```

**Example Commands:**[file:1]
- `docker pull nginx` – pull image from Docker Hub.  
- `docker run nginx` – run container from image.  
- `docker build -t my-app .` – build an image from Dockerfile.

**Conclusion:**  
Docker’s architecture cleanly separates user commands (client), execution (daemon/host), and image storage (registry), making containerized application workflows efficient and portable.[file:1]

---

### Q2. Compare Virtual Machines and Docker containers with diagrams and a comparison table.

**Introduction:**  
Both VMs and containers provide isolation, but they do so in different ways and with different overhead.[file:1]

**Conceptual Difference:**  
- VM: virtualizes **hardware** and runs a **full guest OS** on top.  
- Container: virtualizes at **OS and process level**, sharing host kernel.

**Architecture Diagrams:**[file:1]

```
Virtual Machines:
+-----------------------------+
|           Hardware          |
+-----------------------------+
|          Host OS            |
+-----------------------------+
|         Hypervisor          |
+-----------------------------+
| Guest OS | Guest OS | ...   |
|  App     |  App     |       |
+-----------------------------+

Containers:
+-----------------------------+
|           Hardware          |
+-----------------------------+
|          Host OS            |
+-----------------------------+
|        Docker Engine        |
+-----------------------------+
| Container | Container | ... |
|  App+Libs |  App+Libs |     |
+-----------------------------+
```

**Comparison Table:**[file:1]

| Feature        | Virtual Machine (VM)                        | Container                         |
|---------------|----------------------------------------------|-----------------------------------|
| OS Layer      | Full guest OS per VM                         | Shares host OS kernel             |
| Size          | Large (GBs)                                  | Small (MBs)                       |
| Startup Time  | Slow (minutes)                               | Fast (seconds)                    |
| Overhead      | High (hypervisor + guest OS)                 | Low                               |
| Isolation     | Strong                                       | Moderate                          |
| Examples      | VMware, VirtualBox, Hyper‑V                  | Docker, containerd                |

**Conclusion:**  
Containers provide a lightweight alternative to VMs for many applications, especially microservices, but VMs are still used when full OS isolation is required.[file:1]

---

### Q3. Explain step‑by‑step how to create a Dockerfile, build an image, run a container, and push image to Docker Hub.

**Introduction:**  
Docker supports a clear workflow from Dockerfile to running container and image distribution.[file:1]

**Step 1 – Create Dockerfile:**[file:1]
- Write instructions like base image, working directory, files to copy, commands to run.  
- Example for Node.js shown earlier.

**Step 2 – Build Image:**[file:1]
```bash
docker build -t my-app .
```
- Docker reads Dockerfile and builds an image `my-app`.

**Step 3 – Run Container:**[file:1]
```bash
docker run -d -p 3000:3000 my-app
```
- Creates container, runs app, maps port.

**Step 4 – Manage Containers:**[file:1]
- `docker ps` – list running.  
- `docker stop <id>` – stop.  
- `docker rm <id>` – remove.

**Step 5 – Push to Docker Hub:**[file:1]
```bash
docker tag my-app username/my-app
docker push username/my-app
```

**Conclusion:**  
These simple steps demonstrate full lifecycle: **build → run → share** using Docker.[file:1]

---

### Q4. Explain Kubernetes architecture and how it deploys and manages containerized applications.

**Introduction:**  
Kubernetes coordinates many containers across nodes, providing high availability and scalability.[file:1]

**Architecture Recap:**[file:1]
- Control Plane: API Server, etcd, Scheduler, Controller Manager, (Cloud Controller Manager).  
- Worker Nodes: Kubelet, Kube‑proxy, Container Runtime, Pods.

**Deployment Flow:**[file:1]
1. Developer writes Deployment YAML and applies it using `kubectl apply -f deployment.yaml`.  
2. API Server receives request; state stored in etcd.  
3. Scheduler chooses a suitable node.  
4. Controller Manager ensures requested number of Pods exist.  
5. Kubelet on target node pulls images and runs containers.  
6. Kube‑proxy sets up networking and load balancing.

**Managing Applications:**[file:1]
- Scaling: `kubectl scale deployment nginx-app --replicas=5`.  
- Rolling Updates: change image version in Deployment YAML and re‑apply; Kubernetes updates Pods gradually.  
- Self‑healing: if a Pod crashes, ReplicaSet controller recreates it.

**Conclusion:**  
Kubernetes architecture provides a robust framework to deploy, scale, and heal containerized applications automatically.[file:1]

---

### Q5. Describe the steps for deploying an Nginx web server on Kubernetes using Deployment and Service YAML files.

**Introduction:**  
Deploying Nginx illustrates the basic Kubernetes workflow for any application.[file:1]

**Step 1 – Create Deployment (`deployment.yaml`):**  
- Defines `kind: Deployment`, `replicas: 3`, container image `nginx:latest`, and container port 80.[file:1]

**Step 2 – Create Service (`service.yaml`):**  
- Defines `kind: Service`, `selector: app: nginx`, port mapping, and type (`LoadBalancer` or `NodePort` or `ClusterIP`).[file:1]

**Step 3 – Apply Configurations:**[file:1]
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

**Step 4 – Verify and Manage:**[file:1]
- `kubectl get pods` – list Pods.  
- `kubectl get services` – view service IP and port.  
- `kubectl scale deployment nginx-app --replicas=5` – scale.

**Conclusion:**  
With just two YAML manifests and a few commands, Kubernetes deploys a scalable, load‑balanced Nginx web service.[file:1]

---

## 7. Viva Questions and Answers (Unit 6)

1. **Q:** What is Docker?  
   **A:** Docker is an open platform for building, shipping, and running applications using containerization.[file:1]

2. **Q:** What is a Docker container?  
   **A:** A lightweight, standalone, executable package that includes an application and all its dependencies.[file:1]

3. **Q:** What is a Docker image?  
   **A:** A read‑only template used to create containers.

4. **Q:** Name any two Docker commands.  
   **A:** `docker build`, `docker run`, `docker ps`, `docker pull` (any two).[file:1]

5. **Q:** What is Kubernetes?  
   **A:** An open‑source container orchestration platform for deploying, scaling, and managing containers.[file:1]

6. **Q:** What is a Pod?  
   **A:** The smallest deployable unit in Kubernetes, usually containing one application container.[file:1]

7. **Q:** What is the function of Kubelet?  
   **A:** It runs on each node and ensures containers are running as expected.[file:1]

8. **Q:** What does `kubectl get pods` do?  
   **A:** Lists the Pods currently running in the cluster.[file:1]

9. **Q:** Why do we need Kubernetes if we already use Docker?  
   **A:** Docker runs individual containers; Kubernetes manages many containers across multiple nodes with features like scaling and self‑healing.[file:1]

10. **Q:** What is Docker Hub?  
    **A:** A public Docker image registry.

---

## 8. Quick Revision Summary (Unit 6)

- Docker: open platform for building, shipping, and running containerized apps.[file:1]  
- Containers: self‑contained, isolated, portable, lightweight, scalable.[file:1]  
- VM vs Container: full OS vs shared kernel; heavy vs light; slow vs fast.[file:1]  
- Docker workflow: Dockerfile → `docker build` → `docker run` → optionally `docker push`.[file:1]  
- Kubernetes: orchestration platform handling deployment, scaling, networking, and healing of containers.[file:1]  
- Kubernetes cluster: control plane (API server, etcd, scheduler, controllers) + worker nodes (kubelet, kube‑proxy, runtime, pods).[file:1]  
- Deploy apps with `Deployment` (Pods management) and `Service` (stable networking) YAMLs + `kubectl apply`.[file:1]

---

## 9. Important Exam Tips for Unit 6

- Always include **architecture diagrams** (Docker and Kubernetes) for 10‑mark questions.  
- Use comparison tables for **VM vs Container**.  
- Show at least one simple **Dockerfile** and `docker build/run` sequence.  
- Show at least one **Kubernetes Deployment and Service YAML** example.  
- Emphasize keywords: **containerization, image, container, registry, Pod, Deployment, Service, control plane, kubelet**.

---

## 10. Most Expected Questions – Unit 6

1. Explain Docker architecture with neat diagram.  
2. Compare Virtual Machines and Containers.  
3. Explain how to create Docker images and run containers using Dockerfile and commands.  
4. Explain Kubernetes architecture and components.  
5. Describe steps to deploy and manage an Nginx application on Kubernetes.
