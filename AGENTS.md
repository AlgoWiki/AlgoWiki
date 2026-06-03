# AlgoWiki — contributor & agent guide

AlgoWiki is a wiki dedicated to competitive programming. It is a **learning
resource and reference** meant to cover topics from the basics all the way to
the most advanced uses in competitive programming. Source pages live in
`wiki/*.md` and are rendered by **[gitit](https://github.com/jgm/gitit) +
[pandoc](https://pandoc.org)** (pandoc's extended Markdown) at
<https://wiki.algo.is>.

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
  Linking to a page that does not exist yet is fine and encouraged — gitit turns
  it into a "create this page" link. Link liberally; cross-references are a core
  value of the wiki.
- **Math** is `$inline$` and `$$display$$` (MathJax on the live site).
- **Code** goes in fenced blocks tagged with the language, e.g. `~~~ {.cpp}`.
  C++ is the house language; assume `#include <bits/stdc++.h>` and `using
  namespace std;`.
- **Spoilers / solution sketches** use raw-HTML `<details>` with a blank line
  before and after the inner Markdown so pandoc parses it (math and code work
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

Run from inside `wiki/`:

- **Render**: `pandoc -f markdown -t html "<Page>.md" >/dev/null` — must exit 0
  with no warnings other than `Could not convert TeX math …` (that one is
  harmless; the live site has MathJax). Confirm `<details>` blocks and any
  internal `#anchor` links resolve to generated header IDs.
- **Code**: compile every snippet (`g++ -O2 -std=c++17`) and sanity-check its
  output against known values. Don't ship code you haven't run.

## Git

The working branch is `main` (tracks `origin/main`); recent content commits live
there. Commit and push **only when the user asks**.
