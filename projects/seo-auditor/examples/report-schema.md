# Report schema

The target v2 report is based on structured audit results rather than only a free-form text aggregation.

## Crawl summary

\`\`\`json
{
  "targetUrl": "https://example.com",
  "discoveredCount": 70,
  "successCount": 65,
  "errorCount": 5,
  "crawlCoverage": 92.86
}
\`\`\`

## Page findings

\`\`\`json
{
  "url": "https://example.com/page",
  "status": "success",
  "title": "Example title",
  "titleLength": 13,
  "description": "Example description",
  "descriptionLength": 20,
  "h1": "Example H1",
  "issues": {
    "titleMissing": false,
    "descriptionMissing": false,
    "h1Missing": false,
    "titleDuplicate": false,
    "descriptionDuplicate": false,
    "h1Duplicate": false
  }
}
\`\`\`

## Failed page

\`\`\`json
{
  "url": "https://example.com/missing",
  "status": "error",
  "httpStatus": 404,
  "error": {
    "type": "HTTP_ERROR",
    "message": "Page not found",
    "retryable": false
  }
}
\`\`\`

## AI input

Claude should receive a bounded structured dataset containing:

- crawl summary;
- deterministic SEO findings;
- failed-page summary;
- only page-level details required for interpretation.

## AI output

The final report should contain:

1. crawl coverage;
2. main SEO findings;
3. failed pages;
4. corrective actions;
5. concise conclusion.

The AI output is the interpretation and presentation layer. Underlying counts and rule flags remain deterministic data.
