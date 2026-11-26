---
name: bulk-web-vitals-audit
description: Run real Lighthouse 13 audits on a list of URLs or on the pages of a sitemap, mobile and/or desktop, and get performance, accessibility, best-practices and SEO scores, lab Core Web Vitals metrics (LCP, CLS, TBT, FCP, Speed Index, TTFB), the biggest suggested fixes and optional pass/fail budgets per page. Use when the user wants to audit many pages at once, compare mobile and desktop, find the slowest templates of a site, or check pages against performance budgets, and no Google API key or quota should be needed. Results are lab measurements, not real-user field data. Runs the pay-per-use Apify Actor datatoolsforall/bulk-lighthouse-pagespeed-audit on the user's own Apify account (USD 0.02 per URL per device).
license: MIT
compatibility: Needs this plugin's MCP server (mcp.apify.com with Apify OAuth sign-in), or the standard Apify MCP server with call-actor, or the Apify CLI after apify login. Needs an Apify account.
metadata:
  author: Data Tools for All
  actor: datatoolsforall/bulk-lighthouse-pagespeed-audit
  disclosure: The author built this Actor and earns from paid runs.
---

# Bulk Lighthouse audit (lab Core Web Vitals)

Each audit is one normal page load in a fresh headless Chrome running Lighthouse 13 on Apify, under simulated mobile or desktop conditions. Audits run one at a time so they do not distort each other.

## When to use

- "Audit these 30 URLs", "run Lighthouse on every page of my sitemap", "which of my pages are slowest on mobile".
- Comparing mobile and desktop scores for the same pages.
- Checking many pages against budgets (for example performance >= 70, LCP <= 2500 ms) and listing the ones that fail.
- Finding the most common suggested fixes across a site (`opportunities`).

## When not to use

- **One or two URLs**: Lighthouse in Chrome DevTools or https://pagespeed.web.dev is free. Suggest that first.
- **Real-user data (field Core Web Vitals, INP)**: lab audits cannot measure INP or what real visitors experience. Point to the Chrome UX Report / PageSpeed Insights or Search Console. If the user has their own PageSpeed Insights API key, the `psiApiKey` input adds CrUX field data (`fieldData`), but only when the user supplies the key themselves.
- Pages behind a login, pages that need cookies or form input, load testing, uptime monitoring.
- Sites the user does not own or is not allowed to test.

Never describe the results as "Core Web Vitals assessment passed/failed": that verdict comes from field data. Say "lab LCP", "lab CLS" and "TBT (a lab proxy for responsiveness)".

## Tools

1. This plugin's MCP server exposes the Actor as its own tool, named `datatoolsforall--bulk-lighthouse-pagespeed-audit` (your client may add a prefix, for example `mcp__plugin_datatoolsforall_dtfa__...` in Claude Code). Loading it also loads `get-actor-run`, `get-dataset-items`, `get-key-value-store-record` and `abort-actor-run`.
2. If only the standard Apify MCP server is connected, use `call-actor` with `actor: "datatoolsforall/bulk-lighthouse-pagespeed-audit"`, the same `input`, and `callOptions: { "maxTotalChargeUsd": <cap> }`.

Runs are slow: about 10 seconds for a light page and about 60 seconds for a heavy one in local tests; the platform can be slower. 50 pages can take 10 to 50 minutes. The Actor tool waits up to `waitSecs` seconds (default 30, maximum 45) and then returns the run status and a `nextStep`. Tell the user the run is in progress and how long it may take, then call `get-actor-run` with its `runId` and `waitSecs: 45` until the status is SUCCEEDED, FAILED, ABORTED or TIMED-OUT. Never start a second run for the same request. Read rows with `get-dataset-items` (the `fields` argument keeps large results small).

## Cost and guardrails (billed to the user's Apify account)

- USD 0.02 per successful audit (one URL on one device). Both devices = 2 audits per URL.
- With the user's own PageSpeed Insights key: USD 0.003 per audit (audits then run on Google's API under the user's quota).
- Rows with an `errorCode` are free. Plus a start fee of a fraction of a cent per run.
- Number of audits = (number of `urls` + pages taken from the sitemap, at most `maxUrls`) x number of `strategies`.
- **`maxUrls` only caps sitemap pages.** Every URL in `urls` is audited (up to 1,000). The Actor default `maxUrls` is 50 (USD 1.00 per device). Always set it explicitly.
- If the estimate is above about USD 1, tell the user the number of audits and the cost and wait for a yes. Example: 120 URLs x mobile and desktop = 240 audits = USD 4.80.
- For a first look at a big site, audit a sample of representative templates (home, category, product, article) instead of every page.
- Do not re-run audits to average scores unless the user asks for repeated runs and accepts the cost.

## Input fields

No field is strictly required, but give `urls` or `sitemapUrl`.

