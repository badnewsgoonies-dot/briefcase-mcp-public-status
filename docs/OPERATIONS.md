# Operations — Briefcase MCP status mirror system

Documentation for operators of this system. The `README.md` at the repo root
is auto-updated and should not be edited manually.

## Architecture

Four search-indexable mirrors are kept in sync with the live chain generation
at `http://127.0.0.1:8765/mcp/public/status` by a single updater process on
the Briefcase host:

| Mirror         | URL                                                                                              | Updated every |
|----------------|--------------------------------------------------------------------------------------------------|---------------|
| Primary gist   | https://gist.github.com/badnewsgoonies-dot/2234b9d35c8f61f75f197d670a86120c                      | 5 min         |
| Sentinel gist  | https://gist.github.com/badnewsgoonies-dot/35473d4f6c7ee0f14b49d410c037cc13                      | 5 min         |
| Repo README    | https://github.com/badnewsgoonies-dot/briefcase-mcp-public-status                                | on change*    |
| Pages site     | https://badnewsgoonies-dot.github.io/briefcase-mcp-public-status/                                | on change*    |

\* Repo README commits (and therefore Pages rebuilds) are skipped when the
generation is unchanged AND the last commit is < 30 min old. This keeps us
under Pages' 10-builds/hour limit.

## Search phrases

- Primary: `Briefcase MCP Public Status API` (not globally unique; competes with MCP docs)
- Sentinel: `gctools-briefcase-live-chain-generation-mirror-b2e4a1f7` (globally unique)

## Files on the briefcase host

- Updater source: `https://gist.githubusercontent.com/badnewsgoonies-dot/2234b9d35c8f61f75f197d670a86120c/raw/gist_updater.py`
- Probe source:   `https://gist.githubusercontent.com/badnewsgoonies-dot/2234b9d35c8f61f75f197d670a86120c/raw/freshness_probe.py`
- Env file:       `/etc/briefcase-gist-updater.env` (contains `GITHUB_TOKEN=...`)
- Log:            `/var/log/briefcase-gist-updater.log` under systemd, or `/tmp/briefcase-gist-updater-*.log` under briefcase_process

## Installing as a systemd service

1. Copy `briefcase-gist-updater.service` from the repo's `docs/` folder to `/etc/systemd/system/`.
2. Create `/etc/briefcase-gist-updater.env` with `GITHUB_TOKEN=<classic PAT with gist+repo scope>` (chmod 600).
3. `sudo systemctl daemon-reload && sudo systemctl enable --now briefcase-gist-updater.service`
4. Verify: `sudo journalctl -u briefcase-gist-updater.service -n 20 -f`

Updater pulls the v2 script fresh from the gist on every start, so rolling out a new updater version is:
1. Edit `gist_updater.py` in the primary gist (via web UI or API).
2. `sudo systemctl restart briefcase-gist-updater.service`

## Monitoring

Run `freshness_probe.py` from any machine (not on the briefcase host) every 10 min. Exits 0 if all mirrors are fresh and consistent; exits 1 otherwise. Add to cron:

```
*/10 * * * * /usr/bin/python3 /usr/local/bin/freshness_probe.py >> /var/log/briefcase-freshness.log 2>&1
```

Or fetch it directly in cron:

```
*/10 * * * * curl -sSf https://gist.githubusercontent.com/badnewsgoonies-dot/2234b9d35c8f61f75f197d670a86120c/raw/freshness_probe.py | python3
```

## Failure playbook

| Symptom                                         | Likely cause                       | Fix                                                    |
|-------------------------------------------------|------------------------------------|--------------------------------------------------------|
| Probe FAIL: one mirror stale, others fresh      | GitHub CDN cache of raw URL        | Usually clears within 10 min; if not, PATCH again      |
| Probe FAIL: all mirrors stale                   | Updater died or host lost network  | `systemctl restart briefcase-gist-updater`             |
| Probe FAIL: gens differ by >1                   | Updater in weird partial state     | Restart updater; it will converge on next cycle        |
| Updater log: `primary_gist FAIL http=401`       | Token revoked/expired              | Update token in `/etc/briefcase-gist-updater.env`      |
| Updater log: `fetch failed: HTTPError 403`      | Status endpoint behind Cloudflare  | Updater must use `127.0.0.1:8765`, not public domain   |
| Pages site stops updating but repo commits fine | Hit 10 builds/hr limit             | v2.1 already rate-limits; should not happen            |

## Rotating the GitHub token

1. Create a new classic PAT with `gist` + `repo` scope (or equivalent fine-grained).
2. Edit `/etc/briefcase-gist-updater.env`.
3. `sudo systemctl restart briefcase-gist-updater.service`.
4. Verify: `sudo journalctl -u briefcase-gist-updater.service -n 5` — should show `ok gen=...` within 30 s.
5. Revoke old token at https://github.com/settings/tokens.
