# contact.thatuglyboy.com

Single static page listing contact channels. Cloudflare Worker
(`thatuglyboy-contact`) serving this folder as assets. No build step.

## Deploying

**Pushing to `main` auto-deploys.** Cloudflare builds from the GitHub repo on
push — no manual deploy step, so treat a push as shipping to production.
`wrangler.jsonc` must stay at the repo root.

Setup still outstanding: this repo has no GitHub remote yet, and the subdomain
is not bound to the Worker (the DNS record exists, but it needs a route or
custom domain pointing at `thatuglyboy-contact`).

## Layout

Everything is in `index.html` — markup, inline CSS, and the language script.
`fonts/` and `favicon.png` are **copies** of the main site's; they aren't
referenced across hostnames because that would need CORS headers for the fonts.

Only `ugly_boy` and `avenir` are declared here. If a heading ever needs the
`angst` face, copy it over from `../main-site/fonts/` — it isn't in this repo.

## Language

Reads and writes the same `tub:lang` localStorage key as the main site, so an
EN/ES choice follows visitors across the two hostnames. Strings live in the
`I18N` object at the bottom of `index.html`; keep both languages in sync.

Copy register matches the main site: lowercase, declarative, no second person.

## Related

- `../main-site` — separate repo and Worker for `thatuglyboy.com`. Design tokens
  (colors, fonts) are duplicated here; if they change there, update them here too.
