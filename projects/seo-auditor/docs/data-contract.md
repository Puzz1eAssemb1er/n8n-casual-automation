# Data contract

## Purpose

Successful and failed crawl attempts use one common page-result model so the two branches can be merged without losing traceability.

## PageResult

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

## Error PageResult

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

## Field semantics

| Field | Type | Meaning |
| --- | --- | --- |
| \`url\` | string | Normalized crawl target |
| \`status\` | enum | \`success\` or \`error\` |
| \`httpStatus\` | integer / null | HTTP response status when available |
| \`title\` | string / null | Extracted HTML title |
| \`titleLength\` | integer | Character count |
| \`description\` | string / null | Extracted meta description |
| \`descriptionLength\` | integer | Character count |
| \`h1\` | string / null | Extracted H1 |
| \`error\` | object / null | Error information for unsuccessful processing |

## Audit-level summary

\`\`\`json
{
  "targetUrl": "https://example.com",
  "discoveredCount": 70,
  "successCount": 65,
  "errorCount": 5,
  "crawlCoverage": 92.86,
  "errorsByType": {
    "HTTP_ERROR": 4,
    "TIMEOUT": 1
  }
}
\`\`\`

The numbers above are an example based on the observed 70 / 65 / 5 test and are not hard-coded workflow output.

## Design rule

Downstream processing must be able to answer:

> How many URLs were discovered, how many were processed successfully, and what happened to the rest?

without reading n8n execution logs manually.
