---
name: lab-research-memo
description: Build, edit, restructure, or review Jupyter notebook research memos for empirical, methodological, and conceptual work, using a fixed eight-section lab convention, pre-commit decision or hypothesis rules, required figures and diagrams, TODO-based honesty markers, and a post-render sanity check. Use this skill for memos, write-ups, technical notes, calibration reports, diagnostics, replications, exploratory analyses, design proposals, and empirical results write-ups — including casual phrasings like "write up these results," "document this analysis," "draft a note on X," or "summarize what we found." Do NOT use this skill for journal papers, slide decks, or commit messages.
---

# Lab Research Memo (agent guide)

You are drafting or reviewing a Jupyter notebook research memo (`memo.ipynb`) for the user and their coauthors. The notebook will later be rendered to HTML via Quarto. Your job is to produce or improve the notebook structure, fill in what you know, and mark everything else with explicit `TODO:` items the user and coauthors will resolve. **You do not render, execute, or commit the notebook** — those are the user's actions.

## Mode selection

First decide which mode the request falls into:

- **Drafting mode** — the user wants a new memo (or a substantial restructure). Follow the **Drafting workflow** below.
- **Review mode** — the user has an existing memo and wants feedback or targeted edits. Follow the **Review workflow** below.

If unclear, default to drafting only when no existing notebook is referenced; otherwise default to review and ask for confirmation.

## Drafting workflow

1. **Identify the memo type** (see "Memo types"). Pick a sensible default if the user is ambiguous; don't block on this.
2. **Use the context you have.** If enough context exists to draft a useful skeleton, proceed and mark missing information with `TODO:`. Ask follow-up questions only when the memo type, intended audience, or deliverable is genuinely ambiguous, and batch at most 3 at once.
3. **Build the notebook** cell by cell per the "Notebook construction" recipe.
4. **Mark every uncertain item with `TODO:`** in visible prose — never invent numbers, paths, or results, and never hide TODOs in code comments.
5. **Run the static pre-handoff self-check** against your draft.
6. **Hand back** to the user using the handoff template at the end of this guide.

## Review workflow

When asked to review an existing memo, do not rewrite by default. Audit first, then propose targeted edits.

1. **Audit** the memo against:
    - Presence and ordering of the eight standard sections.
    - Pre-commit norm (criteria/hypotheses stated before results).
    - At least one figure or diagram.
    - Caption + prose reference for every table, figure, diagram.
    - `TODO:` / `[incomplete, ignore for now]` / placeholder rows correctly used.
    - Reproducibility appendix populated (§§8.1–8.5).
    - Static post-render risks (math delimiters, broken cross-references, naming inconsistencies).
2. **Report findings** as a triaged list: **must-fix** (structural or honesty violations), **should-fix** (clarity, completeness), **nice-to-have** (style polish). For each finding, name the section and one-line description.
3. **Propose targeted edits** only for must-fix items by default. Ask the user before rewriting whole sections.

## Inputs to collect from the user

Use the context already provided whenever possible. Ask only for items that are missing **and** genuinely block the draft. Items below in order of priority:

- **Memo type** (decision / diagnostic / exploratory / method / replication / design proposal / empirical results).
- **Title, ISO date, author list** (the user and coauthors, in display order).
- **One-sentence headline takeaway** — recommendation, finding, or claim.
- **Data and code locations** — project-relative paths to inputs and the upstream script(s).
- **Decision-memo specifics:** Primary metric, Guardrail, Tie-breaker.
- **Empirical-results-memo specifics:** the estimand and identification strategy.
- **Known pending pieces** — convert each into a visible `TODO:` in the relevant section.

If the user is vague on any item, leave it as a `TODO:` rather than guessing.

## Memo types

