---
title: "Software Development Team"
subtitle: "Software Development Team Roles, Documentation, and Lifecycle"
author: "Brian King"
revnumber: "v0.5.1"
doctype: "book"
cDay: "Saturday"
cDate: 25
cMonth: 07
cYear: 2026
uDay: "Monday"
uDate: 27
uMonth: 07
uYear: 2026
toc: true
toclevels: 6
sectnums: true
sectnumlevels: 6
icons: "font"
keywords: "Leadership Team, Development Team, Project Manager, Business Analyst, Systems Analyst, Project Architect, BMD, BRD, PRD, CRD, UID, UXD, SSD, Agile, Scrum, SDLC, Software Development, DevOps, QA"
summary: "Defines the leadership structure, individual roles and responsibilities, the suite of documentation required for project success, and the end-to-end development lifecycle."
description: "This document outlines the structured approach to software development within the organization — covering the Leadership Team, Leadership Roles, Development Team, Software Development Documentation (SDDs), and the Software Development Process including Development, Security, Testing, Deployment, Maintenance, and Average Project Metrics."
license: "AGPL 3.0"
attributions: "Not applicable"
copyright: "© Copyright 2020-2026 DigitalCoreNZ. All rights reserved."
status: "Complete"
---

# Software Development Team

This document outlines the structured approach to software development within the organization. It defines the leadership structure, individual roles and responsibilities, the suite of documentation required for project success, and the end-to-end development lifecycle. By establishing these standards, we ensure alignment between client needs, functional requirements, and technical execution.

## Leadership Team

Software projects are led by a small group who, collectively, make up the software leadership team:

- The Project Manager (PM), who interfaces between the client and the leadership team,
- The Business Analyst (BA), who suggests business needs and solutions to the leadership team,
- The Systems Analyst (SA), who suggests the project functionalities to the leadership team, and
- The Project Architect (PA), who suggests the project infrastructure and framework to the leadership team.

### Leadership Roles

The leadership roles form the strategic core of any software project. They are responsible for translating a business vision into a technical reality, ensuring that the project remains on schedule, within scope, and technically sound.

#### The Project Manager

- **Tasks:** Project scheduling, resource allocation, risk management, and client communication.
- **Responsibilities:** Ensuring project delivery within time and budget constraints; managing stakeholder expectations.
- **Outcomes:** Project plans, status reports, and successful delivery of the final product.
- **Contributions:** Provides the organizational framework and client-facing interface that keeps the project moving forward.

#### The Business Analyst

- **Tasks:** Analysing business processes, identifying opportunities for improvement, and documenting business requirements.
- **Responsibilities:** Ensuring that the proposed solutions align with the business goals and provide value to stakeholders, attends the same meetings as, and works closely with, the Project Manager
- **Outcomes:** Business cases, process models, and stakeholder analysis.
- **Contributions:** Provides the business context and justification for project initiatives, ensuring strategic alignment.

#### The Systems Analyst

- **Tasks:** Requirement gathering, business process modelling, and functional gap analysis.
- **Responsibilities:** Bridging the gap between business needs and technical specifications; ensuring functionality meets user requirements.
- **Outcomes:** Business Requirements Documents (BRD), use cases, and functional workflows.
- **Contributions:** Ensures the software solves the right problems and provides tangible value to the end-users.

#### The Project Architect

- **Tasks:** System design, technology stack selection, and defining integration patterns.
- **Responsibilities:** Ensuring the scalability, security, and maintainability of the technical solution.
- **Outcomes:** Software Specifications Documents (SSD), architectural diagrams, and infrastructure plans.
- **Contributions:** Provides the technical blueprint and structural integrity required for a robust and future-proof system.

#### Purpose of the Leadership Team

Together, the leadership team:

- Defines the requirements,
- Builds a flexible development strategy, and
- Creates the documentation that is used during development of the project.

**Leadership Team Collective Responsibilities:**

- **Tasks:** Strategic planning, cross-functional coordination, and final approval of project milestones.
- **Responsibilities:** Maintaining the "Big Picture" view and resolving conflicts between scope, time, and technical feasibility.
- **Outcomes:** A unified project vision and a comprehensive set of guiding documentation.
- **Contributions:** Establishes the foundation for the development team to build upon with clarity and confidence.

## Developer Team

- **Development Manager:** Manages the developers; responsible for code quality, sprint execution, and technical mentorship.
- **Full Stack Developers:** Responsible for implementing front-end and back-end logic, database management, and API integration.
- **QA Manager:** Manages the testing team; responsible for the overall quality assurance strategy and release readiness.
- **Security Engineer:** Responsible for implementing security controls, performing vulnerability assessments, and ensuring data protection.
- **QA/Test Engineers:** Responsible for writing test plans, executing manual/automated tests, and reporting bugs.
- **DevOps Engineer:** Responsible for CI/CD pipelines, environment stability, and deployment automation.
- **Product Manager:** Responsible for the product vision, roadmap, and prioritizing the product backlog based on market needs.
- **Project Manager:** Responsible for project planning, execution, monitoring, and closing, ensuring project objectives are met.
- **UI/UX Designer:** Responsible for creating intuitive and engaging user interfaces and ensuring a seamless user experience.

