# DEPLOY.md — putting dov.global live (Cloudflare Pages)

Status: **prepared, not executed.** The site is one self-contained `index.html` (zero external requests),
so any static host works; these steps assume the domain is on Cloudflare (per the 07-finc invoice).

## 1. One-time setup
1. Cloudflare dashboard → **Workers & Pages → Create → Pages → Upload assets**.
2. Project name: `dov-global`. Drag in a folder containing:
   - `index.html` (the single-page site)
   - `_headers` (below)
   - optional later: `assets/logo.png`, `og.png` (1200×630 share image), `favicon.ico`
3. Deploy → you get `dov-global.pages.dev`. Verify it renders.

CLI alternative: `npx wrangler pages deploy ./site --project-name=dov-global`

## 2. Custom domain
1. Pages project → **Custom domains → Add** → `dov.global` (and `www.dov.global`).
2. Cloudflare auto-creates the CNAME records (domain is already on CF DNS). Both should go **Active** in minutes.
3. SSL/TLS → **Full (strict)**; edge certificates are automatic.

## 3. `_headers` file (real security headers — replaces the in-page CSP meta)
```
/*
  Content-Security-Policy: default-src 'none'; img-src 'self' data:; style-src 'unsafe-inline'; script-src 'unsafe-inline'; base-uri 'none'; form-action mailto:; frame-ancestors 'none'
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: no-referrer
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
  Cross-Origin-Opener-Policy: same-origin
```
(When scripts are later externalized, tighten `script-src` to hashes/nonces and drop `unsafe-inline`.)

## 4. DNS / domain hygiene (Cloudflare dashboard)
- **DNSSEC**: DNS → Settings → enable.
- **CAA record**: `dov.global CAA 0 issue "letsencrypt.org"` + `"pki.goog"` (Cloudflare uses both).
- **Email auth** for @dov.global (whatever mail provider is in use):
  - SPF `TXT "v=spf1 include:<provider> -all"`
  - DKIM per provider
  - DMARC `TXT _dmarc "v=DMARC1; p=quarantine; rua=mailto:braylo@dov.global"`
- Redirect rule: `www.dov.global/*` → `https://dov.global/$1` (301).

## 5. Post-deploy checklist
- [ ] https://dov.global loads; padlock valid; `pages.dev` URL also fine.
- [ ] securityheaders.com scan = A (after `_headers`).
- [ ] Lighthouse: perf ≥ 90 (single file, no requests — should be trivial), a11y ≥ 95.
- [ ] OG preview (paste URL in Slack/X) — add hosted `og.png` + `<meta property="og:image">` for a real card.
- [ ] Enable **Cloudflare Web Analytics** (cookieless, free) if wanted — paste snippet OR use the
      no-JS server-side analytics; note: adding the JS snippet requires loosening CSP `script-src`/`connect-src`.
- [ ] robots: single page, nothing to block; optionally add `/robots.txt` with `Allow: /` + sitemap line.

## 6. Updating the site
Edit CONFIG at the top of `index.html` (claim counts, claimed[] coords, you-are-here label/position,
departure date, email) → re-upload via Pages "Create new deployment". Rollbacks are one click (previous
deployments are kept).

## 7. Later (when commerce lands)
- Stripe Payment Links can replace the mailto CTAs with zero backend.
- Live claim ledger: a tiny Cloudflare Worker + KV storing `claimed[]`, fetched by the page
  (requires CSP `connect-src 'self'`).
- Turnstile on any real form.
