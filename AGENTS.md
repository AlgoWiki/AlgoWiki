# AlgoWiki — contributor & agent guide

AlgoWiki is a wiki dedicated to competitive programming. It is a **learning
resource and reference** meant to cover topics from the basics all the way to
the most advanced uses in competitive programming. Source pages live in
`wiki/*.md`.

The live site at <https://wiki.algo.is> is a **[Gatsby](https://www.gatsbyjs.com)
static site** (separate repo, this repo is included as a git submodule). It
renders pages with `gatsby-transformer-remark` — i.e. **remark / GitHub Flavored
Markdown**, *not* pandoc — plus a custom `gatsby-remark-wiki-link` plugin for
`[Page]()` links, **build-time [KaTeX](https://katex.org)** (via
`gatsby-remark-katex` / `remark-math`) for `$…$` and `$$…$$`, and **Tailwind
Typography** (`prose`) for styling. Knowing this matters: see the math rules
below and the verification notes at the end.

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
- **Math** is rendered at build time by KaTeX, via `remark-math`. Because
  `remark-math` parses `$…$`/`$$…$$` into math *before* Markdown escaping runs,
  backslash sequences like `\{`, `\}`, `\\`, `\#` are now safe inside math (this
  used to be broken under the old MathJax setup — don't reintroduce workarounds).
  Two rules:
    - **Inline math:** `$ … $`. Works anywhere in text.
    - **Display math:** the `$$` fences **must each be alone on their own line,
      with a blank line before and after** — only this form renders as centered
      display math:
      ```

      $$
      \sum_{i=1}^n i = \frac{n(n+1)}{2}
      $$

      ```
      A single-line `$$ … $$`, or `$$` with content on the fence line, renders as
      *inline* math (or breaks the paragraph), so always use the block form above.
  KaTeX is stricter than MathJax (no `\mbox` quirks etc.); the build runs with
  `throwOnError:false`, so a bad formula shows in red rather than failing the
  build — grep the built HTML for `katex-error` to catch these.
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
  blocks. It is *not* the live renderer (pandoc handles math differently), so use
  it only as a structure linter.
- **Math**: confirm every display formula uses the blank-line `$$`-on-own-lines
  block form. The authoritative check is to build the actual site: in the website
  repo, point its `content` submodule at your commit, run `gatsby build` (or
  `gatsby develop`), and grep the built HTML — `katex-error` must be **0**, and
  display formulas should appear as `math-display` divs.
- **Code**: compile every snippet (`g++ -O2 -std=c++17`) and sanity-check its
  output against known values. Don't ship code you haven't run.

## Git

The working branch is `main` (tracks `origin/main`); recent content commits live
there. Commit and push **only when the user asks**.
