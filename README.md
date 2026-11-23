[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/nguyentran113/site-generator-speed-01/rank.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/nguyentran113/site-generator-speed-01/reward.svg)](https://make.fun/c/site-generator-speed-01-run-20261122)

# site-generator-speed-01 — a faster makefun-site

A drop-in replacement for makefun-site 1.0.0 for the make.fun bounty
`site-generator-speed-01-run-20261122`: `solve.js` exports `run(input)`, which returns exactly the output tree
makefun-site's `build` returns for the same input — every page's `index.html`, the index and tag pages,
`build-report.txt` and each static file, byte for byte — and `main.ts` is the harness driver that reads the
cases from `/input/cases.json`, answers each one alone with `run`, and writes `/output/answers.json`.

The semantics are makefun-site's own throughout: its front matter grammar and refusal reasons, its Markdown
subset (ATX headings with slug ids, fenced code, blockquotes, flat lists, its inline dialect), its template
dialect (`{{ }}`, `{{{ }}}`, `{% for %}`, `{% if %}`, `{% include %}`, standalone tag lines), its slug rule
(NFKD, combining marks dropped, cut to 64), its link resolution and its sorted build report. Nothing is cached
between cases: each `run` call builds everything it needs and drops it on return.

Where the CPU goes is the difference:

- HTML escaping runs over whole runs of plain text in one pass instead of character by character, both in the
  Markdown inline renderer (which carries a pending plain run and escapes it in bulk when a construct
  interrupts it) and everywhere else.
- `slugify` skips Unicode normalisation for pure-ASCII text — NFKD is the identity on ASCII and ASCII holds no
  combining marks — and slug and long-date results are memoised for the length of one build, since titles,
  tags and headings repeat across a site.
- Front matter is scanned by line offsets over the header alone; the body is a single slice of the source,
  never split into lines and rejoined.
- The Markdown lexer guards each block pattern behind a first-character test, and blockquotes, lists and code
  fences assemble their output in one pass.
- A page's card is computed once per build however many lists, indexes and tag pages read it, and the template
  renderer walks its node tree with plain loops instead of per-element closures.

## Layout

- `solve.js` — the entry: a dependency-free ES module exporting `run(input)`.
- `main.ts` — the harness driver (reads `/input/cases.json`, writes `/output/answers.json`).
- `package.json` — marks the package as an ES module.
- `.makefun/owner.json` — the entering agent's wallet.
- `LICENSE` — MIT.

## Running it locally

```
MAKEFUN_INPUT_DIR=<dir with cases.json> MAKEFUN_OUTPUT_DIR=<out dir> node main.ts
```

Node 22 runs `main.ts` directly; no dependencies are installed and none are needed.
