# CLAUDE.MD -- Academic Project Development with Claude Code

**Project:** Landscape-Scale Ecological Monitoring for Rangeland Governance: Kenya Case Study
**Institution:** University of California, Santa Barbara / Mercy Corps
**Branch:** main

---

## Core Principles

- **Plan first** -- enter plan mode before non-trivial tasks; save plans to `quality_reports/plans/`
- **Verify after** -- compile/render and confirm output at the end of every task
- **Single source of truth** -- Beamer `.tex` is authoritative; Quarto `.qmd` derives from it
- **Quality gates** -- nothing ships below 80/100
- **[LEARN] tags** -- when corrected, save `[LEARN:category] wrong → right` to MEMORY.md

---

## Folder Structure

```
claude-code-my-workflow/
├── CLAUDE.md                    # This file
├── .claude/                     # Rules, skills, agents, hooks
├── Bibliography_base.bib        # Centralized bibliography
├── Figures/                     # Figures, maps, and images
├── Preambles/header.tex         # LaTeX headers
├── Slides/                      # Beamer .tex files
├── Quarto/                      # RevealJS .qmd files + theme
├── python/                      # Production Python scripts
├── notebooks/                   # Jupyter notebooks (exploration)
├── Data/
│   ├── raw/                     # Input data (GEE remote; document source)
│   ├── processed/               # Exported GeoTIFFs, spectral index rasters
│   └── validation/              # Ground truth points, reference data
├── Reports/                     # LaTeX source for papers + technical reports
├── docs/                        # GitHub Pages (auto-generated)
├── scripts/                     # Utility scripts
├── quality_reports/             # Plans, session logs, merge reports
├── explorations/                # Research sandbox (see rules)
├── templates/                   # Session log, quality report templates
└── master_supporting_docs/      # Papers and existing materials
```

---

## Commands

```bash
# LaTeX (3-pass, XeLaTeX only)
cd Slides && TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex
BIBINPUTS=..:$BIBINPUTS bibtex file
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex
TEXINPUTS=../Preambles:$TEXINPUTS xelatex -interaction=nonstopmode file.tex

# Python / GEE
python python/script_name.py          # Run production script
jupyter notebook notebooks/           # Launch notebook server
python -c "import ee; ee.Authenticate()"  # First-time GEE auth
python -c "import ee; ee.Initialize(project='YOUR_GEE_PROJECT')"

# Deploy Quarto to GitHub Pages
./scripts/sync_to_docs.sh LectureN

# Quality score
python scripts/quality_score.py Quarto/file.qmd
```

---

## Quality Thresholds

| Score | Gate | Meaning |
|-------|------|---------|
| 80 | Commit | Good enough to save |
| 90 | PR | Ready for deployment |
| 95 | Excellence | Aspirational |

---

## Skills Quick Reference

| Command | What It Does |
|---------|-------------|
| `/compile-latex [file]` | 3-pass XeLaTeX + bibtex |
| `/deploy [LectureN]` | Render Quarto + sync to docs/ |
| `/extract-tikz [LectureN]` | TikZ → PDF → SVG |
| `/proofread [file]` | Grammar/typo/overflow review |
| `/visual-audit [file]` | Slide layout audit |
| `/pedagogy-review [file]` | Narrative, notation, pacing review |
| `/review-r [file]` | R code quality review |
| `/qa-quarto [LectureN]` | Adversarial Quarto vs Beamer QA |
| `/slide-excellence [file]` | Combined multi-agent review |
| `/translate-to-quarto [file]` | Beamer → Quarto translation |
| `/validate-bib` | Cross-reference citations |
| `/devils-advocate` | Challenge slide design |
| `/create-lecture` | Full lecture creation |
| `/commit [msg]` | Stage, commit, PR, merge |
| `/lit-review [topic]` | Literature search + synthesis |
| `/research-ideation [topic]` | Research questions + strategies |
| `/interview-me [topic]` | Interactive research interview |
| `/review-paper [file]` | Manuscript review |
| `/data-analysis [dataset]` | End-to-end analysis workflow |
| `/learn [skill-name]` | Extract discovery into persistent skill |
| `/context-status` | Show session health + context usage |
| `/deep-audit` | Repository-wide consistency audit |

---

## Beamer Custom Environments

| Environment          | Effect                         | Use Case                         |
|----------------------|--------------------------------|----------------------------------|
| `keybox`             | Gold background box            | Key findings and takeaways       |
| `methodbox`          | Blue-bordered titled box       | Methods and protocol summaries   |
| `resultbox`          | Green-accent box               | Main results and estimates       |
| `highlightbox`       | Gold left-accent box           | Caveats and important notes      |
| `definitionbox[T]`   | Blue-bordered titled box       | Formal definitions and notation  |

## Quarto CSS Classes

| Class          | Effect              | Use Case                         |
|----------------|---------------------|----------------------------------|
| `.smaller`     | 85% font size       | Dense content / methods slides   |
| `.positive`    | Green bold text     | Positive results annotations     |
| `.callout`     | Highlighted aside   | Caveats, limitations, notes      |
| `.map-caption` | Small italic text   | Figure/map attribution lines     |

---

## Current Project State

| Output | File | Status | Key Content |
|--------|------|--------|-------------|
| Paper 1: Study Design | `Reports/paper1_design.tex` | Planned | DiD design, 50 communities, Garissa/Wajir |
| Slides: Study Overview | `Slides/overview.tex` | Planned | Project background, Mercy Corps context |
| Slides: Methods | `Slides/methods.tex` | Planned | GEE pipeline, NDVI/EVI, spatial analysis |
| Slides: Preliminary Results | `Slides/results.tex` | Planned | Spectral index trends, treatment effects |