## The Software Development Process

Our processes are built upon **Agile methodologies**, specifically leveraging the Scrum framework to ensure iterative delivery, transparency, and rapid adaptation to changing requirements. We operate in fixed-length sprints (typically 2 weeks, although AI has significantly reduced the timeline), beginning with sprint planning and concluding with a sprint review and retrospective. This approach supports high velocity while valuable features are delivered first.

### Development

The development phase is where the technical implementation occurs, guided by the Product Requirements Document and Software Specifications Document.

- **Sprint Execution:** Software Developers work on prioritized backlog items. Daily stand-ups synchronize activities and identify blockers.
- **Code Standards & Peer Review:** All code must adhere to project-specific style guides. Peer reviews (Pull Requests) are mandatory, ensuring code quality, knowledge sharing, and adherence to architectural patterns.
- **Incremental Integration:** Code is integrated into the main branch at the end of each day to:
  - Avoid "integration hell", and
  - Ensure the code is always in a deployable state.
- **Responsible:** Developers, Development Manager.

### Security

Security is not a final step but a continuous thread woven throughout the DevSecOps lifecycle.

- **Secure Coding Practices:** Developers follow OWASP guidelines to prevent common vulnerabilities like SQL injection and XSS.
- **Vulnerability Scanning:** Automated tools scan dependencies and source code for known vulnerabilities during the build process.
- **Access Control & Encryption:** Implementation of robust authentication (OAuth2/OIDC), role-based access control (RBAC), and encryption at rest and in transit.
- **Responsible:** Security Engineer, Systems Architect, Developers.

### Testing

Quality is verified through a multi-layered testing strategy to ensure functional correctness and performance.

- **Automated Testing:** Includes Unit Tests (logic verification), Integration Tests (component interaction), and End-to-End (E2E) Tests (user flow simulation).
- **Manual QA:** Exploratory testing to identify edge cases and UI/UX inconsistencies that automated tools might miss.
- **User Acceptance Testing (UAT):** The final validation step where stakeholders verify that the software meets the business requirements defined in the BRD.
- **Responsible:** QA Team, Developers (for Unit Tests), Client (for UAT).

### Deployment

We utilize automated pipelines to ensure reliable and repeatable releases.

- **Continuous Integration (CI):** Every commit triggers an automated build and test suite.
- **Continuous Deployment (CD):** Successful builds are automatically deployed to staging environments for validation, followed by a controlled promotion to production.
- **Infrastructure as Code (IaC):** Server configurations and cloud resources are managed via scripts to ensure environment parity.
- **Responsible:** DevOps Engineer.

### Maintenance

Post-launch activities focus on stability, performance, and evolution.

- **Monitoring & Alerting:** Real-time tracking of system health, error rates, and performance metrics (e.g., latency, throughput).
- **Incident Management:** A structured process for responding to, and resolving, production issues based on severity.
- **Feedback Loop:** User feedback and analytics are fed back into the Product Backlog for future sprint planning.
- **Responsible:** Support Team, Developers, Product Manager.

### Average Project Metrics

While every project is unique, an "average" mid-sized software product (e.g., a custom enterprise web application or a feature-rich mobile app) typically follows these parameters:

- **Timeline:** 3 to 6 months from inception to initial production release.
  - **Discovery & Planning:** 2 to 4 weeks.
  - **Development (MVP):** 2 to 4 months (8 to 12 sprints).
  - **UAT & Final Hardening:** 1 to 3 weeks.
- **Team Size:** 10 to 15 people.
  - **Leadership:** 3 to 4 members (Project Manager, Business/Systems Analyst, Architect).
  - **Execution:** 6 to 10 Developers (Front-end, Back-end, Full-stack).
  - **Quality & Ops:** 2 to 3 members (QA Engineers, DevOps/Security).

This structured approach minimizes risk, optimizes resource utilization, and maximizes the likelihood of delivering a product that exceeds client expectations.

However, an optimised AI workflow has severely reduced:

1. The expected timeline from months to weeks,
2. The development team to one _experienced_ developer and three (3) AI Entities, and
3. The expectations of delivering quality software with support for continuous security updates.

Quality software is problematic unless an experienced Project Manager or an experienced Software Developer fully commits to becoming an AI Generalist, who:

* Learns how to fine-tune and post-train AI models with open weights and permissive licenses,
* Adopts AI models, engineering tools, frameworks, processes, ideologies, and other AI concepts, and
* Understands the underlying AI infrastructure, is prepared to learn new AI technologies as they drop, and is willing to constantly follow a field where the goalpost is regularly moved.

## Conclusion: Software Development Team

This document provides a comprehensive framework for navigating the complexities of modern software engineering. By clearly defining leadership roles, developer roles, team responsibilities, and rigorous documentation, we establish a foundation for transparency and accountability. The hierarchical transition from high-level business requirements to granular software specifications, executed within an Agile/Scrum lifecycle, ensures that the final product is not only technically robust but also strategically aligned with business objectives.
