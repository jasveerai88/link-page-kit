# Contributing

Contributions are welcome — bug reports, fixes and small focused features.

## Ground rules

- **Standard library only.** link-page-kit's promise is "clone
  and run". Pull requests that add a third-party dependency will
  not be merged.
- **Local only, and the tool never fetches.** No network calls,
  no telemetry, no uploads. That includes "just checking whether
  a link is live" — the checks are structural by design, and a
  liveness feature would break the offline promise the README
  makes. Wording that implies a link was verified live will not
  be merged.
- **The page makes no request the config didn't ask for.** The
  generated HTML may reference only the configured links,
  socials and avatar. No web fonts, no CDN assets, no analytics,
  and no `<script>` — ever. The test suite scans the generated
  page and pins the external-URL set to the config exactly; a
  change that adds a reference fails the suite on purpose.
- **The scheme allowlist does not grow casually.** https, http,
  mailto and tel are the page's whole vocabulary. Adding a
  scheme — especially anything scriptable or local (`data:`,
  `file:`, `blob:`) — needs a security argument in the PR and
  the README, SKILL.md, changelog and tests updated together.
- **Duplicates stay warnings.** A repeated target is a finding
  the user may well intend (a pinned link repeated in socials);
  promoting it to an error, or silently de-duplicating the
  page, changes a stated product choice.
- **Escaping is not optional.** Any new place user text reaches
  the page goes through the same HTML escaping, in text and in
  attributes, and gets a test with markup in the input.
- **Both directions tested.** A behaviour change needs a test
  where good input succeeds and bad input fails cleanly — not
  just one side. Page-weight expectations come from the written
  file's bytes, not from constants copied out of a run.

## Getting started

```bash
git clone https://github.com/jasveerai88/link-page-kit
cd link-page-kit
python3 -m unittest discover -s tests -v
python3 src/link_page_kit.py preview examples/sample-config.json
```

## Pull requests

1. Fork, branch, and keep the change to one topic.
2. Add or update tests under `tests/`; the full suite must pass.
3. Note the change in `CHANGELOG.md` under an "Unreleased" heading.
4. Describe the behaviour change with a real before/after run if
   the CLI output changes.
