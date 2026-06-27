# Architecture

## Purpose

This document describes the high-level architecture of Marketplace Listing Assistant.

The goal is to define the main system responsibilities, workflows and architectural principles before introducing implementation details.

This document should remain concise and evolve only when architectural decisions become stable.

---

## Architecture Status

**Current Phase:** M1 - Software Architecture

**Maturity:** Draft

The project is currently defining the first high-level workflow and the core system boundaries.

Implementation details may change.

---

## High-Level Product Workflow

The application starts from a product provided by the user.

For the MVP, the primary input source is a product image.

```text
Acquire Product Image
        ↓
Analyze Product Image
        ↓
Build Normalized Product Context
        ↓
User Review / Correction
        ↓
Choose Objective
        ↓
Estimate Value and/or Generate Listing
        ↓
Review Output
        ↓
Export Marketplace-Ready Content
```

The first goal is not to automate the entire selling process.

The first goal is to help the user move from product information to a better listing with less repetitive work.

---

## Core Workflow

### 1. Acquire Product Image

The user provides one or more product images.

For the MVP, supported input is limited to image upload.

Future input sources may include:

* text search;
* barcode;
* external product URL;
* marketplace URL;
* clipboard input;
* mobile camera input.

These sources are intentionally excluded from the initial MVP.

---

### 2. Analyze Product Image

The system attempts to identify or classify the product using the uploaded image.

At this stage, the result is only a proposal.

The system should provide:

* detected product information;
* confidence score;
* possible alternatives;
* missing or uncertain fields.

The analysis is not considered authoritative until reviewed by the user.

---

### 3. Build Normalized Product Context

After image analysis, the system builds a normalized product context.

The normalized product context represents the product in a source-independent format.

From this point onward, the system should not depend on how the product was originally provided.

Example input sources:

```text
Image
Text
Barcode
URL
External database
```

All sources should eventually produce the same kind of normalized product context.

---

### 4. User Review and Correction

The user reviews the proposed product context.

The user may:

* accept the proposed result;
* correct wrong fields;
* provide missing information;
* manually override the detected product.

This is a core part of the workflow, not an exception.

The software proposes.

The user confirms.

---

### 5. Choose Objective

Once the product context is available and reviewed, the user chooses what to do next.

Initial supported objectives:

* estimate market value;
* generate marketplace listing;
* do both.

The system should not force the user to choose an objective before the product context exists.

---

### 6. Estimate Value

The system estimates a realistic market value or price range for the product.

For the MVP, this may start as a simplified or manually assisted feature.

Future versions may use:

* marketplace sold listings;
* public price databases;
* historical price trends;
* user-defined pricing rules.

The estimate should expose uncertainty where appropriate.

---

### 7. Generate Listing

The system generates marketplace-ready listing content.

The output may include:

* title;
* description;
* keywords;
* suggested category;
* condition notes;
* warnings;
* listing quality score.

The generated content should be editable before export.

---

### 8. Review Output

The user reviews the generated listing or price suggestion.

The user may modify the result before exporting it.

The system should support expert users who want control, not hide relevant information from them.

---

### 9. Export Content

The MVP will export generated listing content manually.

Supported export may include:

* copy to clipboard;
* plain text;
* structured JSON;
* marketplace-specific text templates.

Direct marketplace publishing is outside the MVP scope.

---

## Main Architectural Concepts

### Product Context

Product Context is the central data structure of the system.

It represents the product after acquisition and normalization.

It should contain:

* product identity;
* product category;
* known attributes;
* uncertain attributes;
* source information;
* confidence level;
* user validation status.

Example:

```json
{
  "productType": "trading_card",
  "name": "SS4 Vegito, Sparking Offense/Defense",
  "game": "Dragon Ball Super Card Game Masters",
  "set": "Beyond Generations",
  "setCode": "BT24",
  "cardNumber": "122",
  "rarity": "SPR",
  "language": "English",
  "condition": "Near Mint",
  "source": "image_analysis",
  "confidence": 0.92,
  "validatedByUser": false
}
```

The exact structure will be refined later.

---

### Listing Draft

A Listing Draft represents the content generated for marketplace publication.

It should be derived from a validated or partially validated Product Context.

It may contain:

