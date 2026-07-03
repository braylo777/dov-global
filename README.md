# dov-global

Public marketing site for **DOV Global PBC** — a Wyoming Public Benefit Corporation.
One man, every country on Earth, 25 years — funded by revenue, not donations.

The entire site is a single self-contained `site/index.html` (≈350 KB): all CSS, JS,
the logo, the favicon, and the world map are inlined as data URIs. **Zero external
requests** — no CDNs, no fonts, no trackers. A strict Content-Security-Policy is set
in-page and re-asserted at the edge via `site/_headers`.

## Deploy (Cloudflare Workers Assets)

```bash
npx wrangler deploy      # publishes ./site → https://dov-global.<subdomain>.workers.dev
```

First time on a new machine: `npx wrangler login` (one browser approval), then deploy.
`wrangler.jsonc` names the project `dov-global` and serves `./site`. The live `dov.global`
custom domain is **not** attached yet — see `DEPLOY.md` for the domain + DNS steps.

## Editing content

All mutable facts live in one `CONFIG` object at the top of the `<script>` in
`site/index.html`:

- `email` — contact address
- `departure` — hero countdown target (ISO 8601)
- `youAreHere` — `{label, lat, lon}` marker on the map
- `countriesClaimed` / `countriesTotal` — the scarcity ledger counts
- `claimed` — array of `[lat, lon]` for claimed Forward Operating Bases

Edit, save, `npx wrangler deploy`. Rollbacks are one deployment in the Cloudflare dashboard.

## CI

`.github/workflows/build-check` verifies `site/index.html` + `site/_headers` exist and
that the page has no external `http(s)` asset references (stays self-contained).

## Open content TODOs

- Real field-story paragraph (Zambia / Kathmandu) replacing the placeholder.
- Field photos + founder portrait.
- Live claim counts (update `CONFIG.countriesClaimed` / `claimed[]`).
