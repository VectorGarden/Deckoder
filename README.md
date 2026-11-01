# Deckoder

Converts Yu-Gi-Oh! decklists between **YDK**, **YDKE** and the **tournament registration JSON** format — in both directions, entirely in the browser.

The awkward part isn't the file formats, it's the identifiers. A `.ydk` file lists 8-digit **passcodes** (the number printed on the card). The registration JSON uses `CardDatabaseId`, which is Konami's internal **card ID** (the `cid` on the official database). They are different numbering systems, so converting between them needs a lookup table. Deckoder builds one on the fly from the YGOPRODeck API and caches it locally.

## Use it

Drop a `.ydk`, paste a `ydke://` URI, or drop a registration `.json`. Deckoder detects the direction and gives you all three formats back, with copy and download buttons for each.

Section 3 shows the deck as card art, grouped into main, extra and side. Hover a card (or tap it on a phone) for its text and stats; click to keep the details open.

Switch section 3 to **Check list** to see every card with its resolved name — check a few before you register. Anything that can't be matched is flagged in red with a box to type the correct ID into; Deckoder remembers what you enter. If anything is unmatched, the check list opens automatically.

## Offline use

Every card Deckoder resolves is stored in a local lookup table. Under **Lookup table & offline use** you can export that table to a file and reimport it elsewhere.

Do one conversion at home and hit **Export lookup table** before an event. The page then works with no network at all — worth doing before a tournament laptop meets venue wifi.

Card text and art are not in the exported table. Both are kept by the browser that fetched them, so open the deck view once on the event laptop itself while it's online. Card text is stored in IndexedDB until you clear the table. Art lives in the browser's HTTP cache for about a month. Offline, any card whose art isn't cached shows as a coloured tile with its name.

## Hosting

Static, no build step. `index.html` runs from `file://` by double-clicking, or from any web server.

Deployed at **https://deckoder.reizu.dev**. Every push to `main` deploys through `.github/workflows/pages.yml`, which publishes the repo root as-is. It can also be run by hand from the Actions tab.

Setup, once per project:

1. **Verify `reizu.dev` on your GitHub account** — profile → Settings → Pages → Add a domain. Add the `_github-pages-challenge-<user>` TXT record it gives you, then Verify. Do this before pointing any DNS, so nobody else can claim a `reizu.dev` subdomain on Pages.
2. **DNS** — one CNAME record: name `deckoder`, target `<user>.github.io`. No A records; those are only for apex domains. If Cloudflare, keep it **DNS Only** (grey cloud) until the certificate issues.
3. **Repo** — Settings → Pages → Source: **GitHub Actions**, then set Custom domain to `deckoder.reizu.dev`.
4. **Enforce HTTPS** — tick it once it stops being greyed out. Not optional: `.dev` is HSTS-preloaded, so the site is unreachable until the certificate is live.

With an Actions deploy, the custom domain lives in the repo's Pages settings, and Pages ignores the `CNAME` file. It's kept so the domain is recorded with the code. Actions deploys don't run Jekyll, so no `.nojekyll` is needed.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire app — HTML, CSS and JS in one file |
| `icon.svg` | Source icon |
| `favicon.ico` | 16/32/48 bundle for older browsers |
| `icon-180.png` | `apple-touch-icon`, iOS home screen |
| `icon-192.png`, `icon-512.png` | Web app manifest icons |
| `site.webmanifest` | Makes it installable to a phone home screen |
| `.github/workflows/pages.yml` | Deploys to GitHub Pages on every push to `main` |
| `CNAME` | Records the custom domain (`deckoder.reizu.dev`); the live setting is in the repo's Pages settings |

The favicon is also embedded in `index.html` as a data URI, so the single file keeps its icon even if you move it somewhere without the PNGs.

## Notes

- Card data comes from the [YGOPRODeck API](https://ygoprodeck.com/api-guide/). Passcode lookups are batched into one request per deck; Konami-ID lookups (JSON → YDK) are one request per card, with a full-database fallback if that endpoint isn't available.
- Alternate-art passcodes resolve to the same card, and the deck view shows that passcode's own art.
- Card art is loaded from `images.ygoprodeck.com`. YGOPRODeck asks that each image be fetched once and stored, and blocks IPs that fetch fast. Their images are cacheable for 31 days, so the browser cache does the storing, and first-time fetches are queued at ten a second. Their image server sends no CORS headers, so the page can't copy the images into its own storage.
- Main deck is split into `Monsters` / `Spells` / `Traps` using the card type from the API, since a `.ydk` doesn't record it. `Extra` and `Side` stay flat, matching the registration format.
- Deck size is checked as you go (main 40–60, extra ≤15, side ≤15).
- Nothing is uploaded anywhere. The only network traffic is card lookups.
