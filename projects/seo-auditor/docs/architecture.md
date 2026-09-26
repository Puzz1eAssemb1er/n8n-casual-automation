# Architecture

## 1. System context

The SEO Auditor is a lightweight workflow automation service implemented in n8n.

A user provides a target website URL. The workflow discovers internal URLs, crawls them, extracts selected on-page metadata, records both successful and failed requests, performs deterministic checks, sends the resulting dataset to Claude for interpretation and delivers an HTML report by email.

## 2. Architectural principle

The system follows:

\`\`\`
Acquire facts → normalize data → validate / classify → aggregate → interpret → deliver
\`\`\`

The key principle is:

> **crawl status is part of the data model.**

A failed HTTP request is not an absence of data. It is a meaningful processing result.

## 3. Logical layers

### Orchestration

n8n controls execution order, item fan-out, branching, merging, transformation and delivery.

### Data acquisition

HTTP Request nodes retrieve the target HTML pages.

### Data extraction

The HTML node extracts:

- \`title\`;
- \`meta[name="description"]\`;
- \`h1\`.

### Deterministic processing

Transformation nodes:

- normalize URLs;
- remove duplicates;
- create stable page result records;
- classify request failures;
- calculate SEO checks;
- aggregate the dataset.

### AI interpretation

Claude receives structured facts and explains the findings.

Claude should not determine whether a page existed, calculate crawl coverage or act as the primary source of basic SEO flags.

### Presentation and delivery

The analysis is formatted as HTML and sent through Gmail.

## 4. Target data flow

\`\`\`
Configuration
    ↓
Homepage
    ↓
Link discovery
    ↓
URL normalization
    ↓
URL deduplication
    ↓
Page crawler
    ├── Success → Metadata extraction → Page Result ─┐
    └── Error   → Error normalization  → Error Result ─┤
                                                       ↓
                                                 Merge Results
                                                       ↓
                                              Deterministic checks
                                                       ↓
                                                 Audit dataset
                                                       ↓
                                                  Claude AI
                                                       ↓
                                                 HTML report
                                                       ↓
                                                   Email
\`\`\`

## 5. Why Merge Results exists

The crawler creates two execution streams:

- successful page responses;
- failed page requests.

The Merge stage converts these streams into **one complete crawl result set**.

For the observed test case:

\`\`\`
65 success + 5 errors = 70 crawl results
\`\`\`

The output can therefore distinguish:

- successfully inspected pages;
- pages that could not be inspected;
- total crawl coverage.

n8n's Merge node supports **Append** mode, which keeps items from multiple inputs in sequence. This matches the requirement to preserve both result streams. urln8n Merge node documentationhttps://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.merge/\u0023append\u002dmode\u0029

## 6. Responsibility boundary

| Component | Responsibility |
| --- | --- |
| n8n | Orchestration, transport, extraction, deterministic processing, branching, merge and delivery |
| Target website | Source of HTML and link structure |
| Claude | Interpretation, summary and recommendations |
| Gmail | Report transport |

## 7. Failure boundary

The design distinguishes:

**Workflow-level failure**

A critical dependency or configuration prevents the audit from continuing.

**Page-level failure**

One page cannot be fetched or processed while the audit itself can continue.

A page-level failure is represented as an item in the audit dataset.

## 8. Target observability

The workflow should expose at least:

- discovered URL count;
- crawl target count;
- success count;
- error count;
- error categories;
- retry count;
- final analyzed count;
- report delivery status.
