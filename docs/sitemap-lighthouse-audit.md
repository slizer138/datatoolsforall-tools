---
title: Audit every page of a sitemap with Lighthouse for about $1
description: Take 50 pages from a sitemap, run a mobile Lighthouse audit on each, and rank them by performance score, LCP, CLS and TBT. Real field names and honest cost math.
---

# Audit every page of a sitemap with Lighthouse for about $1

This guide takes the pages listed in a website's sitemap, runs a mobile Lighthouse audit on 50 of them, and ranks them by performance. It uses two Apify Actors that I built and earn from: [Sitemap Extractor](https://apify.com/datatoolsforall/website-sitemap-extractor) and [Bulk Lighthouse & PageSpeed Audit](https://apify.com/datatoolsforall/bulk-lighthouse-pagespeed-audit). If you only need one or two pages, use Lighthouse in Chrome DevTools or [pagespeed.web.dev](https://pagespeed.web.dev) instead: they are free.

## What it costs

| Step | Count | Price | Cost |
|---|---|---|---|
| List the sitemap's URLs (optional) | up to 500 URLs | $0.0002 per URL | at most $0.10 |
| Lighthouse, mobile only | 50 audits | $0.02 per audit | $1.00 |
| Apify start fee | 1 or 2 runs | a fraction of a cent | under $0.01 |
| **Total** | | | **about $1.00 to $1.10** |

Variations:

- Mobile **and** desktop: 100 audits = $2.00.
- With your own Google PageSpeed Insights API key (`psiApiKey`): $0.003 per audit, so 50 audits = $0.15, and you also get real-user Chrome UX Report data when Google has it. Audits then run on Google's API under your key and quota.
- Rows that end in an error (`TIMEOUT`, `BLOCKED`, `HTTP_ERROR`, ...) are not charged.

Apify's free plan includes $5 of monthly usage credit, which covers this guide several times.

## Before you start

- A free Apify account.
- A site you own or are allowed to test. Each audit is a normal page load from Apify's servers.
- About 10 to 50 minutes. Audits run one at a time so they do not distort each other. In local tests a light page took about 10 seconds and a heavy, script-rich page about 60 seconds; on Apify it can take longer.

## Step 1 (optional): choose the 50 pages

The Lighthouse Actor can read the sitemap itself (Step 2, option B), but it then takes the **first** 50 pages it finds, which are often all of one type. A better sample covers each page template. List the URLs first:

```json
{
  "startUrls": ["https://www.example.com"],
  "maxUrlsPerSite": 500,
  "excludePatterns": ["/tag/", "/page/\\d+"]
}
```

Run [Sitemap Extractor](https://apify.com/datatoolsforall/website-sitemap-extractor) with this input (at most $0.10). Each output row has `url`, `lastmod`, `sitemapUrl`, `changefreq` and `priority`. Group the URLs by path pattern (`/blog/...`, `/product/...`, `/category/...`) and pick a few from each group, up to 50. To focus on recent changes, add `"lastmodAfter": "30 days"`.

## Step 2: run the audits

Option A, your own list (recommended):

```json
{
  "urls": ["https://www.example.com/", "https://www.example.com/pricing", "..."],
  "strategies": ["mobile"],
  "budgets": { "performanceScore": 70, "lcpMs": 2500, "cls": 0.1 }
}
```

Option B, straight from the sitemap:

```json
{
  "sitemapUrl": "https://www.example.com/sitemap.xml",
  "maxUrls": 50,
  "strategies": ["mobile"],
  "budgets": { "performanceScore": 70, "lcpMs": 2500, "cls": 0.1 }
}
```

`maxUrls` limits only the pages taken from the sitemap. Every URL in `urls` is audited, so count them before you run.

Three ways to start the run:

- **Apify Console:** open [the Actor](https://apify.com/datatoolsforall/bulk-lighthouse-pagespeed-audit), switch the input to JSON, paste one of the inputs above, set **Maximum cost per run** to `1.10` in the run options, and start.
- **An AI agent:** with the [MCP server or plugin](https://github.com/slizer138/datatoolsforall-tools#install) connected, ask: *"Run a mobile Lighthouse audit on the first 50 pages of https://www.example.com/sitemap.xml, flag pages with LCP over 2.5 s, and cap the cost at $1.10."* The plugin's skill estimates the cost and asks before running anything over about $1.
- **The Apify API:** start the run, then fetch the results when it has finished (a 50-page run takes longer than the 5-minute synchronous endpoint allows).

```bash
curl -X POST "https://api.apify.com/v2/acts/datatoolsforall~bulk-lighthouse-pagespeed-audit/runs?maxTotalChargeUsd=1.10" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"sitemapUrl":"https://www.example.com/sitemap.xml","maxUrls":50,"strategies":["mobile"]}'
# The response contains data.id (run ID) and data.defaultDatasetId.
curl -H "Authorization: Bearer $APIFY_TOKEN" "https://api.apify.com/v2/actor-runs/<RUN_ID>"   # repeat until status is SUCCEEDED
curl -H "Authorization: Bearer $APIFY_TOKEN" "https://api.apify.com/v2/datasets/<DATASET_ID>/items?format=csv&fields=url,strategy,performanceScore,lcpMs,cls,tbtMs,seoScore,accessibilityScore,passedBudgets,errorCode" > audit.csv
```

`maxTotalChargeUsd` is a hard cap: the Actor checks the remaining budget before each audit and stops cleanly when it runs out.

## Step 3: read the results

Each row is one URL on one device. The fields you will use most:

| Field | Meaning |
|---|---|
| `url`, `finalUrl` | The URL you gave, and where it ended up after redirects |
| `performanceScore`, `accessibilityScore`, `bestPracticesScore`, `seoScore` | Lighthouse category scores, 0 to 100 |
| `lcpMs`, `cls`, `tbtMs`, `fcpMs`, `speedIndexMs`, `ttfbMs` | Lab metrics: Largest Contentful Paint, Cumulative Layout Shift, Total Blocking Time, First Contentful Paint, Speed Index, time to first byte |
| `opportunities` | The biggest suggested fixes: `[{id, title, savingsMs, savingsBytes}]` |
| `passedBudgets`, `budgetFailures` | Whether the page met your budgets, and which ones it missed, for example `"lcpMs: 4517 > 2500"` |
| `errorCode`, `error`, `warnings` | Why an audit failed (not charged), and Lighthouse warnings such as a page that never finished loading |

A simple way to work through them:

1. Sort by `performanceScore`, lowest first. The bottom ten pages are your list.
2. Group the bottom pages by template. If every product page is slow, fix the template once.
3. Count how often each `opportunities[].id` appears across pages. A fix that shows up on 40 of 50 pages (for example unused JavaScript or oversized images) is usually worth more than any single-page fix.
4. Filter `passedBudgets = false` and read `budgetFailures` for the exact misses. Google describes an LCP of 2,500 ms or less and a CLS of 0.1 or less as "good".

## What this does not tell you

- **Real-user experience.** These are lab measurements of one page load under simulated mobile conditions. Google's Core Web Vitals assessment in Search uses field data from real Chrome users, including INP (Interaction to Next Paint), which a lab audit cannot measure. TBT is the lab proxy for responsiveness. For field data, use Search Console, the Chrome UX Report, or the `psiApiKey` option above.
- **Exact numbers.** Scores move by a few points between runs because of network and server timing. Compare medians of several runs before acting on small differences.
- **Pages behind a login** or pages that need cookies or form input.

## Do it again next month

Save the input as an Apify task and add a schedule, or ask your agent to rerun the same list. Keep the same URLs and device so the numbers are comparable.
