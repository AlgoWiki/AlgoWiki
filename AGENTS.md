# AlgoWiki — contributor & agent guide

AlgoWiki is a wiki dedicated to competitive programming. It is a **learning
resource and reference** meant to cover topics from the basics all the way to
the most advanced uses in competitive programming. Source pages live in
`wiki/*.md`.

The live site at <https://wiki.algo.is> is a **[Gatsby](https://www.gatsbyjs.com)
static site** (separate repo, this repo is included as a git submodule). It
renders pages with `gatsby-transformer-remark` — i.e. **remark / GitHub Flavored
Markdown**, *not* pandoc — plus a custom `gatsby-remark-wiki-link` plugin for
`[Page]()` links, **client-side [MathJax](https://www.mathjax.org)** for `$…$`,
and **Tailwind Typography** (`prose`) for styling. Knowing this matters: see the
math caveat below and the verification notes at the end.

Most pages today are sparse stubs — often just a list of problems. The goal is to
flesh them out into complete, self-contained articles. Do **not** match the thin
existing style; aim for the quality bar described below.

## Markdown conventions

- **Frontmatter** (optional but expected on content pages):
  ```
  ---
  categories: Number theory, Algorithm techniques
  ---
  ```
  Reuse existing category names where possible (`Graph theory`, `Graph
  algorithms`, `Algorithm techniques`, `Data structures`, `Combinatorics`,
  `Geometry`, `Number theory`, `String data structures`, …).
- **Wiki links** use the empty-target form `[Page Name]()`, or `[label](Page
  Name)` for custom text, and `[label](Page Name#anchor)` to target a section.
  Linking to a page that does not exist yet is fine and encouraged. Link
  liberally; cross-references are a core value of the wiki.
- **Math** is `$inline$` and `$$display$$`, rendered client-side by MathJax.
  **Caveat (important):** there is currently no remark math plugin, so remark
  treats `$…$` as ordinary text and applies Markdown's backslash-escaping *before*
  MathJax runs. That silently strips the backslash from any escaped ASCII
  punctuation — `\#`, `\{`, `\}`, `\%`, `\_`, `\&`, and `\\` (turned into a single
  `\`). So `$\#\{x\}$` reaches MathJax as `$#{x}$` and breaks. **Avoid those in
  math:** prefer plain `{`/`}` for grouping, rephrase to drop `\#`/`\{ \}`
  set-builder notation, and avoid `\\` line breaks inside `array`/`cases`.
  Sequences like `\log`, `\sum`, `\frac`, `\binom`, `a_i`, `x^2` are safe (the
  char after `\` is not punctuation). The clean long-term fix is to add a remark
  math plugin to the site; until then, author math defensively.
- **Code** goes in fenced blocks tagged with the language, e.g. `~~~ {.cpp}`.
  C++ is the house language; assume `#include <bits/stdc++.h>` and `using
  namespace std;`.
- **Spoilers / solution sketches** use raw-HTML `<details>` with a blank line
  before and after the inner Markdown so remark parses it (math and code work
  inside):
  ```
  <details>
  <summary>Solution sketch — Problem Name</summary>

  Body Markdown, with $math$ and `code` as needed.

  </details>
  ```

## Page template

Order sections roughly like this (omit any that don't apply, add others as the
topic needs):

1. **Intro paragraph** — bold the term on first use; one or two sentences saying
   what it is, its complexity, and where it fits. Link related pages.
2. `## Description` — how it works, with a clean reference implementation. Fold in
   a short complexity argument (a `### Complexity` subsection is fine).
3. `## Applications` — what it's used for, each bullet linking relevant pages.
4. `## Variants` — extensions and specializations, each with its own short
   explanation and code where useful.
5. `## Problems` — a **curated** list, grouped by the idea each problem
   exercises. Draw from multiple judges (Kattis, CSES, Codeforces, SPOJ, Project
   Euler, ICPC archives…), not just one. Attach a `<details>` solution sketch to
   representative entries; the sketch should focus **only** on how this page's
   topic is used, not be a full editorial.
6. `## See also` — wiki links to closely related pages, each with a few words of
   why.
7. `## External links` — high-quality tutorials/references (cp-algorithms,
   Codeforces blogs, Wikipedia, …).

## Quality bar

- Write for a learner: motivate the idea, then make it precise. Progress from
  basics to advanced within the page.
- Prefer correct, idiomatic, compilable C++ over pseudocode. Watch for the usual
  traps (integer overflow, off-by-one, array bounds).
- Make cross-references dense and accurate.

## Verify before committing

- **Structure (quick local check)**: `pandoc -f gfm -t html "<Page>.md" >/dev/null`
  is a handy sanity check for headings, lists, fenced code, and `<details>`
  blocks. **But pandoc is _not_ the live renderer and does _not_ reproduce the
  math caveat above** — pandoc parses `$…$` as math and keeps the backslashes,
  whereas the live remark/GFM build strips them. So a page can look fine in pandoc
  and still render broken math on the site. Treat pandoc as a structure linter
  only.
- **Math**: eyeball every `$…$`/`$$…$$` for the risky escapes listed in the math
  caveat (`\#`, `\{`, `\}`, `\\`, …). The authoritative check is to build/preview
  the actual site (`gatsby develop` in the website repo, with this repo checked
  out as its `content` submodule at your commit) and look at the rendered math.
- **Code**: compile every snippet (`g++ -O2 -std=c++17`) and sanity-check its
  output against known values. Don't ship code you haven't run.

## Git

The working branch is `main` (tracks `origin/main`); recent content commits live
there. Commit and push **only when the user asks**.
