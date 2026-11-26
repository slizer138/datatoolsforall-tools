# Data Tools for All

Four data tools for Claude and other AI agents, packaged as one plugin. Each tool is a pay-per-use Actor on the Apify platform that runs on **your own Apify account**:

- **Bulk domain availability**: check which domain names are free, straight from the official registries (RDAP), with registrar, expiry date and age for taken names.
- **Bulk Lighthouse audits**: run real Lighthouse on a list of URLs or every page of a sitemap, mobile and desktop, and get scores, lab Core Web Vitals (LCP, CLS, TBT and more) and the top suggested fixes.
- **US business entity search**: look up LLCs and corporations by name or state ID, or pull new filings by date, from the official open data of New York, Texas, Colorado, Oregon and Connecticut.
- **Sitemap URL extraction**: list every URL in a website's sitemaps with lastmod and hreflang, and see which pages are new or removed since the last run.

The plugin adds one remote MCP server (Apify's hosted server at `mcp.apify.com`, limited to these four Actors) and four skills that tell Claude when to use each tool, which inputs to set, how to keep cost under control and how to handle errors.

## Install

In Claude Code:

```text
/plugin install datatoolsforall --marketplace slizer138/datatoolsforall-tools
```

On older Claude Code versions, first run `/plugin marketplace add slizer138/datatoolsforall-tools`, then `/plugin install datatoolsforall@datatoolsforall`. Afterwards open `/mcp`, select the plugin's `dtfa` server and sign in to Apify.

## Disclosure

The author, Data Tools for All, built these four Actors and earns a share of what you pay for them through Apify. The skills tell Claude when **not** to use them and what the free alternative is.

## How sign-in and billing work

The first time Claude calls a tool, your MCP client opens Apify's own sign-in page in your browser (OAuth). Your client stores the resulting credential. This plugin contains no credential, never asks you for one, and the author never sees it.

Every run is billed to your Apify account per result, at the prices below. Apify's free plan includes a monthly usage credit. You can see and stop every run in Apify Console.

## Prices

| Tool | Apify Actor | Price | Free |
|---|---|---|---|
| Domain availability | `datatoolsforall/bulk-domain-availability-checker` | $0.001 per domain answered (available or registered) | rows with status unknown, unsupported_tld or invalid |
| Lighthouse audit | `datatoolsforall/bulk-lighthouse-pagespeed-audit` | $0.02 per URL per device, or $0.003 if you add your own PageSpeed Insights key | rows with an error code |
| Business entity search | `datatoolsforall/state-business-entity-search` | $0.0015 per entity record or definitive "ID not found" answer | error rows and invalid IDs |
| Sitemap URLs | `datatoolsforall/website-sitemap-extractor` | $0.0002 per URL row, plus $0.0005 per URL if you turn on the HTTP status check | most error rows |

Apify also charges a start fee of a fraction of a cent per run. The skills always set a result cap and ask you before any run they estimate at more than about $1.

## What is sent where

- **Your agent to Apify.** Tool calls go to `mcp.apify.com`, which Apify operates. Apify starts the Actor run on its platform with the input Claude prepared: domain names or keywords, page URLs or a sitemap URL, company names or state IDs, or website addresses. Apify stores the run's input and results in your Apify account.
- **The Actors to public sources.** The domain tool asks the official registry RDAP servers and public DNS-over-HTTPS resolvers (Google and Cloudflare). The Lighthouse tool loads each page you list in a headless Chrome browser on Apify, or sends the URLs to Google's PageSpeed Insights API only when you provide your own key. The business search tool queries the state open-data portals (data.ny.gov, data.colorado.gov, data.ct.gov, data.oregon.gov, data.texas.gov). The sitemap tool fetches robots.txt and sitemap files from the websites you name, and sends one HEAD request per URL only when you turn on the status check.
- **Nothing else.** The plugin has no hooks, no scripts and no other servers. The author operates no server in this path.

## Data notes

- Lighthouse results are **lab** measurements under simulated conditions, not real-user field data.
- Texas business data is the Texas Comptroller's "Active Franchise Tax Permit Holders" dataset, not the Secretary of State register. The New York and Oregon datasets list active entities only.
- Audit only websites you own or are allowed to test, and use any personal names in business records only for lawful purposes such as KYB and due diligence.

## Links

- Source, install guides for other clients, and support: https://github.com/slizer138/datatoolsforall-tools
- Privacy policy: https://slizer138.github.io/datatoolsforall-tools/privacy
- Terms: https://slizer138.github.io/datatoolsforall-tools/terms
- Contact: datatoolsforall@gmail.com

License: MIT.
