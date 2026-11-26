---
name: us-business-entity-search
description: Search registered LLCs, corporations, nonprofits and partnerships by name or by state ID, or pull every new filing since a date, from official state open data for New York, Texas, Colorado, Oregon and Connecticut, normalized to one schema (status, entity type, formation date, addresses, registered agent where published, source link). Use for KYB and vendor checks, due diligence, confirming a company exists and is active in one of these five states, or building a feed of newly formed businesses. Texas data is the Texas Comptroller's franchise-tax permit holder list, and the New York and Oregon datasets list active entities only. Runs the pay-per-use Apify Actor datatoolsforall/state-business-entity-search on the user's own Apify account (USD 0.0015 per record).
license: MIT
compatibility: Needs this plugin's MCP server (mcp.apify.com with Apify OAuth sign-in), or the standard Apify MCP server with call-actor, or the Apify CLI after apify login. Needs an Apify account.
metadata:
  author: Data Tools for All
  actor: datatoolsforall/state-business-entity-search
  disclosure: The author built this Actor and earns from paid runs.
---

# US business entity search (NY, TX, CO, OR, CT)

Queries each state's official open-data portal (Socrata SODA API) and returns one normalized row per entity, with a `sourceUrl` that reproduces the record at the source.

| State | Publisher and dataset | Coverage note |
|---|---|---|
| NY | NY Department of State, "Active Corporations: Beginning 1800" (data.ny.gov) | Active entities only |
| CO | CO Department of State, "Business Entities in Colorado" (data.colorado.gov) | All statuses |
| CT | CT Secretary of the State, "Business Registry - Business Master" plus agents and principals (data.ct.gov) | All statuses; rejected filings and name reservations excluded |
| OR | OR Secretary of State, "Active Businesses - ALL" (data.oregon.gov) | Active entities only; assumed business names (DBAs) included as type `Other` |
| TX | **TX Comptroller of Public Accounts**, "Active Franchise Tax Permit Holders" (data.texas.gov) | Not the Secretary of State register: some entity types are missing, no agents or officers, mailing address only |

Never describe Texas results as Secretary of State data. For the legal standing of a Texas entity, point the user to the Texas Comptroller or Secretary of State.

## When to use

- "Is ACME Cleaning LLC registered in Colorado, and is it active?"
- Looking up exact records by state ID (NY DOS ID, CO entity ID, CT account number, OR registry number, TX taxpayer number or TX SOS file number).
- "New LLCs formed in Colorado in the last 7 days" (new-filings feed with `formedAfter` alone).
- Vendor onboarding and KYB checks for companies in these five states.

## When not to use

- Any state other than NY, TX, CO, OR and CT: say this tool does not cover it and point to that state's Secretary of State business search.
- Inactive or dissolved entities in New York or Oregon: they are not in those datasets, so "not found" there can mean "no longer active".
- Certified good-standing certificates, filed documents, ownership or beneficial-owner data: not in these datasets.
- A single lookup the user can do on the state's own free search page: suggest it if they only need one record.
- Collecting personal data for marketing or profiling. Person names (agents, officers) are off by default and only for lawful purposes such as KYB and due diligence.

## Tools

1. This plugin's MCP server exposes the Actor as its own tool, named `datatoolsforall--state-business-entity-search` (your client may add a prefix, for example `mcp__plugin_datatoolsforall_dtfa__...` in Claude Code). Loading it also loads `get-actor-run`, `get-dataset-items`, `get-key-value-store-record` and `abort-actor-run`.
2. If only the standard Apify MCP server is connected, use `call-actor` with `actor: "datatoolsforall/state-business-entity-search"`, the same `input`, and `callOptions: { "maxTotalChargeUsd": <cap> }`.
3. Without MCP: `apify call datatoolsforall/state-business-entity-search -i '<input JSON>' -s -o` after `apify login`.

The Actor tool waits up to `waitSecs` seconds (default 30, maximum 45) and then returns the run status and a `nextStep`. Most runs finish within that wait. If not, call `get-actor-run` with its `runId` and `waitSecs: 45` until the status is SUCCEEDED, FAILED, ABORTED or TIMED-OUT, and never start a second run for the same request. Read rows with `get-dataset-items`.

## Cost and guardrails (billed to the user's Apify account)

- USD 0.0015 per entity record, and per definitive "ID not found" answer for an ID the user asked for. Error rows (`RATE_LIMITED`, `SOURCE_UNAVAILABLE`, `TIMEOUT`, ...) and `INVALID_ID` rows are free. Plus a start fee of a fraction of a cent per run.
- Cost = records returned x 0.0015, at most `states` x `maxResultsPerState` x 0.0015 for name and date searches, plus one record per requested ID.
- **Always set `maxResultsPerState` explicitly.** The current default is 100 per state (at most USD 0.15 per state, USD 0.75 for all five), but set the value the task needs: for a "does this company exist" check, 20 is plenty; the maximum, 50,000, would be up to USD 75 per state.
- Search only the states the user cares about.
- If the estimate is above about USD 1, tell the user and wait for a yes. Example: a 30-day new-filings feed for Colorado can return several thousand records.

