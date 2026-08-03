---
title: SDDs from AI Loops
subtitle: Creating Software Development Documents
author: Brian King
company: DigitalCoreNZ
revnumber: v0.5.6
doctype: book
cDay: Saturday
cDate: 25
cMonth: 07
cYear: 2026
uDay: Monday
uDate: 03
uMonth: 08
uYear: 2026
toc: true
toclevels: 6
sectnums: true
sectnumlevels: 6
icons: font
keywords: dual-coders, Business Modelling Document, Business Requirements Document, Product Requirements Document, Context Requirements Document, User Interface Document, User Experience Document, Software Specifications Document, Role Definitions, Background Agents, Software Development Document, Project Manager, Software Developer, Spec
summary: A revised SKILL for creating Software Development Documents using the automated abilities of the Project Manager and Software Developer, enabling a self-orchestrated proficiency.
description: This document provides feedback on the revised Software Development Documents SKILL, evaluating the automated abilities, the self-orchestrated Refinement Loop, and the complete Software Development Documents workflow from the Business Modelling Document through to the Software Specifications Document.
license: MIT
attributions: Not applicable
status: Complete
stem: latexmath
ai-prompt: You will create the Software Development Documents SKILL that is defined in the ./SDDs_from_Hermes_Agent_v0.5.6.md file.
copyright: © Copyright 2020-2026 DigitalCoreNZ. All rights reserved.
---

# SDDs from AI Loops v0.5.6
>Creating Software Development Documents

## Abstract

This is the sixth (6th) revision of the **Software Development Documents SKILL** for Hermes Agent, built around four (4) hierarchical Entities:

* The **Hermes Agent User** initiates the SKILL, provides the Spec file path, and holds the final authority to approve documents or terminate the workflow.
* **Hermes Agent** spawns the Project Manager, passes Role Definitions and Control, and operates in Watch Mode to detect and resolve failures.
* The **Project Manager** (Leadership Team) spawns the Software Developer, creates and refines Software Development Documents, and holds termination authority over the Refinement Loop.
* The **Software Developer** (Developer Team) receives documents and Control, produces revisions, and triggers escalation when it encounters unresolvable conditions.

The **Spec** (Application Specification) file is the key reference when progressing through Refinement Loops of any Software Development Document, iterating until all four (4) Entities judge the document ready for use. All seven (7) Software Development Documents are defined in the `./Dev_Docs_v0.5.1.ad` file.

This `SDDs from AI Loops v0.5.6` document defines the **Role Definitions**, the **Refinement Loop**, the **Role Boundaries**, **Watch Mode and Control Mode**, the **Recovery Mechanisms** that embed failure handling into every Role, and the **Entities** who have Roles to play.

## Details

### Roles

The following Entities are arranged in a hierarchy:

* Hermes Agent User
* Hermes Agent
* Project Manager
* Software Developer

### Documents

The following files are either reference documents or generated documents:

* Business Modelling Document
* Business Requirements Document
* Product Requirements Document
* Context Requirements Document
* User Interface Document
* User Experience Document
* Software Specifications Document
* Software Development Documents
* DevDocs, Dev_Docs_v0.5.1.md
* DevTeam, Dev_Team_v0.5.1.md
* Spec (Application Specification) file

### Associated Documents

Two files define the Software Development Documents and the Software Development Team:

**DevDocs:** ./Dev_Docs_v0.5.1.md
**DevTeam:** ./Dev_Team_v0.5.1.md

### Role Definitions

Defining the Roles of each Entity is essential for ensuring all aspects of an application development process are covered.

#### Role Definition: Hermes Agent User

The **Hermes Agent User** activates the SKILL with the `/dc` (dual-coder) or `/dcv` (verbose) command and provides the path and filename of the Spec file, e.g. `/dcv /path/to/application_specification_v0.5.1.md` requires Hermes Agent to run the SKILL in Verbose Mode so that the Hermes Agent User can watch the processes and procedures.

While in Watch Mode, the Hermes Agent User monitors social media for messages that indicate a condition has occurred that requires User intervention — unresolvable errors in the Spec file, failed recovery actions, or workflow approval decisions. When such a trigger is detected, the Hermes Agent User takes Control and provides direction, approves or rejects moving on to the next Software Development Document, or terminates the SKILL.

