# link-page-kit

**A list of links should not cost $288 a year.**

Linktree charges up to $288/yr for what is, fundamentally, a
list of links with a pretty header — and takes 9% seller fees
on top, and offers a custom domain on **no** plan, not even the
top one. The cheaper hosted rivals keep dying underneath their
users: Shopify discontinued Linkpop in July 2025, and Linktree
acquired and folded both Koji and Bento. Close the account, or
watch the platform fold, and every link you ever shared is dead.

`link-page-kit` generates the page instead. One config JSON in,
**one self-contained HTML file** out — all CSS inline, no
JavaScript, no trackers, no external requests except the links
themselves and your avatar. Host it on GitHub Pages, your own
site, any static host, forever, for free. It cannot be shut
down, folded, or repriced, because there is no platform — it's
a file, and it's yours.

- Pure Python 3 standard library — no dependencies, no install step
- Local only — the tool never fetches the network, not even to
  "check" your links; validation is structural, and says so
- Script-friendly exit codes: `0` = clean, `1` = built with
  warnings (duplicate URLs), `2` = error, nothing written

## The config

```json
{
  "name": "Maya Chen",
  "bio": "Trail rider and host of The Kelowna Trailcast.",
  "avatar": "https://example.org/maya/avatar.jpg",
  "theme": {"accent": "#2e7d4f", "mode": "light"},
  "links": [
    {"label": "Watch on YouTube",
     "url": "https://youtube.com/@KelownaTrailcast",
     "note": "New video every Friday"},
    {"label": "Email me", "url": "mailto:maya@example.org"}
  ],
  "socials": [
    {"label": "TikTok", "url": "https://tiktok.com/@kelownatrailcast"}
  ]
}
```

| Field | Meaning |
|---|---|
| `name` | your name (required, non-empty) |
| `bio` | one or two lines under the name (optional) |
| `avatar` | an http(s) URL or a local path (optional) — **referenced** in an `<img>`, never embedded, so the page stays small and the image can live beside it |
| `theme.accent` | hex colour, `#rgb` or `#rrggbb` (default `#2563eb`) |
| `theme.mode` | `"light"` (default) or `"dark"` |
| `links` | at least one `{"label", "url", "note"?}` — label and url required |
| `socials` | same shape, optional — rendered as a pill row under the links |

The config is validated on every run, and every error names
the field: `links[2].url`, `theme.accent`. A malformed config
is a clean exit-2 error, never a traceback.

## Validation is the product

Every URL is checked before the page exists:

- **Scheme allowlist** — `https`, `http`, `mailto`, `tel` only.
  `javascript:`, `data:`, `file:`, scheme-relative `//host` and
  scheme-less values are **refused**: a link page that can carry
  a `javascript:` href is a phishing kit, not a link page.
- **Shape** — http(s) URLs need a host and no whitespace;
  `mailto:` needs an address; `tel:` needs digits.
- **Duplicates are flagged** — after normalising scheme/host
  case and a trailing slash, two entries pointing at the same
  target produce a warning. The page still builds (exit 1);
  you decide whether the repetition is deliberate.
- **Labels are required** — a link with no label is a button
  nobody can read.

`--check-links` lists the structural check for every URL (the
scheme and the host/address/number it found). It is **offline
by design**: the tool never fetches the network, so it
validates *shape*, never *liveness*. A well-shaped link can
still be dead; a liveness claim from a tool that makes no
requests would be a lie, so it doesn't make one.

## The page itself

- **One file, everything inline.** No external stylesheets,
  fonts or scripts — a system font stack, so there is nothing
  to wait for. The sample page below weighs **3,122 bytes**.
- **Zero JavaScript.** There is no `<script>` in the output,
  ever — nothing to phone home, nothing to break.
- **Escaped, always.** Every name, label, note and URL is
  HTML-escaped in text and in attributes. A label containing
  `<script>alert(1)</script>` renders as visible text.
- **Accessible by construction.** Semantic HTML (`header`,
  `main`, labelled `nav` landmarks, a real list), alt text on
  the avatar, visible `:focus-visible` outlines, mobile-first
  layout.

