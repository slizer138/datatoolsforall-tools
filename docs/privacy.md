---
title: Privacy policy
description: What the Data Tools for All plugin, skills and Apify Actors send where, and what the author can and cannot see.
---

# Privacy policy

Last updated: 2026-11-26

This policy covers the files in the [datatoolsforall-tools](https://github.com/slizer138/datatoolsforall-tools) repository (the Claude Code plugin, the skills, the MCP configuration and the MCP Registry entries) and the four Apify Actors they call, published by Data Tools for All ("we").

## Short version

- The plugin and skills are configuration and instructions. They contain no code that runs on your computer, no analytics and no credentials.
- When your agent uses a tool, your request goes to Apify's hosted MCP server and runs on Apify's platform, under your own Apify account. We do not operate any server in that path.
- We never receive your Apify credential. Your MCP client gets it from Apify's sign-in page and stores it.

## What is sent where

1. **Your AI client to Apify.** Tool calls go to `mcp.apify.com`, operated by Apify Technologies s.r.o. The call contains the input your agent prepared, for example domain names or keywords, page URLs or a sitemap URL, business names or state IDs, or website addresses. Apify starts an Actor run in your account and stores the run's input, log and results there. Apify's handling of this data is described in [Apify's privacy policy](https://apify.com/privacy-policy).
2. **The Actors to public sources.** During a run, each Actor contacts only the public sources needed for the job:
   - Domain availability: the registries' RDAP servers (found through IANA's RDAP bootstrap files) and public DNS-over-HTTPS resolvers from Google and Cloudflare.
   - Lighthouse audit: each web page you list, loaded once per device in a headless Chrome browser on Apify, and the sitemap you name. Only if you add your own PageSpeed Insights API key are the URLs sent to Google's PageSpeed Insights API under your key.
   - Business entity search: the official open-data portals of New York, Colorado, Connecticut, Oregon and Texas (data.ny.gov, data.colorado.gov, data.ct.gov, data.oregon.gov, data.texas.gov).
   - Sitemap extraction: robots.txt and sitemap files of the websites you name, and, only if you turn on the status check, one HEAD request per listed URL.
3. **Nothing else.** The plugin has no hooks, scripts or telemetry, and the skills do not tell your agent to send data anywhere else.

## What we can see

As the publisher of the Actors, Apify gives us usage and earnings statistics for them (for example run counts and charged events). We do not receive your Apify credential, and we do not ask you for personal information. If you contact us by email or open a GitHub issue, we see what you send there and use it only to answer you.

## Personal data in results

- The domain tool returns registry data about domains; it does not return registrant contact details.
- The business search tool returns what the states publish. Names and addresses of individual registered agents and officers are left out unless you turn on `includeAgentAndOfficerNames`. Use such data only for lawful purposes such as KYB and due diligence.
- The Lighthouse and sitemap tools return technical data about web pages.

You control your results in your Apify account and can delete runs and datasets there at any time.

## Children

These tools are not directed at children under 18.

## Changes and contact

We will update this page if the data flow changes, and note the date above. Questions: datatoolsforall@gmail.com.
