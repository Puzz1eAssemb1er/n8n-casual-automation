# Technical notes and improvement backlog

The current implementation should be presented as an MVP rather than as a complete production crawler.

## Current limitations

### URL normalization
Raw links are deduplicated before absolute URL normalization. Equivalent relative and absolute forms can therefore remain as separate crawl targets.

**Next step:** normalize first, then deduplicate on the canonical URL.

### Crawl depth
The workflow discovers URLs from the homepage only. It does not recursively crawl discovered pages to find additional URLs.

**Next step:** introduce configurable crawl depth and a visited-URL set.

### Error handling
Page downloads continue after individual request failures, but the current graph does not produce a dedicated error dataset.

**Next step:** preserve failed URL, status/error and retry information and include them in the final report.

### SEO rules
Length classification is currently delegated to the LLM.

**Next step:** implement deterministic checks for missing fields, duplicates and configurable character thresholds, then use the LLM for explanation and prioritization.

### Scalability
A large website can produce a large aggregated prompt and high execution cost.

**Next step:** add page limits, batching, structured JSON output and optionally a persistent data store.

## Target page contract

A stronger next version should produce a record similar to:

```json
{
  "url": "https://example.com/page",
  "statusCode": 200,
  "title": "Example title",
  "titleLength": 13,
  "description": "Example description",
  "descriptionLength": 20,
  "h1": "Example H1",
  "error": null
}
```

This structure would make the workflow easier to test and extend toward a database, spreadsheet or BI layer.

## Security

Public exports must not contain API keys, OAuth tokens, personal recipient addresses or n8n instance-specific credential metadata. The repository version therefore uses placeholders and n8n's credential mechanism.
