# Statement of Work (SOW)

## Project Title

Tutoring Student Management System (TSIS)

🔗 [Project Roadmap](https://github.com/users/AnsonCheng03/projects/3/)

## Author

Anson Cheng

## Date

June 2025

## Purpose

The purpose of this project is to design and develop a full-featured education management platform tailored for private tutoring centers. While most educational platforms are designed for large schools and involve heavy setup and administrative overhead, TSIS focuses on simplicity, flexibility, and usability—making it ideal for tutoring centers operating with lean teams and fast-paced schedules.

This system aims to streamline core functions such as class scheduling, attendance tracking, AI-assisted homework feedback, billing, and academic analytics. The architecture emphasizes modular design, cloud-native deployment, and role-based experiences for Admins, Tutors, and Students.

## Objectives

- Deliver a practical SaaS platform optimized for tutoring centers, not schools
- Automate operational workflows like billing, grading, and notifications
- Support AI-based tools to assist educators and reduce workload
- Ensure scalability, modularity, and observability in backend services
- Provide a responsive, cross-platform user experience via Angular and Flutter
- Maintain project transparency and accountability through public GitHub tracking

## Deliverables

| Deliverable                 | Description                                                              |
| --------------------------- | ------------------------------------------------------------------------ |
| Functional Web Dashboard    | Angular-based admin panel for managing classes, users, and billing       |
| Mobile Application          | Flutter-based app for students to access classes, homework, and feedback |
| Backend Microservices       | Node.js services with PostgreSQL, MongoDB, Redis, RabbitMQ               |
| Authentication System       | Integrated Keycloak SSO with JWT and role-based access                   |
| AI-Assisted Homework Module | AI integration for feedback, scoring, and analysis                       |
| Reporting Dashboard         | Read-model dashboards with exportable reports and data visualization     |
| Documentation               | BRD, SOW, Architecture, BDD, Test Plans, and User Guide                  |
| QA & UAT Readiness          | End-to-end test plans and UAT scenarios for verification and acceptance  |
| Deployment Support          | Docker + Kubernetes manifests for local and cloud deployment             |

## Project Timeline (12-Month Plan)

Detailed timelines are documented in the  
📄 [Project Plan](plan.md)

> 📌 As a personal side project, this schedule is aspirational and may shift based on available time.

## Scope Reference

Detailed inclusion and exclusion of features are documented in the  
📄 [Business Requirements Document (BRD)](BRD.md)

## Technology Reference

Technical stack and architectural design decisions are documented in  
📄 [System Architecture](architecture.md) and [Microservice Overview](microservices.md)

## Success Criteria

- All BRD-defined user stories are implemented, tested, and traceable
- Core user flows work end-to-end across platforms (class → attendance → homework → billing)
- AI feedback is integrated and credit-tracked properly
- Admin and student portals are responsive and role-isolated
- System is deployable in a containerized environment (Docker + Kubernetes)
- Documentation is comprehensive and the project is demonstrable publicly
