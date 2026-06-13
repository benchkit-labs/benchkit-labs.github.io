# Benchkit Labs — site architecture & deployment

How the org homepage and each tool's pages map to domains. One repo per tool,
each served as its own static GitHub Pages site under its own subdomain. DNS is
on Cloudflare.

## Layout

| Repo | Serves | Domain |
|---|---|---|
| `benchkit-labs.github.io` | org homepage | `benchkit-labs.dev` (apex) + `www` |
| `gapps-embed` | home + launcher + builder | `gapps-embed.benchkit-labs.dev` |
| `eoscard-400plus-prep` | (CLI tool — repo + README only for now) | — |

Each repo owns its own landing page(s) next to its code, README, and issues.
There is **no** central site repo aggregating the tool pages — the only shared
thing is the visual style (the navy/teal CSS), which is copied inline into each
page rather than hosted centrally. When the brand changes (rare), update each
page by hand.

A tool gets a subdomain when it has a web page worth serving. CLI-only tools
(like `eoscard-400plus-prep`) live as repos and are linked from the org homepage;
they don't need a subdomain until/unless they grow a hosted page.

## Per-repo GitHub Pages setup

For each repo that serves a site:

1. **Settings → Pages → Source:** Deploy from a branch → `main` / `/ (root)`.
2. Add a **`CNAME`** file at the repo root containing the exact hostname:
   - `benchkit-labs.github.io` → `benchkit-labs.dev`
   - `gapps-embed`             → `gapps-embed.benchkit-labs.dev`
3. **Settings → Pages → Enforce HTTPS:** on (after the cert provisions, ~minutes).

> ⚠️ Once a custom domain is set, the site serves at the **root** of that domain,
> not under the old `/<repo>/` path. Keep all internal links **relative**
> (`gapps-launcher.html`, not `/gapps-embed/gapps-launcher.html`) so they survive
> the move. Absolute embed URLs in docs/snippets use the subdomain root, e.g.
> `https://gapps-embed.benchkit-labs.dev/gapps-launcher.html`.

## Cloudflare DNS

In the `benchkit-labs.dev` zone:

| Type | Name | Content | Proxy |
|---|---|---|---|
| A | `@` | `185.199.108.153` | DNS only (grey cloud) |
| A | `@` | `185.199.109.153` | DNS only |
| A | `@` | `185.199.110.153` | DNS only |
| A | `@` | `185.199.111.153` | DNS only |
| CNAME | `www` | `benchkit-labs.github.io` | DNS only |
| CNAME | `gapps-embed` | `benchkit-labs.github.io` | DNS only |

Every new tool subdomain = **one CNAME record** → `benchkit-labs.github.io`
(plus that repo's `CNAME` file + Pages enabled).

> **Proxy must be "DNS only" (grey cloud), not proxied (orange).** GitHub Pages
> provisions its own Let's Encrypt cert for the custom domain; Cloudflare's proxy
> in front of Pages causes cert-validation and redirect-loop issues unless you
> configure Cloudflare's own TLS to match. Grey-cloud is the simplest correct
> setup. (If you later want Cloudflare's CDN/WAF in front, switch to Full-strict
> TLS and confirm the Pages cert still validates first.)

## Adding a new tool with a web page

1. Create the repo under `benchkit-labs`, build its page(s) with relative links,
   copy the inline CSS theme from an existing page.
2. Add a `CNAME` file: `<tool>.benchkit-labs.dev`.
3. Enable Pages (branch `main`, root).
4. Cloudflare: add CNAME `<tool>` → `benchkit-labs.github.io`, DNS only.
5. Add a project card to the org homepage (`index.html`) linking
   `https://<tool>.benchkit-labs.dev`.