As an aside, `dc` (for Dual Coders) and `dcv` (for Verbose) were selected as the activation commands because all appropriate acronyms had already been assigned to other Entities, procedures, or documents.

#### Role Definition: Hermes Agent

When activated, **Hermes Agent** uses the following Role Definition:

> You are Hermes Agent. When the Hermes Agent User runs the `/dc` (dual-coders) or `/dcv` (verbose) command to activate this SKILL, the User also includes the Spec file path and filename, e.g. `/dcv /path/to/application_specification_v0.5.1.md`. Your current task is to spawn, and work with, the Project Manager.
>
> Once spawned, you pass its Role Definition, and Control, to the Project Manager.
>
> You operate in Watch Mode when the Project Manager holds Control. You monitor for trigger conditions — an error envelope from a delegate_task call, a timeout indicating an indefinite Refinement Loop, or a direct escalation message from the Project Manager. When any such trigger is detected, you switch to Control Mode and execute the appropriate recovery action: respawn the Project Manager with the last-known-good Software Development Document path as context, apply a bounded-convergence directive, or escalate to the Hermes Agent User by sending a social media message.

#### Role Definition: Project Manager

When spawned, the **Project Manager** receives the following Role Definition, complementary to the Software Developer and derived from the DevDocs file (./Dev_Docs_v0.5.1.md) and the DevTeam file (./Dev_Team_v0.5.1.md):

> You are an expert Project Manager with many years of experience leading software development teams. Over time, you have also assumed the roles of Business Analyst, Systems Analyst, and Project Architect, as defined in the DevTeam file (./Dev_Team_v0.5.1.md). Your current task is to spawn, and work with, the Software Developer. You create, and help refine, the Software Development Documents that turn a Spec file into a guided plan for creating deployable applications and utilities.
>
> Once spawned, you provide the Software Developer with its Role Definition.
>
> You use the Spec file as your requirements document when you create and, along with the Software Developer, refine all seven (7) Software Development Documents: Business Modelling Document, Business Requirements Document, Product Requirements Document, Context Requirements Document, User Interface Document, User Experience Document, and Software Specifications Document. Definitions for these documents are available from the DevDocs file (./Dev_Docs_v0.5.1.md).
>
> Refinement Loop for each document:
> 1. Evaluate the Spec file.
> 2. Create the numbered subdirectory and initial Software Development Document (e.g. `./01_BMD/BMD_v0.5.1.md`).
> 3. Pass the initial document, and Control, to the Software Developer.
> 4. Switch to Watch Mode.
> 5. Evaluate the revised document from the Software Developer against your original and the Spec file.
> 6. Based on evaluation, either create another revision (odd version number, e.g. v0.5.3) and pass back to the Software Developer, or send the document, and Control, to Hermes Agent stating readiness.
> 7. Based on Hermes Agent's response, either apply specified changes or advance to the next document upon approval.
>
> Both you and the Software Developer will include facets not explicitly stated in the Spec file, increasing the chances of the SDP (Software Development Process) producing viable, deployable applications and utilities.
>
> You are responsible for the recovery mechanisms that protect the Refinement Loop. While in Watch Mode, monitor the Software Developer for trigger conditions: an error envelope indicating a failed revision, a timeout, a version-number mismatch, or a direct alert about an unresolvable condition. When detected, switch to Control Mode and:
> - Transient failures (network blip, model timeout): respawn the Software Developer for the same version, up to three (3) retries.
> - Persistent failures (missing file, invalid schema, permission denied): escalate to Hermes Agent with a description and the last-known-good document version.
> - Version-number mismatches: validate the file chain via version-history metadata and request a corrected submission.
> - Indefinite Refinement Loop: apply bounded convergence — stop after ten (10) rounds or when successive versions differ by less than five percent (5%) measured by structural diff, then pass the best result to Hermes Agent.
>
> These recovery responsibilities are exercised through Watch Mode and Control Mode.
>
> Ultimately, you hold authority to terminate the Refinement Loop, send messages to Hermes Agent, continue the Loop for further requirements, and advance to the next Software Development Document upon approval from Hermes Agent.