| Type | Body (Sections 4–7) |
|---|---|
| **Decision memo** | 4. Candidates / 5. Decision rule / 6. Baseline evaluation / 7. Main comparison |
| **Diagnostic memo** | 4. Symptom / 5. Hypotheses / 6. Tests / 7. Resolution |
| **Exploratory memo** | 4. Approach / 5. Findings / 6. Robustness / 7. Open questions |
| **Method / explainer memo** | 4. Setup / 5. Method (with mandatory architecture diagram) / 6. Worked example / 7. Edge cases |
| **Replication / audit memo** | 4. Original claim / 5. Replication setup / 6. Results / 7. Differences |
| **Design proposal memo** | 4. Motivation / 5. Proposed design (with diagram) / 6. Risks / 7. Success criteria |
| **Empirical results memo** | 4. Empirical strategy / 5. Main results / 6. Robustness and diagnostics / 7. Interpretation and limitations |

### Default visuals by memo type

Include at least the visual listed for the memo's type, in addition to any others the analysis warrants:

- **Decision memo** — candidate comparison plot in §6 or §7.
- **Diagnostic memo** — residual, failure-rate, or error-by-bin plot in §6 or §7.
- **Exploratory memo** — sample composition, distribution, or subgroup comparison plot in §5.
- **Method / explainer memo** — architecture or pipeline diagram in §5 (mandatory).
- **Replication / audit memo** — original-vs-replicated estimate plot in §6.
- **Design proposal memo** — proposed-design diagram in §5 (mandatory).
- **Empirical results memo** — coefficient plot, event-study plot, calibration curve, or subgroup-result plot in §5.

## Standard structure (eight sections)

```
1. Executive Summary
   1.1 Key takeaways
   1.2 Section roadmap
2. Data / Inputs
   2.1 Source
   2.2 Sample
   2.3 Covariates / Variables
   2.4 Documentation Link
3. Problem Statement and Baseline
4–7. Body (varies by memo type)
8. Reproducibility Appendix
   8.1 Inputs and upstream generation
   8.2 Outputs written by this memo
   8.3 Configuration knobs
   8.4 Archived artifacts
   8.5 Session info
```

Never rename or reorder these. If a section genuinely does not apply, keep the heading and write one sentence saying so.

## Notebook construction

Build the notebook in this cell order. Default language is Python; if the project uses R, swap to an R kernel and use R equivalents (`library(tidyverse)`, etc.).

1. **Raw cell at top: YAML front matter** for Quarto:

    ```yaml
    ---
    title: "<Memo Title>"
    author:
      - "<User>"
      - "<Coauthor 1>"
    date: "YYYY-MM-DD"
    format:
      html:
        toc: true
        number-sections: true
        code-fold: true
        embed-resources: true
    ---
    ```

2. **Markdown cell: Section 1 (Executive Summary).** Two subsections (1.1 Key takeaways, 1.2 Section roadmap). The last bullet of §1.1 states the concrete takeaway with a number when applicable, or a `TODO:` if pending. Include a scope/limitation sentence in §1.1.

3. **Code cell: imports and paths.** Standard library, project-internal imports, `from pathlib import Path`. Follow with a `# === parameters ===` block listing tunable knobs (thresholds, sample sizes, top-N, candidate list).

4. **Markdown cell: Section 2.1 (Source).** Inline-code each input path. Group paths semantically. Name the upstream script(s). For non-trivial data flow (>2 upstream scripts or branching merges), include a Mermaid pipeline diagram.

5. **Code cell: data loading and join.**

6. **Markdown cell: Section 2.2 (Sample).** State N and relevant subgroup splits. If a transformed target is introduced, define it with `$$...$$` and define every symbol on the following line.

7. **Code cell: sample summary statistics.** Produce `summary_table_2_2` as a DataFrame.

8. **Markdown cell: Section 2.3 (Covariates / Variables).** Group semantically, list each variable as inline `code`.

9. **Markdown cell: Section 2.4 (Documentation Link).** Point to `setup_memo.md`, `PIPELINE.md`, `global_parameters.yaml`, or equivalents.

