---
name: flesh-out-wiki-page
description: Expand an AlgoWiki page (wiki/*.md) from a stub into a complete, learning-oriented article — with description, complexity, variants, curated problems with spoiler solution sketches, cross-references, and verification. Use when asked to flesh out, improve, write, or fill in a wiki page/topic.
---

# Flesh out a wiki page

Turn a sparse AlgoWiki page into a complete article. The conventions, page
template, and quality bar live in the repo's `AGENTS.md` — **read it first** and
treat it as the source of truth; this skill is the procedure for applying it.

## 1. Orient (once per session is enough)

- Read `AGENTS.md` at the repo root for format, template, and the quality bar.
- Confirm the toolchain is available: `which pandoc g++`.
- Skim two or three well-developed pages to match tone, e.g. `wiki/Binary
  search.md` and `wiki/Trie.md`.

## 2. Pick / confirm the page

- If the user named a page, use it. Otherwise propose one: prefer a **foundational
  topic that is currently a stub**, is reasonably self-contained, and is rich in
  cross-references and available problems. Briefly confirm the choice (and the
  spoiler-sketch approach) before investing heavily.
- Read the existing file. Treat its content as a starting point only — the wiki is
  intentionally incomplete, so rewrite freely rather than preserving the stub.

## 3. Research the content

- Cover the topic basics → advanced: core idea, a reference implementation,
  complexity, applications, and the important variants/extensions.
- Gather a **curated** problem set from multiple judges (Kattis, CSES, Codeforces,
  SPOJ, Project Euler, ICPC archives). Use WebSearch/WebFetch to find strong,
  canonical problems and to confirm links resolve. Prefer problems that each
  isolate a distinct facet of the topic.
- Identify pages to cross-reference (both existing and worth-creating).

## 4. Write the page

Follow the section template in `AGENTS.md` (intro → Description → Applications →
Variants → Problems → See also → External links). Specifically:

- Frontmatter `categories:` reusing existing category names.
- Bold the term on first use; lead with what/why/complexity.
- Reference implementations in `~~~ {.cpp}` fences.
- Group the Problems list by the idea each exercises, and attach `<details>`
  solution sketches to representative entries. Each sketch focuses **only** on how
  this page's topic is used — not a full editorial. Pattern:
  ```
  <details>
  <summary>Solution sketch — Problem Name</summary>

  ...topic-relevant reasoning, with $math$ / `code` as needed...

  </details>
  ```
- Cross-reference liberally with `[Page Name]()`; create-links to not-yet-existing
  pages are fine.

## 5. Verify (do not skip)

The live site is remark/GFM + client-side MathJax + Tailwind `prose`, **not**
pandoc — see the rendering + math caveat in `AGENTS.md`. So:

- **Structure**: `pandoc -f gfm -t html "<Page>.md" >/tmp/out.html` as a quick
  linter. Confirm every `<details>` block is present, empty-target wiki links
  survived (`href=""`), and any internal `(#anchor)` links match a generated
  `id="…"`. Remember pandoc does **not** reveal math problems.
- **Math**: scan every `$…$`/`$$…$$` for backslash-escaped ASCII punctuation
  (`\#`, `\{`, `\}`, `\\`, `\_`, …), which remark strips before MathJax. Rephrase
  to avoid them. For real confidence, preview in the website repo
  (`gatsby develop`, this repo as its `content` submodule at your commit).
- **Code**: extract every C++ snippet into one program, `g++ -O2 -std=c++17`, run
  it, and check outputs against known values (e.g. known counts, a worked example,
  spot-checked function values). Fix anything that doesn't match before shipping.

## 6. Report and (only if asked) commit

- Summarize what changed and the verification you ran.
- Commit/push **only when the user asks**. Stage just the page(s) you changed. The
  working branch is `main` (tracks `origin/main`).

## Reference

The first run of this workflow produced `wiki/Sieve of Eratosthenes.md` — use it
as the worked example of the target quality and structure.