* title;
* description;
* keywords;
* price suggestion;
* marketplace-specific fields;
* quality score;
* warnings;
* generated alternatives.

---

### Price Suggestion

A Price Suggestion represents a proposed value or selling range.

It may contain:

* minimum price;
* suggested price;
* maximum price;
* currency;
* confidence;
* source references;
* notes.

For the MVP, the pricing model may be simple and evolve later.

---

### Listing Score

Listing Score evaluates the quality of a generated or manually edited listing.

It may consider:

* completeness;
* keyword coverage;
* title quality;
* description clarity;
* price consistency;
* marketplace-specific requirements.

The score must be explainable.

A numeric value without explanation is not useful to a power user.

---

## High-Level Components

### User Interface

Responsible for:

* accepting product images;
* showing analysis results;
* allowing user corrections;
* showing price and listing outputs;
* exporting content.

The UI decides how much information to show.

The UI does not own the core business logic.

---

### API Layer

Responsible for:

* exposing application use cases;
* validating requests;
* coordinating services;
* returning structured responses to the UI.

The API layer should remain thin.

Business logic should live in the application/core layer.

---

### Core Engine

Responsible for:

* managing Product Context;
* coordinating listing generation;
* coordinating price estimation;
* calculating listing score;
* applying marketplace-independent rules.

The Core Engine should always produce the highest useful level of information available.

Different UIs may display different subsets of this information.

---

### Product Recognition Service

Responsible for:

* analyzing uploaded product images;
* proposing product identity and attributes;
* returning confidence and alternatives.

The recognition result is not final until reviewed by the user.

---

### Pricing Service

Responsible for:

* estimating market value;
* exposing confidence and uncertainty;
* providing price-related notes.

This component may initially be mocked or simplified.

---

### Listing Generation Service

Responsible for:

* generating title;
* generating description;
* suggesting keywords;
* adapting content to target marketplaces.

The first version should prioritize quality and explainability over automation.

---

### Storage Layer

Responsible for:

* storing product contexts;
* storing listing drafts;
* storing uploaded images or image references;
* storing generated outputs.

Storage requirements will be refined after the first vertical slice.

---

## Architectural Principles

### Source Independence

The system should not depend on the original input source after Product Context has been built.

The same downstream workflow should work regardless of whether the product came from:

* an image;
* manual input;
* barcode;
* URL;
* external database.

For the MVP, only image input is implemented.

---

### User-Controlled Automation

Automation should reduce repetitive work without removing control from the user.

The system should support correction, review and override.

This is especially important because the initial target user is an experienced marketplace seller.

---

### Confidence-Aware Workflow

Recognition, pricing and listing suggestions may be uncertain.

The system should expose uncertainty instead of hiding it.

Low-confidence results should trigger review or correction, not silent failure.

---

### Explainable Output

Scores, warnings and suggestions should be understandable.

The system should avoid producing unexplained numeric values or opaque recommendations.

Power users need to know why something is being suggested.

---

### MVP Simplicity

The MVP should avoid:

* direct marketplace publishing;
* browser automation;
* automatic repricing;
* persistent learning from user corrections;
* complex multi-user features;
* advanced analytics.

These features may be introduced later only if they support the core product value.

---

## Initial System Boundary

Inside the system:

```text
Image upload
Product analysis
Product context normalization
User correction
Price estimation
Listing generation
Listing scoring
Manual export
```

Outside the MVP:

```text
Marketplace publishing
Marketplace account management
Inventory management
Automatic repricing
Historical analytics
Persistent learning layer
Mobile application
Browser extension
```

---

## First Vertical Slice

The first vertical slice should prove the core workflow with the smallest useful implementation.

Target flow:

```text
Upload image
        ↓
Return mocked or simplified product context
        ↓
Allow user confirmation/correction
        ↓
Generate basic listing draft
        ↓
Show listing score
        ↓
Export content
```

This slice should validate the system shape before investing in complex recognition, pricing or marketplace integrations.

---

## Open Questions

* What exact fields are required for the first Product Context model?
* Which product category should be used for the first vertical slice?
* Should the first recognition service be mocked, AI-based, or manually assisted?
* Which marketplace template should be implemented first?
* Which scoring rules are useful enough for the MVP?
* What should be stored permanently in the first version?

These questions should be resolved before or during the first vertical slice.