10. **Markdown cell: Section 3 (Problem Statement and Baseline).** Open with a **bold** one-sentence problem statement. Define every metric on first use. Declare Primary vs. Secondary metrics.

11. **Code cell: baseline metrics and figure(s).** Produce at least one figure or diagram. Save figures to `outputs/figures/<name>.png` and tables to `outputs/tables/<name>.csv` via the helpers `save_figure()` / `save_table()` (or project equivalents).

12. **Markdown + code cells: Sections 4–7 (body).** One Markdown header cell followed by supporting code cells per section. For decision memos: §4 candidates + impact table, §5 decision rule **before any results**, §6 baseline evaluation against the rule, §7 main comparison with same metric family + target-consistency rule. For empirical results memos: §4 strategy and identification **before any estimates**, §5 main results table + coefficient or event-study plot, §6 pre-specified robustness, §7 limitations and scope.

13. **Markdown cell: Section 8 (Reproducibility Appendix §§8.1–8.4).**

14. **Code cell: §8.5 Session info.** Python: `%load_ext watermark` + `%watermark -iv -m`. R kernel: `sessionInfo()`. The cell must execute; never paste static text.

### Output naming convention

Name generated artifacts consistently so prose references stay stable:

- Tables: `summary_table_2_2`, `table_6_primary`, `table_7_main_comparison`.
- Figures: `fig_3_baseline`, `fig_5_main_results`, `fig_7_event_study`.
- Pattern: `<table|fig>_<section>_<short_descriptor>`.
- Save under `outputs/tables/` and `outputs/figures/`.

## Section-content rules

**Section 1 — Executive Summary.** The last bullet of §1.1 is the concrete takeaway; if unfinalized, write `TODO:` naming the specific item. **Include at least one scope or limitation sentence in §1.1.** §1.2 lists every downstream section in one sentence each.

**Section 2 — Data / Inputs.** A collaborator with filesystem access should reconstruct the analysis dataset from this section alone. **When no dataset is involved**, keep all four subsections and use them for conceptual inputs: 2.1 conceptual setup / framework references; 2.2 notation and assumptions; 2.3 variables, parameters, or operators introduced; 2.4 documentation link.

**Section 3 — Problem Statement and Baseline.** Lead with a **bold** plain-language sentence stating the problem before any jargon, then show baseline metrics overall and by relevant subgroups.

**Sections 4–7 — Body.** Pre-commit norm: criteria, hypotheses, identification strategies, or questions stated in early body sections must match what is evaluated in later body sections. **Section 7 (or its equivalent) must include an explicit limitations or scope paragraph** for empirical, replication, and exploratory memos.

**Section 8 — Reproducibility Appendix.** §8.1 names the upstream script for every input file. §8.2 names the output directory and the helpers that write tables and figures. §8.3 points to the `# === parameters ===` block from cell 3. §8.4 points to `archive/legacy_outputs/`. §8.5 is the live `watermark` / `sessionInfo()` cell.

## Style and conventions

- **Voice:** "we." The memo speaks for the project, not for you (the agent) or any single author.
- **Bold for headline claims.** Use `**bold**` for the load-bearing claim of a bullet or the first sentence of a section.
- **Short paragraphs.** Three to five sentences max.
- **Define on first use.** Spell out MAE, R², GOF, ATE, IV, etc., the first time they appear.
- **No marketing words:** avoid "novel," "powerful," "state-of-the-art," "robust" (unless used technically).
- **Math:** inline `$...$`, display `$$...$$`. Never `\(...\)`. Define every symbol on the line following the equation.
- **Tables:**
    - Every table has a caption. If the notebook framework doesn't render native captions for a DataFrame output, place a **bold caption sentence immediately above or below the table** in the surrounding Markdown cell.
    - Numeric precision: **4 decimals for fit metrics, coefficients, standard errors, p-values, and slopes; 2 decimals for percentages; integers for counts.**
