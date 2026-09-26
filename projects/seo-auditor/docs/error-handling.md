# Error handling design

## Goal

A page-level HTTP failure must not disappear from the audit.

For every crawl target:

\`\`\`
1 URL in → 1 logical result out
\`\`\`

The result is either:

- \`status=success\`;
- \`status=error\`.

## Current observation

The current workflow produced:

\`\`\`
70 crawl targets
65 success
5 error
\`\`\`

The five failed items are therefore part of audit coverage.

## Target branch

\`\`\`
📥 Скачать все страницы
        │
        ├── Success → 🏷️ Извлечь мета-теги
        │                    ↓
        │              ✅ Сформировать результат
        │
        └── Error   → ⚠️ Нормализовать ошибку
                             ↓
                       ❗ Error Result
                             │
                             └───────┐
                                     ↓
                           🔀 Объединить результаты
\`\`\`

## Error result contract

\`\`\`json
{
  "url": "https://example.com/missing",
  "status": "error",
  "httpStatus": 404,
  "title": null,
  "titleLength": 0,
  "description": null,
  "descriptionLength": 0,
  "h1": null,
  "error": {
    "type": "HTTP_ERROR",
    "message": "Page not found",
    "retryable": false,
    "attempt": 1
  }
}
\`\`\`

The exact raw error fields should be mapped from the n8n Error output available in the actual execution.

## Retry classification

\`\`\`
Transient
├── timeout
├── network failure
├── 429
└── 5xx
      ↓
    retry

Permanent
├── 4xx (business-rule dependent)
└── unrecoverable validation errors
      ↓
  final error
\`\`\`

Retry is a separate step. First preserve the error record and merge it into the final dataset.

## Merge

Use a Merge node in **Append** mode:

\`\`\`
Input 1 = successful PageResult items
Input 2 = normalized ErrorResult items

Output = complete crawl result set
\`\`\`

For the observed run:

\`\`\`
65 + 5 = 70
\`\`\`

## Reporting requirement

The final report must expose:

- total crawl targets;
- successful pages;
- failed pages;
- crawl coverage;
- failed URLs;
- error type / HTTP status when available.

This prevents the report from presenting incomplete crawl coverage as a complete audit.