## Usage

```bash
# 1. Build the page (default output: index.html)
python3 src/link_page_kit.py build examples/sample-config.json \
    -o maya-links.html

# 2. Preview first: links in order, page weight, checks passed
python3 src/link_page_kit.py preview examples/sample-config.json

# 3. See the structural check behind every URL
python3 src/link_page_kit.py preview examples/sample-config.json \
    --check-links

# 4. Drive it from a script: exit 1 means duplicate warnings
python3 src/link_page_kit.py build page.json || echo "check warnings"
```

Then put the HTML file anywhere static: GitHub Pages, your
existing website, Netlify/Vercel drag-and-drop, an S3 bucket.
Point your bio at your own domain once, and you never have to
update a shared link again.

## Use with AI agents

Every command takes `--json`: one JSON object with the same
facts and exit codes as the text view (errors come back as
`{"error": ...}`, argparse usage errors included):

```bash
python3 src/link_page_kit.py build examples/sample-config.json \
    -o maya-links.html --json
```

Agents: read [SKILL.md](SKILL.md) first — commands, the full
JSON schemas, the exit codes, and the honesty rules (checks
are structural, never liveness; a duplicate is a warning, not
an error; the page makes no request the config didn't ask
for).

## Real run (verbatim)

The bundled sample is a creator page: 6 links (YouTube,
Instagram, a shop product, the podcast, a trail map, one
mailto) and 2 socials. The preview:

```text
$ python3 src/link_page_kit.py preview examples/sample-config.json
link-page-kit v0.1.0 — preview · config sample-config.json
Maya Chen · 6 link(s), 2 social(s) · page weight 3,122 bytes

Links, in order:
  1. Watch on YouTube — https://youtube.com/@KelownaTrailcast — New video every Friday
  2. Follow on Instagram — https://instagram.com/kelownatrailcast
  3. Trailcast stickers — my shop — https://example.org/shop/stickers — Ships within Canada
  4. Listen to the podcast — https://example.org/trailcast
  5. Trail conditions map — https://example.org/trail-conditions
  6. Email me — mailto:maya@example.org — Sponsorships and ride invites

Socials:
  1. TikTok — https://tiktok.com/@kelownatrailcast
  2. X — https://x.com/kelownatrailcast

Checks passed:
  ✓ name present
  ✓ 6 link(s), every label present
  ✓ every URL uses an allowed scheme (https, http, mailto, tel)
  ✓ single file, no external requests besides the configured links, socials and avatar
  ✓ avatar is referenced, not embedded
  ✓ accent colour is a valid hex value
  ✓ no duplicate URLs
Warnings: none
(exit code: 0)
```

The structural detail behind every URL (`--check-links`):

```text
Link checks (structural, offline — shape, not liveness):
  links[0] 'Watch on YouTube' — https://youtube.com/@KelownaTrailcast · scheme https · host youtube.com
  links[1] 'Follow on Instagram' — https://instagram.com/kelownatrailcast · scheme https · host instagram.com
  links[2] 'Trailcast stickers — my shop' — https://example.org/shop/stickers · scheme https · host example.org
  links[3] 'Listen to the podcast' — https://example.org/trailcast · scheme https · host example.org
  links[4] 'Trail conditions map' — https://example.org/trail-conditions · scheme https · host example.org
  links[5] 'Email me' — mailto:maya@example.org · scheme mailto · address maya@example.org
  socials[0] 'TikTok' — https://tiktok.com/@kelownatrailcast · scheme https · host tiktok.com
  socials[1] 'X' — https://x.com/kelownatrailcast · scheme https · host x.com
```

The build writes exactly the page the preview weighed:

```text
$ python3 src/link_page_kit.py build examples/sample-config.json -o maya-links.html
link-page-kit v0.1.0 — build · config sample-config.json
Wrote maya-links.html — 3,122 bytes · 6 link(s), 2 social(s)
Checks passed:
  ✓ name present
  ✓ 6 link(s), every label present
  ✓ every URL uses an allowed scheme (https, http, mailto, tel)
  ✓ single file, no external requests besides the configured links, socials and avatar
  ✓ avatar is referenced, not embedded
  ✓ accent colour is a valid hex value
  ✓ no duplicate URLs
Warnings: none
(exit code: 0)
```

