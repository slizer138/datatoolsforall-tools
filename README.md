# Data Tools for All: data tools for AI agents and people

Four pay-per-use data tools that Claude, Codex, Cursor, VS Code or any MCP client can call, and that you can also run by hand or from Python:

| Task | Apify Actor | Price |
|---|---|---|
| [Check if hundreds of domain names are available](#check-if-many-domain-names-are-available) | [`datatoolsforall/bulk-domain-availability-checker`](https://apify.com/datatoolsforall/bulk-domain-availability-checker) | $0.001 per domain answered |
| [Run Lighthouse on every page of a sitemap](#run-lighthouse-on-many-pages-or-a-whole-sitemap) | [`datatoolsforall/bulk-lighthouse-pagespeed-audit`](https://apify.com/datatoolsforall/bulk-lighthouse-pagespeed-audit) | $0.02 per URL per device |
| [Search LLCs and corporations in NY, TX, CO, OR and CT](#search-us-business-entities-in-ny-tx-co-or-and-ct) | [`datatoolsforall/state-business-entity-search`](https://apify.com/datatoolsforall/state-business-entity-search) | $0.0015 per record |
| [Get every URL from a website's sitemap](#get-every-url-from-a-websites-sitemap) | [`datatoolsforall/website-sitemap-extractor`](https://apify.com/datatoolsforall/website-sitemap-extractor) | $0.0002 per URL |

This repository holds the agent-side files: a Claude Code plugin with four skills, an Agent Plugins manifest for Codex and Cursor, MCP Registry entries, an llms.txt, and copy-paste setup for each client. The tools themselves run on Apify.

> **Disclosure.** I (Data Tools for All) built these four Actors and earn a share of what you pay for them through Apify. The skills and this page say when **not** to use them and what the free alternative is.

- Website: https://slizer138.github.io/datatoolsforall-tools/
- Tutorial: [Audit every page of a sitemap with Lighthouse for about $1](https://slizer138.github.io/datatoolsforall-tools/sitemap-lighthouse-audit)
- Privacy: https://slizer138.github.io/datatoolsforall-tools/privacy · Terms: https://slizer138.github.io/datatoolsforall-tools/terms
- Contact: datatoolsforall@gmail.com · Issues: https://github.com/slizer138/datatoolsforall-tools/issues

## How it works

All four tools are reachable through **Apify's hosted MCP server**. This URL loads exactly these four tools and nothing else:

```text
https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor
```

- **Sign-in:** the first call opens Apify's sign-in page in your browser (OAuth). No token to copy. You need a free Apify account.
- **Billing:** every run bills per result to *your* Apify account, at the prices above. Rows that end in an error are free. Apify's free plan includes $5 of monthly usage credit. You can see and abort every run in Apify Console.
- **Tool names:** each Actor becomes one MCP tool named `datatoolsforall--<actor-name>`. Apify also adds `get-actor-run`, `get-dataset-items`, `get-key-value-store-record` and `abort-actor-run` so the agent can follow long runs and page through results.
- **One tool only:** replace the `tools=` list with a single Actor, for example `https://mcp.apify.com/?tools=datatoolsforall/website-sitemap-extractor`.

## Install

### Claude Code (plugin with skills)

```bash
claude plugin marketplace add slizer138/datatoolsforall-tools
claude plugin install datatoolsforall@datatoolsforall
```

Inside a session the same works as `/plugin marketplace add slizer138/datatoolsforall-tools` and `/plugin install datatoolsforall@datatoolsforall`, or in one step on Claude Code 2.1.275 or later: `/plugin install datatoolsforall --marketplace slizer138/datatoolsforall-tools`. Then run `/mcp`, select the plugin's `dtfa` server and sign in to Apify. The plugin adds the MCP server and four skills (`bulk-domain-availability`, `bulk-web-vitals-audit`, `us-business-entity-search`, `sitemap-url-extractor`) that teach Claude the inputs, the cost caps and the failure modes.

MCP server only, without the skills:

```bash
claude mcp add --transport http dtfa "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor"
```

### Claude.ai and Claude Desktop

Settings > Connectors > Add custom connector, paste the MCP URL above, then sign in to Apify when asked.

### Codex

```bash
codex plugin marketplace add slizer138/datatoolsforall-tools
```

or the MCP server alone:

```bash
codex mcp add dtfa --url "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor"
codex mcp login dtfa
```

The Codex plugin route uses the [Agent Plugins](https://agent-plugins.org) files in `plugins/datatoolsforall/` (`plugin.json`, `mcp.json`) and `.agents/plugins/marketplace.json`. I have not tested it in Codex yet; please open an issue if it fails.

### Cursor

Add this to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "dtfa": {
      "url": "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor"
    }
  }
}
```

Or paste this install link into your browser's address bar:

```text
cursor://anysphere.cursor-deeplink/mcp/install?name=dtfa&config=eyJ1cmwiOiJodHRwczovL21jcC5hcGlmeS5jb20vP3Rvb2xzPWRhdGF0b29sc2ZvcmFsbC9idWxrLWRvbWFpbi1hdmFpbGFiaWxpdHktY2hlY2tlcixkYXRhdG9vbHNmb3JhbGwvYnVsay1saWdodGhvdXNlLXBhZ2VzcGVlZC1hdWRpdCxkYXRhdG9vbHNmb3JhbGwvc3RhdGUtYnVzaW5lc3MtZW50aXR5LXNlYXJjaCxkYXRhdG9vbHNmb3JhbGwvd2Vic2l0ZS1zaXRlbWFwLWV4dHJhY3RvciJ9
```

### VS Code (GitHub Copilot agent mode)

```bash
code --add-mcp '{"name":"dtfa","type":"http","url":"https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor"}'
```

or in `.vscode/mcp.json`:

```json
{
  "servers": {
    "dtfa": {
      "type": "http",
      "url": "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor"
    }
  }
}
```

### Skills only (skills CLI, any agent)

```bash
npx skills add slizer138/datatoolsforall-tools
```

The skills are plain `SKILL.md` files in [`plugins/datatoolsforall/skills/`](plugins/datatoolsforall/skills). They call the tools through the MCP server above, through the standard Apify MCP server's `call-actor` tool, or through the Apify CLI (`apify login`, then `apify call datatoolsforall/<actor> -i '<json>'`).

### Any other MCP client

Use the MCP URL above with the Streamable HTTP transport and let the client run OAuth. The same four tools are listed one by one in the [official MCP Registry](https://registry.modelcontextprotocol.io) as `io.github.slizer138/bulk-domain-availability-checker`, `io.github.slizer138/lighthouse-pagespeed-audit`, `io.github.slizer138/sitemap-url-extractor` and `io.github.slizer138/secretary-of-state-business-search` (see [`servers/`](servers)).

### Headless and CI: use an Apify API token

Only if your client cannot do OAuth (CI jobs, servers, scripts). Create a token in Apify Console > Settings > Integrations, keep it in an environment variable or secret store, and send it as a header. **Never put the token in the URL** (`?token=`) or commit it.

Claude Code:

```bash
claude mcp add --transport http dtfa "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor" --header "Authorization: Bearer $APIFY_TOKEN"
```

Cursor (`mcp.json`):

```json
{
  "mcpServers": {
    "dtfa": {
      "url": "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor",
      "headers": { "Authorization": "Bearer ${env:APIFY_TOKEN}" }
    }
  }
}
```

Codex (`~/.codex/config.toml`):

```toml
[mcp_servers.dtfa]
url = "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor"
bearer_token_env_var = "APIFY_TOKEN"
```

VS Code (`.vscode/mcp.json`, prompts for the token once and stores it securely):

```json
{
  "inputs": [
    { "type": "promptString", "id": "apify-token", "description": "Apify API token", "password": true }
  ],
  "servers": {
    "dtfa": {
      "type": "http",
      "url": "https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker,datatoolsforall/bulk-lighthouse-pagespeed-audit,datatoolsforall/state-business-entity-search,datatoolsforall/website-sitemap-extractor",
      "headers": { "Authorization": "Bearer ${input:apify-token}" }
    }
  }
}
```

### Python agent frameworks

LangChain (`pip install langchain-apify`, reads `APIFY_TOKEN`):

```python
from langchain_apify import ApifyActorsTool

sitemap_tool = ApifyActorsTool("datatoolsforall/website-sitemap-extractor")
result = sitemap_tool.invoke({"run_input": {"startUrls": ["https://example.com"], "maxUrlsPerSite": 500}})
```

CrewAI (`pip install 'crewai[tools]' langchain-apify`, reads `APIFY_API_TOKEN`):

```python
from crewai_tools import ApifyActorsTool

lighthouse_tool = ApifyActorsTool(actor_name="datatoolsforall/bulk-lighthouse-pagespeed-audit")
domain_tool = ApifyActorsTool(actor_name="datatoolsforall/bulk-domain-availability-checker")
```

Both wrappers build the tool's input schema from the Actor, so the field names below apply.

### Apify API (no agent)

```bash
curl -X POST "https://api.apify.com/v2/acts/datatoolsforall~website-sitemap-extractor/run-sync-get-dataset-items?maxTotalChargeUsd=0.20" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"startUrls":["https://example.com"],"maxUrlsPerSite":500}'
```

`maxTotalChargeUsd` caps what the run can cost; the Actors stop cleanly when they reach it.

## The tools

### Check if many domain names are available

`datatoolsforall/bulk-domain-availability-checker` asks the official registry RDAP servers (the successor of WHOIS), with a public-DNS prefilter, whether each name is registered. Paste up to 5,000 domains, or combine keywords with TLDs, prefixes and suffixes.

- **Input:** `{"domains": ["brightparcel.com", "brightparcel.io"], "maxDomains": 10}` or `{"keywords": ["brightparcel"], "tlds": ["com", "io", "ai"], "maxDomains": 50}`
- **Output fields:** `domain`, `status` (`available`, `registered`, `unknown`, `unsupported_tld`, `invalid`), `registrar`, `createdDate`, `expirationDate`, `daysUntilExpiry`, `domainAgeDays`, `statusCodes`, `nameservers`, `note`, `errorCode`, and more.
- **Price:** $0.001 per domain answered available or registered ($1 per 1,000). Unknown, unsupported and invalid rows are free. `maxDomains` defaults to 500 (at most $0.50).
- **Limits:** "available" means the registry has no registration; premium or reserved names can still be unavailable or cost more at a registrar. No prices, no purchases, no registrant contact data.
- **Free alternative for one or two names:** your registrar's search box.

### Run Lighthouse on many pages or a whole sitemap

`datatoolsforall/bulk-lighthouse-pagespeed-audit` runs Lighthouse 13 in headless Chrome on Apify, one audit at a time, for mobile, desktop or both. No Google API key or quota is needed.

- **Input:** `{"urls": ["https://example.com/"], "strategies": ["mobile"]}` or `{"sitemapUrl": "https://example.com/sitemap.xml", "maxUrls": 50}`
- **Output fields:** `url`, `strategy`, `performanceScore`, `accessibilityScore`, `bestPracticesScore`, `seoScore`, `lcpMs`, `fcpMs`, `tbtMs`, `cls`, `speedIndexMs`, `ttfbMs`, `opportunities`, `budgetFailures`, `passedBudgets`, `errorCode`, and more.
- **Price:** $0.02 per URL per device, or $0.003 if you add your own PageSpeed Insights key. Error rows are free. `maxUrls` (pages taken from a sitemap) defaults to 50 (at most $1.00 per device); every pasted URL is audited.
- **Limits:** these are **lab** metrics under simulated conditions, not real-user field data; INP needs field data (the optional own-key mode adds Chrome UX Report data when Google has it). Audit only sites you own or may test. About 10 to 60 seconds per audit.
- **Free alternative for one page:** Lighthouse in Chrome DevTools or pagespeed.web.dev.

### Search US business entities in NY, TX, CO, OR and CT

`datatoolsforall/state-business-entity-search` searches LLCs, corporations, nonprofits and partnerships by name or state ID, or pulls new filings since a date, from official state open data, normalized to one schema with a link to each source record.

- **Input:** `{"states": ["CO"], "searchNames": ["acme"], "maxResultsPerState": 20}` or `{"states": ["CO"], "formedAfter": "7 days", "maxResultsPerState": 500}`
- **Output fields:** `state`, `entityId`, `entityName`, `entityType`, `status`, `isActive`, `formationDate`, `principalAddress`, `mailingAddress`, `registeredAgent`, `matchedBy`, `sourceDataset`, `sourceUrl`, `errorCode`, and more.
- **Price:** $0.0015 per record ($1.50 per 1,000), including a definitive "not found" for an ID you asked for. Error rows are free. `maxResultsPerState` defaults to 100 (at most $0.15 per state); set it to what you need.
- **Sources and limits:** NY Department of State, CO Department of State, CT Secretary of the State and OR Secretary of State datasets; **Texas data is the Texas Comptroller's "Active Franchise Tax Permit Holders" dataset**, not the Secretary of State register. The New York and Oregon datasets list active entities only. Person names (agents, officers) are off by default and only for lawful uses such as KYB.
- **Free alternative for one lookup:** the state's own business search page.

### Get every URL from a website's sitemap

`datatoolsforall/website-sitemap-extractor` finds a site's sitemaps through robots.txt and common paths, follows sitemap indexes and .xml.gz files, and returns one row per URL. It can also check the HTTP status of each URL and report new and removed URLs since the previous run.

- **Input:** `{"startUrls": ["https://example.com"], "maxUrlsPerSite": 500}`
- **Output fields:** `url`, `site`, `sitemapUrl`, `lastmod`, `changefreq`, `priority`, `alternates`, `imageCount`, `videoCount`, `statusCode`, `redirectLocation`, `diffStatus`, `errorCode`, and more.
- **Price:** $0.0002 per URL ($0.20 per 1,000), plus $0.0005 per URL with the status check. Most error rows are free. `maxUrlsPerSite` defaults to 10,000 (at most $2.00 per site), so set it.
- **Limits:** reads sitemaps only, it is not a crawler; pages missing from the sitemap are not found. Sites that block bots get a `BLOCKED` row; nothing is bypassed.
- **Free alternative for a small site:** open `/sitemap.xml` in your browser.

## Cost guards the skills follow

- Always set the cap input: `maxDomains`, `maxUrls`, `maxUrlsPerSite` or `maxResultsPerState`.
- Estimate the cost before running, and ask the user before any run estimated at more than about $1.
- With the standard Apify MCP server, pass `callOptions.maxTotalChargeUsd` to `call-actor`; with the API, the `maxTotalChargeUsd` query parameter.
- One run per request, no repeated runs to double-check, and long runs are followed with `get-actor-run` instead of being started again.

## Repository layout

```text
.claude-plugin/marketplace.json   Claude Code marketplace (this repo)
.agents/plugins/marketplace.json  Codex marketplace
plugins/datatoolsforall/          the plugin: .claude-plugin/plugin.json, .mcp.json,
                                  plugin.json + mcp.json (Agent Plugins 1.0.0), README, LICENSE, icon,
                                  skills/<skill>/SKILL.md
servers/<name>/server.json        official MCP Registry entries, one per tool
docs/                             GitHub Pages site: overview, tutorial, privacy, terms, llms.txt
llms.txt                          short index for LLMs
skills.sh.json                    tells the skills CLI where the skills are
```

## License

MIT, see [LICENSE](LICENSE). The Actors are separate products on Apify; running them is subject to Apify's terms and the prices above.
