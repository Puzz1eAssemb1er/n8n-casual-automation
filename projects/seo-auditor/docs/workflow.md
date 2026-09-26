# Workflow walkthrough

## Phase 1 — Start and configure

### \`▶️ Запуск аудита\`

Manual trigger for one audit execution.

### \`🌐 Адрес сайта\`

Stores the target website in \`baseUrl\`.

## Phase 2 — Discover URLs

### \`📥 Скачать главную страницу\`

Downloads the homepage HTML.

### \`🔗 Собрать ссылки\`

Extracts \`a[href]\` values from the homepage.

### \`📋 Разбить на список\`

Converts the link array into individual n8n items.

### \`🧹 Фильтр ссылок\`

Keeps relative links and links belonging to the configured site.

### \`🔂 Убрать дубли\`

**Current:** removes duplicate raw URL values.

**V2:** normalization must happen before deduplication so equivalent URLs become one crawl target.

### \`🔧 Сделать полные адреса\`

**Current:** creates \`fullUrl\` from \`baseUrl\` and a relative link.

**V2:** move URL normalization before deduplication and use the normalized URL as the crawl key.

## Phase 3 — Crawl

### \`📥 Скачать все страницы\`

Requests every crawl target.

The current execution demonstrates the need for explicit error handling:

\`\`\`
70 inputs
├── 65 → Success
└── 5  → Error
\`\`\`

The Error output must become part of the data pipeline.

## Phase 4 — Success path

### \`🏷️ Извлечь мета-теги\`

Extracts:

\`\`\`
title
description
h1
\`\`\`

### Target node: \`✅ Сформировать результат страницы\`

Create one stable record per successful page:

\`\`\`json
{
  "url": "https://example.com/page",
  "status": "success",
  "httpStatus": 200,
  "title": "Example title",
  "titleLength": 13,
  "description": "Example description",
  "descriptionLength": 20,
  "h1": "Example H1",
  "error": null
}
\`\`\`

## Phase 5 — Error path

### Target node: \`⚠️ Нормализовать ошибку загрузки\`

Convert the raw error output into the same page-result structure.

The node must preserve:

- original URL;
- status = \`error\`;
- HTTP status when available;
- error type;
- readable message;
- retryable flag;
- attempt number.

## Phase 6 — Merge

### Target node: \`🔀 Объединить результаты\`

Connect both result branches:

\`\`\`
Success → Page Result ─┐
                       ├→ Merge / Append
Error   → Error Result ┘
\`\`\`

Target invariant:

\`\`\`
successCount + errorCount = crawlTargetCount
\`\`\`

Observed test:

\`\`\`
65 + 5 = 70
\`\`\`

## Phase 7 — Deterministic SEO checks

### Target node: \`🔎 Проверить SEO-правила\`

Calculate facts independently of the LLM:

- missing Title;
- missing Description;
- missing H1;
- Title length;
- Description length;
- duplicate Title;
- duplicate Description;
- duplicate H1;
- success count;
- error count.

Thresholds should be explicit configuration.

## Phase 8 — Structured dataset

### Target node: \`📝 Собрать структурированный отчёт\`

The current workflow builds a large text payload.

V2 should create structured audit data first, then generate the LLM prompt from that dataset.

## Phase 9 — Claude analysis

### \`🤖 Анализ Claude AI\`

Claude receives deterministic findings and explains them.

The AI layer should produce:

- summary;
- interpretation;
- recommendations.

It should not be responsible for primary crawl counts or deterministic rule calculation.

## Phase 10 — Presentation and delivery

### \`🎨 Оформить HTML\`

Converts the AI response into email-ready HTML.

### \`✉️ Отправить на email\`

Delivers the final report.

## Target end-to-end flow

\`\`\`
Start
 ↓
Configure audit
 ↓
Discover URLs
 ↓
Normalize
 ↓
Deduplicate
 ↓
Crawl
 ├─ Success → Extract → Page Result ─┐
 └─ Error   → Normalize → Error Result ─┤
                                       ↓
                                   Merge Results
                                       ↓
                               Deterministic Checks
                                       ↓
                               Structured Dataset
                                       ↓
                                    Claude
                                       ↓
                                  HTML / Email
\`\`\`
