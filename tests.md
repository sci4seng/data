# 100 regression tests for the sci4seng/{core,data,lifts} split

Each test is one invariant. "Pre-split" = behavior of this monorepo
as of 2026-06-01. "Post-split" = the three sci4seng/* repos. The
split must preserve every line below.

Group A–J, 10 each.

---

## A. Repo layout + size (10)

1. `sci4seng/core` clone size < 50 MB (was: 9 GB monorepo).
2. `sci4seng/lifts` clone size < 20 MB.
3. `sci4seng/data` git-tracked size < 5 MB (manifest + README + dropzone only).
4. `sci4seng/core` contains `paper/`, `docs/`, `scripts/`, top-level `*.md`. No `data/`, no `extract/data/`, no `extract/lifts/*.Rmd`.
5. `sci4seng/lifts` contains `conf/`, `R/`, `vignettes/`, sample `tools.yml`. No `data/`, no `*.html`.
6. `sci4seng/data` contains `README.md`, `manifest.yml`, `dropzone/`. Nothing else tracked.
7. Each repo carries a root `.gitignore` covering `data/*/`, `__MACOSX/`, `.DS_Store`, `*.zip`, `*.bak`.
8. Each repo carries a root `README.md` explaining purpose + cross-links to siblings.
9. `CLAUDE.md` lives at root of `sci4seng/core`; lifts + data carry stripped variants.
10. No `.git` subdir (from a nested project clone) is tracked anywhere in any repo. `git ls-files --stage | awk '$1==160000'` returns empty in all 3.

## B. SD framework correctness (10)

11. `sd.py` in core defines the 35 named models from pre-split (diapers, brooks, brooksq, bugs, debt, sir, rework, learn, defmap, aiwork, flaky, dora, micro, teamtopo, burnout, aidebt, archpat, congruence, congruence_motif, little, coordn2, entropy, costchange, pareto, linus, mirroring, orgchurn, ownership, ossfail, deprot, scope, ctxswitch, limits, successful, maturity).
12. For each model M, `M().rq()` (single-shot, default bg, seed=1) returns the same `{verdict, gap}` as pre-split `full_audit.csv` cols 2-3.
13. For each M, `M().init` keys + default + lo + hi + units 4-tuples identical to pre-split.
14. `stress(M, n=200, seed=1, dist='triangular')` returns the same `{CONFIRM, REFUTE, neutral}` counts as pre-split `inp_cnt/par_cnt`.
15. `stress_matrix(M, n=200, seed=1)` returns the same 2×2 cell label as pre-split `full_audit.csv` col 9.
16. `rq_n(M, n=100, seed=1, dist='triangular', target='all')` returns the same `{verdict_n, gap_n, sd0_n, sd1_n, eps_n}` as pre-split cols 4-8.
17. `verdict()` threshold formula `max(|y0|*0.05, 0.5)` unchanged.
18. `verdict_n()` uses Cliff's δ + KS + median-ε where ε = 0.35*sd(y0s); thresholds unchanged.
19. `sample(rng, default, lo, hi, dist)` supports `triangular` (peaks at default) + `uniform` (equal weight) and returns same draws given same seed.
20. `run()` with `mode='clip'` vs `mode='reject'` produces same trajectory where reject doesn't trip; trajectory identical to pre-split for default bg.

## C. V&V test bank (10)

21. `tests.py` exposes the 8 structural tests: `boundary_adq`, `anomaly_check`, `extreme_eqn`, `mr_zero_input`, `mr_monotone`, `mr_dt_halving`, `mr_bound_consist`, `mr_scale`.
22. Each structural test returns `{test, status, detail}` per pre-split for every M (regression by row in `full_audit.csv`).
23. `stress_matrix()` majority thresholds (≥50% CONFIRM → axis CONFIRM; ≥20% REFUTE → axis REFUTE) unchanged.
24. `cell_label()` mapping (in both `tests.py` and `full_audit.py`) unchanged. Known off-diagonal label swap between the two files persists (do not "fix" mid-split — propagation risk).
25. `ALL_TESTS` list = 8 items same as pre-split.
26. `boundary_adq` runs at `tmax=80` unchanged.
27. `mr_dt_halving` tolerance `tol=0.10` unchanged.
28. `mr_monotone` grid size `n=5` unchanged.
29. PASS / FAIL / SKIP enum strings unchanged (used by gen_rich.py CSS classes).
30. `ERR:<ExceptionName>` error-format unchanged (used by gen_rich.py + audit_staleness).

## D. Audit pipeline (10)

31. `full_audit.py` produces a 35-row `full_audit.csv`.
32. CSV columns identical to pre-split: `model, verdict, gap, verdict_n, gap_n, sd0_n, sd1_n, eps_n, cell, inp_cnt, par_cnt, boundary_adq, anomaly_check, extreme_eqn, mr_zero_input, mr_monotone, mr_dt_halving, mr_bound_consist, mr_scale`.
33. Every model row's `verdict` field matches pre-split (per-model regression). Diff `full_audit.csv` before/after; expect 0 rows differing.
34. `full_audit.py` wall-clock < 5 s on Mac M-series (was ~3 s).
35. `boundary_check.py` produces same N-row CSV as pre-split, same per-row `status` (in_range / at_boundary / out_of_range).
36. `calibrate.py` produces same 9-model `calibrated_verdicts.csv` with same `(default_verdict, calib_verdict, changes_verdict)` triple per model.
37. `cross_project.py` produces same per-(model, project) metric grid; `lo`/`hi`/`status` cols unchanged.
38. `melt_lifts.py` no-op guard intact: when no `lift_<m>_<p>.csv` files exist in outputs/, do NOT wipe the frozen `lifts.csv`.
39. `melt_lifts.py` with sources: emits same 435-row long-form CSV as pre-split.
40. `scripts/refresh.py` green end-to-end on empty dropzone (re-validates pipeline, prints "No (cell, verdict, verdict_n, gap) changes").

## E. Lift CSVs preserved (10)

41. `lifts.csv` (frozen survivor): 435 rows, byte-identical to pre-split git HEAD.
42. `brooks_tax_median` per project unchanged (F3 11× spread finding intact: 0.029 → 0.311).
43. `brooksq.leak_rate` per project unchanged (F1: hi=0.5 violated on 7/8 projects, range 0.42 → 0.93).
44. `debt.pay_rate_median` per project unchanged (F2: convergent 0.36–0.59 on 5 Java projects).
45. `archpat.{Patterned_n, Legacy_n, Drift_n}` per project unchanged (helix 149/384/0; ambari 381/1890/0).
46. `congruence.{Brokers_n, Clusters_n}` per project unchanged (helix 3/4 from radio-silence).
47. `learn.{Jr_n, Tr_n, Sr_n, train_rate, promote_rate}` per project unchanged.
48. `dora.{batch_size, arrival_rate, rec_rate, cfr}` per project unchanged.
49. `rework.failrate_median` per project unchanged (all 7 lifted < 0.5 thesis trigger).
50. `defmap.tst_proxy` per project unchanged.

## F. Docs site (10)

51. `docs/index.html` exists in `sci4seng/core` and renders at `sci4seng.github.io/core/`.
52. 35 `docs/models/<name>.html` pages exist (one per model).
53. Typology row counts match `full_audit.csv` (currently 23 universal / 7 process-cond / 4 fragile / 1 world-cond).
54. Per-model card CSS class on index.html matches that model's cell label.
55. Per-model scorecard carries the standard 18-row schema (8 structural + 4 effect + 1 cell + 5 data-tier).
56. GitHub Pages build green (no Jekyll errors) for `sci4seng/core` post-split.
57. URL pattern `sci4seng.github.io/core/models/<m>.html` returns 200 for every model.
58. Internal `<a href="...">` links between model pages all resolve (no 404s).
59. Per-model "References" table present + populated for each of the 35 pages.
60. No broken images / missing SVGs (e.g., `grid_aiwork.svg` loads on `aiwork.html` Panel 6).

## G. Scorecard generation (10)

61. `gen_rich.py` reads `full_audit.csv` + `boundary_check.csv` + `calibrated_verdicts.csv` + `lifts.csv`. Pipeline order preserved.
62. `M[]` dict in `gen_rich.py` has 35 entries — same set of model keys as pre-split.
63. `brooks` + `diapers` retain `manual=True` (no auto-overwrite).
64. `render_scorecard_table()` emits 18 rows per page: 8 structural, `rq()`, `rq_n`, `stress(inputs)`, `stress(params)`, `2×2 cell`, then 5 data-tier (`param_plausibility`, `boundary_adq_data`, `calibrated_rq_rerun`, `family_member_coherence`, `behavior_reproduction`).
65. `param_plausibility` row derives counts (in_range / at_boundary / out_of_range) from `boundary_check.csv` per model.
66. `boundary_adq_data` row reports PASS iff any lifted value reaches or exceeds declared `[lo, hi]`; warn iff all strictly inside; N/A iff no lift rows.
67. `calibrated_rq_rerun` reports `N/A · default=<v>; no overridable params` when notes contain "no calib applied" (brooks case).
68. `family_member_coherence` reports `<N> projects lifted` from distinct project count in `lifts.csv` for that model.
69. `behavior_reproduction` = `not run` globally on all 35 pages.
70. `gen_rich.py` runs without error on a cold clone (no stale state needed).

## H. Cross-references (10)

71. `CLAUDE.md` path refs valid post-split (paper/, scripts/, docs/ all resolve in core).
72. `STATE.md` refs resolve in core: `sd.py`, `MODELS_README.md`, `findings.md`, `TIMETABLE.md`, `sanity.md`.
73. `TODO.md` line-anchor refs resolve: `gen_rich.py:727`, `sd.py:789`, `paper/Makefile:96`, etc.
74. Each `vignettes/*.Rmd` in `sci4seng/lifts` `parse_config("../tools.yml")` resolves to `lifts/tools.yml`.
75. Each Rmd `source("functions.R")` resolves to `lifts/R/functions.R` (or wherever R/ lives in lifts repo).
76. SME's kaiaulu PR Rmd paths map cleanly to `sci4seng/lifts/vignettes/<topic>_<method>.Rmd` naming (per TODO SS).
77. Cross-repo refs use stable HTTPS URLs (`https://github.com/sci4seng/core/...`), not relative `../../`.
78. References from docs to lift notebooks updated to point at `sci4seng/lifts/vignettes/...` (was `extract/lifts/...`).
79. Internal anchors (`#typology`, `#findings`, `#models`) resolve on index.html.
80. External DOI / arXiv links unchanged (Sterman, Forrester, Mauerer 2022, Catolino 2019, Tsantalis 2006, etc.).

## I. Build / render (10)

81. `paper/Makefile` targets in core all run: `inference`, `audit-orphans`, `audit-staleness`, `findings`, `render`, `refresh`, `refresh-lifts`, `refresh-dry`, `push`.
82. `audit_staleness.py` exits 0 on green state of refreshed CSVs.
83. `check_pages.py` exits 0 on `docs/models/*.html` post-regen.
84. `make push` gates on both checks (exits non-zero if either fails).
85. `python3 -c "from sd import *"` succeeds in core (all 35 model factories importable).
86. R environment in lifts repo finds `kaiaulu` package (or fails with clear error) when running any vignette.
87. `tools.yml` paths in lifts repo are either: absolute paths user must edit OR `${SCI4SENG_TOOLS}` env-var indirection. Documented in README.
88. `perceval`, `scc`, `RefactoringMiner`, `pattern4.jar`, `Depends` paths all resolve via `tools.yml`.
89. `extract/scripts/szz_pass.py`, `archpat_lift.py`, `parse_pattern4_xml.py` still run (move to `lifts/scripts/`).
90. `scripts/refresh.py --dry-run` prints the correct 8-step pipeline plan without running anything.

## J. Carlos PR hygiene + anonymisation (10)

91. **No `*.html` in `sci4seng/lifts`** (Carlos email 2026-06-01: "html files should be hosted elsewhere, not in PR"). CI rejects PRs that add HTML to lifts.
92. **No `data/` in `sci4seng/lifts`** (Carlos rule: he doesn't host data in kaiaulu).
93. Each vignette Rmd carries a YAML header naming its 1:1 knit-target HTML at the canonical Pages URL.
94. Per-vignette kaiaulu PR contains exactly 3 file types: `conf/<project>.yml`, `R/<helper>.R`, `vignettes/<topic>_<method>.Rmd`.
95. Knit HTML output lands at `sci4seng/core/docs/lifts/<name>.html` (or external Drive folder) — never inside `sci4seng/lifts`.
96. Anonymisation pass on `sci4seng/core` strips identifiers from prose: "Tim", "Carlos", "Rick", "Umar", "Menzies", "Ric" → "Author N" or removed.
97. Anonymisation strips `github.com/timm/*` URLs from `docs/index.html` (GitHub ribbon), `CLAUDE.md`, `TODO.md`, `STATE.md`.
98. Anonymisation zip excludes `diary/` (collaborator emails) + `meta/session_*.md` (Claude chat logs) entirely.
99. Git commit history scrubbed via `git-filter-repo --name-callback` (committer name → "Anonymous") before pushing to anon mirror.
100. Final anonymous.4open.science zip < 100 MB and renders the index page cleanly without identity leaks.

---

## How to run these

- Tests A, F, H, I = shell + curl + git commands. Wrappable as `bash test_split.sh` script.
- Tests B, C, D, E, G = Python regression. Wrap as `pytest test_regression.py` against pinned pre-split `full_audit.csv` + `lifts.csv` etc. snapshots.
- Tests J = manual checklist for the anonymisation pass; some scriptable (no-html-in-lifts CI rule), some review-by-eye.

**Recommended order**:
1. Run A first (layout sanity — fastest to fix if wrong).
2. Run B–G (correctness — these are the regression core).
3. Run H–I (build/links — needs everything else in place).
4. Run J only when ready to submit the anonymous artifact.

**Acceptance**: all 100 pass on a fresh `git clone sci4seng/core && cd core && make refresh && bash test_split.sh && pytest test_regression.py` on a Mac M-series with no pre-existing state.

---

## K. Human-eyeball runbook — step-by-step, what URL, what to look at

Each step: **GO** to URL → **DO** action → **CHECK** what passes / fails.

Replace `<host>` with one of:
- `https://sci4seng.github.io/core/` (live GH Pages, post-split)
- `http://localhost:8000/` (local: `cd docs && python3 -m http.server 8000`)
- For now (pre-split): `https://timm.github.io/icse27theories/`



Not scriptable. Open the deployed GH Pages site (or local preview)
and verify each by reading / clicking / resizing.

### Subgroup K-IDX — Index page (5 min)

**K1.** GO `<host>/`. DO scroll to top. CHECK title "MYTHS" + sub "Models Yielding Testable Hypotheses in Software" both visible without scrolling. **Fail if** either missing or truncated.

**K2.** GO `<host>/`. DO scroll to "Headline findings". CHECK 5 or 6 bullets visible, each starting with a bold "F0 —", "F1 —", etc. **Fail if** lead-ins not bold, or numbering breaks.

**K3.** GO `<host>/#typology`. DO scroll to typology table. CHECK 4 rows (universal / process-cond / fragile / world-cond), counts column right-aligned, total adds to 35. **Fail if** any row missing or count column overflows past table edge.

**K4.** GO `<host>/#models`. DO scroll the model grid. CHECK 35 cards visible (count them — should be one per model). Each card has: model name (top), year in light grey, one-line description, coloured badge (cell name), lift-status text. **Fail if** count ≠ 35 or any card missing any of those 5 fields.

**K5.** GO `<host>/#models`. DO scan card colours. CHECK 4 distinct cell colours used (universal / process / fragile / world). "dark" cards (no lift available) appear dimmed/greyer than "lifted" ones. **Fail if** all cards look identical or "dark" doesn't visually pop.

**K6.** GO `<host>/#models`. DO hover mouse over a card (without clicking). CHECK cursor turns to a pointer, card border darkens OR shadow appears. **Fail if** no visual feedback on hover.

**K7.** GO `<host>/`. DO scroll to the very bottom. CHECK footer reads "Anonymous submission · ICSE 2027" (or your venue/year). For double-blind: there should be NO "Fork me on GitHub" ribbon top-right. **Fail if** ribbon visible OR footer mentions a name.

**K8.** GO `<host>/`. DO click "typology" link in top nav. CHECK page smooth-scrolls to the typology table; URL bar shows `<host>/#typology`. Repeat for "findings" and "models". **Fail if** any nav link doesn't jump.

**K9.** GO `<host>/`. DO resize browser window to ~600 px wide (phone width). CHECK page reflows: cards stack 1-up or 2-up, no horizontal scrollbar. **Fail if** content goes off-screen requiring horizontal scroll.

**K10.** GO `<host>/`. DO right-click → "View page source". CHECK no inline `<style>` blocks (CSS should be in `css/style.css`). No comments leaking timestamps, paths, usernames. **Fail if** source contains "/Users/timm/..." or commit hashes.

### Subgroup K-MOD — Per-model page (10 min — sample 5 models)

For each of these 5 URLs:
- `<host>/models/brooks.html`     (manual=True hand-tuned)
- `<host>/models/archpat.html`    (canonical lift)
- `<host>/models/aiwork.html`     (carries grid figure)
- `<host>/models/congruence_motif.html`  (newest model)
- `<host>/models/diapers.html`    (toy / manual=True)

**K11.** GO each of the 5 URLs. DO let page load. CHECK title + 6-tab switcher (Summary / Model / Lift / Inputs / Scorecard / Results) visible. **Fail if** page errors out, or tab count ≠ 6.

**K12.** GO `<host>/models/brooks.html`. DO click each tab in sequence. CHECK content swaps; URL fragment updates (e.g. `#tab-3`). **Fail if** clicking does nothing OR URL doesn't update.

**K13.** GO `<host>/models/brooks.html`. DO note the cell-typology badge under the title. GO back to `<host>/#models`, find the brooks card. CHECK badge text matches between the two. Repeat for archpat + aiwork. **Fail if** badge ≠ card class on any of the 3.

**K14.** GO `<host>/models/brooks.html`. DO click Tab 3 (Lift). CHECK code blocks render in monospace with a subtle background. No horizontal scroll for normal-length lines. **Fail if** code looks like prose or scrollbars appear.

**K15.** GO `<host>/models/archpat.html`. DO click Tab 5 (Scorecard). CHECK the full 18-row table is visible without horizontal scroll on a desktop (1280+ px wide). **Fail if** table overflows on desktop. Mobile may scroll horizontally — acceptable.

**K16.** GO `<host>/models/archpat.html`. DO scroll the Scorecard table. CHECK the 5 "data-tier" rows (param_plausibility, boundary_adq_data, calibrated_rq_rerun, family_member_coherence, behavior_reproduction) appear visually separated from the upper structural+effect rows (separator row "data-tier checks" in italic). **Fail if** no separator visible.

**K17.** GO `<host>/models/brooks.html`. DO click Tab 6 (References). DO click 3 reference links at random. CHECK each opens a real paper at the DOI/URL shown. **Fail if** any link is dead (404) or wrong paper.

**K18.** GO `<host>/models/brooks.html`. DO read the Summary tab. GO `<host>/models/archpat.html` (auto-gen). DO read its Summary tab. CHECK tone + density similar; brooks shouldn't read as obviously hand-crafted vs the others. **Fail if** brooks reads like a different writer / template.

**K19.** GO `<host>/models/diapers.html`. DO read the Summary tab. CHECK there's a visible "toy / canary / demonstrator" disclaimer near the top. **Fail if** reader could mistake it for a real empirical model.

**K20.** GO `<host>/models/aiwork.html`. DO click Tab 6 (Results). CHECK an SVG figure renders showing a 6×6 grid (red-to-blue colour gradient). Diagonal contour from upper-left (red, negative) to lower-right (blue, positive). 3 circles annotate "GitHub RCT 2024", "GitClear 2024", "METR 2025" with a legend on the right. **Fail if** image broken / missing OR legend not labelled.

### Subgroup K-LIFT — Per-Rmd lift output (5 min)

**K21.** GO `<host>/lifts/lift_archpat_helix.html` (and similar for other knit outputs). CHECK page loads in < 2 s, scrollable. **Fail if** > 5 s or 404.

**K22.** GO each lift HTML. DO scroll to top. CHECK a banner explains methodological substitutions (where any heuristic stands in for a canonical step). **Fail if** results shown without banner naming the substitutions.

**K23.** GO any lift HTML containing figures. DO open the figure in a new tab / zoom in browser. CHECK image at print-readable resolution (axis labels legible, not pixelated). **Fail if** any chart is too low-res to read axis labels.

**K24.** GO any lift HTML. DO scroll to bottom. CHECK download links to the source CSV resolve (click → file downloads or renders). **Fail if** link dead.

### Subgroup K-PR — Carlos PR workflow on kaiaulu (5 min, when submitting)

**K25.** GO `https://github.com/sailuh/kaiaulu/pulls`. DO open your draft PR. DO click "Files changed". CHECK the file list contains ONLY: one `conf/<project>.yml`, one or more `R/*.R`, one `vignettes/<topic>_<method>.Rmd`. NO `*.html`, NO `data/`, NO `.original.md`, NO `meta/`, NO `outputs/`. **Fail if** any HTML or data file in diff.

**K26.** GO same PR page. DO read PR description. CHECK it links the Pages-hosted knit output (e.g. `https://sci4seng.github.io/core/lifts/<name>.html`). DO click the link. **Fail if** missing link OR link 404s.

**K27.** GO same PR's "Files changed" view. DO click the `vignettes/*.Rmd` file. CHECK GitHub renders it as markdown (chunks shown as fenced code, prose readable). DO click a prose line → "+" icon should appear allowing inline comment. **Fail if** Rmd shown as raw text or inline-comment not offered.

**K28.** GO same Rmd in the PR. DO Ctrl-F for `parse_config(`. CHECK the path is `"../tools.yml"` (canonical). DO Ctrl-F for `source(`. CHECK references `"functions.R"` from the same vignettes dir. **Fail if** paths point at /Users/timm/... or other machine-specific paths.

### Subgroup K-ANON — Anonymous-submission preview (5 min, before send)

**K29.** GO local unzipped anon-mirror dir. DO open EVERY `docs/**/*.html` in a browser (or grep entire dir). DO Ctrl-F each of these strings: `Tim`, `Carlos`, `Rick`, `Umar`, `Ric`, `Menzies`, `github.com/timm`. CHECK zero matches across all files. **Fail if** any hit (then re-run the anonymisation pass).

**K30.** GO local anon-mirror `<host>/`. DO scroll to footer. DO open 3 random model pages and scroll to their footers + reference sections. CHECK every footer says "Anonymous submission · <venue> <year>" (your wording). NO stray "Tim et al." in any caption, acknowledgement, or self-citation. **Fail if** any leak.

---

## How to use the K group

Carlos / Rick / reviewers can do these in ~15 minutes once the site
is up. Walk a checklist; mark fail conditions with a screenshot for
the issue tracker. Most failures here are CSS / layout regressions
that scriptable tests A–J would miss because they only check
file-content invariance, not rendered appearance.

Suggested cadence:
- After every `make refresh` that produces visible HTML changes: run K11–K20 against a local preview server (e.g. `python3 -m http.server` from `docs/`).
- After every Pages deploy: run K1–K10 against the live URL.
- Before each kaiaulu PR submit: run K25–K28.
- Before each anonymous-submission send: run K29–K30.
