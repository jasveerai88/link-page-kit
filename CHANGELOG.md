# Changelog

## v0.1.0 — 2026-11-04

First release.

- **One config JSON in, one owned link page out** — a config
  holding `name`, `bio`, `avatar`, `theme` (`accent`, `mode`),
  `links` and `socials` generates a complete link-in-bio page:
  one static HTML file, all CSS inline in a single `<style>`
  block, zero JavaScript, a system font stack, and no external
  requests except the configured links, socials and avatar.
  Semantic HTML with labelled nav landmarks, avatar alt text,
  `:focus-visible` outlines, mobile-first layout. The bundled
  sample page weighs 3,122 bytes.
- **`build CONFIG [-o page.html]`** — validates, then writes
  the page (default output `index.html`). Exit 0 = clean,
  1 = built with duplicate warnings, 2 = error and nothing
  written.
- **`preview CONFIG`** — the summary without writing anything:
  the links in order, the socials, the page weight in bytes
  (exactly the byte length of the file `build` writes) and
  the checks passed.
- **Validation is the product** — every URL checked offline
  (structural only; the tool never fetches the network and
  never claims liveness): schemes https/http/mailto/tel only,
  with `javascript:`, `data:`, `file:`, scheme-relative and
  scheme-less values refused; http(s) URLs need a host and no
  whitespace; `mailto:` needs an address; `tel:` needs
  digits; labels are required; every error names its field.
  `--check-links` lists the structural check behind every URL.
- **Duplicates flagged, not fatal** — URLs are compared after
  normalising scheme/host case and a trailing slash, across
  links and socials combined; a duplicate is a warning (the
  page builds, exit 1) naming both entries.
- **Escaping everywhere** — every user-supplied string is
  HTML-escaped in text and attributes; a `<script>` in a
  label renders as text, and the generated page contains no
  `<script>` element at all.
- **The avatar is referenced, never embedded** — an http(s)
  URL or a local path in an `<img src>`, so the page stays
  one small file.
- **`--json` on every command** — one JSON object with the
  same facts and exit codes as the text view; errors are
  `{"error": ...}` objects, argparse usage errors included.
- **`SKILL.md`** at the repo root for AI coding agents:
  commands, the JSON schemas, exit codes, and the honesty
  rules (structural never live; a duplicate is a warning, not
  an error; no external requests beyond the config's).
- Bundled sample config (`examples/sample-config.json`): a
  creator page — 6 links (YouTube, Instagram, a shop product,
  a podcast, a trail map, one mailto) and 2 socials.
- 62 stdlib-unittest tests: the happy path, the page's
  no-script and external-URL-scan contracts, escaping, every
  refusal naming its field, duplicate detection exact /
  normalised / cross-list, page-weight equality with the
  written file, and the clean-error shape in both modes.
- Pure Python standard library; local only; MIT licence.
