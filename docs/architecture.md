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

The system attempts to analyze the uploaded product image.

The analysis phase may consist of multiple internal steps, including:

- text extraction;
- attribute recognition;
- interpretation according to the expected card structure;
- draft product context generation.

At this stage, the result is only a system-generated proposal.

The analysis is not considered authoritative until reviewed by the user.

### Attribute Recognition

Not every product attribute can be reliably extracted from an image.

Some attributes may require user confirmation or manual input, even when the product itself has been correctly identified.

For example, trading card games often distinguish between:

- printed rarity (editorial classification);
- finish (physical surface treatment).

These concepts are independent in several games, such as Pokémon, and may require different recognition strategies.

The MVP intentionally allows users to manually confirm or provide these attributes instead of attempting automatic recognition.

Future versions may introduce computer vision techniques to recognize surface finishes when technically reliable.
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

## Initial Domain Object Graph

The current MVP workflow starts from a single uploaded media asset.

For the MVP, the relationship between `AnalysisSession` and `MediaAsset` is intentionally restricted to one media asset per session.

```text
AnalysisSession
│
├── MediaAsset (1)
│
├── TextExtractionResult (0..1)
│
├── DraftProductContext (0..1)
└── ProductContext (0..1)
        │
        ├── PriceSuggestion (0..1)
        │
        └── ListingDraft (0..N)



### Notes
* `AnalysisSession` represents the user-started analysis workflow.
* `MediaAsset` represents the uploaded file used as input.
* `TextExtractionResult` represents the raw or semi-structured text extracted from the media asset.
* `DraftProductContext` represents the system-generated structured proposal derived from the extracted text.
* `ProductContext` represents the user-reviewed and normalized product information.
* `PriceSuggestion` is generated from a ProductContext.
* `ListingDraft` is generated from a ProductContext.

### MVP Constraint

```text
1 AnalysisSession = 1 MediaAsset = 1 product candidate
```

Multiple media assets per session are intentionally excluded from the MVP.

Supporting multiple assets would require the system to determine whether the uploaded files represent:

* the same product;
* different products;
* front and back images;
* additional details;
* accidental uploads.

Future support for multiple media assets may require an explicit concept such as `MediaGroup` or `ProductEvidence`.



## Main Architectural Concepts

### DraftProductContext

`DraftProductContext` represents the system-generated structured proposal created during the analysis workflow.

It is derived from the extracted text and other available recognition signals.

Its purpose is to help identify the product.

It should contain only the information needed to determine what the product is, not every possible product attribute.

Typical fields may include:

- product type;
- card name;
- set or expansion;
- collector number;
- language;
- selected disambiguation attributes.

`DraftProductContext` is not authoritative.

It may contain incomplete, uncertain or incorrect values.

The user may review, correct or reject it before a final `ProductContext` is created.

The key distinction is:

```text
DraftProductContext identifies.
ProductContext describes.
```

### Product Context
`ProductContext` represents the user-reviewed and normalized product information.

It is created after the user has confirmed or corrected the `DraftProductContext`.

Unlike `DraftProductContext`, which exists to identify the product, `ProductContext` exists to describe the product with enough detail to support downstream workflows such as price estimation and listing generation.
Product Context is the central data structure of the system.

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
}
```

The exact structure will be refined later.

For the MVP, `ProductContext` should contain the minimum information required to generate a useful listing and support price estimation:

- card name;
- set name;
- set code;
- collector number;
- language;
- printed rarity;
- finish;
- condition.

`condition` is manually provided or confirmed by the user.

Future versions may add grading-related fields, such as:

- grading company;
- grade value;
- certification number.

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

