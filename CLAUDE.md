# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Palladium is a basic/intermediate algebra textbook written in **PreTeXt** (XML-based authoring
language for open textbooks, https://pretextbook.org/). It is not a software project — there is no
compiling application code, and "building" means rendering the XML source into HTML/LaTeX/PDF/WeBWorK.

The content originates from **ORCCA** (Open Resources for Community College Algebra, copyright
Portland Community College, github.com/PCCMathSAC/orcca/) and is being reorganized/edited into a new
book structure. Per the initial commit message, the ORCCA content was "shuffled" into this project's
structure and is being trimmed and edited chapter by chapter (see commit history: "trim content",
"lots of spring 2026 edits", "summer 2026 edits").

## Build commands

```
pretext build html      # build the html target
pretext build latex     # build the latex target
pretext build pdf       # build the pdf target (xelatex)
pretext build pg        # build WeBWorK (.pg) problem sets
pretext view html -b    # serve the built html locally and open a browser
pretext build --deploys # build all targets configured for deployment (used in CI)
```

Targets are defined in `project.ptx`. All four targets share the same source
(`src/palladium.ptx`) and the same `publication/publication.xml`.

`requirements.txt` pins the PreTeXt CLI version used by CI (`pretext == 2.41.2`); a newer CLI may be
installed locally (check with `pretext --version`). Install with `pip install -r requirements.txt`.

There is no test suite or linter. The closest thing to validation is `pretext build`/`pretext
generate` itself, which reports schema/XML errors (missing labels, disallowed elements, malformed
WeBWorK problems, etc.) for the files it processes.

## Source structure

`src/palladium.ptx` is the book root. It `xi:include`s `bookinfo.ptx` (docinfo: LaTeX macros, element
renames like "solution" → "Explanation", cross-reference style, WeBWorK PG macro setup) and
`frontmatter.ptx`, then the chapters, then `backmatter.ptx`.

**Only a subset of the source is actually wired into the book.** `src/` contains far more
`chapter-*.ptx`, `section-*.ptx`, and `review-*.ptx` files (inherited from the ORCCA reshuffle) than
are currently `xi:include`d anywhere. Right now `src/palladium.ptx` only includes:
`chapter-linear.ptx`, `chapter-quadratic.ptx`, `chapter-rational-functions-and-equations.ptx`,
`chapter-radical-expressions-and-equations.ptx`. The other ~10 `chapter-*.ptx` files exist on disk but
are not part of the built book yet — when asked to "add" or "restore" a chapter, check whether it
needs an `xi:include` added to `src/palladium.ptx` (and whether its own sections need including). Some
includes are also individually commented out inside a chapter file (e.g. a `review-*.ptx` include in
`chapter-variables-expressions-equations-and-inequalities.ptx`) — check for that too before assuming a
file is dead.

`src/retired/` holds content fully pulled out of circulation (old appendices, chapters, sections) —
distinguish this from the merely-not-yet-included files described above.

File naming conventions:
- `chapter-<name>.ptx` — a `<chapter>`; has an `<introduction>` then `xi:include`s its `section-*.ptx`
  files and usually ends with its `review-*.ptx` include.
- `section-<name>.ptx` — a `<section>`, the actual lesson content.
- `review-<name>.ptx` — the end-of-chapter review section for a chapter.
- `appendix-<name>.ptx` — backmatter appendices (only `appendix-basic-algebra-review`'s sections are
  currently included via `backmatter.ptx`; `appendix-ccogs.ptx` and `appendix-unit-conversions.ptx`
  exist but aren't included).

### Section internal structure

A typical `section-*.ptx` runs: `<introduction>` → topic subsections (each usually with its own
`<introduction>`) → `<reading-questions>` → `<exercises>`, where `<exercises>` is split into
`<subexercises>` groups titled (in order) **Review and Warmup**, **Skills Practice**,
**Applications**, and **Challenge**. Exercises may be plain PreTeXt `<exercise>` (with
`<statement>`/`<hint>`/`<answer>`/`<solution>`) or `<webwork>` exercises embedding WeBWorK PG code
(`<pg-macros>`/`<setup>`/`<pg-code>`/`<statement>`).

The checklist left at the bottom of `src/palladium.ptx` describes the target conventions for finished
sections (every exercise has `@label`, the Review/Skills/Applications/Challenge subexercises
structure, multipart exercises use `<task>`, exercise groups have titles, indexing, CCOG indexing,
every section has reading questions, every chapter has an introduction) — useful as a checklist when
editing or reviewing a section, since not all existing sections meet it yet.

`src/KarasChallengeProblems.xml` is a standalone bank of WeBWorK challenge `<exercisegroup>`s, not
currently `xi:include`d into the book.

## Publication and styling config

`publication/publication.xml` controls cross-cutting output behavior (not part of `src/`):
- `<source>`: asset directories are `../assets` (external, checked in) and `../generated-assets`
  (generated at build time, gitignored).
- `<webwork>`: points at a local `pg-location` and the `webwork.pcc.edu` server/course for
  static-processing WeBWorK problems.
- `<html>`: base URL, knowl settings, GeoGebra calculator, index page.
- `<latex>`: custom page geometry (non-standard paper size) for the print/PDF target.

`publication/css/` holds several `colors_sapphire_*.css` theme variants plus `orcca.css`, inherited
from ORCCA. `xsl/` holds custom XSL stylesheets (`orcca-html.xsl`, `orcca-latex.xsl`,
`orcca-print.xsl`, etc.) and a LaTeX preamble under `xsl/latex-preamble/` — note these are **not**
currently referenced by `project.ptx` (no `<xsl>` element on any target), so confirm whether a given
custom stylesheet is actually wired in before assuming it affects the build.

## Other root files

- `codechat_config.yaml` — config for the CodeChat live-preview extension used when editing in VS Code
  (see `.devcontainer/devcontainer.json`, which also lists the recommended VS Code extensions:
  `oscarlevin.pretext-tools`, `CodeChat.codechat`, spell checker, etc.).
- `palladium.tex` / `jingreport.txt` — stray build artifacts (a generated LaTeX file and a schema-
  validation error log) currently untracked at the repo root; not part of the authored source.
- `.github/workflows/pretext-cli.yml` — runs on PRs, builds all deploy targets and stages them as a
  downloadable artifact (and optionally to Cloudflare Pages / GitHub Pages if configured).
- `.github/workflows/pretext-deploy.yml` — manual (`workflow_dispatch`) build-and-push to the
  `gh-pages` branch.
