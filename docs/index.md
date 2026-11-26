---
title: Data Tools for All
description: Pay-per-use data tools for AI agents and people - bulk domain availability, bulk Lighthouse audits, US business registry search and sitemap URL extraction, on your own Apify account.
---

# Data Tools for All

Four pay-per-use data tools that AI agents (Claude, Codex, Cursor, VS Code and any MCP client) can call, and that people can run by hand. Each tool is an Actor on the Apify platform and runs on **your own Apify account**.

| Task | Apify Actor | Price |
|---|---|---|
| Check if many domain names are available (registry RDAP), with registrar and expiry | [bulk-domain-availability-checker](https://apify.com/datatoolsforall/bulk-domain-availability-checker) | $0.001 per domain answered |
| Run Lighthouse on a URL list or the pages of a sitemap, mobile and desktop (lab metrics) | [bulk-lighthouse-pagespeed-audit](https://apify.com/datatoolsforall/bulk-lighthouse-pagespeed-audit) | $0.02 per URL per device |
| Search LLCs and corporations in NY, TX, CO, OR and CT state open data, or pull new filings | [state-business-entity-search](https://apify.com/datatoolsforall/state-business-entity-search) | $0.0015 per record |
| Get every URL from a website's sitemaps, with new and removed pages between runs | [website-sitemap-extractor](https://apify.com/datatoolsforall/website-sitemap-extractor) | $0.0002 per URL |

Rows that end in an error are free. Apify's free plan includes $5 of monthly usage credit.

**Disclosure:** Data Tools for All built these Actors and earns a share of what you pay for them through Apify.

## Connect an AI agent

Add this MCP server URL to your client and sign in to Apify in the browser when asked (no token to copy):

```text
https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor
```

Claude Code users can install the plugin, which adds four skills with cost caps and error handling:

```bash
claude plugin marketplace add slizer138/datatoolsforall-tools
claude plugin install datatoolsforall@datatoolsforall
```

Setup for Claude.ai, Codex, Cursor, VS Code, LangChain, CrewAI and the Apify API is in the [README on GitHub](https://github.com/slizer138/datatoolsforall-tools#readme).

## Guides

- [Audit every page of a sitemap with Lighthouse for about $1](sitemap-lighthouse-audit)

## Honest limits

- Lighthouse results are **lab** measurements under simulated conditions, not real-user field data.
- Texas business data is the Texas Comptroller's "Active Franchise Tax Permit Holders" dataset, not the Secretary of State register. The New York and Oregon datasets list active entities only.
- "Available" domains can still be premium or reserved at a registrar.
- The sitemap tool reads sitemaps; it does not crawl pages that are missing from them.

## About

- Source and issues: [github.com/slizer138/datatoolsforall-tools](https://github.com/slizer138/datatoolsforall-tools)
- Apify profile: [apify.com/datatoolsforall](https://apify.com/datatoolsforall)
- [Privacy policy](privacy) · [Terms](terms) · [llms.txt](llms.txt)
- Contact: datatoolsforall@gmail.com