- **File paths:** project-relative from the repository root, inline-coded (e.g., `` `data/processed/panel.parquet` ``). Use absolute paths only when the user explicitly provides or requires them.

## Figures and diagrams

**Every memo must include at least one figure or diagram.** See "Default visuals by memo type" above for the minimum expected visual.

- **Plots** — generated in code cells (matplotlib/seaborn/plotly for Python; ggplot2 for R). Reference every plot in prose within a sentence of where it appears. State filters or zoom explicitly in the caption.
- **Diagrams** — use Mermaid (Quarto renders it natively, no plugins). Drop the Mermaid block directly into a Markdown cell:

    ````
    ```{mermaid}
    flowchart LR
        raw[raw_panel.csv] --> clean[clean.py]
        clean --> processed[processed_panel.parquet]
        processed --> model[fit_model.py]
        model --> preds[predictions.parquet]
        preds --> memo[this memo]
    ```
    ````

Standard diagram locations: §2.1 pipeline diagram (>2 upstream scripts); §4 comparison diagram (candidates differ in non-obvious ways); §5 architecture diagram (method memos, mandatory); §3 state/process diagram (multi-stage baseline).

## Honesty conventions

- **Inline `TODO:` for every uncertain item.** Whenever a number, path, or claim is not finalized, write `TODO: <specific item the user needs to provide>` in the visible prose. **Never hide TODOs in code comments**, and never silently omit a pending item.
- **Section-level incomplete markers.** Add `[incomplete, ignore for now]` to any section heading whose conclusions can't yet be trusted.
- **Placeholder rows in tables.** When predictions or results are pending, include a placeholder row with `status = pending` and the expected output path.
- **Explicit scope and limitations.** State scope in **bold** whenever the conclusion is narrower than the reader might assume. **At least one limitation belongs in §1.1, and a limitations subsection in §7 (or equivalent) for empirical, replication, and exploratory memos.**

## Privacy and shareability

Memos are shared with coauthors and sometimes more widely. The agent must not leak data the user wouldn't share verbatim.

- **No row-level identifying data in rendered output.** Prefer aggregate summaries, masked IDs, bucketed values, or small synthetic examples.
- **No raw API keys, credentials, or auth tokens** in any cell.
- **No verbatim restricted-source content** beyond brief, properly attributed quotation. For data with terms of use (proprietary panels, licensed datasets), confirm with the user before showing extracts.
- When unsure whether content is shareable, leave a `TODO:` flagging the question rather than including it.

## Anti-patterns

Do not:

