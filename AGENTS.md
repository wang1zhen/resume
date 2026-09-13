# AGENTS.md

Guidance for any AI coding agent working in this repo. This repo is its own git root, so an
agent launched here sees only this file — it is written to stand alone.

Part of a two-repo workspace at `~/career`; the sibling is `career-ops/`, an AI job-search
system holding the same personal facts. See `~/career/AGENTS.md` when work spans both.

## What this is

A personal LaTeX CV in two languages, built with XeLaTeX.

| File | Role |
|------|------|
| `main.tex` | Japanese CV, for 日系 applications |
| `main_en.tex` | English CV, for 外資 applications |
| `resume.sty` | All shared styling: palette, `\eduentry`, `\cvsection`, `\furiname` |
| `1.jpg` | Photo, used by the Japanese version only |
| `resume_王_一臻_jp.pdf`, `resume_WANG_Yizhen_en.pdf` | Tracked build output at the repo root |
| `LaTeX.aux/` | Gitignored build directory |

## Build

```bash
latexmk -xelatex -outdir=LaTeX.aux -interaction=nonstopmode main.tex
cp LaTeX.aux/main.pdf "resume_王_一臻_jp.pdf"        # required, see below

latexmk -xelatex -outdir=LaTeX.aux -interaction=nonstopmode main_en.tex
cp LaTeX.aux/main_en.pdf resume_WANG_Yizhen_en.pdf
```

**The copy-out step is not optional.** latexmk writes into the gitignored outdir, while the
PDFs are tracked at the repo root. Skip it and the committed PDF silently goes stale against
the `.tex` it claims to represent.

**Both files must stay at one page.** Check after every content edit:

```bash
pdfinfo resume_WANG_Yizhen_en.pdf | grep -i pages
```

**Treat overfull boxes as errors, not warnings.** Count them:

```bash
grep -c Overfull LaTeX.aux/main.log
```

An overfull `\hbox` in a CJK line means text was **silently clipped**, not wrapped — see the
xeCJK note below. An overfull `\vbox` means the page spilled past the text block into the
margin. Both should be zero.

## Two traps that cost real work

**`xeCJK` is load-bearing.** `fontspec` alone gives XeTeX no break opportunities inside a
CJK run, so any Japanese line longer than its box is **silently truncated** — no error, just
missing text. `resume.sty` loads `xeCJK` and sets `\setCJKmainfont`. Do not remove it.

**The font family is `Noto Serif CJK JP`.** There is no family called "Noto Sans Serif CJK";
that string resolves to Latin-only `Noto Sans` and every CJK glyph disappears. Verify what
actually got embedded rather than trusting the source:

```bash
pdffonts resume_王_一臻_jp.pdf | grep -oE 'NotoSerifCJKjp|NotoSansCJKjp'
```

Both faces ship in Debian's `fonts-noto-cjk`, which is what CI installs.

## The two versions deliberately diverge

They used to be one document in two languages. Since 2026-09-01 they serve different markets
and different conventions. **Keep the facts in sync; do not re-align the structure.**

| | `main.tex` (JA, 日系) | `main_en.tex` (EN, 外資) |
|---|---|---|
| Class | `article` 10pt | `extarticle` 9pt — English runs longer at the same size |
| Photo | Yes, Japanese convention | No — not a Western convention, and images break some ATS |
| Opening | Straight to 学歴 | `Profile` summary block |
| Interests / 趣味 | Yes | No |
| Company names | Japanese only (犀牛国際教育・欧美科教育) | English only (X-NEW Education · OMEKA Education) |

Japanese text uses Japanese kanji forms, not simplified: 国**際**, never 国际.

## Content rules

The facts here are mirrored in `career-ops/cv.md`, `config/profile.yml`,
`modes/_profile.md`, `modes/_brief.md`, `article-digest.md` and the LinkedIn draft under
`career-ops/output/`. A terminology change usually touches eight files. `career-ops/modes/_custom.md`
holds the authoritative list of claims that must never drift. The ones that bite most often:

- The rocket-propulsion coupling is **conjugate heat transfer**, never FSI or
  fluid-structure interaction. Thermal only, no structural deformation.
- The combustion work is **reacting-flow CFD with detailed chemical kinetics**
  (Chemkin-format mechanisms in OpenFOAM), not "combustion coupling".
- **75% drag reduction and 40% power saving rate are maxima.** Never round, never add "net".
- Japanese is **JLPT N1, business level**, obtained Aug 2026. The 2020–2023 entry says he
  *began* studying Japanese and was *approaching* prospective supervisors — not that he
  reached business level or "secured" a place then.
- The Melbourne withdrawal never appears without its reason.
- No SOLIZE client names. Domains only.
- American spelling: modeling, program, enrollment, license.

## CI and releases

`.github/workflows/build-resume.yml` builds both files on pushes to `main` that touch
`.tex` / `.sty` / `1.jpg` / the workflow, on `v*` tags, and on manual dispatch. It installs
`fonts-noto-cjk` before compiling.

**Release asset names must be ASCII.** GitHub replaces non-ASCII characters in an asset name
with dots, which once published an unusable `resume_._._jp.pdf`. The repo keeps the CJK
filename; the workflow publishes romanized copies:

```
resume_WANG_Yizhen_jp.pdf
resume_WANG_Yizhen_en.pdf
```

They are **listed explicitly, not globbed** — a `resume_*.pdf` glob also matches the repo's
own tracked CJK-named file and re-uploads the mangled asset on every run.

A `v*` tag cuts a versioned release; any other build refreshes the rolling `latest`
prerelease.

## Pushing

The `origin` remote is SSH and no key is present in the agent environment. Push over HTTPS
instead; `gh` is authenticated:

```bash
git push https://github.com/wang1zhen/resume.git main
```

Avoid `git add -A` here. It has already swept an unrelated stray file into a commit once.