| Field | Type | Default | Notes |
|---|---|---|---|
| `urls` | array of URL strings (up to 1,000) | none | Every URL is audited once per device. Rows show the URL exactly as typed. |
| `sitemapUrl` | string | none | A sitemap.xml, sitemap index, .xml.gz or plain-text sitemap. Up to `maxUrls` pages are taken in sitemap order, in addition to `urls`. |
| `strategies` | array of `mobile`, `desktop` | `["mobile"]` | Each device is one audit per URL. |
| `categories` | array of `performance`, `accessibility`, `best-practices`, `seo` | all four | Lab metrics and opportunities need `performance`. |
| `maxUrls` | integer 1 to 1,000 | 50 | Pages taken from the sitemap only. |
| `maxOpportunities` | integer 0 to 20 | 5 | Biggest suggested fixes per page; 0 turns them off. |
| `budgets` | object | none | Minimums `performanceScore`, `accessibilityScore`, `bestPracticesScore`, `seoScore` (0 to 100); maximums `lcpMs`, `fcpMs`, `tbtMs`, `cls`, `speedIndexMs`, `ttfbMs`. |
| `saveFullReport` | boolean | false | Saves the full Lighthouse HTML report per page; link in `reportUrl`. |
| `psiApiKey` | string (secret) | none | The user's own Google PageSpeed Insights key. Only if the user configures it themselves; never ask for it in chat and never echo it. |
| `throttlingMethod` | `simulate` or `devtools` | `simulate` | Keep `simulate` (same as PageSpeed Insights). `devtools` is slower and noisier. |
| `timeoutSecs` | integer 60 to 300 | 120 | Per audit. A timed-out audit is a free `TIMEOUT` row. |

Minimal input: `{"urls": ["https://example.com/"], "strategies": ["mobile"]}` (USD 0.02).

## Output fields (one dataset row per URL per device)

`url`, `finalUrl`, `strategy`, `engine` (`lighthouse-local` or `pagespeed-insights`), `lighthouseVersion`, `performanceScore`, `accessibilityScore`, `bestPracticesScore`, `seoScore` (0 to 100), `lcpMs`, `fcpMs`, `tbtMs`, `cls`, `speedIndexMs`, `ttfbMs` (all lab), `fieldData` (`{lcpP75Ms, inpP75Ms, clsP75, fcpP75Ms, ttfbP75Ms, overallCategory, origin}`, only with the user's PSI key, else `null`), `opportunities` (`[{id, title, savingsMs, savingsBytes}]`, largest saving first), `budgetFailures` (for example `"lcpMs: 4517 > 2500"`), `passedBudgets` (`true`/`false`, `null` without budgets), `reportUrl`, `errorCode`, `error`, `warnings`, `auditedAt`.

How to present results: a table sorted by `performanceScore` (worst first) with `url`, `strategy`, `performanceScore`, `lcpMs`, `cls`, `tbtMs`; then the opportunities that appear on the most pages. Mention that scores vary by a few points between runs.

## Errors (all free)

| `errorCode` | Meaning | What to do |
|---|---|---|
| `INVALID_URL` | Not a public http(s) address (localhost and private networks are refused). | Fix the URL. |
| `DNS_FAILURE` | Domain does not resolve. | Check spelling. |
| `BLOCKED` | The site answered 401, 403 or 429. | Do not try to get around it; the user can allow-list the audit on their own site. |
| `HTTP_ERROR` | 404, 500 and similar. | Report the broken URL. |
| `NAVIGATION_ERROR` | Certificate or connection error. | Report it. |
| `TIMEOUT` | Audit exceeded `timeoutSecs`. | Retry once with a higher `timeoutSecs` if the user wants. |
| `NO_CONTENT`, `NOT_HTML` | Blank page, or a file instead of a page. | Remove from the list. |
| `INCOMPLETE_AUDIT` | A selected score could not be computed (mostly with `devtools` throttling). | Use `simulate`. |
| `AUDIT_ERROR` | Lighthouse failed on the page. | Report it. |
| `PSI_QUOTA`, `PSI_ERROR` | Only with the user's key: quota used up or API error. | Retry later or without the key. |

A warning such as "The page loaded too slowly to finish within the time limit" means Lighthouse audited what had loaded after up to 45 seconds; scores are still computed with `simulate`.

## Example prompts

- "Run Lighthouse on these 12 landing pages, mobile and desktop, and tell me which are worst." -> `{"urls": [...12 URLs], "strategies": ["mobile", "desktop"]}` = 24 audits = USD 0.48.
- "Audit the first 50 pages of https://example.com/sitemap.xml on mobile and flag anything with LCP over 2.5 s." -> `{"sitemapUrl": "https://example.com/sitemap.xml", "maxUrls": 50, "strategies": ["mobile"], "budgets": {"lcpMs": 2500}}` = USD 1.00; confirm first.
- "Check our blog templates against a performance score of 80." -> pick 5 to 10 representative URLs (the `sitemap-url-extractor` skill can list them), then `budgets: {"performanceScore": 80}`.

Out of scope: "Is my site passing Core Web Vitals in Google Search?" needs field data from real users (Search Console or CrUX); lab audits cannot answer it.
