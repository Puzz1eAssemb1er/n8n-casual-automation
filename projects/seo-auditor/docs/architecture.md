# Architecture

## Context

The workflow is a lightweight SEO-audit service implemented as an n8n pipeline. A user supplies a website URL; the workflow discovers internal pages, collects selected on-page metadata, sends a bounded dataset to Claude for interpretation and delivers the resulting report by email.

## Processing layers

### 1. Orchestration
n8n controls execution order, item fan-out, transformations, model binding and delivery.

### 2. Data acquisition
HTTP Request nodes retrieve HTML from the target site. HTML extraction nodes convert markup into structured fields.

### 3. Deterministic transformation
JavaScript nodes filter links, normalize URLs and aggregate page-level records.

### 4. AI analysis
Claude receives the consolidated metadata dataset. The model is used for interpretation and recommendations, not for primary HTML extraction.

### 5. Presentation and delivery
The LLM response is transformed into simple HTML and passed to Gmail as the delivery channel.

## Logical data contracts

| Object | Key fields | Purpose |
|---|---|---|
| Configuration | `baseUrl` | Target website |
| Discovery item | `links` | Raw `href` extracted from homepage |
| Crawl target | `fullUrl` | Normalized page URL |
| Page record | `title`, `description`, `h1` | SEO metadata |
| AI input | `report` | Consolidated audit dataset |
| Delivery payload | `html` | Email-ready report |

## End-to-end flow

```text
Manual trigger
  ↓
Target URL
  ↓
Homepage fetch
  ↓
Link extraction
  ↓
Split into items
  ↓
Internal-link filter
  ↓
Deduplication
  ↓
Absolute URL creation
  ↓
Page crawl
  ↓
SEO metadata extraction
  ↓
Report aggregation
  ↓
Claude analysis
  ↓
HTML formatting
  ↓
Email delivery
```

## Responsibility boundary

**n8n:** orchestration, transport, extraction and deterministic transformation.  
**Target website:** source of HTML and link structure.  
**Claude:** semantic interpretation and recommendations.  
**Gmail:** output delivery channel.
