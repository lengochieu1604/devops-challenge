# Notion format for this task
+ https://defiant-august-ce6.notion.site/Problem-2-Building-Castle-In-The-Cloud-1a285b7592a68100b240f05496c22dee

# Problem 2: Building Castle In The Cloud
<aside>
⏰ Duration: You should not spend more than **12 hours** on this problem.
*Time estimation is for internship roles, if you are a software professional you should spend significantly less time.*

</aside>

# Task

Architecture a **highly available trading system** with similar features to the Binance trading platform [https://www.binance.com](https://www.binance.com/en). This system will need to be **resilient** to **failures**, **scalable**, and **cost-effective**. You should **design** the architecture, choose the **appropriate technologies**, and **explain** your approach for maintaining **high availability and scalability.** 

Due to time constraint, you will not be able to cover every feature on the reference platform, so choose some features that will help you demonstrate your mastery of cloud infrastructure. 

### Deliverables

Your submission should include:

- An overview **diagram** of the **services used** and what **role** they play in the system.
- **Elaboration** on why each cloud service is used and what are the **alternatives** considered.
- **Plans for scaling** when the product grows beyond your current setup.

### Specifications

You are limited to the following constraints for your plan.

| Cloud Provider | Amazon Web Services (**AWS**) |
| --- | --- |
| Throughput | **500 RPS** (requests per second) |
| Response Time | p99 response time of <100ms |

# Analysis

## **System component**

- Environment: DEV, QAC, STG, PRD
- Programming:
    - Fe: React
    - Be: Node.js
- Network
    - CDN: Cloud Front + S3
    - Domain: Route53
    - Network: VPC (03 layers), InternetGW, NATGW, ElasticIP, PublicIP, etc.
- Active Directory: AWS Cloud Diretory
- SVC: GitLab server
- CICD
    - CI: GitLab CI
    - CD: ArgoCD
- Microservices:
    - API gateway: AWS APIGW
    - Front-end
        - Portal: EKS
    - Back-end
        - Message broker: AWS S3 - SNS - SQS
        - Business logic: EKS
            - SSO, Sign up, Login, Logout, Forget password, trading view portal, crypto transaction, deposit (crypto, money, P2P), withdraw money, statistics (profit, asset, etc), asset management (earn, spot, futures, wallet, etc), trade, futures, notification, news, support service, refers, events, gift & campain, etc.
        - Notification: AWS SES
    - Database
        - SQL: AWS RDS (MySQL, Postgres, etc)
        - NoSQL: Amazon DocumentDB, AWS DynamoDB
    - Storage
        - VM disk
        - Metadata: AWS S3
    - Cache: ElastiCache

## Deployments strategy

- Rolling
- Blue/Green
- Canary

## Monitoring & Logging & Alerting

- Monitoring: Prometheus, Grafana, etc
- Logging: Loki, Promtail
- Alerting: Alert Manager

## Security

- DevSecOps
- AWS WAF, AWS Shield, Security Group, NACL, AppArmor, Kube-bench, KubeSec, etc

## Application flow

- Traffic
    - External → Application
        - End user → Domain → AWS Route53 → AWS CloudFront → AWS WAF → VPC → Internet GW Public subnet → AWS ALB → NAT → Private subnet → Ingress → K8s cluster ↔ DB
    - Internal
        - Ingree ↔ K8s cluster → Portal → Message broker → Business logic → DB

## Failures strategy

https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html

- Backup & Restore
- Pilot light
- Warn standby
- Multi site active/active

## Scalable

- Horizontal Scaling
- Database Scaling
- Load Balancing
- Asynchronous Processing
- Edge Caching

## Cost-effective

- Cost components
    - Resources cost
    - Traffic outbound
    - Operation cost
- Cost estimation report
- Highly available
    - Multi zone

# Solution: Trading System Architecture Microservices on AWS EKS cluster

This architecture provides a **resilient, highly available, scalable, and cost-effective** trading platform similar to Binance, hosted on **AWS**. The design leverages **microservices, containerization, messaging systems, and scalable databases** to handle **high transaction volumes** with **low latency (<100ms response time at p99), resilience against failures, and satisfy within 500 RPS**.

## **1. Architecture**

### **Architecture Diagram**

Here is the high-level **architecture diagram** for the trading system on AWS. It visually represents the key components, network flow, and AWS infrastructure used for **scalability, resilience, and cost efficiency**. Divided to four environments: **DEV** (Development), **QAC** (Quality Assurance), **STG** (Staging), and **PRD** (Production).

**STG/PRD Architecture**

- Zone 1 has **more worker nodes (three nodes just for the illustration)** compared to DEV/QAC.
- This aligns with the **high availability (HA) and performance** requirement for staging and production environments.
<img width="856" alt="image" src="https://github.com/user-attachments/assets/e7f9df41-00a6-409c-9484-2ae6d3a5659f" />

**DEV/QAC Architecture**

- Zone 1 has **fewer worker nodes** compared to STG/PRD.
- This aligns with **cost optimization** while still maintaining some level of HA.
<img width="856" alt="image" src="https://github.com/user-attachments/assets/3bccdae5-ddec-430d-a4a9-99f53d46f061" />

### Architecture components and explaination

| **Category** | **Technology Stack** | **Purpose** | **Alternatives Considered** |
|-------------|----------------------|------------|----------------------------|
| **Cloud Provider** | AWS | Provides global scalability, redundancy, and cost efficiency. | Azure / GCP |
| **Environments** | DEV, QAC, STG, PRD | Isolated environments for development, testing, pre-production, and production. | N/A |
| **Source Version Control (SVC)** | GitLab Server | Provides private repositories, issue tracking, and wikis. | GitHub Server |
| **Programming** | - **Frontend**: React <br> - **Backend**: Node.js | Frontend for user interfaces <br> Backend for trading logic | N/A |
| **CI/CD** | - **CI**: GitLab CI <br> - **CD**: ArgoCD | Automated CI and CD to build an application image and deploy to container orchestration. | - **CI**: GitHub Actions <br> - **CD**: Harness CD |
| **Active Directory** | AWS Directory Service for Microsoft Active Directory | Create directories for organizational charts, device registries, SSO, and identity management. | N/A |
| **Network & Domain** | AWS VPC (Three layers), Internet GW, NAT GW, Elastic IP, Route 53, etc. | Isolated and secure networking architecture. | CloudFlare |
| **CDN** | AWS CloudFront | Securely deliver content with low latency and high transfer speeds. | CloudFlare |
| **Load Balancing** | AWS ALB | Ensures even request distribution and load balancing. | AWS NLB |
| **API Gateway** | AWS API Gateway | Reduces latency and improves API response times with rate-limiting. | Kong Gateway |
| **Microservices** | Kubernetes (EKS) | Business logic services such as SSO, sign-up, login, logout, trading view portal, crypto transactions, deposits (crypto, money, P2P), withdrawals, statistics, asset management, trade, futures, notifications, news, support, referral programs, events, and campaigns. | ECS Fargate type |
| **Message Broker** | AWS SNS + SQS | Asynchronous communication for order execution and notifications with decoupled event-driven communication. | Kafka |
| **Storage** | AWS S3 | Stores metadata, trading logs, and historical records. | MinIO |
| **Databases** | - **SQL**: AWS RDS (PostgreSQL) <br> - **NoSQL**: Amazon DocumentDB | - **SQL**: Transactional data storage <br> - **NoSQL**: High-speed lookups for orders & balances | - **SQL**: Self-managed PostgreSQL on EC2 <br> - **NoSQL**: MongoDB Atlas |
| **Caching** | AWS ElastiCache (Redis) | Caches frequent queries to reduce DB load. | Redis |
| **Monitoring** | AWS CloudWatch, AWS QuickSight | Observability and real-time performance tracking. | Prometheus, Grafana, Loki |
| **Logging** | AWS CloudWatch | Aggregates logs. | Promtail, Loki |
| **Notification** | AWS SES | Manages and triggers alerts. | Alert Manager |
| **Security** | AWS WAF, AWS Shield, IAM, Kube-bench, Kube-sec, KubeLinter, AppArmor, Cilium, Istio, Falco, seccomp | Protection against DDoS and unauthorized access. | Nginx ModSecurity WAF |

**Constrains**

| Throughput | **500 RPS** (requests per second) |
| --- | --- |
| Response Time | p99 response time of <100ms |

**List services and pricing plan that is suitable for the above constrains**

| **Services** | **Default RPS can be processed** | **Pricing Plan** |
| --- | --- | --- |
| API Gateway | 10,000 | Default |
| AWS ALB | Virtually unlimited number | Default |
| AWS SQS | 3,000 | Default |

After deploy an application, we can perform a Load Test to calculate the resources need for each RPS by using the below steps. Then calculate the total of resources for the AWS EKS Cluster for optimizing resources and cost

- **Run a Load Test Using JMeter**
    - Simulate API Traffic at different RPS levels (e.g., 100 RPS, 200 RPS, 500 RPS).
    - Measure the latency, error rate, and system performance.
- **Calculate Resource Consumption Per RPS**
    - **vCPU per request** = Total vCPU usage during test / Total RPS
    - **Memory per request** = Total RPS Total RAM usage during test (MB) / Total RPS
    - **Concurrency** = The number of RPS * Response time
            
**Refers**

- https://aws.amazon.com/blogs/containers/how-to-use-multiple-load-balancer-target-group-support-for-amazon-ecs-to-access-internal-and-external-service-endpoint-using-the-same-dns-name/
- https://aws.amazon.com/blogs/networking-and-content-delivery/integrating-your-directory-services-dns-resolution-with-amazon-route-53-resolvers/
- https://docs.aws.amazon.com/whitepapers/latest/active-directory-domain-services/design-consideration-for-aws-managed-microsoft-active-directory.html
- https://aws.amazon.com/blogs/database/configuring-amazon-elasticache-for-redis-for-higher-availability/
- https://medium.com/geekculture/how-to-calculate-server-max-requests-per-second-38a39bb96a85
- https://docs.oracle.com/en/cloud/paas/integration-cloud/oracle-integration-oci/calculate-requests-second.html            

---

## **2. Failures Strategy & High Availability**

Based on the requirements of this task, Warm Standby is the best disaster recovery strategy because it ensures rapid failover, scalability, and high availability while remaining cost-effective. It keeps a secondary AZ partially active, allowing for quick recovery with minimal downtime and near-zero data loss, unlike slower Backup & Restore or costly Multi-Site Active/Active strategy.

| **Strategy** | **Failures** | **Scalability** | **Cost-Effectiveness** | **Availability** | **RTO (Recovery Time Objective)** | **RPO (Recovery Point Objective)** |
| --- | --- | --- | --- | --- | --- | --- |
| **Backup & Restore** | ❌ Poor | ❌ Slow | ✅ Low-cost | ❌ Low | ⏳ Hours | ⏳ Hours (Data Loss Possible) |
| **Pilot Light** | ⚠️ Partial | ⚠️ Needs Scaling | ✅ Low-cost | ⚠️ Medium | ⏳ Minutes to Hours | ⏳ Minutes to Hours |
| **Warm Standby** ✅ **Best Balance** | ✅ Fast | ✅ Can Scale | ✅ Cost-efficient | ✅ High | ⏳ **1-5 mins** | ⏳ **Milliseconds to Seconds** (Using Continuous Replication) |
| **Multi-Site Active/Active** | ✅ No downtime | ✅ Best | ❌ Expensive | ✅✅ **Highest** | ⏳ **Instant** | ⏳ **Zero (No Data Loss)** |

For another components like metadata, DB, compute, etc. that should have the corresponding DR strategy to make sure a failures requirements:

| **Strategy** | **Implementation** |
| --- | --- |
| **Backup & Restore** | RDS Snapshots, S3 Versioning, PITR in DynamoDB |
| **Multi-Zone Deployment** | RDS, DynamoDB, and EKS deployed across multiple AZs |
| **Failover Mechanism** | Warm Standby in a separate AWS AZs |
| **Auto-Failover** | RDS Multi-AZ, Kubernetes pod rescheduling |

**Refers**

- https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html

## **3. Scaling Strategy**

Below is the plans for scaling when the product grows beyond current infrastructure.

| **Scaling Strategy** | **Implementation** |
|----------------------|--------------------|
| **Horizontal Scaling** | - **Microservices**: Deployments, Horizontal Pod Autoscaler (HPA) <br> - **EKS Nodes**: Cluster Autoscaler |
| **Database Scaling** | - Read replicas and partitioning in Amazon RDS |
| **Load Balancing** | - **AWS ALB**: Load balancing between multiple availability zones <br> - **AWS Global Accelerator**: Improves worldwide traffic distribution |
| **Asynchronous Processing** | - **SNS + SQS**: Decouples workloads and distributes loads efficiently |
| **Edge Caching** | - **CloudFront**: Low-latency content delivery and caching at the edge |

---

---

## **4. Cost Optimization**

| **Cost Factor** | **Optimization Strategy** |
| --- | --- |
| **Compute Costs** | Use Spot Instances for non-critical workloads |
| **Database Costs** | Optimize queries, enable RDS auto-scaling |
| **Storage Costs** | S3 lifecycle policies for archive management |
| **Traffic Costs** | Reduce outbound traffic using CloudFront caching |

---

## 5. CICD pipeline

### Diagram

![Diagrams-99Tech CICD V1 0 0 (1)](https://github.com/user-attachments/assets/e6ec5651-9354-4272-9856-d573a78f1c2e)

### DEV and QAC

This pipeline is primarily designed for continuous integration (CI) and continuous deployment (CD) in the **development (DEV)** and **quality assurance control (QAC)** environments.

**CI**

- **Trigger**: Automatically triggered when an engineer commits and pushes changes to GitLab.
- **Stages:**
    1. **Software Composition Analysis (SCA)** - Scans for vulnerabilities in dependencies.
        - If it fails → Pipeline exits.
    2. **Static Application Security Testing (SAST)** - Analyzes code for security vulnerabilities.
        - If it fails → Pipeline exits.
    3. **Testing Stage**:
        - Code Linter, Unit Tests, Regression Tests, etc.
        - If any test fails → Pipeline exits.
    4. **Build & Push**:
        - The application is built and pushed to **Elastic Container Registry (ECR)**.
        - If this step fails → Pipeline exits.

**CD**

- **Trigger**: Automatically starts once the CI pipeline completes successfully.
- **Stages:**
    1. **ArgoCD pulls HelmChart**.
        - If fails → Pipeline exits.
    2. **Deploy to Amazon EKS (Elastic Kubernetes Service)**.
        - If fails → Pipeline exits.
    3. **PostSync Validation**:
        - End-to-End (E2E) Testing.
        - Dynamic Application Security Testing (DAST).
        - If any test fails → Pipeline exits.

### STG and PRD

This pipeline is designed for **staging (STG)** and **production (PRD)** environments.

**CI**

- **Trigger**: **Manual** trigger based on a *semantic version tag (V.*.*)**.
- **Stages:**
    1. **Software Composition Analysis (SCA)**.
        - If it fails → Pipeline exits.
    2. **Static Application Security Testing (SAST)**.
        - If it fails → Pipeline exits.
    3. **Testing Stage**:
        - Code Linter, Unit Tests, Regression Tests, etc.
        - If any test fails → Pipeline exits.
    4. **Build & Push**:
        - The application is built and pushed to **Elastic Container Registry (ECR)**.
        - If this step fails → Pipeline exits.

**CD**

- **Trigger**: **Manual webhook trigger** initiated by an engineer.
- **Stages:**
    1. **ArgoCD pulls HelmChart**.
        - If fails → Pipeline exits.
    2. **Deploy to Amazon EKS**.
        - If fails → Pipeline exits.
    3. **PostSync Validation**:
        - End-to-End (E2E) Testing.
        - Dynamic Application Security Testing (DAST).
        - If any test fails → Pipeline exits.

### **Comparison (difference)**

| **Category** | **DEV/QAC CI/CD** | **STG/PRD CI/CD** |
|-------------|-------------------|-------------------|
| **Trigger Type (CI)** | Automatic trigger on commit | Manual trigger with semantic versioning |
| **Trigger Type (CD)** | Automatically starts after CI completes | Requires manual webhook trigger by engineer |
| **Deployment Approach** | Continuous integration and deployment | More controlled and manual approach for stability |
| **Environment Segregation** | Early-stage testing and validation | More stable and intended for production releases |
| **Versioning Strategy** | Does **not** use semantic versioning (runs on commit) | Requires **semantic version tag** before execution |

## 6. WBS

<img width="963" alt="image" src="https://github.com/user-attachments/assets/3cce4e38-1c8b-4d20-8fd5-cff00b87bacf" />

This **Work Breakdown Structure (WBS) for Blockchain Application V1.0.0** outlines the tasks and timeline for a **6-week project**. The breakdown consists of four major **epics/tasks**, each with specific work items and an associated timeline (marked across weeks W1 to W6).

- **Duration**
    - 6 Weeks
- **Assumption**
    - 1 week = 5 working days
    - 1 day = 8 hours

### **Proof of Concept (POC)**

**Goal**: Define scope, architecture, and CI/CD strategy.

- **Planning (W1)**
    - Define project scope, objectives, and definition of done (DoD).
- **Project Kick-off (W1)**
    - Design an architecture.
    - Design a CI/CD flow.

### **Functional Development**

**Goal**: Set up infrastructure and deploy the application.

- **Infrastructure (W2-W3)**
    - Develop Terraform modules for network, compute, etc.
    - Build infrastructure using Terraform modules.
- **Application Deployment (W4)**
    - Build CI/CD pipelines.
    - Deploy the application using CI/CD pipelines.

### **Testing and Refinement**

**Goal**: Ensure system performance, stability, and enhancements.

- **Testing (W5)**
    - Perform load, stress, and business tests.
- **Refinement (W5)**
    - System refinement and enhancements.

### **Handover**

**Goal**: Documentation and final handover.

- **Document (W6)**
    - Build and hand over documentation to the team.
