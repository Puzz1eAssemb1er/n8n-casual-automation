# Technical notes

## Current implementation vs target design

The current workflow is an MVP. It proves the end-to-end automation path, while the v2 backlog focuses on correctness, traceability and resilience.

## URL normalization

**Current:** raw URLs are deduplicated before absolute URL normalization.

**Target:** normalize first, then deduplicate using normalized \`fullUrl\` as the unique crawl key.

## Page result contract

Every crawl target should produce exactly one logical result:

\`\`\`
status = success | error
\`\`\`

Success results contain extracted metadata.

Error results contain the crawl URL and available error information.

This establishes a one-input / one-result rule for the crawl stage.

## Error handling

The current crawler continues after individual page request failures. The missing component is a dedicated Error branch that converts failed items into the same data contract as successful items.

Target:

\`\`\`
Success → PageResult ─┐
                      ├→ Merge / Append
Error   → ErrorResult ┘
\`\`\`

## Retry policy

A later iteration should retry only transient failures:

| Error class | Default handling |
| --- | --- |
| Timeout / network error | Retry |
| HTTP 429 | Retry with delay |
| HTTP 5xx | Retry with limit |
| HTTP 4xx | Record as final error unless business rules require otherwise |
| Unknown | Record and inspect |

Retries must be bounded.

## Deterministic SEO checks

Current AI analysis is asked to identify missing fields, suspicious lengths and duplicates.

Target: compute these facts in n8n before the LLM stage.

This gives reproducible results and prevents the LLM from becoming the source of truth for numeric counts and boolean checks.

## Structured aggregation

The current aggregation creates one large text report.

Target:

- build structured JSON records;
- calculate summary statistics;
- generate the LLM prompt from those records;
- keep the original structured data available for future exports.

## Crawl limits

A production-oriented implementation should expose:

- maximum number of crawl targets;
- per-request timeout;
- maximum retry count;
- optional delay between requests;
- optional crawl depth.

## Testing strategy

Minimum test matrix:

1. all pages successful;
2. mixed 200 + 4xx;
3. timeout;
4. 5xx;
5. 429;
6. duplicate URLs;
7. missing SEO metadata;
8. duplicate metadata;
9. empty link set;
10. homepage request failure.

## Security

Never commit:

- API keys;
- OAuth tokens;
- exported credentials;
- personal recipient addresses;
- instance-specific secret metadata.

Use n8n credentials and public placeholders.
