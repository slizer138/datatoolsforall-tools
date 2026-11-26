---
name: sitemap-url-extractor
description: List every page URL a website publishes in its sitemaps (found via robots.txt and common paths, nested indexes and .xml.gz followed) with lastmod, changefreq, priority, hreflang alternates and image/video counts, optionally the HTTP status of each URL, and new/removed URLs compared with the previous run. Use when the user wants all URLs of a site, a page inventory for an SEO audit or migration, pages changed since a date, pages added or removed since last time, or a URL list to feed into a Lighthouse audit or a crawl. Runs the pay-per-use Apify Actor datatoolsforall/website-sitemap-extractor on the user's own Apify account (USD 0.0002 per URL row).
license: MIT
compatibility: Needs this plugin's MCP server (mcp.apify.com with Apify OAuth sign-in), or the standard Apify MCP server with call-actor, or the Apify CLI after apify login. Needs an Apify account.
metadata:
  author: Data Tools for All
  actor: datatoolsforall/website-sitemap-extractor
  disclosure: The author built this Actor and earns from paid runs.
---

# Sitemap URL extractor

Reads only the site's own discovery files (robots.txt, sitemap.xml, sitemap indexes, .xml.gz, plain-text and RSS/Atom sitemaps). It does not crawl pages, so it returns exactly what the site declares in its sitemaps, one row per URL.

## When to use

- "Get all URLs of example.com", "how many pages does this site have in its sitemap", "export the sitemap to CSV".
- Building a page inventory for an SEO audit, a migration redirect map or a content review.
- "Which pages changed in the last 30 days" (`lastmodAfter`).
- "Which pages were added or removed since last week" (`compareWithPreviousRun`, best on a schedule).
- Finding redirected or broken URLs listed in the sitemap (`checkUrlStatus`).
- Getting a URL list to pass to the `bulk-web-vitals-audit` skill.

## When not to use

- The user needs pages that are **not** in any sitemap (orphans, faceted URLs): this is not a crawler. Say so and suggest a crawler.
- The site has a small sitemap the user can open in the browser: fetching `https://site/sitemap.xml` yourself (if you have a web fetch tool) is free.
- Sitemaps behind a login, HTML-only "sitemap pages", or sites that block bots (the Actor returns `BLOCKED` and does not bypass anything).
- The user wants page content, titles or meta tags: this tool returns URLs and sitemap metadata only.

## Tools

1. This plugin's MCP server exposes the Actor as its own tool, named `datatoolsforall--website-sitemap-extractor` (your client may add a prefix, for example `mcp__plugin_datatoolsforall_dtfa__...` in Claude Code). Loading it also loads `get-actor-run`, `get-dataset-items`, `get-key-value-store-record` and `abort-actor-run`.
2. If only the standard Apify MCP server is connected, use `call-actor` with `actor: "datatoolsforall/website-sitemap-extractor"`, the same `input`, and `callOptions: { "maxTotalChargeUsd": <cap> }`.

The Actor tool waits up to `waitSecs` seconds (default 30, maximum 45) and then returns the run status and a `nextStep`. If the run has not finished, call `get-actor-run` with its `runId` and `waitSecs: 45` until the status is SUCCEEDED, FAILED, ABORTED or TIMED-OUT; never start a second run for the same request. Then read rows with `get-dataset-items` (use `limit` and `offset`, or `fields`, for large results).

## Cost and guardrails (billed to the user's Apify account)

- USD 0.0002 per stored URL row (`url-extracted`). In comparison mode, each `removed` row is also one event. A website that has no sitemap at all (`NO_SITEMAP_FOUND`) is one event.
- `checkUrlStatus: true` adds USD 0.0005 per URL that got an HTTP answer.
- Free rows: `SITEMAP_NOT_FOUND`, `ROBOTS_DISALLOWED`, `SITEMAP_EMPTY`, `CHILD_SITEMAPS_FAILED`, `INDEX_NOT_FOLLOWED`, `DNS_NOT_FOUND`, `BLOCKED`, `FETCH_FAILED`, `INTERNAL_ERROR`. Plus a start fee of a fraction of a cent per run.
- **Always set `maxUrlsPerSite`.** The Actor default is 10,000 (up to USD 2.00 per website, or USD 7.00 with status checks). Estimate: `sites x maxUrlsPerSite x 0.0002` (plus `x 0.0005` with status checks).
- If the estimate is above about USD 1, tell the user the estimate and the cap and wait for a yes before running.
- Start small when the site size is unknown: `maxUrlsPerSite: 500` costs at most USD 0.10 and tells you whether the site is bigger (the SUMMARY record shows `truncatedBy`).
- One run per request. Do not re-run a successful run to "double check".

## Input fields

Required: `startUrls`.