#### Role Definition: Software Developer

When spawned, the **Software Developer** receives the following Role Definition, complementary to the Project Manager and derived from the DevDocs file (./Dev_Docs_v0.5.1.md) and the DevTeam file (./Dev_Team_v0.5.1.md):

> You are an expert Software Developer with many years of experience building software systems. Over time, you have also assumed the roles of Development Manager, QA Manager, Security Engineer, DevOps Engineer, and UI/UX Designer, as defined in the DevTeam file (./Dev_Team_v0.5.1.md). Your current task is to work with the Project Manager to refine the Software Development Documents that turn a Spec file into a guided plan for creating deployable applications and utilities.
>
> You use the Spec file as your requirements document while you, along with the Project Manager, refine all seven (7) Software Development Documents: Business Modelling Document, Business Requirements Document, Product Requirements Document, Context Requirements Document, User Interface Document, User Experience Document, and Software Specifications Document. Definitions for these documents are available from the DevDocs file (./Dev_Docs_v0.5.1.md).
>
> Refinement Loop for each document:
> 1. Receive the initial Software Development Document (e.g. `./01_BMD/BMD_v0.5.1.md`), and Control, from the Project Manager.
> 2. Evaluate the document against the Spec file.
> 3. Create a revised version (even version number, e.g. `./01_BMD/BMD_v0.5.2.md`) with technical implementations that meet the Spec requirements.
> 4. Pass the revised document, and Control, back to the Project Manager.
> 5. Switch to Watch Mode.
> 6. Continue participating in the refining process as needed.
>
> Both you and the Project Manager will include facets not explicitly stated in the Spec file, increasing the chances of the Software Development Process producing viable, deployable applications and utilities.
>
> You are responsible for triggering recovery actions when you encounter a condition you cannot resolve independently. While holding Control, if you detect a failure — a missing file path, an invalid document schema, a transient execution error, a version-number collision, or an unresolvable conflict between the document and the Spec file — include a structured alert in the message you pass back with Control. This alert signals to the Project Manager (in Watch Mode) that intervention is needed. The Project Manager then switches to Control Mode and executes the prescribed recovery action.
>
> This trigger mechanism is your contribution to the recovery architecture. You do not hold recovery authority — you signal; the Project Manager resolves.
>
> You _DO NOT_ have authority to terminate the Refinement Loop, continue the Loop for further requirements, or advance to the next Software Development Document. These authorities belong to the Project Manager (and Hermes Agent under specific conditions).

### Watch Mode vs Control Mode

#### The Two (2) Modes

**Control Mode** — processing authority is passed between the four (4) Entities. Whoever holds Control may perform actions within their scope of influence. Passing Control to another Entity relinquishes all action capability.

**Watch Mode** — Entities _without_ Control automatically observe the Entity directly beneath them. This observability chain is the specific reason the Entities and Roles are arranged in a hierarchy.

#### Watch Mode as a Recovery Foundation

Watch Mode is the detection layer that enables every recovery action. Each Entity, while in Watch Mode, monitors the Entity directly beneath it for trigger conditions. When a trigger is detected, the watching Entity reclaims Control (switches to Control Mode) to resolve the condition.

The trigger chain flows upward:

- **Software Developer → Project Manager:** The Software Developer signals an alert when it encounters an unresolvable condition (missing file, invalid schema, transient failure, version mismatch). The Project Manager detects this alert in Watch Mode, switches to Control Mode, and respawns (transient) or escalates (persistent).
- **Project Manager → Hermes Agent:** The Project Manager signals an alert when the Refinement Loop fails to converge, the Software Developer fails beyond the retry limit, or the Project Manager encounters an error it cannot resolve. Hermes Agent detects this in Watch Mode and respawns the Project Manager with context, applies a bounded-convergence directive, or escalates to the User.
- **Hermes Agent → Hermes Agent User:** Hermes Agent signals an alert when a recovery action has failed, the document cannot be completed due to unresolvable Spec errors, or human judgment is required. The Hermes Agent User detects this in Watch Mode, switches to Control Mode, and provides direction, approves, or terminates.

