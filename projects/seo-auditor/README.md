# n8n SEO Auditor

> An n8n-based on-page SEO auditing pipeline that discovers internal URLs, crawls pages, extracts SEO metadata, preserves crawl failures, performs deterministic checks and uses Claude for interpretation and recommendations.

## Project status

**Current stage:** MVP workflow + v2 architecture in progress.

The current workflow already performs homepage discovery, internal-link filtering, deduplication, page crawling, metadata extraction, Claude analysis, HTML formatting and email delivery.

During testing the crawler produced:

\`\`\`
70 crawl targets
├── 65 success
└── 5 error
\`\`\`

The next implementation step is to preserve those 5 failed pages as first-class audit results instead of losing them before the final report.

The repository therefore separates:

- the **current executable workflow export** in \`workflows/seo-auditor.json\`;
- the **target v2 architecture and implementation plan** documented in \`docs/\`.

## Business problem

A manual SEO audit requires repetitive work:

1. collect URLs;
2. open pages;
3. inspect \`Title\`, \`meta description\` and \`H1\`;
4. identify missing, duplicated or suspicious metadata;
5. prepare a report.

The workflow converts this into a repeatable pipeline with explicit inputs, processing stages, error handling and automated delivery.

## Input / output

**Input**

- Target website URL (\`baseUrl\`).

**Output**

A structured audit containing:

- total discovered URLs;
- successful and failed crawl counts;
- page-level \`Title\`, \`Description\` and \`H1\` data;
- deterministic SEO findings;
- failed URL records with error information;
- Claude-generated interpretation and recommendations;
- HTML email report.

## Target architecture

\`\`\`mermaid
flowchart LR
    A[Manual Trigger] --> B[Audit Configuration]
    B --> C[Download Homepage]
    C --> D[Extract Links]
    D --> E[Normalize URLs]
    E --> F[Deduplicate]
    F --> G[Download Pages]

    G -->|Success| H[Extract Metadata]
    G -->|Error| I[Normalize Error]

    H --> J[Page Result]
    I --> K[Error Result]

    J --> L[Merge Results]
    K --> L

    L --> M[Deterministic SEO Checks]
    M --> N[Structured Audit Dataset]
    N --> O[Claude Analysis]
    P[(Claude Model)] -.-> O
    O --> Q[HTML Report]
    Q --> R[Email Delivery]
\`\`\`

## Processing stages

| Stage | Responsibility | Result |
| --- | --- | --- |
| Configuration | Define target and runtime parameters | Audit configuration |
| Discovery | Download homepage and extract links | Raw URL candidates |
| URL processing | Normalize and deduplicate URLs | Unique crawl targets |
| Crawling | Request every target | Success / Error |
| Success handling | Extract metadata and build stable result | \`status=success\` |
| Error handling | Preserve failed URL and error information | \`status=error\` |
| Merge | Combine both result streams | Complete crawl dataset |
| Deterministic audit | Calculate SEO facts | Machine-checkable findings |
| AI analysis | Explain findings and recommendations | Human-readable analysis |
| Delivery | Render and send report | HTML email |

## Why the Error branch matters

The observed run demonstrates a data-completeness problem:

\`\`\`
70 targets
├── 65 successful
└── 5 failed
\`\`\`

A report built only from the 65 successful pages cannot reliably answer how many URLs were actually checked.

The target design therefore follows:

\`\`\`
Success → Page Result ─┐
                       ├→ Merge Results → Complete Audit Dataset
Error   → Error Result ┘
\`\`\`

This establishes a simple invariant:

\`\`\`
successCount + errorCount = crawlTargetCount
\`\`\`

## Deterministic vs AI responsibility

**n8n / deterministic layer**

- URL normalization;
- deduplication;
- crawl execution;
- success / error status;
- HTTP and error metadata;
- missing-field checks;
- character counts;
- duplicate detection;
- summary statistics.

**Claude**

- interpret findings;
- summarize the audit;
- explain patterns;
- formulate recommendations.

Claude is therefore an interpretation layer, not the source of truth for basic counts or boolean SEO checks.

## Current scope

### Included

- homepage link discovery;
- internal-link filtering;
- URL deduplication;
- page crawling;
- \`Title\` extraction;
- \`meta description\` extraction;
- \`H1\` extraction;
- AI-assisted interpretation;
- HTML email report.

### Next implementation scope

- preserve failed crawl targets;
- common PageResult contract;
- Merge / Append of success and error records;
- deterministic SEO rules;
- structured audit dataset;
- bounded retry policy.

### Future scope

- recursive crawling;
- \`robots.txt\` / \`sitemap.xml\`;
- canonical / noindex / nofollow checks;
- redirect analysis;
- image \`alt\` coverage;
- Core Web Vitals;
- persistent storage.

## Repository structure

\`\`\`
projects/seo-auditor/
├── README.md
├── .gitignore
├── workflows/
│   └── seo-auditor.json
├── docs/
│   ├── architecture.md
│   ├── workflow.md
│   ├── data-contract.md
│   ├── error-handling.md
│   ├── implementation-plan.md
│   └── technical-notes.md
└── examples/
    └── report-schema.md
\`\`\`

## Installation

1. Import \`workflows/seo-auditor.json\` into n8n.
2. Configure the Claude/Anthropic credential.
3. Configure the Gmail credential.
4. Set the target URL in \`🌐 Адрес сайта\`.
5. Run \`▶️ Запуск аудита\`.

The public export must not contain credentials, tokens or personal recipient data.

## Portfolio value

The project demonstrates business-process decomposition, workflow orchestration, HTTP integration, HTML parsing, JavaScript transformation, explicit success/error paths, data contracts, deterministic rule design, LLM integration and automated reporting.

The key portfolio artifact is the architecture decision to make the automation **traceable and complete**, including unsuccessful processing results.
