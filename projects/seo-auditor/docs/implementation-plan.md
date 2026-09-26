# Implementation plan

## Objective

Complete the SEO Auditor workflow without losing failed crawl targets, then evolve it into a traceable AI-automation / BA portfolio case.

## Phase 0 — Freeze the baseline

- Keep the current sanitized workflow export.
- Record the observed test: **70 crawl targets / 65 success / 5 error**.
- Define a common \`PageResult\` contract.
- Define acceptance criteria before implementation.

**Done when:** the current MVP remains reproducible.

## Phase 1 — Fix URL processing order

1. Normalize relative and absolute URLs.
2. Store the normalized URL as \`fullUrl\`.
3. Deduplicate using normalized \`fullUrl\`.
4. Use the result as the canonical crawl target.

**Acceptance:** equivalent URLs produce one crawl target.

## Phase 2 — Build the success result contract

Add a node after metadata extraction:

\`\`\`
🏷️ Извлечь мета-теги
        ↓
✅ Сформировать результат страницы
\`\`\`

Target fields:

\`\`\`
url
status=success
httpStatus
title
titleLength
description
descriptionLength
h1
error=null
\`\`\`

**Acceptance:** one successful crawl input produces one stable result record.

## Phase 3 — Build the Error branch

Connect the Error output of \`📥 Скачать все страницы\` to:

\`\`\`
⚠️ Нормализовать ошибку загрузки
\`\`\`

Responsibilities:

- preserve the original URL;
- set \`status=error\`;
- capture HTTP status when available;
- classify the error;
- store a readable message;
- mark whether retry is appropriate;
- store attempt number.

**Acceptance:** failed page items are not dropped.

## Phase 4 — Merge success and error

Add:

\`\`\`
Success → Page Result ─┐
                       ├→ 🔀 Объединить результаты
Error   → Error Result ┘
\`\`\`

Use Merge / Append.

Core invariant:

\`\`\`
successCount + errorCount = crawlTargetCount
\`\`\`

For the observed run:

\`\`\`
65 + 5 = 70
\`\`\`

## Phase 5 — Add deterministic SEO rules

Calculate before Claude:

- missing Title;
- missing Description;
- missing H1;
- Title length;
- Description length;
- duplicate Title;
- duplicate Description;
- duplicate H1.

Thresholds must be explicit configuration.

**Acceptance:** identical input produces identical flags without calling Claude.

## Phase 6 — Replace free-form aggregation

Replace the current large text-only aggregation with structured data.

Target model:

\`\`\`
AuditSummary
├── targetUrl
├── discoveredCount
├── successCount
├── errorCount
├── crawlCoverage
└── errorsByType

PageResult[]
└── url / status / metadata / issues / error
\`\`\`

**Acceptance:** the dataset can be reused by another delivery channel without reparsing text.

## Phase 7 — Reframe Claude as interpretation

Claude receives:

- deterministic summary statistics;
- deterministic issue counts;
- page findings within a defined prompt budget;
- failed-page summary.

Claude produces:

- executive summary;
- interpretation;
- recommendations.

**Acceptance:** removing Claude does not change crawl counts or SEO flags.

## Phase 8 — Add bounded retries

Retry only transient errors:

- timeout / network failures;
- HTTP 429;
- HTTP 5xx.

Use a maximum attempt count and bounded delay/backoff.

Permanent errors remain in the final dataset.

## Phase 9 — Reporting and observability

Final report must show:

- target URL;
- discovered URLs;
- successful pages;
- failed pages;
- crawl coverage;
- SEO issue counts;
- failed URL list;
- recommendations.

## Phase 10 — Portfolio packaging

Required artifacts:

- architecture diagram;
- workflow walkthrough;
- data contract;
- error-handling design;
- implementation roadmap;
- sanitized workflow export;
- report schema;
- test cases.

## Definition of Done for v2

- [ ] URL normalization precedes deduplication.
- [ ] Successful pages produce PageResult records.
- [ ] Failed pages produce Error PageResult records.
- [ ] Success and error records are merged.
- [ ] No crawl target disappears silently.
- [ ] Deterministic SEO checks are implemented.
- [ ] Structured audit data is created before Claude.
- [ ] Claude is used for interpretation, not basic counting.
- [ ] Final report exposes crawl coverage and failed URLs.
- [ ] Workflow export is updated and sanitized.
- [ ] GitHub documentation matches the implemented workflow.
