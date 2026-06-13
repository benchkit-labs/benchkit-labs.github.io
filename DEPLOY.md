# BenchKit Labs — site architecture & deployment

How the org homepage and each tool's pages map to domains. One repo per tool,
each served as its own static GitHub Pages site under its own subdomain. DNS is
on Cloudflare.

## Layout

Repo names match the subdomains (`gapps` repo → `gapps.*`; `snapp` repo →
`snapp.*`). Page files inside `gapps` are unprefixed (`launcher.html`,
`builder.html`) since the subdomain already carries the product name.

| Repo | Serves | Domain |
|---|---|---|
| `benchkit-labs.github.io` | org homepage | `benchkit-labs.dev` (apex) + `www` |
| `gapps` | home + launcher + builder | `gapps.benchkit-labs.dev` |
| `snapp` | landing page (extension ships via stores) | `snapp.benchkit-labs.dev` |
| `eoscard-400plus-prep` | (CLI tool — repo + README only for now) | — |

> CDN pins frozen at the `v1.0.0` tag still reference the old paths
> (`gh/benchkit-labs/gapps-launcher.html`) because the tag predates the rename
> — that's correct, leave them. GitHub redirects the old `gapps-embed` /
> `snapp-ext` repo URLs to the new names indefinitely.

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
   - `gapps-embed`             → `gapps.benchkit-labs.dev`
3. **Settings → Pages → Enforce HTTPS:** on (after the cert provisions, ~minutes).

> ⚠️ Once a custom domain is set, the site serves at the **root** of that domain,
> not under the old `/<repo>/` path. Keep all internal links **relative**
> (`gapps-launcher.html`, not `/gapps-embed/gapps-launcher.html`) so they survive
> the move. Absolute embed URLs in docs/snippets use the subdomain root, e.g.
> `https://gapps.benchkit-labs.dev/gapps-launcher.html`.

## Cloudflare DNS

We run **Cloudflare-proxied (orange cloud)** in front of GitHub Pages. Cloudflare
terminates TLS at its edge with its own cert; Pages is the origin. This is the
setup that's live — *not* the DNS-only / GitHub-cert path.

In the `benchkit-labs.dev` zone:

| Type | Name | Content | Proxy |
|---|---|---|---|
| A | `@` | `185.199.108.153` | Proxied (orange) |
| A | `@` | `185.199.109.153` | Proxied |
| A | `@` | `185.199.110.153` | Proxied |
| A | `@` | `185.199.111.153` | Proxied |
| CNAME | `www` | `benchkit-labs.github.io` | Proxied |
| CNAME | `gapps` | `benchkit-labs.github.io` | Proxied |
| CNAME | `snapp` | `benchkit-labs.github.io` | Proxied |

(Cloudflare may show the proxied apex/subdomain resolving to its own IPs —
`104.21.x` / `172.67.x` — rather than the `185.199.x` Pages IPs. That's expected
with the proxy on.)

Every new tool subdomain = **one CNAME record** → `benchkit-labs.github.io`,
orange-cloud (plus that repo's `CNAME` file + Pages enabled).

> **SSL/TLS mode must be Full or Full (strict), never Flexible.** With the proxy
> on, Cloudflare → origin encryption is set under **SSL/TLS → Overview**. Pages
> presents a valid cert, so **Full (strict)** is correct. *Flexible* makes
> Cloudflare talk to Pages over plain HTTP and is the usual cause of redirect
> loops — avoid it.
>
> Because Cloudflare owns the cert, leave GitHub's **Settings → Pages → Enforce
> HTTPS** as-is; the public TLS the visitor sees is Cloudflare's edge cert, not
> the Pages Let's Encrypt one.

## Adding a new tool with a web page

1. Create the repo under `benchkit-labs`, build its page(s) with relative links,
   copy the inline CSS theme from an existing page.
2. Add a `CNAME` file: `<tool>.benchkit-labs.dev`.
3. Enable Pages (branch `main`, root).
4. Cloudflare: add CNAME `<tool>` → `benchkit-labs.github.io`, proxied (orange).
5. Add a project card to the org homepage (`index.html`) linking
   `https://<tool>.benchkit-labs.dev`.