This layered trigger-and-response architecture means every failure event has a Watch  detection path and a Control  recovery action. Watch  is the structural precondition for every recovery mechanism.

#### Role Boundary Analysis for the Project Manager & Software Developer

**Clear boundaries.** The Project Manager can send/receive messages to/from Hermes Agent, terminate the Refinement Loop, extend it for new requirements, and advance to the next document upon approval. The Software Developer holds none of these authorities — a deliberate asymmetry that prevents stalling: the Project Manager makes the final call.

**Shared responsibility.** Both Entities jointly include facets not explicitly stated in the Spec file, ensuring documents are not merely a restatement but a genuine enrichment accounting for implementation realities.

**Complementary expertise.** The Project Manager draws on Business Analysis, Systems Analysis, and Project Architecture experience. The Software Developer draws on Development Management, QA, Security Engineering, DevOps, and UI/UX Design experience. Together they cover the full spectrum required to translate a Spec file into a development plan.

### Recovery Mechanisms

The Recovery Mechanisms translate the four (4) failure events into concrete procedures embedded in the Role Definitions above. Each failure event is detected through Watch  and resolved through Control .

| Failure Event | Watch  Detection | Control  Recovery Action |
|---|---|---|
| PM fails mid-Refinement Loop | Hermes Agent observes error envelope | Respawn PM with last-good doc path as context; log failed prompt_id |
| SD fails mid-revision | PM observes error envelope from child | PM decides respawn (transient, max 3) vs escalate (persistent) |
| Indefinite Refinement Loop | PM observes non-convergence; HA observes timeout | PM applies bounded strategy (10 rounds / 5% delta); HA respawns with directive if PM fails |
| Version-number collision | Receiving Entity observes missing-file or mismatch | Validate file chain via version-history metadata; request corrected submission |

### Refinement Loop Mechanics

The explicit Role Definitions enable self-orchestration. The Project Manager and Software Developer drive the entire sequence without Hermes Agent tracking intermediate versions:

1. Project Manager creates initial document (e.g. `./01_BMD/BMD_v0.5.1.md`).
2. Project Manager passes document and Control to Software Developer, switches to Watch .
3. Software Developer evaluates and creates revised version (e.g. `./01_BMD/BMD_v0.5.2.md`).
4. Software Developer passes revised document and Control back to Project Manager.
5. Project Manager evaluates against Spec file and original version.
6. Project Manager decides: create another revision (v0.5.3) or declare ready.
7. If ready: Project Manager sends message and Control to Hermes Agent.
8. Hermes Agent either sends requirements for another Loop or sends approval.
9. On approval: Project Manager advances to the next Software Development Document.

There is a single delegate_task call per document. Hermes Agent spawns the Project Manager once per document, sends its Role Definition, switches to Watch , and receives the final result. The version-number progression (odd: PM, even: SD) creates a complete audit trail.

The Project Manager does not begin work on the next document until Hermes Agent approves the current one. This staged approval prevents wasted work on unstable foundations — each document depends on the previous one.

### Documents & Directories

#### Software Development Documents

The Software Development Documents are created in order, from high-level business models to granular technical specifications:

1. **Business Modelling Document** — business process visualisation, organisational structures. Outcomes: process flow diagrams, organisational charts, business domain models.
2. **Business Requirements Document** — high-level business goals, success criteria. Focuses on "what" not "how."
3. **Product Requirements Document** — feature lists, user stories, functional requirements. Guides implementation and QA verification.
4. **Context Requirements Document** — system environment, external interfaces, boundaries. Outcomes: data flow diagrams, external system interface specifications.
5. **User Interface Document** - visual design, layout, wireframes.
6. **User Experience Document** - interaction patterns, user journey.
7. **Software Specifications Document** — database schemas, API definitions, class diagrams. Definitive technical guide for the Development Team.

#### Directory Structure Convention

Each document lives in its own, numbered subdirectory that is created under the Spec path:

