# Architecture of TSIS

## Tech Stack Overview

| ADR      | Layer                   | Tech                                                 | Purpose                                                                                                          |
| -------- | ----------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| [ADR-001](#adr-001-adopt-angular-for-web-frontend)  | Frontend                | **Angular**                                          | Role-based dashboards for Admin, Tutor, and Student                                                              |
| [ADR-002](#adr-002-adopt-flutter-for-mobile-app)   | Mobile                  | **Flutter**                                          | Native-style student app to watch classes, submit homework, and check attendance                                 |
| [ADR-003](#adr-003-implement-a-bff-layer-with-nodejs-express) | Backend (BFF)           | **Node.js (Express)**                                | Backend-for-Frontend layer exposing REST APIs, handling session/auth, role-based access, and request aggregation |
| [ADR-004](#adr-004-microservices-with-nodejs-express)         | Microservices           | **Node.js (Express)**                                | Each service (e.g., auth, class, homework) is independently deployed and owns its business logic                 |
| [ADR-005](#adr-005-restful-apis-with-swagger-openapi)         | API Design              | **RESTful API + Swagger (OpenAPI)**                  | Services expose documented REST APIs via Swagger UI                                                              |
| [ADR-006](#adr-006-containerization-with-docker--kubernetes)  | Containerization        | **Docker + Kubernetes (K8s)**                        | Each service runs in an isolated container; managed via K8s for orchestration and scaling                        |
| [ADR-007](#adr-007-aws--cloudflare-for-infrastructure)        | Cloud Infra             | **AWS (EC2, Lambda, RDS, S3) + Cloudflare**          | AWS for compute/storage/serverless; Cloudflare for CDN, HTTPS, and DDoS protection                               |
| [ADR-008](#adr-008-networking-with-aws-alb-and-vpc-setup)     | Networking              | **AWS ALB (Application Load Balancer)**              | Distributes incoming traffic across BFF/API services for high availability and scaling                           |
| [ADR-008](#adr-008-networking-with-aws-alb-and-vpc-setup)     | VPC Networking          | **NAT Gateway + Private/Public Subnet Split**        | Allows internal services to access the internet securely while keeping critical infra private                    |
| [ADR-009](#adr-009-cqrs-architecture-with-postgresql-write--mongodb-read) | Database                | **PostgreSQL (write) + MongoDB (read)**              | CQRS + Outbox Pattern: PostgreSQL for transactional writes, MongoDB for fast reads and dashboards                |
| [ADR-010](#adr-010-event-driven-data-sync-via-redis-pubsub--outbox-pattern) | Data Sync               | **Redis Pub/Sub**                                    | Event-driven sync from write DB to read DB                                                                       |
| [ADR-011](#adr-011-centralized-logging--observability-with-rabbitmq--elk) | Logging & Observability | **RabbitMQ + ELK (Elasticsearch, Logstash, Kibana)** | Centralized, scalable log ingestion and real-time observability dashboards                                        |
| [ADR-012](#adr-012-use-keycloak-for-authentication--authorization) | Auth                    | **Keycloak (JWT-based)**                             | Centralized identity and access management, using stateless JWT tokens                                           |
| [ADR-013](#adr-013-testing-frameworks-postman--cypress)      | Testing                 | **Postman, Cypress**                                 | API and end-to-end UI testing to ensure reliability and regressions                                              |
| [ADR-014](#adr-014-automate-cicd-with-github-actions--aws-deployment) | CI/CD                   | **GitHub Actions + AWS Deployment**                  | Automated linting, testing, build, and deployment pipelines on push or PR                                        |
| [ADR-015](#adr-015-gitops-practices-via-github)               | GitOps                  | **GitHub (PRs, branch policies, issue workflows)**   | Treat code and infra as Git-managed artifacts with reviews and auditability                                      |

## System Architecture Diagram

![System Architecture Diagram](./diagram/SystemDiagram.png)
[Draw.io Raw File](./raw/SystemDiagram.drawio)

## Architectural Decision Records (ADR)

Each ADR captures a significant architectural decision for TSIS, documenting the context, decision, and its consequences.

### ADR-001: Adopt Angular for Web Frontend

**Status**: Accepted  
**Context**: We need a robust framework for building role-based dashboards (Admin, Tutor, Student) with structured components, dependency injection, and built-in routing.  
**Decision**: Use **Angular** as the primary web frontend framework.  

**Consequences**:  

- **Positive**: Strong typing with TypeScript, modular architecture, and RxJS for reactive data streams.  
- **Positive**: Comprehensive CLI and officially supported component libraries (Angular Material).  
- **Negative**: Steeper learning curve compared to simpler libraries.  
- **Negative**: Larger initial bundle sizes; requires lazy loading and build optimization.  

**Considered Options**:  

- **React**  
  - *Pros*: Massive ecosystem, flexible UI library with Hooks and Context API; wide community plugin support.  
  - *Cons*: Unopinionated—requires choosing routing (React Router), state management (Redux, MobX), and UI toolkit separately; can lead to inconsistent patterns.  
- **Vue.js**  
  - *Pros*: Incremental adoption path; built-in solutions for routing (Vue Router) and state (Vuex); simpler learning curve.  
  - *Cons*: Smaller enterprise adoption; ecosystem fragmentation between v2 and v3; fewer large-scale case studies.  
- **Svelte**  
  - *Pros*: Compile-time reactivity with minimal runtime overhead; very small bundle sizes and excellent performance.  
  - *Cons*: Emerging ecosystem; limited mature tooling and fewer enterprise-grade component libraries; smaller talent pool.  

**Rationale**:  
Angular provides a complete, opinionated framework out-of-the-box, with first-class TypeScript support, dependency injection, and a cohesive CLI. This ensures consistency across multiple dashboards and reduces architectural decisions left to individual teams, which is crucial for a project of TSIS’s scope.  

---

### ADR-002: Adopt Flutter for Mobile App

**Status**: Accepted  
**Context**: We require a native-style mobile app for students to watch classes, submit homework, and check attendance on both iOS and Android, with high performance and consistent UI.  
**Decision**: Use **Flutter** for cross-platform mobile development.  

**Consequences**:  

- **Positive**: Single codebase for both iOS and Android, reducing development and maintenance effort.  
- **Positive**: Hot-reload speeds up iteration and debugging.  
- **Positive**: Rich, customizable widget library delivers near-native performance.  
- **Negative**: Dependency on the Dart ecosystem; team must learn and maintain Dart skills.  
- **Negative**: Larger binary size compared to some native solutions; may require code-size optimization.  

**Considered Options**:  

- **React Native**  
  - *Pros*: Leverages JavaScript/TypeScript skills; large ecosystem and community plugins; supports native modules.  
  - *Cons*: Bridge overhead can introduce performance bottlenecks; fragmentation in maturity of libraries; requires separate handling for some native features.  
- **Native (Kotlin + Swift)**  
  - *Pros*: Full access to platform APIs; best possible performance and smallest binary.  
  - *Cons*: Two separate codebases increase development and QA effort; divergence of feature parity; higher maintenance cost.  
- **Ionic (Capacitor)**  
  - *Pros*: Web-tech stack (Angular/React/Vue); rapid prototyping with existing web components; easy to share code with web frontend.  
  - *Cons*: Webview-based performance can feel less native; limited access to some device APIs; UX inconsistencies across platforms.  
- **Xamarin**  
  - *Pros*: C#/.NET ecosystem; good access to native APIs; shared business logic.  
  - *Cons*: Smaller community; larger app sizes; less momentum compared to Flutter or React Native.  

**Rationale**:  
Flutter strikes the best balance between performance, DX (developer experience), and UI consistency. Its rich widget system and single-codebase model maximize productivity and ensure a uniform look and feel across platforms—critical for delivering a seamless student experience.  

---

### ADR-003: Implement a BFF Layer with Node.js (Express)

**Status**: Accepted  
**Context**: Frontend and mobile clients (Angular, Flutter) require tailored APIs, consistent session/auth handling, and aggregation of data from multiple microservices without coupling them directly to each service’s contract.  
**Decision**: Build a **Backend-for-Frontend (BFF)** service using **Node.js** (Express) that exposes REST endpoints specific to each client’s needs, handles authentication/authorization, and orchestrates calls to underlying microservices.  

**Consequences**:  

- **Positive**: Simplifies client code by providing client-specific payloads and hiding service churn.  
- **Positive**: Centralizes security, rate-limiting, and versioning per client.  
- **Negative**: Additional deployment surface and operational overhead.  
- **Negative**: Potential latency overhead due to an extra network hop.  

**Considered Options**:  

- **AWS API Gateway + Lambda**  
  - *Pros*: Fully managed, auto-scaling, built-in caching and authorizers.  
  - *Cons*: Vendor lock-in, cold starts, increased per-request cost, more fragmented observability.  
- **GraphQL Gateway (Apollo Server)**  
  - *Pros*: Single endpoint, flexible queries, built-in schema stitching and data batching.  
  - *Cons*: Complexity of schema maintenance, over-fetching risk if not carefully designed, learning curve.  
- **Client-side Aggregation**  
  - *Pros*: No extra service layer, direct calls can reduce infra.  
  - *Cons*: Repeated round trips, duplicated logic across clients, tight coupling to microservice contracts.  

**Rationale**:  
A dedicated Node.js/Express BFF strikes the best balance between control, performance, and developer familiarity. It leverages our existing backend skillset, provides low-latency tailored APIs, and keeps clients decoupled from microservice evolution.  

### ADR-004: Microservices with Node.js (Express)

**Status**: Accepted  
**Context**: TSIS requires a set of independently deployable services (e.g., auth, class, homework) so that teams can iterate and scale each domain without impacting others.  
**Decision**: Implement all backend services as **Node.js** applications using the **Express** framework.  

**Consequences**:  

- **Positive**: Rapid development leveraging JavaScript/TypeScript and a mature npm ecosystem.  
- **Positive**: Unified tech stack across BFF and microservices simplifies hiring and code sharing.  
- **Negative**: Higher memory usage compared to some compiled languages; requires vigilant monitoring and tuning.  
- **Negative**: Single-threaded nature of Node.js means CPU-bound tasks must offload work or use worker threads.  

**Considered Options**:  

- **Spring Boot (Java)**  
  - *Pros*: Mature ecosystem, strong typing, built-in dependency injection, rich enterprise features.  
  - *Cons*: Longer startup times, heavier resource footprint, steeper learning for TypeScript-centric teams.  
- **Go (Golang)**  
  - *Pros*: Native binaries, excellent performance and concurrency model (goroutines), small memory footprint.  
  - *Cons*: Less mature web framework ecosystem, manual dependency management, steeper learning curve for JavaScript-focused developers.  
- **.NET Core (C#)**  
  - *Pros*: High performance, robust framework, first-class support for microservices patterns.  
  - *Cons*: Requires C# expertise, less common in Node/JS-centric organizations, larger container images.  

**Rationale**:  
Choosing Node.js/Express keeps our backend consistent with the BFF layer and leverages existing team skills in JavaScript/TypeScript. It accelerates development, fosters code reuse, and aligns with the lightweight, event-driven nature of our services.  

---

### ADR-005: RESTful APIs with Swagger (OpenAPI)

**Status**: Accepted  
**Context**: We need clear, discoverable, and maintainable service contracts between microservices and clients. Endpoints must be self-documenting and allow for automated client SDK generation.  
**Decision**: Use **RESTful** API conventions and document all services with **Swagger (OpenAPI)** specifications.  

**Consequences**:  

- **Positive**: Auto-generated, interactive API documentation via Swagger UI.  
- **Positive**: Strong contract enforcement with request/response schemas.  
- **Negative**: Must keep OpenAPI specs in sync with implemented code—adds overhead.  
- **Negative**: REST can lead to multiple round trips for composite operations.  

**Considered Options**:  

- **GraphQL**  
  - *Pros*: Single endpoint, clients request exactly the data they need; built-in schema introspection.  
  - *Cons*: Overhead of maintaining a GraphQL schema and resolvers; risk of complex queries impacting performance; caching more complex.  
- **gRPC**  
  - *Pros*: High performance, binary protocol, built-in code generation for many languages; bi-directional streaming support.  
  - *Cons*: Less human-readable than JSON/REST; requires HTTP/2; browser support via proxies only.  
- **Async API/Message Contracts**  
  - *Pros*: Good for event-driven parts of system; decouples producer/consumer.  
  - *Cons*: Not suited for synchronous client interactions; tooling less mature for request/response patterns.  

**Rationale**:  
REST + OpenAPI strikes the best balance between developer familiarity, ecosystem maturity, and ease of integration. Swagger ensures our APIs are well-documented, discoverable, and can be validated early in the development cycle—critical for coordinating multiple teams and third-party integrations.  

---

### ADR-006: Containerization with Docker & Orchestration via Kubernetes

**Status**: Accepted  
**Context**: Services must run in isolated environments, be reproducible across dev/staging/production, and scale to handle varying loads.  
**Decision**: Package each service as a **Docker** container and deploy on **Kubernetes (K8s)** for orchestration.  

**Consequences**:  

- **Positive**: Consistent runtime environment across teams and stages; “works on my machine” eliminated.  
- **Positive**: Kubernetes provides auto-scaling, rolling updates, self-healing, and service discovery out-of-the-box.  
- **Negative**: Introduces operational complexity; requires team training in container lifecycle and K8s constructs.  
- **Negative**: Resource overhead of the control plane and node agents; potential for “debugging in prod” if configs mismanaged.  

**Considered Options**:  

- **Docker Swarm**  
  - *Pros*: Simpler setup; uses the same Docker CLI; built-in clustering.  
  - *Cons*: Less feature-rich than K8s; smaller community and fewer managed offerings.  
- **AWS ECS/Fargate**  
  - *Pros*: Fully managed AWS service; abstracts away cluster management; tight AWS integration.  
  - *Cons*: Vendor lock-in; limited portability; less control over scheduling nuances.  
- **HashiCorp Nomad**  
  - *Pros*: Lightweight scheduler; multi-region support; straightforward job definitions.  
  - *Cons*: Smaller ecosystem; requires separate tooling for service mesh, ingress, and storage.  

**Rationale**:  
Kubernetes is the de facto standard for container orchestration, with a rich ecosystem and broad community support. Its declarative API and robust feature set (auto-scaling, rolling updates, health checks) align perfectly with TSIS’s needs for resilience and scalability, despite the steeper learning curve.  

### ADR-007: AWS + Cloudflare for Infrastructure

**Status**: Accepted  
**Context**: TSIS requires highly available, scalable compute and storage, plus global content delivery and edge security to protect against DDoS and accelerate static assets.  
**Decision**: Host core services and data on **AWS** (EC2, Lambda, RDS, S3) and leverage **Cloudflare** for CDN, HTTPS termination, DNS, and DDoS mitigation.  

**Consequences**:  

- **Positive**:  
  - AWS managed services (EC2 autoscaling, Lambda serverless, RDS managed databases, S3 durable storage) reduce operational burden.  
  - Cloudflare caches static assets at the edge, improving latency for global users and absorbing volumetric attacks.  
- **Negative**:  
  - Multi-vendor complexity—teams must manage both AWS and Cloudflare consoles, IAM policies, and billing.  
  - Potential for configuration drift between platforms if not centrally tracked.  

**Considered Options**:  

- **Azure + Azure CDN**  
  - *Pros*: Single vendor for compute (VMs, Functions), storage (Blob), and edge caching; native identity integration (Azure AD + Front Door).  
  - *Cons*: Less mature edge network compared to Cloudflare; potentially higher egress costs; steeper learning curve if team unfamiliar with Azure.  
- **GCP + Cloud CDN**  
  - *Pros*: Integrated with Google’s backbone, easy peering with GKE and Cloud Functions; unified IAM and billing.  
  - *Cons*: Edge footprint smaller than Cloudflare’s; fewer security edge features (WAF/Rate Limiting) out of the box.  
- **On-Premise + Akamai**  
  - *Pros*: Full control over hardware and network; Akamai offers top-tier global CDN and security.  
  - *Cons*: High capital expenditure; long provisioning cycles; increased ops burden for hardware maintenance and network configuration.  

**Rationale**:  
AWS provides the most mature, widely adopted managed services for compute, storage, and serverless. Pairing it with Cloudflare gives us the broadest global edge network and richest security features, ensuring both performance and protection without building or managing our own edge layer.  

---

### ADR-008: Networking with AWS ALB and VPC Setup

**Status**: Accepted  
**Context**: Ensure high availability, secure access, and isolation between public-facing services and internal systems.  
**Decision**: Route incoming traffic through an **AWS Application Load Balancer (ALB)** and place resources inside a **VPC** with public and private subnets, using a **NAT Gateway** for outbound internet access from private resources.  

**Consequences**:  

- **Positive**:  
  - ALB provides SSL termination, path-based routing, and health checks.  
  - Public subnets host load balancers and NAT Gateway; private subnets secure databases and internal services.  
- **Negative**:  
  - Increased AWS networking costs (NAT Gateway, Elastic IPs).  
  - Added complexity in VPC configuration and subnet management.  

**Considered Options**:  

- **Classic Load Balancer (ELB)**  
  - *Pros*: Simple setup for HTTP/HTTPS.  
  - *Cons*: Lacks advanced routing rules and slower feature updates compared to ALB.  
- **Network Load Balancer (NLB)**  
  - *Pros*: Ultra-low latency and static IP support for TCP/UDP.  
  - *Cons*: No application-level routing or SSL offloading; more complex certificate management.  
- **Self-managed Proxy (e.g., NGINX on EC2)**  
  - *Pros*: Full control over routing logic and middleware.  
  - *Cons*: Operational overhead—patching, scaling, high-availability configuration.  

**Rationale**:  
AWS ALB strikes the right balance between managed features (SSL offload, path routing, health checks) and ease of integration with Auto Scaling Groups. Coupled with a VPC design that separates public and private resources, it delivers both security and scalability with minimal operational burden.  

---

### ADR-009: CQRS Architecture with PostgreSQL (write) & MongoDB (read)

**Status**: Accepted  
**Context**: We need transactional integrity for writes and fast read performance for dashboards without impacting write throughput.  
**Decision**: Implement **CQRS**: use **PostgreSQL** for all transactional writes and **MongoDB** as a denormalized read model.  

**Consequences**:  

- **Positive**:  
  - Write operations remain ACID-compliant, ensuring data integrity.  
  - Read operations against MongoDB are fast and can be scaled independently for reporting and analytics.  
- **Negative**:  
  - Increased system complexity due to separate read/write data stores.  
  - Eventual consistency means reads may lag behind writes.  

**Considered Options**:  

- **Single Database (PostgreSQL only)**  
  - *Pros*: Simpler architecture, single source of truth, strong consistency.  
  - *Cons*: Read-heavy queries could impact write performance; complex joins for analytics can be slow.  
- **Read Replica of PostgreSQL**  
  - *Pros*: Leverages built-in replication, consistent schema, strong read scalability.  
  - *Cons*: Read replicas lag unpredictably; replicas can’t easily reshape data for reporting.  
- **Elasticsearch for Read Model**  
  - *Pros*: Powerful search and aggregation capabilities, near real-time indexing.  
  - *Cons*: Schema mapping overhead, eventual consistency, more operational overhead, less suited for general-purpose document reads.  

**Rationale**:  
CQRS with PostgreSQL writes and MongoDB reads offers a clear separation: writes remain transactional while reads can be optimized and scaled for diverse query patterns. MongoDB’s flexible schema and rich query capabilities make it ideal for dashboard data without overloading the primary database.  

### ADR-010: Event-driven Data Sync via Redis Pub/Sub & Outbox Pattern

**Status**: Accepted  
**Context**: To maintain a denormalized read model (MongoDB) in sync with the transactional write store (PostgreSQL) without tight coupling or excessive polling.  
**Decision**: Use the **Outbox Pattern** in PostgreSQL to atomically record domain events on write, then publish these events via **Redis Pub/Sub** to downstream consumers that update MongoDB.  

**Consequences**:  

- **Positive**: Guarantees reliable event emission (atomic write & outbox insert) and decouples services through a lightweight message broker.  
- **Positive**: Low-latency propagation of changes to the read model, enabling near real-time dashboards.  
- **Negative**: Introduces operational complexity—must monitor and scale Redis, ensure idempotent consumers, and purge or archive outbox entries.  
- **Negative**: Adds development overhead to implement transactional outbox logic and consumer error handling.  

**Considered Options**:  

- **Direct DB Triggers with Kafka**  
  - *Pros*: Leverages robust, partitioned streaming; strong delivery guarantees and replay capability.  
  - *Cons*: More heavyweight infrastructure; steeper learning curve; Kafka not as easy to operate for small teams.  
- **Change Data Capture (CDC) via Debezium**  
  - *Pros*: No code changes in services; taps the database WAL for event streams; supports multiple sinks.  
  - *Cons*: Adds Debezium and Kafka dependencies; greater latency and operational overhead.  
- **Polling-Based Sync**  
  - *Pros*: Simple to implement; no additional messaging infrastructure.  
  - *Cons*: Inefficient (constant querying), higher load on the write database, and increased latency.  

**Rationale**:  
The Outbox Pattern combined with Redis Pub/Sub offers a balanced solution: reliable, atomic event emission without the operational weight of a full streaming platform. Redis is lightweight to manage and integrates easily with our existing stack, while outbox entries ensure no events are lost even if the publisher crashes.  

### ADR-011: Centralized Logging & Observability with RabbitMQ + ELK

**Status**: Accepted  
**Context**: Need centralized logs for debugging, auditing, and performance monitoring.  
**Decision**: Send logs through RabbitMQ to a Logstash pipeline feeding Elasticsearch and Kibana.  
**Consequences**:  

- **Positive**: Scalable and searchable log storage.  
- **Positive**: Real-time dashboards for observability.  
- **Negative**: Additional infrastructure and maintenance overhead.

**Considered Options**:  

- **Direct Elasticsearch Ingestion**  
  - *Pros*: Fewer moving parts; services write logs directly to Elasticsearch.  
  - *Cons*: No buffering; lost logs if Elasticsearch is down; tight coupling.  
- **Kafka-based Pipeline**  
  - *Pros*: High throughput, persistence, replay capability.  
  - *Cons*: Heavier infrastructure; steeper operational burden.  
- **Cloud-managed Logging (AWS CloudWatch)**  
  - *Pros*: Fully managed; integrates with AWS services.  
  - *Cons*: Vendor lock-in; higher cost at scale; less query flexibility.

**Rationale**: RabbitMQ provides reliable buffering and decoupling of producers and consumers. ELK offers rich search and visualization, fitting our messaging-centric architecture with manageable overhead.

---

### ADR-012: Use Keycloak for Authentication & Authorization

**Status**: Accepted  
**Context**: Require centralized identity management, SSO, and fine-grained access control.  
**Decision**: Deploy Keycloak with JWT-based tokens.  
**Consequences**:  

- **Positive**: Standards-based SSO and RBAC.  
- **Positive**: Extensible identity federation and user management.  
- **Negative**: Additional service to deploy and maintain.

**Considered Options**:  

- **Auth0**  
  - *Pros*: Fully managed, quick integration, multi-tenant support.  
  - *Cons*: SaaS cost; vendor lock-in; limited customization.  
- **AWS Cognito**  
  - *Pros*: Native AWS integration; serverless.  
  - *Cons*: Complex setup; limited theming; scaling quirks.  
- **Custom JWT Service**  
  - *Pros*: Full control; lightweight.  
  - *Cons*: Reinvents user management; increased security risk.

**Rationale**: Keycloak is open-source, feature-rich, and supports standard protocols (OIDC/SAML). It aligns with our self-hosted approach and offers full control over user data and customization.

---

### ADR-013: Testing Frameworks: Postman & Cypress

**Status**: Accepted  
**Context**: Ensure reliability through automated tests at both API and UI levels.  
**Decision**: Use Postman for API testing and Cypress for end-to-end browser tests.  
**Consequences**:  

- **Positive**: Postman offers data-driven API tests with easy scripting.  
- **Positive**: Cypress provides fast, reliable UI tests with time-travel debugging.  
- **Negative**: Maintaining test suites as features evolve requires ongoing effort.

**Considered Options**:  

- **JUnit/TestNG + Selenium**  
  - *Pros*: Mature Java ecosystem; wide CI support.  
  - *Cons*: Slower, brittle UI tests; more boilerplate.  
- **Playwright**  
  - *Pros*: Cross-browser support, network interception, parallelism.  
  - *Cons*: Newer, smaller community than Cypress.  
- **REST Assured**  
  - *Pros*: Fluent Java API testing.  
  - *Cons*: Less accessible for JS/TS-centric teams; no UI testing.

**Rationale**: Postman and Cypress integrate seamlessly with our JS/TS stack, covering both API and UI needs while delivering strong developer ergonomics.

---

### ADR-014: Automate CI/CD with GitHub Actions & AWS Deployment

**Status**: Accepted  
**Context**: Need automated linting, testing, and deployment on code changes.  
**Decision**: Configure GitHub Actions pipelines to run on push/PR, performing lint, test, build, and deploy to AWS environments.  
**Consequences**:  

- **Positive**: Rapid feedback; consistent, repeatable deployments.  
- **Positive**: Native integration with GitHub repos and secrets.  
- **Negative**: Complex workflows and secret management require careful organization.

**Considered Options**:  

- **Jenkins**  
  - *Pros*: Highly customizable; plugin ecosystem.  
  - *Cons*: Self-hosted; maintenance overhead.  
- **CircleCI**  
  - *Pros*: Cloud-hosted, parallelism.  
  - *Cons*: Cost at scale; separates platform.  
- **GitLab CI/CD**  
  - *Pros*: Built-in with GitLab; powerful pipelines.  
  - *Cons*: Requires GitLab; diverges from GitHub.

**Rationale**: GitHub Actions aligns with our GitHub-centric workflow, minimizing context switches and operational overhead while providing powerful CI/CD capabilities.

---

### ADR-015: GitOps Practices via GitHub

**Status**: Accepted  
**Context**: Treat both code and infrastructure changes uniformly with review workflows and versioning.  
**Decision**: Use GitHub as the single source of truth; enforce branch policies, PR reviews, and link deployments to pull requests.  
**Consequences**:  

- **Positive**: Traceable changes; auditability; easy rollbacks via Git history.  
- **Positive**: Peer review enforces quality and collaboration.  
- **Negative**: Review process may add overhead for small fixes.

**Considered Options**:  

- **Argo CD**  
  - *Pros*: Declarative GitOps for Kubernetes; automated sync.  
  - *Cons*: Additional component to manage; learning curve.  
- **Flux CD**  
  - *Pros*: Lightweight; native GitHub integration.  
  - *Cons*: Less mature UI; YAML-heavy.  
- **Terraform Cloud**  
  - *Pros*: Remote state management; policy enforcement.  
  - *Cons*: Separate UI; infra-only focus.

**Rationale**: Leveraging GitHub for GitOps aligns with existing practices. Once workflows stabilize, we can introduce Argo CD or Flux CD for automated deployment synchronization.  
