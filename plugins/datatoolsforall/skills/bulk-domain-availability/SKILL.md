---
name: bulk-domain-availability
description: Check whether many domain names are available or registered, from the official registry RDAP services (the WHOIS successor) with a public-DNS prefilter, and get registrar, created and expiry dates, days until expiry, domain age, status codes and nameservers for taken names. Takes a list of up to 5,000 domains, or keywords combined with TLDs, prefixes and suffixes for name brainstorming. Use when the user wants to find a free domain for a product or company name, check a list of domains in bulk, or look up expiry dates and registrars for a portfolio. Runs the pay-per-use Apify Actor datatoolsforall/bulk-domain-availability-checker on the user's own Apify account (USD 0.001 per domain answered; unknown, unsupported and invalid rows are free).
license: MIT
compatibility: Needs this plugin's MCP server (mcp.apify.com with Apify OAuth sign-in), or the standard Apify MCP server with call-actor, or the Apify CLI after apify login. Needs an Apify account.
metadata:
  author: Data Tools for All
  actor: datatoolsforall/bulk-domain-availability-checker
  disclosure: The author built this Actor and earns from paid runs.
---

# Bulk domain availability (RDAP)

Free-looking names are confirmed with the registry's RDAP server. Taken names are usually answered from public DNS (Google and Cloudflare DNS-over-HTTPS) first, and then from RDAP when registration details are wanted.

## When to use

- "Is brightparcel.com free? What about .io and .ai?" or "find me an available .com for these 20 name ideas".
- Checking a pasted list of domains (up to 5,000) in one go.
- Expiry dates, registrar and age for a list of domains the user owns or is watching.

## When not to use

- **One or two names**: a registrar's search box or a single RDAP lookup is free. Suggest that first (you can still run it if the user prefers).
- **Price or purchase**: "available" means the registry has no registration. Premium, reserved or blocked names can still be unavailable or cost more at a registrar. This tool does not buy domains and does not show prices.
- Owner contact details (registrant name, email): RDAP for most gTLDs redacts them, and this tool does not return them.
- Trademark clearance: suggest a trademark search; availability says nothing about legal use.

## Tools

1. This plugin's MCP server exposes the Actor as its own tool, named `datatoolsforall--bulk-domain-availability-checker` (your client may add a prefix, for example `mcp__plugin_datatoolsforall_dtfa__...` in Claude Code). Loading it also loads `get-actor-run`, `get-dataset-items`, `get-key-value-store-record` and `abort-actor-run`.
2. If only the standard Apify MCP server is connected, use `call-actor` with `actor: "datatoolsforall/bulk-domain-availability-checker"`, the same `input`, and `callOptions: { "maxTotalChargeUsd": <cap> }`.

The Actor tool waits up to `waitSecs` seconds (default 30, maximum 45) and then returns the run status and a `nextStep`. Small runs usually finish within that wait. For larger runs (hundreds of domains with registration details), call `get-actor-run` with its `runId` and `waitSecs: 45` until the status is SUCCEEDED, FAILED, ABORTED or TIMED-OUT, and never start a second run for the same request. Read rows with `get-dataset-items`.

## Cost and guardrails (billed to the user's Apify account)

- USD 0.001 per domain with a definitive answer (`available` or `registered`). Rows with `unknown`, `unsupported_tld` or `invalid` are free. Plus a start fee of a fraction of a cent per run.
- Domains checked = pasted `domains` + `keywords` x (1 + number of `prefixes` + number of `suffixes`) x number of `tlds`, capped by `maxDomains`. Hyphen variants add more.
- **Always set `maxDomains`.** The Actor default is 500 (at most USD 0.50). The maximum is 5,000 (USD 5.00).
- If the estimate is above about USD 1 (more than about 1,000 domains), tell the user the count and cost and wait for a yes.
- For brainstorming, start with few TLDs (for example `com`, `io`, `ai`) and add more only for the names the user likes.

## Input fields

All fields are optional, but give `domains` or `keywords`.

