# MCP Registry entries

One `server.json` per tool, in the [official MCP Registry](https://registry.modelcontextprotocol.io) format (schema `2025-12-11`). Each entry points at Apify's hosted MCP server, limited to one Actor with the `?tools=` parameter. The registry stores metadata only; there is no code to run here.

| Folder | Registry name | Remote URL |
|---|---|---|
| `bulk-domain-availability-checker` | `io.github.slizer138/bulk-domain-availability-checker` | `https://mcp.apify.com/?tools=datatoolsforall/bulk-domain-availability-checker` |
| `lighthouse-pagespeed-audit` | `io.github.slizer138/lighthouse-pagespeed-audit` | `https://mcp.apify.com/?tools=datatoolsforall/bulk-lighthouse-pagespeed-audit` |
| `sitemap-url-extractor` | `io.github.slizer138/sitemap-url-extractor` | `https://mcp.apify.com/?tools=datatoolsforall/website-sitemap-extractor` |
| `secretary-of-state-business-search` | `io.github.slizer138/secretary-of-state-business-search` | `https://mcp.apify.com/?tools=datatoolsforall/state-business-entity-search` |

The `Authorization` header is optional: clients that support OAuth sign in to Apify in the browser instead. Runs bill per event to the caller's own Apify account.

## Publishing (maintainer)

Using `mcp-publisher` from https://github.com/modelcontextprotocol/registry/releases:

```bash
mcp-publisher login github          # device flow: enter the code at github.com/login/device
for d in bulk-domain-availability-checker lighthouse-pagespeed-audit sitemap-url-extractor secretary-of-state-business-search; do
  (cd "servers/$d" && mcp-publisher validate && mcp-publisher publish)
done
curl -s "https://registry.modelcontextprotocol.io/v0.1/servers?search=io.github.slizer138"
```

For an update, raise `version` in the file (never reuse a version) and publish again.