Scanning the generated page for external references returns
exactly the eight URLs the config asked for — the avatar, six
links, two socials — and nothing else (the mailto is not an
http reference):

```text
$ grep -oE 'https?://[^" <>]+' maya-links.html | sort -u
https://example.org/maya/avatar.jpg
https://example.org/shop/stickers
https://example.org/trail-conditions
https://example.org/trailcast
https://instagram.com/kelownatrailcast
https://tiktok.com/@kelownatrailcast
https://x.com/kelownatrailcast
https://youtube.com/@KelownaTrailcast
```

A refused URL — note the error names the field, and no file
is written:

```text
$ python3 src/link_page_kit.py build bad.json -o bad.html
ERROR: links[0].url: scheme 'javascript' is not allowed (allowed: https, http, mailto, tel): 'javascript:alert(1)'
(exit code: 2)
```

And a duplicate — a warning, not an error: the page builds
and the run exits 1:

```text
$ python3 src/link_page_kit.py build dup.json -o dup.html
link-page-kit v0.1.0 — build · config dup.json
Wrote dup.html — 1,924 bytes · 2 link(s), 0 social(s)
Checks passed:
  ✓ name present
  ✓ 2 link(s), every label present
  ✓ every URL uses an allowed scheme (https, http, mailto, tel)
  ✓ single file, no external requests besides the configured links, socials and avatar
Warnings:
  ! duplicate URL: 'https://example.org/page' (links[1] 'b') is the same target as links[0] 'a'
(exit code: 1)
```

## Tests

```bash
python3 -m unittest discover -s tests -v
# Ran 62 tests — OK
```

The suite covers both directions: the happy path (one file,
all links and socials rendered, notes, dark palette, local-path
avatar); the generated page's contract (no `<script>` anywhere,
the external-URL scan matching the configured set exactly,
semantic landmarks, alt text, focus styles); escaping (a
`<script>` label renders as text, quotes and ampersands in
names and URLs are attribute-safe); every refusal (bad schemes,
scheme-relative, host-less, whitespace, address-less mailto,
digit-less tel) naming its field with nothing written;
duplicates exact, normalised and cross-list (exit 1, page
built); page weight equal to the written file's byte length;
and the error shape (malformed JSON, missing files and
argparse errors all exit 2 cleanly, as `{"error": ...}` under
`--json`, never a traceback).

## Dual use

- **For creators (the Sponsors surface):** a link page you own
  outright, generated locally — the free, permanent answer to a
  $288/yr subscription, a 9% seller fee, and platforms that fold.
- **For its author:** link pages for the author's own YouTube
  channels, and a standard deliverable in Focus Brilliant
  website builds — clients get a link page on their own domain
  instead of a Linktree recommendation that bills them forever.

## Known limits

- **Structural, not live.** The tool validates that a URL is
  well-formed and safe-schemed. Whether the target is up today
  is a network question this tool deliberately never asks —
  run a link checker separately if you need liveness.
- **One page per config.** No batch mode, no multi-page sites,
  no hosting integration — deliberately. The tool makes the
  file; you already have somewhere to put a file.
- **The avatar is referenced, not embedded.** An http(s) avatar
  is one external request (yours, to your host); a local-path
  avatar must sit beside the page where you host it. Embedding
  would bloat every page with a copy of the image.
- **Styling is one accent colour and a light/dark mode.** This
  is a link page, not a website builder — if you need full
  design control, you need a website (which is the point of
  owning the file: you can hand-edit it).

## Privacy

What it touches: only the config file you point it at. What it
sends: nothing — there is no network code in this tool. The
page it generates sends nothing either: no analytics, no
trackers, no fonts — the only requests a visitor's browser
makes are your links and your avatar.

## Support

If this replaces a link-in-bio subscription for you, consider
sponsoring: **https://github.com/sponsors/jasveerai88**

## Licence

MIT — see [LICENSE](LICENSE).
