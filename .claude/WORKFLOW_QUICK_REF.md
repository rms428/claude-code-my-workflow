# Workflow Quick Reference

**Model:** Contractor (you direct, Claude orchestrates)

---

## The Loop

```
Your instruction
    ↓
[PLAN] (if multi-file or unclear) → Show plan → Your approval
    ↓
[EXECUTE] Implement, verify, done
    ↓
[REPORT] Summary + what's ready
    ↓
Repeat
```

---

## I Ask You When

- **Design forks:** "Option A (fast) vs. Option B (robust). Which?"
- **Code ambiguity:** "Spec unclear on X. Assume Y?"
- **Replication edge case:** "Just missed tolerance. Investigate?"
- **Scope question:** "Also refactor Y while here, or focus on X?"

---

## I Just Execute When

- Code fix is obvious (bug, pattern application)
- Verification (tolerance checks, tests, compilation)
- Documentation (logs, commits)
- Plotting (per established standards)
- Deployment (after you approve, I ship automatically)

---

## Quality Gates (No Exceptions)

| Score | Action |
|-------|--------|
| >= 80 | Ready to commit |
| < 80  | Fix blocking issues |

---

## Non-Negotiables

- **CRS:** Always reproject to UTM Zone 37N (EPSG:32637) before area calculations or spatial joins. Input data from GEE is WGS84 — never compute areas in degrees.
- **GEE collection:** Always document the ImageCollection ID, filterDate range, filterBounds region, and cloud threshold used. Comment these in every script.
- **Reproducibility:** Pin GEE collection versions in comments. Export key intermediate products as GEE assets. Log asset IDs and export dates in metadata.
- **Figure standard:** All maps must include scale bar, north arrow, legend with units, and coordinate labels. Minimum 300 DPI for exports.
- **NDVI tolerance:** Same-date, same-sensor NDVI agreement must be within ±0.005. Larger discrepancies indicate a scale factor or masking error.
- **Path convention:** Use `pathlib.Path` and `PROJECT_ROOT` — no hardcoded absolute paths in scripts.
- **Seeds:** `random.seed(42)` + `np.random.seed(42)` at top of any stochastic script.

---

## Preferences

<!-- Fill in as you discover your working style -->

**Visual:** Maps must be publication-ready (scale bar, north arrow, colorbar with units, 300 DPI). Use viridis or RdYlGn palettes — no jet.
**Reporting:** Concise bullets for summaries; detailed prose in reports; code output on request.
**Session logs:** Always (post-plan, incremental, end-of-session)
**Replication:** Flag NDVI discrepancies > ±0.005. Flag area discrepancies > 0.1 ha. Investigate before reporting near-misses.

---

## Exploration Mode

For experimental work, use the **Fast-Track** workflow:
- Work in `explorations/` folder
- 60/100 quality threshold (vs. 80/100 for production)
- No plan needed — just a research value check (2 min)
- See `.claude/rules/exploration-fast-track.md`

---

## Next Step

You provide task → I plan (if needed) → Your approval → Execute → Done.