## Input fields

Required: `states`.

| Field | Type | Default | Notes |
|---|---|---|---|
| `states` | array of `NY`, `CO`, `CT`, `OR`, `TX` (unique, up to 5) | none | Registries to search. |
| `searchNames` | array of strings (up to 20, at least 2 characters each) | none | Substring match on the name, not case sensitive; `acme` finds `ACME CLEANERS, INC.` Shorter terms find more. |
| `entityIds` | array of strings (up to 100) | none | Exact IDs, digits only. Prefix a state to search one state: `NY:4424185`. Every requested ID gets a row (a `NOT_FOUND` row if missing). |
| `formedAfter` | string | none | `YYYY-MM-DD` or relative (`7 days`, `2 weeks`, `1 month`). Alone, it gives a feed of new filings, newest first. |
| `formedBefore` | string | none | Same format. |
| `maxResultsPerState` | integer 1 to 50,000 | 100 | Name matches or new filings per state, newest formation date first. IDs are returned on top of this limit. |
| `onlyActive` | boolean | false | Skip dissolved, withdrawn and revoked records (no effect in NY and OR). |
| `entityTypes` | array of `LLC`, `Corporation`, `Nonprofit`, `LP`, `Other` | all | Normalized types. Leave out `Other` to drop Oregon DBAs. |
| `includeAgentAndOfficerNames` | boolean | false | Adds individual registered agents and officer or principal names where the state publishes them (NY chairman, CT principals, OR representatives). Only for lawful purposes. |
| `socrataAppToken` | string (secret) | none | Optional quota token from a Socrata portal. Only if the user configures it themselves; never ask for it in chat. |

Minimal input: `{"states": ["CO"], "searchNames": ["acme"], "maxResultsPerState": 20}` (at most USD 0.03).

## Output fields (one dataset row per entity)

`state`, `entityId`, `secondaryId` (TX SOS file number; CT internal id), `entityName`, `nameNote` (CO status note split from the name), `entityTypeRaw`, `entityType`, `status` (as published), `isActive` (`true`, `false` or `null` when the source status cannot be interpreted), `sosStatusCode` (TX only, as published, not interpreted), `formationDate` (`YYYY-MM-DD`), `jurisdictionOfFormation`, `county` (NY only), `principalAddress` and `mailingAddress` (`{line1, line2, city, state, zip, country}`), `registeredAgent` (`{agentType, name, organizationName, address}`), `officers` (`[{name, title}]`, only with `includeAgentAndOfficerNames`), `matchedBy` (`entityId`, `name:<term>` or `formationDate`), `dataWarnings`, `sourceDataset`, `sourceUrl`, `dataUpdatedAt`, `retrievedAt`, `error`, `errorCode`.

How to present results: name, state, type, status, formation date and city, with the `sourceUrl` so the user can verify. State the source dataset for each state, and the NY/OR active-only and Texas Comptroller caveats when relevant. Texas `status` is `Active` only for right-to-transact code `A`; other codes are shown raw with `isActive: null`, so do not call those inactive.

## Errors and what to do

`errorCode` values: `NOT_FOUND`, `PLACEHOLDER_ID`, `INVALID_ID`, `SCAN_LIMIT`, `BLOCKED`, `RATE_LIMITED`, `SOURCE_UNAVAILABLE`, `TIMEOUT`, `NETWORK_ERROR`, `SCHEMA_CHANGED`, `DATASET_NOT_FOUND`, `INVALID_APP_TOKEN`, `BAD_RESPONSE`, `HTTP_ERROR`.

- `NOT_FOUND` for an ID (billed): the ID is not in that dataset. In NY and OR the entity may be inactive; in TX it may not hold a franchise-tax permit.
- `INVALID_ID` (free): the ID has letters or the wrong form; all five states use digits-only IDs. CT account numbers keep their leading zero (`0293297`).
- `RATE_LIMITED`, `SOURCE_UNAVAILABLE`, `TIMEOUT`, `NETWORK_ERROR` (free): the state portal is busy or down. Say which state failed and offer one retry later; other states' rows are still valid.
- `SCHEMA_CHANGED`, `DATASET_NOT_FOUND` (free): the state changed its dataset. Report it; do not retry in a loop.
- No match for a name: try a shorter or different part of the name (for example without "LLC" or punctuation).

## Example prompts

- "Is there an active company called Bright Parcel in New York or Connecticut?" -> `{"states": ["NY", "CT"], "searchNames": ["bright parcel"], "onlyActive": true, "maxResultsPerState": 20}`.
- "New LLCs formed in Colorado in the last 7 days." -> `{"states": ["CO"], "formedAfter": "7 days", "entityTypes": ["LLC"], "maxResultsPerState": 500}` (at most USD 0.75).
- "Look up NY DOS ID 4424185 and Colorado entity 20191234567." -> `{"states": ["NY", "CO"], "entityIds": ["NY:4424185", "CO:20191234567"]}`.

Out of scope: "Find the owner's phone number and email for these LLCs" is not in this data and is not a purpose this tool supports.
