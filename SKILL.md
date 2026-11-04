---
name: link-page-kit
description: Turn one config JSON file (name, bio, avatar, theme, links, socials) into a link-in-bio page you own — ONE self-contained static HTML file, all CSS inline, zero JavaScript, no external requests except the configured links, socials and avatar. `build` validates the config then writes the page (default index.html); `preview` prints the links in order, the page weight in bytes and the checks passed, writing nothing. URL validation is the product: schemes https/http/mailto/tel only (javascript:, data:, file:, scheme-relative and scheme-less values are REFUSED with exit 2 and nothing written); http(s) needs a host; mailto needs an address; tel needs digits; duplicate URLs (normalised for scheme/host case and a trailing slash, across links AND socials) are warnings — the page still builds, exit 1. All checks are OFFLINE and STRUCTURAL: the tool never fetches the network, so it validates shape, never liveness — never report a link as "live" or "working" from this tool. Every user string is HTML-escaped; a <script> in a label renders as text. Use when a creator or a small business needs a link page without a Linktree-style subscription. Exit 1 is a finding (duplicates), not a failure.
---

# link-page-kit — agent skill

## Purpose

A link-in-bio page is a list of links with a header. Hosted
services charge up to $288/yr for it, take seller fees, offer
custom domains on no plan, and keep folding (Linkpop
discontinued, Koji and Bento folded). This tool generates the
page as one static HTML file from one config JSON — ownable,
hostable anywhere, permanent. Pure Python standard library;
local only; the config file is the only thing it reads, and
nothing it produces phones home.

**The honesty core of this tool:** its checks are structural,
never live — the tool makes no network requests, so it can say
"this URL is well-formed and safe-schemed", never "this link
works". And the page's contract is exact: the only external
references in the output are the links, socials and avatar in
the config. If an agent adds a font, a CDN stylesheet or an
analytics snippet, the tool's reason to exist is gone.

## When to use / not use

Use when:
- A creator or business needs a link-in-bio page they own:
  write a config, `build`, put the file on any static host.
- A pipeline needs a yes/no gate before publishing a page:
  `build` exit 0 = clean; exit 1 = published-with-duplicates
  (decide if deliberate); exit 2 = do not publish, the error
  names the field to fix.
- An agent assembles a page from data it already holds (a
  channel's links, a client's contact links) and needs the
  result checked for unsafe schemes and duplicates before it
  ships.

Do NOT use when:
- You need to know whether links are ALIVE — this tool never
  fetches. Use a link checker for liveness; use this for
  shape, safety and generation.
- You need a multi-page website, a store, or a blog — this
  makes one link page per config, deliberately.
- You need the avatar embedded in the file — avatars are
  referenced (`<img src>`), never embedded, by design.

## Commands

Every command takes the config file as a positional argument
and supports `--json`. Both support `--check-links`, which
lists the structural check (scheme + host/address/number) for
every URL.

```bash
# Build the page (exit 1 with duplicate warnings; exit 2 and
# NO file written on any validation error)
python3 src/link_page_kit.py build CONFIG.json \
    [-o page.html] [--check-links]

# Preview: name, links in order, socials, page weight in
# bytes, checks passed — writes nothing
python3 src/link_page_kit.py preview CONFIG.json [--check-links]
```

Config fields: `name` (required, non-empty string), `bio`
(optional string), `avatar` (optional: http(s) URL or local
path; any other scheme is refused), `theme` (optional object:
`accent` = `#rgb` or `#rrggbb`, default `#2563eb`; `mode` =
`"light"` default or `"dark"`), `links` (required, non-empty
list of `{"label", "url", "note"?}`), `socials` (optional
list, same shape). The config is validated on every run;
errors name the field (`links[2].url`, `theme.accent`,
`socials[0].url`).

## URL rules (exact)

