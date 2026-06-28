# Decisions

This document describes the current project conventions and major technical decisions.

Git already preserves the historical evolution of these choices, so this file should describe the current agreed state of the project, not every previous discussion.

---

## Project Conventions

### Git Workflow

The default branch is `master`.

`master` should remain clean and represent a stable state of the project.

For documentation-only changes during the initial planning phase, changes may be committed directly to `master`.

For software development, each feature should be developed on a dedicated branch.

Recommended branch naming:

```text
docs/<topic>
backend/<feature>
frontend/<feature>
core/<feature>
infra/<feature>
experiment/<topic>
```

Examples:

```text
backend/fastapi-bootstrap
frontend/listing-draft-page
core/listing-score-engine
infra/docker-compose
experiment/ocr-provider-test
```

Branches should have one clear goal.

A branch should not mix unrelated changes.

---

### Commit Policy

Commits should represent meaningful logical changes.

The project should avoid excessive commits created only because a file was edited.

During exploratory or decision-making phases, changes should be consolidated before committing.

Bad examples:

```text
fix
fix2
update
changes
remove file
add file again
```

Good examples:

```text
docs: define initial project vision
docs: define project conventions
backend: bootstrap FastAPI application
infra: add local docker compose
core: add listing score model
```

Before merging feature branches, the local history may be cleaned using interactive rebase when useful.

The goal is to keep the Git history readable and useful.

---

### Documentation Policy

Documentation must be useful, concise and easy to maintain.

The project should avoid unnecessary fragmentation.

Before creating a new document, check whether the information can fit into an existing one.

Each piece of information should have one clear home.

Current documentation structure:

```text
docs/
├── diagrams/
├── architecture.md
├── roadmap.md
├── vision.md
└── decisions.md
```

Document responsibilities:

```text
vision.md        Why the project exists.
architecture.md How the system is designed.
roadmap.md      What will be built, in which order, and why.
decisions.md    Project conventions and stable technical decisions.
diagrams/       PlantUML diagrams referenced by the documents.
```

The following documents are intentionally avoided for now:

```text
data-model.md
workflows.md
adr/
```

Their content should be included in `architecture.md` or `decisions.md` until there is a strong reason to split them.

---

### Project Philosophy

The project follows a divide-and-conquer approach.

Large problems should be decomposed into smaller and more manageable sub-problems.

Each task should be evaluated using:

```text
Priority
Value
Complexity
Dependencies
```

Not everything can be high priority.

The project should avoid treating all features as equally urgent.

Since there is no external client or strict deadline, the project should prefer clarity, correctness and maintainability over artificial pressure.

---

### Milestone Strategy

The project should use approximate timelines only as orientation, not as hard deadlines.

Milestones should be defined as vertical slices.

A vertical slice should produce something demonstrable across multiple layers of the stack, even if internally simplified.

Example:

```text
Frontend
↓
Backend API
↓
Database
↓
Mocked AI logic
↓
Generated listing output
```

The preferred approach is to build small working flows instead of isolated technical modules that cannot be tested end-to-end.

---

### Engineering Principles

Start simple.

Avoid premature architecture.

Avoid premature optimization.

Avoid introducing infrastructure before there is a real need.

Prefer replaceable components.

Prefer explicit decisions over implicit assumptions.

When possible, validate assumptions with practical tests.

The first goal is to prove that the system can generate better marketplace listings, not to build a complete marketplace management platform.

### User-Centric Design

The project is primarily designed for experienced marketplace users.

The initial goal is not to teach users how to create listings, but to reduce the amount of repetitive work while preserving the quality of their listings.

The software should automate repetitive tasks without removing control from the user.

Future simplified workflows for less experienced users may be introduced only after the core experience has been validated.

---

### Core Engine First

The business logic should remain independent from the user interface.

The Core Engine should always produce the highest level of information available, including:

* listing title;
* description;
* keywords;
* quality score;
* suggested price range;
* warnings;
* alternative suggestions;
* confidence indicators.

Different user interfaces may expose different subsets of this information depending on the target user.

The amount of information shown is a responsibility of the UI, not of the Core Engine.

---

### Feature Growth

New ideas should not interrupt the current milestone.

When a useful feature is identified but does not contribute to the current objective, it should be added to the project roadmap under a future milestone.

The project follows the principle:

> Capture ideas immediately.
> Implement them only when their priority arrives.

This approach helps preventing feature creep while preserving valuable ideas for future iterations.


---

## Architecture Decisions

### Backend Technology

The backend will initially be built using Python and FastAPI.

Reasoning:

```text
Python has a strong ecosystem for AI, OCR, data processing and automation.
FastAPI provides a simple way to build typed HTTP APIs.
The initial project goal benefits more from fast experimentation than from enterprise-heavy structure.
```

Status: accepted.

---

### Database

PostgreSQL will be the primary database.

Reasoning:

```text
It is reliable, well-known, locally available and suitable for structured product, listing and scoring data.
It also leaves room for future full-text and analytical features.
```

MariaDB is available locally but is not part of the initial stack.

Status: accepted.

---

### Frontend

The frontend will initially use React or Next.js.

Reasoning:

```text
The product needs a practical UI for uploading images, reviewing generated listings and exporting content.
A pure CLI would be useful for testing but not sufficient for validating the actual user experience.
```

Status: accepted.

---

### Documentation Diagrams

PlantUML will be used for technical diagrams.

Reasoning:

```text
It is already available, text-based, versionable and suitable for architecture, workflow and sequence diagrams.
```

Status: accepted.

---

### Marketplace Integration

The initial version will not publish directly to marketplaces.

The first target is manual export of optimized listing content.

Reasoning:

```text
Marketplace APIs and browser automation can introduce fragility and platform-specific risk.
The core value of the product must first be validated through listing generation, keyword suggestions and quality scoring.
```

Status: accepted.

### Product Input Strategy
The Core Engine should work from a normalized Product Context.

```text
The initial UI will create this context from images only.
Future input sources may create the same context from text, URLs, barcodes, or external data.
```

### Product Context

...

### AI Collaboration Model

...

### Recognition Failure Strategy

...

### Media Asset Cardinality

For the MVP, an AnalysisSession supports exactly one MediaAsset.

Although the model may evolve to support multiple media assets in the future, the first implementation intentionally avoids this complexity.

Multiple media assets introduce ambiguity:

- they may represent the same product;
- they may represent different products;
- they may be front/back images;
- they may produce conflicting recognition results.

The initial workflow follows the rule:

```text
1 AnalysisSession = 1 MediaAsset = 1 product candidate
Future support for multiple media assets will require an explicit concept such as MediaGroup or ProductEvidence.
```
