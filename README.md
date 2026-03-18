# Phoenix Current Software (PCS)

Service Fusion replacement — built by Phoenix Electric LLC.

**"Current"** = electrical current + up to date.

## What's Here

### `/docs/servicefusion/`
- `SERVICEFUSION_MCP_REWRITE_BRIEF.md` — Complete MCP rewrite specification. Compiled 2026-03-05 by Echo Pro. Covers correct API endpoints, auth flow, what to build (16 tools), what's broken in the old package, and live data snapshots.

### `/references/`
- `servicefusion-api-complete-spec.md` — 17,929-line structured API reference (75 types, 26 endpoints, 6,084 field definitions). Generated 2026-03-17 from RAML spec.
- `servicefusion-api-spec.json` — Raw 4.3MB RAML API specification scraped from SF docs.
- `servicefusion-web-scrape.json` — Scraped web documentation (official docs, Zendesk KB, blog posts).

## Service Fusion API Quick Reference

- **Base URL:** `https://api.servicefusion.com/v1/`
- **Auth:** OAuth 2.0 Client Credentials Grant
- **Token endpoint:** `POST https://api.servicefusion.com/oauth/access_token`
- **Operations:** GET + POST only. NO PUT, PATCH, or DELETE.
- **Rate limit:** 120 req/min
- **Tenant:** Phoenix Electric (ID 4324869397)

## Status

Repository created 2026-03-17. SF documentation gathered and organized. MCP rewrite brief ready for implementation.