- Bury the takeaway past Section 1.
- Show results before stating the decision rule, hypothesis, or identification strategy.
- Rename or reorder the eight sections.
- Ship a memo with no figures or diagrams when the analysis is non-trivial.
- Omit limitations or scope on empirical work.
- Invent numbers, paths, file names, or results — use `TODO:` instead.
- Hide TODOs in code comments.
- Write tables without captions or figures without prose references.
- Use `\(...\)` for inline math.
- Include row-level identifying data in rendered output.
- Render, execute, or commit the notebook (those are the user's actions).

## Pre-handoff self-check (static)

Run this against the notebook content. **Do not execute code cells, render the notebook, or validate runtime outputs** — verify only what is statically visible in the cells.

- [ ] All eight sections present, in order, with the correct names.
- [ ] Section 1 ends with a concrete takeaway, or a clearly marked `TODO:`.
- [ ] **The headline takeaway in §1.1 is traceable to a downstream table, figure, or `TODO:`** elsewhere in the notebook.
- [ ] At least one scope/limitation sentence appears in §1.1.
- [ ] For empirical, replication, or exploratory memos: a limitations subsection appears in §7 (or equivalent).
- [ ] Pre-commit norm visible: criteria/hypotheses/strategy stated in early body sections match what later body sections evaluate.
- [ ] At least one figure or diagram is included, and matches the memo type's default visual (see table above).
- [ ] Every table, figure, and diagram has a caption (Markdown-prose caption if the framework lacks native captions) and is referenced in surrounding prose.
- [ ] Every symbol in every equation is defined on the following line.
- [ ] Inline math uses `$...$`, display math uses `$$...$$`. No `\(...\)`.
- [ ] All file paths are project-relative from the repo root, inline-coded; no vague paths.
- [ ] No invented numbers, paths, or results — every uncertain item is a visible `TODO:`.
- [ ] §8.5 contains a live `watermark` / `sessionInfo()` cell, not pasted text.
- [ ] Output naming follows the convention (`summary_table_2_2`, `fig_3_baseline`, etc.).
- [ ] No row-level identifying data or credentials in any cell.
- [ ] **Notebook file validity:** the file is well-formed `.ipynb` (valid JSON), all cells have correct `cell_type` set (`markdown`, `code`, `raw`), the YAML front matter sits in a Raw cell at the top, and there are no empty placeholder cells left over.

## Deliverable format

- **If notebook-creation tooling is available**, create a valid `.ipynb` file at the path the user specifies (or `memo.ipynb` in the working directory by default).
- **If notebook-creation tooling is not available**, return the notebook content as an ordered list of cells, each clearly labeled with its type (`# Raw cell`, `# Markdown cell`, `# Code cell`) and the cell body in a fenced block. The user will assemble the notebook.

## Handoff template

When finished, return the notebook (or the labeled cells) and a structured handoff message:

---

> **Memo drafted: `<filename>.ipynb`** (`<memo type>` for `<authors>`)
>
> **Structure:** eight sections built per the lab-memo convention. <One sentence summarizing what each body section will contain.>
>
> **TODOs for you and coauthors to resolve before rendering:**
>
> 1. <Specific TODO 1 — e.g., "§1.1 last bullet: confirm the headline numeric value">
> 2. <Specific TODO 2 — e.g., "§2.1: verify the project-relative path to `data/processed/panel.parquet`">
> 3. <...>
>
> **Next steps:**
>
> 1. Open `<filename>.ipynb` in JupyterLab, VS Code, or Cursor.
> 2. Fill in each `TODO:` listed above.
> 3. Run **Kernel → Restart and Run All** to execute every cell top-to-bottom.
> 4. Render with `quarto render <filename>.ipynb`.
> 5. Open the resulting HTML in a real browser (not the IDE preview) and run the post-render sanity check below.
>
> **Post-render sanity check (run after Quarto renders):**
>
> - **Equations:** every `$...$` rendered as math, not raw text; no stray dollar signs; `\theta`-style backslashes don't leak through; long display equations don't overflow at 1280-px width.
> - **Figures and diagrams:** every figure rendered (no raw Mermaid source visible); no broken-image icons; captions appear under their figures.
> - **Tables:** captions present (either native or as bold-prose above/below); numeric precision correct (4 dp for fit metrics, coefficients, SEs, p-values, slopes; 2 dp for %, integers for counts); wide tables don't blow out the page width.
> - **Rendering integrity:** floating TOC present and clickable; section numbering applied; no raw YAML or cell metadata visible.
> - **Cross-references:** §1.2 roadmap matches actual section content; "see Section N" references resolve correctly; figure/table names cited in prose match the named outputs.
> - **Numerical consistency:** N in §2.2 matches N in §3 / §6 / §7 tables; headline number in §1 matches the corresponding cell downstream; subgroup totals add up; percentages sum to 100% within rounding.
> - **Honesty markers:** every `TODO:` and `[incomplete, ignore for now]` is visible in the rendered HTML; limitations clearly stated.
> - **§8.5:** contains real `watermark` / `sessionInfo()` output, not a placeholder.

---

Stop after the handoff message. Wait for the user's response before doing anything else.
