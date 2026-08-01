# contact.thatuglyboy.com

Single static page listing contact channels. Cloudflare Worker
(`thatuglyboy-contact`) serving this folder as assets. No build step.

## Deploying

**Pushing does NOT deploy.** Unlike `../main-site`, this repo is not connected
to Workers Builds — the dashboard shows "Manually deployed", not a commit
message. A push updates GitHub and nothing else. Deploy by hand:

```bash
npx wrangler deploy
```

The Worker is `websitecontact`; `wrangler.jsonc` must say exactly that, or a
deploy silently creates a second, unrouted Worker and the live site never
changes.

Remote: `github.com/RagininW/thatuglyboy_website_contact`

The Worker is named `websitecontact` in Cloudflare (not `thatuglyboy-contact`
as `wrangler.jsonc` says — the dashboard name is what's live).

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
is scoped per *origin*, and every hostname is its own origin. An earlier
version of this file claimed the shared `tub:lang` key made the choice follow
visitors across hostnames — it never did. Language crosses origins **in the
URL** only:

- main site → here: it links to `contact.` or `contacto.` per current language
- here → main site: the back link carries `?lang=en` / `?lang=es`, which
  `app.js` consumes, stores, and strips via `replaceState`

`tub:lang` is still written, but it only helps repeat visits to the *same*
hostname. It's the fallback where neither hostname applies (workers.dev,
localhost), which is also the only place the EN/ES buttons swap text in place —
on the real hostnames they navigate, so the URL stays truthful.

Strings live in the `I18N` object at the bottom of `index.html`; keep both
languages in sync. Adding a language means adding a hostname, not just a key.

Copy register matches the main site: lowercase, declarative, no second person.

## Related

- `../main-site` — separate repo and Worker for `thatuglyboy.com`. Design tokens
  (colors, fonts) are duplicated here; if they change there, update them here too.
