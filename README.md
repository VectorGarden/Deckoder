# Deckoder

Converts Yu-Gi-Oh! decklists between **YDK**, **YDKE** and the **tournament registration JSON** format — in both directions, entirely in the browser.

The awkward part isn't the file formats, it's the identifiers. A `.ydk` file lists 8-digit **passcodes** (the number printed on the card). The registration JSON uses `CardDatabaseId`, which is Konami's internal **card ID** (the `cid` on the official database). They are different numbering systems, so converting between them needs a lookup table. Deckoder builds one on the fly from the YGOPRODeck API and caches it locally.

## Use it

Drop a `.ydk`, paste a `ydke://` URI, or drop a registration `.json`. Deckoder detects the direction and gives you all three formats back, with copy and download buttons for each.

Section 3 lists every card with its resolved name — check a few before you register. Anything that can't be matched is flagged in red with a box to type the correct ID into; Deckoder remembers what you enter.

## Offline use

Every card Deckoder resolves is stored in a local lookup table. Under **Lookup table & offline use** you can export that table to a file and reimport it elsewhere.

Do one conversion at home and hit **Export lookup table** before an event. The page then works with no network at all — worth doing before a tournament laptop meets venue wifi.

## Hosting

Static, no build step. `index.html` runs from `file://` by double-clicking, or from any web server.

Deployed at **https://deckoder.reizu.dev**.

Setup, once per project:

1. **Verify `reizu.dev` on your GitHub account** — profile → Settings → Pages → Add a domain. Add the `_github-pages-challenge-<user>` TXT record it gives you, then Verify. Do this before pointing any DNS, so nobody else can claim a `reizu.dev` subdomain on Pages.
2. **DNS** — one CNAME record: name `deckoder`, target `<user>.github.io`. No A records; those are only for apex domains. If Cloudflare, keep it **DNS Only** (grey cloud) until the certificate issues.
3. **Repo** — Settings → Pages → Source: Deploy from a branch, `main`, `/ (root)`, then set Custom domain to `deckoder.reizu.dev`.
4. **Enforce HTTPS** — tick it once it stops being greyed out. Not optional: `.dev` is HSTS-preloaded, so the site is unreachable until the certificate is live.

The `CNAME` file in this folder holds the domain, so it survives redeploys. `.nojekyll` stops Pages running the files through Jekyll.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire app — HTML, CSS and JS in one file |
| `icon.svg` | Source icon |
| `favicon.ico` | 16/32/48 bundle for older browsers |
| `icon-180.png` | `apple-touch-icon`, iOS home screen |
| `icon-192.png`, `icon-512.png` | Web app manifest icons |
| `site.webmanifest` | Makes it installable to a phone home screen |
| `.nojekyll` | Stops Pages running the files through Jekyll |
| `CNAME` | Custom domain for GitHub Pages (`deckoder.reizu.dev`) |

The favicon is also embedded in `index.html` as a data URI, so the single file keeps its icon even if you move it somewhere without the PNGs.

## Notes

- Card data comes from the [YGOPRODeck API](https://ygoprodeck.com/api-guide/). Passcode lookups are batched into one request per deck; Konami-ID lookups (JSON → YDK) are one request per card, with a full-database fallback if that endpoint isn't available.
- Alternate-art passcodes resolve to the same card.
- Main deck is split into `Monsters` / `Spells` / `Traps` using the card type from the API, since a `.ydk` doesn't record it. `Extra` and `Side` stay flat, matching the registration format.
- Deck size is checked as you go (main 40–60, extra ≤15, side ≤15).
- Nothing is uploaded anywhere. The only network traffic is card lookups.
