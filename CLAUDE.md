# contact.thatuglyboy.com

Single static page listing contact channels. Cloudflare Worker
(`websitecontact`) serving this folder as assets. No build step.

## Deploying

**Pushing to `main` auto-deploys.** Connected to Workers Builds, which runs
`npx wrangler deploy` in the cloud on push — a build lands in well under a
minute. Treat a push as shipping to production.

Two things this depends on, both easy to break:

- `wrangler.jsonc` must say `name: "websitecontact"`. Any other name deploys to
  a phantom Worker with no custom domains: every build goes green and the live
  site never changes.
- `wrangler.jsonc` must stay at the repo root.

Deploying by hand is **not** an option on the current machine — local Node is
v14, and wrangler needs 18+ (it dies with `Unexpected token '{'` before
printing anything useful). Cloud builds sidestep that entirely; to restore the
local path, upgrade Node.

Remote: `github.com/RagininW/thatuglyboy_website_contact`

### Gotcha: routes on the main-site Worker shadow this one

A Worker on a **route** runs before a Worker on a **custom domain**, and a
static-assets Worker never falls through — it just answers. So a route on the
`thatuglyboy` Worker that matches this subdomain silently serves the main site
here instead, even though DNS points the hostname at this Worker.

This already happened once: `contact.thatuglyboy.com` served the main site
until the offending route was removed from `thatuglyboy`. If this page ever
starts looking like the main site again, check that Worker's routes first, then
purge cache. Diagnostic — `app.js` only exists on the main site:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://contact.thatuglyboy.com/app.js
```

`404` is correct. `200` means the main-site Worker is answering.

## Layout

Everything is in `index.html` — markup, inline CSS, and the language script.
`fonts/` and `favicon.png` are **copies** of the main site's; they aren't
referenced across hostnames because that would need CORS headers for the fonts.

Only `ugly_boy` and `avenir` are declared here. If a heading ever needs the
`angst` face, copy it over from `../main-site/fonts/` — it isn't in this repo.

## Language: the hostname is the language

This one page is served on two hostnames — `contact.` is English, `contacto.`
is Spanish — and `langFromHost()` reads the language off `location.hostname`.
Test `contacto.` **before** `contact.`, and anchor both at index 0, or
`websitecontact.…workers.dev` matches too.

`localStorage` cannot carry the choice between the main site and this one: it
is scoped per *origin*, and every hostname is its own origin. A **cookie** can,
because it is scoped per *domain* — `persistLang()` writes `tub_lang` against
`.thatuglyboy.com`, which every host under it receives.

So arriving on `contacto.` now sets Spanish for `thatuglyboy.com` and
`ochre.thatuglyboy.com` as well. The hostname still decides the language *here*
(the URL has to stay truthful), and it is written to the cookie on arrival.

That cookie code is `../main-site/lang-store.js` and `../ochre/lang-store.js`
inlined, since this site is one file. **Change one, change all three.**

The URL hand-off remains underneath, for blocked cookies:

- main site → here: it links to `contact.` or `contacto.` per current language
- here → main site: the back link carries `?lang=en` / `?lang=es`, which
  `app.js` consumes, stores, and strips via `replaceState`

`tub:lang` in localStorage is still written, as the fallback where the cookie
domain does not apply (workers.dev, localhost) — which is also the only place
the EN/ES buttons swap text in place; on the real hostnames they navigate.

Strings live in the `I18N` object at the bottom of `index.html`; keep both
languages in sync. Adding a language means adding a hostname, not just a key.

Copy register matches the main site: lowercase, declarative, no second person.

## Related

- `../main-site` — separate repo and Worker for `thatuglyboy.com`. Design tokens
  (colors, fonts) are duplicated here; if they change there, update them here too.
- `../ochre` — separate repo and Worker for `ochre.thatuglyboy.com`. It runs its
  own palette (the game's), so it is *not* a place to propagate token changes —
  but it inherits the route-shadowing gotcha above.
