# Briefcase MCP Public Status API

**Briefcase MCP Public Status API** — live chain state for the Briefcase MCP server at `api.gctools.dpdns.org`.

**Current chain generation: 41642**

Last updated: 2026-04-18T23:12:46Z

Raw status text: Briefcase MCP is up. The current chain generation is 41642.

## Purpose

This repository mirrors the live Briefcase MCP status endpoint into a search-indexed artifact so that search-only clients (e.g. voice-mode AI assistants) can discover the current chain generation via web search.

Search for the phrase `Briefcase MCP Public Status API` and this repository's README should surface with the current generation number in the snippet.

Alternate sentinel phrase (guaranteed unique): `gctools-briefcase-live-chain-generation-mirror-b2e4a1f7`

## Source endpoints

- `https://api.gctools.dpdns.org/mcp/public/status` — main status (plain text)
- `https://api.gctools.dpdns.org/mcp/public/ping` — health check
- `https://api.gctools.dpdns.org/mcp/public/current_generation` — generation only
- `https://api.gctools.dpdns.org/mcp/public/` — index

## Update cadence

Auto-updated every 5 minutes by a systemd-supervised process on the Briefcase host. If the "Last updated" timestamp above is more than 15 minutes stale, the updater has failed and the number is likely out of date.

## Related mirror

A GitHub Gist mirror also exists:
https://gist.github.com/badnewsgoonies-dot/2234b9d35c8f61f75f197d670a86120c
