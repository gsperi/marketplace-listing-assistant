# Roadmap

## Current Phase

Architecture Definition and Domain Modeling

Status: Completed

Next Phase:
Development Environment

### Completed Objectives

Define the high-level software architecture before implementing the first production components.

* define the first user workflow;
* identify the core system responsibilities;
* define the Product Context model;
* define the first vertical slice;
* identify the initial system boundaries.

---

## Next Milestones

### Development Environment

**Priority:** High

Goal:

Prepare a reproducible development environment.

Expected deliverables:

* backend bootstrap;
* frontend bootstrap;
* Docker environment;
* PostgreSQL setup;
* first runnable application.

---

### Core Infrastructure

**Priority:** High

Goal:

Build the initial application skeleton.

Expected deliverables:

* FastAPI bootstrap;
* React bootstrap;
* Docker integration;
* PostgreSQL connection;
* health endpoint;
* first API endpoint.

---

### First Vertical Slice

**Priority:** High

DraftProductContext generation

↓

User validation

↓

ProductContext generation

Goal:

Validate the complete workflow with the smallest useful implementation.

Expected deliverables:

* image upload;
* mocked product recognition;
* DraftProductContext generation;
* manual review;
* basic listing generation;
* export.

---

### Product Intelligence

**Priority:** Medium

Goal:

Replace mocked services with real implementations.

Possible deliverables:

* AI-assisted product recognition;
* PriceSuggestion engine;
* market price estimation;
* keyword generation;
* improved listing quality.

---

## Future Ideas

Ideas intentionally postponed.

Current list:

* Quick Mode
* Batch Listing Creation
* Text-based product input
* Barcode input
* URL-based product import
* Marketplace API integration
* Browser extension
* Mobile application
* Learning layer based on user confirmations
* Local AI models
* User search history
* Media library
* Saved product analyses
* Automatic finish recognition using computer vision: the system may identify foil patterns and surface finishes (e.g. Reverse Holo, Cosmos Holo, Cracked Ice) from product images.

This feature is intentionally excluded from the MVP.
---

## Parking Lot

Ideas that may become relevant in future iterations but are intentionally excluded from the current roadmap.

Examples:

* marketplace ranking analysis;
* automatic repricing;
* advanced analytics;
* recommendation engine;
* inventory management.