Problems (exit 2, nothing written): scheme not in {https,
http, mailto, tel} — `javascript:`, `data:` and `file:`
included; scheme-relative `//host`; no scheme; http(s) with
no host; any whitespace in the URL; `mailto:` with no `@`
address; `tel:` with no digits; missing or empty `label` or
`url`; missing/empty `name`; missing, empty or non-list
`links`; bad `theme.accent` or `theme.mode`.

Warnings (exit 1, page still built): duplicate URLs. Two
URLs are duplicates when their normalised forms match:
scheme and host lowercased, one trailing path slash removed
(`HTTPS://Example.ORG/page/` == `https://example.org/page`).
Duplicates are detected across `links` and `socials`
combined. Different query strings are different targets.

Checks passed are reported by name: name present; every label
present; every URL on an allowed scheme; single file with no
external requests besides the configured links, socials and
avatar; avatar referenced-not-embedded (when set); accent
valid (when custom); no duplicate URLs (only when true — the
line is absent, not faked, when duplicates exist).

## The page (exact)

One `.html` file: `<!DOCTYPE html>`, `<html lang="en">`,
charset + viewport meta, `<title>{name} — links</title>`, a
meta description from the bio, ALL CSS in one inline `<style>`
(system font stack; accent colour; `:focus-visible` outlines;
mobile-first, max-width 640px column). Body: `header` (avatar
`<img>` with `alt="Portrait of {name}"`, `h1` name, bio),
`main` with `<nav aria-label="Links">` holding the link list
(each link a full-width card; `note` rendered as a sub-line)
and, when socials exist, `<nav aria-label="Social profiles">`
with a pill row. **No `<script>` element exists in the
output.** Every user-supplied string is HTML-escaped in text
and attributes. Page weight = `len(page.encode("utf-8"))` —
`preview`'s number is exactly the byte length of the file
`build` writes.

## Output JSON schema (`--json`)

Common fields: `tool` (`"link-page-kit"`), `version`,
`command`, `config_file` (file name), `name`, `page_bytes`,
`links` — each `{label, url, note?}` in config order —
`socials` (same shape), `checks_passed` (list of strings),
`warnings` (list of strings; duplicates appear here with both
entries named). `build` adds `output_file` (the path written,
as given or `index.html`).

On error, stdout is a single object: `{"error": "<message>"}`
with exit code 2 — including argparse usage errors in `--json`
mode. Never plain text, never a traceback.

## Exit codes

- `0` — clean: `build` wrote the page with no warnings;
  `preview` found nothing to flag.
- `1` — a finding: duplicate URLs. The page WAS built (for
  `build`) — act on the warning, don't treat it as a crash.
- `2` — ERROR: config missing or not valid JSON, a required
  field missing, an unsafe or malformed URL, a bad theme
  value, an unwritable output path, an invalid flag. For
  `build`, NO file was written. No output facts may be quoted
  from a failed run.

## Limits & honesty rules the agent must respect

- **Structural, never live.** Report checks as "well-formed"
  or "valid shape" — never "the link works", "the page is
  reachable" or "verified live". The tool makes no requests;
  neither may your summary imply it did.
- **A duplicate warning is not an error and not nothing.**
  Exit 1 means the page exists AND carries repeated targets.
  Surface the warning verbatim; let the user decide whether
  the repetition is deliberate. Never silently drop the
  duplicate entries — the tool doesn't, and you shouldn't
  rewrite its output by hand.
- **Exit 2 from `build` means no page exists.** Do not
  reconstruct the HTML yourself from the config to "help" —
  the refused scheme or missing label is exactly why the gate
  exists. Fix the config and re-run.
- **The no-external-requests contract is the product.** If
  you post-process the page, do not add fonts, CDNs, trackers
  or scripts. The configured links, socials and avatar are
  the only external references allowed.
- **The avatar is a reference.** A local-path avatar must be
  hosted beside the page; an http(s) avatar is fetched from
  wherever the config points. Never claim the page is fully
  self-contained when an avatar URL is configured — say "one
  file plus the avatar it references".
- Local only: the tool reads only the config file named on
  the command line and sends nothing anywhere.