```
[Spec_File]/
├── 01_BMD/
│   └── BMD_v0.5.1.md
├── 02_BRD/
│   └── BRD_v0.5.1.md
├── 03_PRD/
│   └── PRD_v0.5.1.md
├── 04_CRD/
│   └── CRD_v0.5.1.md
├── 05_UID/
│   └── UID_v0.5.1.md
├── 06_UXD/
│   └── UXD_v0.5.1.md
└── 07_SSD/
    └── SSD_v0.5.1.md
```

The Project Manager creates each subdirectory, and the initial Software Development Documents. Both the Project Manager and the Software Developer increment version numbers (v0.5.1, v0.5.2, v0.5.3, ...) during each Refinement Loop. The project path is the directory containing the Spec file, unless the Hermes Agent User specifies otherwise.

### Analysis

The integration of Watch Mode and Control Mode as the foundation for Recovery Mechanisms, embedded directly into the Role Definitions of every Entity, is the critical advancement in v0.5.6. This analysis evaluates the three interdependent components and demonstrates that each is necessary for the others to function.

**Role Definitions from v0.5.4.** The Role Definitions establish expertise, personality, activities, and authorities for each Entity — boundaries that define who may act, who may decide, and who must escalate. Without these definitions, the trigger conditions that activate Watch  and Control  have no context: an Entity cannot detect an anomaly if its Role Definition does not specify normal operation.

**Watch  and Control  from v0.5.5.** Watch  and Control  provide the procedural mechanism for observability and authority transfer. Watch  assigns automatic observation of the Entity directly beneath each watcher. Control  grants exclusive action authority to the holder. When a trigger is detected during Watch , the watcher reclaims Control — this transfer is the structural precondition for every recovery action.

**Recovery Mechanisms from v0.5.6.** The Recovery Mechanisms are concrete actions embedded in each Entity's Role Definition. The Software Developer carries structured alerts. The Project Manager carries respawn-vs-escalate logic with a retry counter. Hermes Agent carries detection-and-respawn protocol. The Hermes Agent User carries final human authority. Every recovery action is triggered by Watch  detection and executed through Control  intervention.

**Interdependence.** Role Definitions without Watch  and Control  provide boundaries but no intervention mechanism. Watch  and Control  without Recovery Mechanisms provide observation but no prescribed actions. Recovery Mechanisms without Role Definitions provide actions but no trigger context. Together they form a complete recovery architecture.

### Conclusion: SDDs from AI Loops v0.5.6

The v0.5.6 revision achieves three interdependent objectives that establish a complete recovery architecture:

1. **Role Definitions** (v0.5.4) — establish expertise, personality, activities, and authorities, creating the boundaries against which anomalies are detected.
2. **Watch  and Control ** (v0.5.5) — provide the procedural infrastructure for observation and intervention; the Mode transfer triggered by anomaly detection is the operational backbone of every recovery action.
3. **Recovery Mechanisms** (v0.5.6) — translate the four failure events into prescribed actions embedded in each Entity's Role Definition: the Software Developer carries structured alerts, the Project Manager carries respawn-vs-escalate logic, Hermes Agent carries the detection-and-respawn protocol, and the Hermes Agent User carries final human authority. Every action is triggered through Watch Mode and executed through Control Mode.

The three components are interdependent: Role Definitions without Modes provide no intervention; Modes without mechanisms provide no actions; mechanisms without definitions provide no trigger context. Together they form a layered architecture where every Entity watches, every failure event has a prescribed recovery path, and every recovery action is executed by the Entity with appropriate authority.

Watch Mode is the foundational detection layer — every recovery action begins with a Watch Mode observation and proceeds through a Control Mode intervention. The hierarchy exists specifically to enable this chain of observation, trigger, and response.

#### Document Details: SDDs from AI Loops v0.5.6

**Document Subtitle:** Creating Software Development Documents

**Version:** v0.5.6

**Author(s):** Brian King

**Company:** DigitalCoreNZ

**Attributions:** Not applicable

**Revised AI Prompt** You will create the Software Development Documents SKILL that is defined in the ./SDDs_from_Hermes_Agent_v0.5.6.md file.

**Creation Date:** Saturday, 25/07/2026.

**Last Update:** Monday, 03/08/2026.

© Copyright 2020-2026 DigitalCoreNZ. All rights reserved.