| Field | Type | Default | Notes |
|---|---|---|---|
| `domains` | array of strings (up to 5,000) | none | Full names such as `example.com`. URLs are fine (`https://`, paths and `www.` are removed). IDNs such as `müller.de` work. |
| `keywords` | array of strings (up to 200) | none | Name ideas without an ending; spaces are removed (`bright parcel` -> `brightparcel`). |
| `tlds` | array of strings (up to 100) | `com, net, org, io, ai, co, dev, app` | Endings without the dot, for example `co.uk`. Used only with `keywords`. Non-existent endings are skipped with a warning. |
| `prefixes` | array of strings (up to 50) | none | Adds `get` + keyword, `try` + keyword, ... |
| `suffixes` | array of strings (up to 50) | none | Adds keyword + `app`, keyword + `hq`, ... |
| `includeHyphenVariants` | boolean | false | Also `bright-parcel`, `get-brightparcel`; they are dropped first when `maxDomains` is reached. |
| `includeRegistrationDetails` | boolean | true | Registrar, dates, age and status codes for taken names from the `domains` list. Turn off for a faster available/registered-only answer. |
| `detailsForKeywordNames` | boolean | false | Also look up details for taken names generated from keywords (more registry traffic). |
| `skipDnsPrefilter` | boolean | false | Leave off. When on, free .co, .de, .io and .me names come back `unknown`, and taken names under TLDs without RDAP come back `unsupported_tld`. |
| `maxDomains` | integer 1 to 5,000 | 500 | Cap on names checked in this run, counting generated combinations. |

Minimal input: `{"domains": ["brightparcel.com", "brightparcel.io"], "maxDomains": 10}`.

## Output fields (one dataset row per domain)

`domain` (ASCII/punycode), `unicodeDomain`, `tld`, `input`, `status` (`available`, `registered`, `unknown`, `unsupported_tld`, `invalid`), `checkedVia` (`rdap`, `dns`, `rdap+dns`), `rdapServer`, `registrar`, `registrarIanaId`, `createdDate`, `expirationDate`, `lastChangedDate`, `domainAgeDays`, `daysUntilExpiry`, `statusCodes`, `nameservers`, `dnssecSigned`, `note`, `errorCode`, `error`, `checkedAt`.

Taken names generated from keywords are answered from DNS only by default, so their registrar and dates are `null` unless `detailsForKeywordNames` is on. Some registries (for example DENIC for .de) do not publish an expiry date.

How to present results: list `available` names first, then `registered` with registrar and `expirationDate`, then anything `unknown` with its `errorCode`. Repeat the `note` caveat that premium or reserved names may still be unavailable.

## Errors and what to do

`errorCode` values: `INVALID_DOMAIN`, `TLD_NOT_IN_ROOT`, `NO_RDAP_SERVICE`, `TIMEOUT`, `NETWORK_ERROR`, `RATE_LIMITED`, `SOURCE_ERROR`, `HOST_UNAVAILABLE`, `BLOCKED`, `UNEXPECTED_STATUS`, `CONFLICT`, `UNCONFIRMED`, `DNS_ERROR`.

- `status: unknown` (free): the registry gave no reliable answer (timeouts, rate limits, conflicting sources). Re-check only those names later, in one small run.
- `status: unsupported_tld` (free): the TLD has no RDAP service and DNS showed no registration; tell the user to check that TLD at a registrar.
- `status: invalid` (free): not a valid domain name; show the user the `input` value.
- If the final status message says `maxDomains` was reached, tell the user which combinations were left out and offer a follow-up run.

## Example prompts

- "Which of these 300 domains are available? [list]" -> `{"domains": [...], "maxDomains": 300}` (at most USD 0.30).
- "Find an available name for a parcel-tracking startup: try brightparcel, parcelpal and shipnest with .com, .io and .ai, plus get- and -app variants." -> `{"keywords": ["brightparcel", "parcelpal", "shipnest"], "tlds": ["com", "io", "ai"], "prefixes": ["get"], "suffixes": ["app"], "maxDomains": 50}` = 3 x 3 x 3 = 27 names, at most USD 0.03.
- "When do my 40 domains expire, and who is the registrar?" -> `{"domains": [...40], "includeRegistrationDetails": true, "maxDomains": 40}`, then sort by `daysUntilExpiry`.

Out of scope: "Buy brightparcel.com for me" or "how much does it cost?" needs a registrar; this tool only reports registry status.