| Field | Type | Default | Notes |
|---|---|---|---|
| `startUrls` | array of URL strings (1 to 1,000) | none | Website roots (`https://example.com`) or direct sitemap links. A page address such as `https://example.com/blog` is reduced to the site root; to keep one section use `includePatterns` or that section's sitemap URL. Objects `{"url": "..."}` are also accepted. |
| `maxUrlsPerSite` | integer 1 to 1,000,000 | 10000 | Billable rows per website, also the cost cap per website. |
| `includePatterns` | array of regex strings | empty | Keep only matching URLs (case-sensitive), for example `/blog/`. |
| `excludePatterns` | array of regex strings | empty | Drop matching URLs, applied after includes, for example `/tag/`. |
| `lastmodAfter` | string | none | `YYYY-MM-DD` or relative such as `30 days`. URLs without lastmod are dropped when set. |
| `followSitemapIndexes` | boolean | true | Keep on. Off returns only URLs listed directly in the files found. |
| `includeAlternates` | boolean | true | hreflang alternates in `alternates`. |
| `includeMediaCounts` | boolean | true | `imageCount` and `videoCount`. |
| `checkUrlStatus` | boolean | false | One HEAD request per URL; adds `statusCode` and `redirectLocation`. Paid add-on, slower. |
| `compareWithPreviousRun` | boolean | false | Adds `diffStatus` = new, unchanged or removed. Removed URLs are detected only after a complete read (no cap reached, no failed sitemap, no `lastmodAfter`). |
| `stateKey` | string `[a-zA-Z0-9_.-]{1,100}` | none | Name for the remembered URL list, for example `competitor-weekly`. Reuse it so runs compare with each other. |
| `maxSitemapsPerSite` | integer 1 to 10,000 | 500 | Safety limit on sitemap files per website. |
| `requestTimeoutSecs` | integer 5 to 300 | 30 | Per-request wait; failed requests retry 3 times. |

Minimal input: `{"startUrls": ["https://example.com"], "maxUrlsPerSite": 500}` (at most USD 0.10).

## Output fields (one dataset row per URL)

`url`, `site`, `sitemapUrl`, `lastmod` (ISO 8601; date-only values stay `YYYY-MM-DD`; a time without a zone keeps no `Z`), `changefreq`, `priority`, `alternates` (`[{hreflang, href}]`), `imageCount`, `videoCount`, `newsTitle`, `statusCode`, `redirectLocation`, `statusCheckError` (`ROBOTS_DISALLOWED`, `TIMEOUT`, `FETCH_FAILED`, `DNS_NOT_FOUND`), `diffStatus`, `discoveredAt`, `error`, `errorCode`, `sitemapsTried`.

Every row has all keys; values that do not apply are `null`. The run's key-value store has a `SUMMARY` record (read it with `get-key-value-store-record`, key `SUMMARY`) with per-site `outcome` (`OK` or `PARTIAL`), sitemaps found and failed, `truncatedBy`, `filteredOut`, `duplicatesSkipped`, `rowsStored` and `rowsCharged`.

## Errors and what to do

| `errorCode` | Meaning | What to tell or do |
|---|---|---|
| `NO_SITEMAP_FOUND` | robots.txt and common locations show no sitemap (billed as one row). | The site has no sitemap; suggest a crawler. |
| `SITEMAP_NOT_FOUND` | A sitemap link the user gave does not work. | Retry with the website root instead so the Actor discovers sitemaps. |
| `SITEMAP_EMPTY` | Sitemaps exist but list no URLs. | Report it; nothing to extract. |
| `INDEX_NOT_FOLLOWED` | Only index files found and `followSitemapIndexes` was off. | Re-run with it on (ask first if the cost is notable). |
| `CHILD_SITEMAPS_FAILED`, `FETCH_FAILED` | Downloads failed or were cut off. | Report which file (`error`, `sitemapsTried`); one later retry is reasonable. |
| `BLOCKED` | 401/403 or a bot challenge. | Do not try to get around it. Ask the user for an allowed source. |
| `DNS_NOT_FOUND` | The domain does not exist. | Check the spelling with the user. |
| `ROBOTS_DISALLOWED` | robots.txt disallows the sitemap locations. | Respect it. |

If `outcome` is `PARTIAL`, say that some URLs are missing and why, instead of presenting the list as complete. If the row count equals `maxUrlsPerSite`, say the cap was reached and the site has more URLs.

## Example prompts

- "List every URL in example.com's sitemap that changed in the last 30 days." -> `{"startUrls": ["https://example.com"], "lastmodAfter": "30 days", "maxUrlsPerSite": 2000}`
- "Track new and removed pages on competitor.com every week." -> `{"startUrls": ["https://competitor.com"], "compareWithPreviousRun": true, "stateKey": "competitor-weekly", "maxUrlsPerSite": 5000}`; suggest an Apify schedule for weekly runs.
- "Find redirected or broken URLs in our sitemap (about 800 pages)." -> `{"startUrls": ["https://our-site.com"], "checkUrlStatus": true, "maxUrlsPerSite": 1000}` (at most USD 0.70), then filter rows where `statusCode` is not 200.

Out of scope: "Find all pages on this site, including ones not in the sitemap" needs a crawler; explain that this tool only reads sitemaps.
