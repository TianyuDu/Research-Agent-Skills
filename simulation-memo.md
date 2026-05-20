---
name: simulation-memo
description: Build a Jupyter notebook memo specifying a simulation study for the user and their coauthors, following the lab-memo conventions of a fixed eight-section structure, a pre-committed decision rule, mandatory data-generating-process specification, mandatory sanity-check protocol, and a post-render checklist. Use this skill whenever the user asks you to draft, edit, restructure, or review a simulation design, simulation memo, Monte Carlo plan, synthetic-data experiment, calibration study, recovery check, power analysis plan, or any pre-registered numerical experiment. Trigger this even when the user says "write up the simulation we're planning," "document the DGP," "draft a sim study for X," or "write a memo about what the simulation will test." Do NOT use this skill for real-data analyses (use lab-research-memo), journal papers, slide decks, or commit messages.
---

# Simulation Memo (agent guide)

You are drafting a Jupyter notebook simulation memo (`sim_memo.ipynb`) for the user and their coauthors. The notebook will later be rendered to HTML via Quarto. Your job is to produce the notebook structure, specify the simulation in full enough detail that a coauthor can replicate, audit, and interpret it, and mark everything else with explicit `TODO:` items the user and coauthors will resolve. **You do not render, execute, or commit the notebook** — those are the user's actions.

A simulation memo serves three audiences at once: a **replicator** (someone who needs to re-run from scratch), an **auditor** (someone checking the logic for bugs or misspecification), and an **interpreter** (someone reading the results to update their beliefs). Every section must serve at least one of these audiences, and the memo as a whole must serve all three.

## Workflow

Follow these steps in order:

1. **Identify the simulation stage** (see "Simulation stages" below). Ask the user if unclear.
2. **Gather context** by asking the user the questions in "Inputs to collect" below — only those not already obvious from the conversation.
3. **Build the notebook** cell by cell per the "Notebook construction" recipe.
4. **Mark every uncertain item with `TODO:`** — never invent DGP parameters, seed counts, results, or paths.
5. **Run the pre-handoff self-check** against your own draft.
6. **Hand back** to the user using the template at the end of this guide.

## Inputs to collect from the user

Before drafting, ask the user for the items below that aren't already known. Batch the questions — ask at most 4 at once.

- **Simulation stage** (design / pilot / main run / post-mortem).
- **Title.**
- **Last-updated date** (ISO format; defaults to today).
- **Current focus** — one sentence describing what is actively being worked on right now (the part of the memo most likely to change in the next iteration).
- **Linked GitHub issue** — if the conversation references one, capture the issue number and URL. If none is mentioned, omit the status-header field rather than asking.
- **One-sentence research question** the simulation is designed to answer.
- **Pre-committed decision rule** — what outcome will be interpreted as evidence for vs. against the claim. If not yet finalized, ask the user to state the form ("we will conclude X holds if metric M exceeds threshold T").
- **Success criteria for the simulation itself** — the conditions under which the *run* is trustworthy enough that its decision should be believed (sanity checks passed, budget held, no failure-mode flags triggered). Distinct from the decision rule.
- **Estimand** — what quantity the simulation targets, in formal notation.
- **Code locations** — paths to the simulation script(s) and config files.
- **Compute budget** — cluster, hours, expected wall-clock per condition.
- **Known pending pieces** — anything the user already knows should be marked incomplete.

If the user is vague on any item, write it as `TODO:` in the notebook rather than guessing.

## Simulation stages

The same eight-section spine accommodates several stages. Pick one before drafting; it determines the emphasis of the body (Sections 4–7).

| Stage | Body emphasis (Sections 4–7) |
|---|---|
| **Design memo** *(default; written before running)* | 4. DGP / 5. Mechanism / 6. Conditions and metrics / 7. Sanity-check protocol |
| **Pilot memo** *(small-scale exploratory run)* | 4. DGP / 5. Mechanism / 6. Pilot results / 7. Decisions for main run |
| **Main-run memo** *(full execution with conclusions)* | 4. DGP / 5. Mechanism / 6. Conditions and results / 7. Sanity checks and limitations |
| **Post-mortem memo** *(diagnosis of a failed or surprising run)* | 4. DGP as run / 5. Symptom / 6. Hypotheses tested / 7. Resolution |

## Standard structure (eight sections)

Every simulation memo has these eight top-level sections in this order. Sections 1, 2, 3, 8 are the **spine** (always present, same name). Sections 4–7 are the **body** (named per the stage above).

```
1. Executive Summary
   1.1 Key takeaways
   1.2 Section roadmap
2. Setup and Notation
   2.1 Unit space and primitives
   2.2 Estimand
   2.3 Symbol table
   2.4 Documentation link
3. Research Question, Decision Rule, and Success Criteria
   3.1 Research question
   3.2 Decision rule
   3.3 Success criteria for the simulation
4. Data-Generating Process (DGP)
   4.1 Distributional specification
   4.2 Latent / hidden variables
   4.3 Outcome model
   4.4 Known ground truth
5. Mechanism and Estimator(s)
   5.1 Action / policy mechanism
   5.2 Reward or loss
   5.3 Estimator(s) under comparison
   5.4 Hyperparameters
6. Conditions, Metrics, and Results
   6.1 Factors and factorial structure
   6.2 Sample sizes and seeds
   6.3 Primary metric
   6.4 Secondary and diagnostic metrics
   6.5 Results (for pilot / main / post-mortem stages)
7. Sanity Checks, Validation, and Limitations
   7.1 Null and placebo conditions
   7.2 Recovery check on known truth
   7.3 Failure-mode probes
   7.4 Limitations and threats to validity
8. Reproducibility Appendix
   8.1 Code entry point and commit
   8.2 Compute environment
   8.3 Random seeds and seed protocol
   8.4 Outputs written by this memo
   8.5 Configuration knobs
   8.6 Session info
```

Never rename or reorder these. If a subsection genuinely does not apply (e.g., §4.2 latent variables for a fully observed DGP), keep the heading and write one sentence saying so.

## Notebook construction

Build the notebook in this cell order:

1. **Raw cell (very top): YAML front matter** for Quarto. Fill in `title` and `date`. Use:

    ```yaml
    ---
    title: "<Memo Title>"
    date: "YYYY-MM-DD"
    format:
      html:
        toc: true
        number-sections: true
        code-fold: true
        embed-resources: true
    ---
    ```

2. **Markdown cell: Status header.** A short, visible block at the very top of the rendered memo (after the YAML, before §1) capturing the live state of the memo. Use a Quarto callout for visibility:

    ```
    ::: {.callout-note title="Status"}
    - **Stage:** <design / pilot / main run / post-mortem>
    - **Last updated:** YYYY-MM-DD
    - **GitHub issue:** [#<n>](<url>)   <!-- omit this line if no issue -->
    - **Currently:** <one sentence on what is actively being worked on>
    :::
    ```

    Rules:
    - The status header is metadata about the memo, not content; it is **not** numbered.
    - The `Currently` line is the most volatile field and will change between iterations — keep it to one sentence.
    - Include the GitHub issue line only if the user has referenced one in the conversation. Do not invent or guess an issue link.
    - The `Last updated` field is what readers check to decide whether the memo is stale; update it on every meaningful edit.

3. **Markdown cell: Section 1 (Executive Summary).** Two subsections (1.1 Key takeaways, 1.2 Section roadmap). Key takeaways are 4–5 bullets, each opening with a **bold** lead claim. At minimum: (i) the question, (ii) the pre-committed decision rule, (iii) the success criteria for the run itself, (iv) the headline result or expected result with a `TODO:` if pending — explicitly noting whether success criteria were met, (v) the dominant limitation. Roadmap is one bullet per downstream section, each beginning with **bold** "Section N".

4. **Code cell: imports and paths.** Standard library, project-internal imports, and `from pathlib import Path` for path objects. Then a `# === parameters ===` block with the tunable knobs of the simulation (DGP parameters, seed range, factor levels, sample sizes). This is what §8.5 will point to.

5. **Markdown cell: Section 2.1 (Unit space and primitives).** Define every primitive object the simulation manipulates. Each definition uses display math `$$ \ldots $$` followed by a one-line symbol gloss on the next line. Cover at minimum:
    - the unit space (the type of object being simulated, e.g., $\mathcal{X}$ for a unit space of feature vectors, time series, text, images, graphs);
    - the index set or sample dimensions (number of units $n$, time horizon $T_{\max}$ if any, number of seeds, number of replications per seed);
    - every random variable and its role (observed / latent / counterfactual);
    - any auxiliary spaces the mechanism touches (action space, treatment space, message space — name whichever apply).

    Pick symbols consistent with the project's prior notation. If notation has not been fixed, propose and explicitly mark choices as `TODO: confirm symbol choice` so coauthors can vote.

6. **Markdown cell: Section 2.2 (Estimand).** State the target estimand in display math. Define every symbol on the line following the equation. State whether the estimand is point-identified, partially identified, or only well-defined under the simulation's known truth. Tie back to §2.1: every symbol that appears here must already exist in §2.1.

7. **Markdown cell: Section 2.3 (Symbol table).** A three-column table mapping every symbol used downstream to its (a) definition, (b) type/space, (c) source — whether observed, latent, parameter, hyperparameter, or known truth. This is non-negotiable for simulation memos — coauthors will reference it constantly.

8. **Markdown cell: Section 2.4 (Documentation link).** Point to upstream config files, related memos, GitHub issue threads, and the relevant section of the project's main writeup.

9. **Markdown cell: Section 3 (Research Question, Decision Rule, and Success Criteria).** Three subsections.

    - **§3.1 Research question.** Open with a **bold** one-sentence question.

    - **§3.2 Decision rule.** State the pre-committed decision rule. Use the schema $(M, T, D, S, C)$ with each symbol defined immediately:
      - $M$ — the realized value of the primary metric on the primary condition (a scalar);
      - $T$ — the pre-committed threshold;
      - $D \in \{\uparrow, \downarrow\}$ — the pre-committed direction (does crossing mean exceeding or falling below $T$);
      - $S$ — the maximum acceptable seed-level standard error of $M$;
      - $C$ — the conclusion drawn if the criterion is met.

      The rule:

      $$
      \text{If } \bigl[(D = \uparrow \wedge M > T) \vee (D = \downarrow \wedge M < T)\bigr] \wedge \widehat{\mathrm{SE}}(M) \leq S, \text{ then conclude } C; \text{ else conclude } \neg C.
      $$

      This rule is committed before any results are generated. If any of $M$, $T$, $D$, $S$, $C$ is not yet finalized, write `TODO:` with the specific item. (Note: $T$ here is the threshold, distinct from any time horizon $T_{\max}$ in §2.1.)

    - **§3.3 Success criteria for the simulation.** State the conditions under which the *run itself* is trustworthy. The decision rule is invoked only if all success criteria are met; otherwise the run is rejected and the decision rule does not apply. Use this exact form:

      > **Success criteria.** The simulation is considered successful and its decision rule applicable iff all of the following hold:
      > 1. **Sanity checks pass.** Null/placebo metric values lie within their pre-stated tolerance (§7.1); recovery-check estimate lies within its pre-stated tolerance of the planted truth (§7.2).
      > 2. **No failure-mode flags triggered.** Diagnostic statistics for each anticipated failure mode (§7.3) stay below their pre-stated thresholds across all seeds.
      > 3. **Budget and coverage held.** All planned conditions ran; seed count met the pre-committed target; no condition was silently dropped.
      > 4. **Numerical integrity.** No `NaN`/`Inf` in primary or secondary metrics; seed-level dispersion is finite.

      Each criterion has a pre-stated threshold. Distinguish three outcomes: **pass** (apply decision rule), **conditional pass** (apply decision rule with stated caveat), **fail** (do not apply decision rule; diagnose and revise the design).

10. **Markdown cells + code cells: Section 4 (DGP).** This is the load-bearing technical section. Specify the entire data-generating process at a level of detail sufficient for an independent re-implementation. Build:

    - **§4.1 Distributional specification.** Every random variable gets a distribution in display math; spell out the family and parameters (Gaussian with explicit mean and covariance, mixture with explicit component weights, empirical bootstrap from a named dataset $\mathcal{D}$, etc.). For sequence-, panel-, or graph-structured data, also specify dependency structure (Markov order, autocorrelation, edge formation rule).

    - **§4.2 Latent / hidden variables and observability.** Explicitly state, for every variable defined in §4.1, which components of the system observe it: the mechanism (§5.1), the estimator (§5.3), the evaluator (§7), the analyst (when interpreting). An observability matrix (rows = variables, columns = components, cells = observed/hidden) is the cleanest format.

    - **§4.3 Outcome model.** Specify the outcome equation in display math (e.g., $Y = f(\cdot) + \varepsilon$) with the functional form of $f$ and the noise distribution $P_\varepsilon$ fully explicit. State the support and any monotonicity, sparsity, or smoothness assumptions.

    - **§4.4 Known ground truth.** State the quantity the simulation knows that estimators do not — this is the point of simulating. Examples: the true value of the estimand from §2.2; a planted effect size; a known oracle baseline. This subsection is what makes the recovery check in §7.2 possible.

    §4 must include a DGP diagram (see "Figures and diagrams" below): a DAG or graphical model showing variable dependencies and observability.

11. **Markdown cells + code cells: Section 5 (Mechanism and Estimator(s)).** Specify:
    - **§5.1 Mechanism.** The process that maps inputs to actions, treatments, decisions, or predictions inside the simulation. Specify parameterization, what is frozen vs. trainable, and the rule that maps mechanism outputs to whatever quantity feeds into the outcome model.
    - **§5.2 Reward / loss / objective.** Exact functional form, in display math, with every symbol defined.
    - **§5.3 Estimator(s) under comparison.** List each by a short distinguishing name and give the full update or fitting rule in display math. Treat baselines as estimators in this list — never relegate them to a footnote.
    - **§5.4 Hyperparameters.** A table with one row per estimator and one column per knob actually used by the code. No curated subsets.

12. **Markdown cells + code cells: Section 6 (Conditions, Metrics, Results).**
    - **§6.1 Factors and factorial structure.** List factors and their levels; state whether the design is full-grid, fractional, or one-at-a-time.
    - **§6.2 Sample sizes and seeds.** Number of units $n$ per condition, number of seeds per condition, and total compute budget. State the master seed list explicitly (e.g., `seeds = [0, 1, ..., 19]`).
    - **§6.3 Primary metric.** One metric, defined formally, tied directly to $M$ in the §3.2 decision rule.
    - **§6.4 Secondary and diagnostic metrics.** Each defined; clearly labeled as not load-bearing for the decision.
    - **§6.5 Results.** Pilot/main/post-mortem stages only. Tables of mean ± seed SE per condition. Bold the row corresponding to the primary condition.

    For design-stage memos, §6.5 is replaced by a single sentence: "*Results will populate this section after the run; no result is shown here.*"

13. **Markdown cells + code cells: Section 7 (Sanity Checks, Validation, Limitations).**
    - **§7.1 Null and placebo conditions.** Explicit specification of each, with the metric value expected under the null and the tolerance band around it.
    - **§7.2 Recovery check on known truth.** When ground truth from §4.4 is planted, does the estimator recover it within tolerance?
    - **§7.3 Failure-mode probes.** For each anticipated failure mode, name an explicit statistic that flags it and the threshold beyond which the flag fires. Examples of failure modes vary by application: degeneracy of the optimizer, collapse onto a trivial solution, divergence, distribution shift between train and eval.
    - **§7.4 Limitations.** Where the simulation departs from the real system, stationarity / i.i.d. assumptions, the scope of the conclusions, and anticipated objections with how the design does or does not address each.

14. **Markdown cell + code cell: Section 8 (Reproducibility Appendix).** Six subsections.
    - §8.1: code entry point (script path), commit hash, branch.
    - §8.2: compute environment (cluster, GPU type, container or uv lockfile path).
    - §8.3: random seed list and the seed-control protocol (what gets reseeded across replications: data draw, init, rollout sampling).
    - §8.4: output schema — file paths, columns, units, written by which script.
    - §8.5: pointer to the `# === parameters ===` block from the imports cell (cell 4 in the construction recipe).
    - §8.6: live `%load_ext watermark; %watermark -iv -m` cell. Never paste static text.

## Section-content rules

These rules apply regardless of stage.

**Section 1 — Executive Summary.** The decision rule (second bullet) and the headline result or expected result (third bullet) are the load-bearing ones. The third bullet is `TODO:` for design memos and a concrete number for pilot/main/post-mortem memos.

**Section 2 — Setup and Notation.** Goal: a coauthor can read §2 in isolation and know what every symbol in the rest of the memo means. The symbol table (§2.3) is non-negotiable.

**Section 3 — Research Question, Decision Rule, and Success Criteria.** The decision rule (§3.2) must be stated in the exact `(M, T, D, S, C)` form. If the user resists pre-committing, push back once with the reminder that post-hoc rules are not pre-committed rules. The success criteria (§3.3) are separate: they govern whether the *run* is trustworthy enough that the decision rule should be applied at all. A run that fails its success criteria does not produce a decision — it produces a diagnosis. Every success criterion must have a pre-stated threshold, and each threshold must be the same one used downstream in §7.

**Section 4 — DGP.** Goal: a coauthor with no other context could re-implement the DGP from §4 alone. Every distribution explicit. Every dependency in §4.1 reflected in the §4 diagram.

**Section 5 — Mechanism and Estimators.** State estimator names before update rules. The hyperparameter table in §5.4 must list every knob actually used in the code — not a curated subset.

**Section 6 — Conditions, Metrics, Results.** The primary metric is single. If you find yourself listing two primary metrics, one of them is secondary and the user has not yet decided which. Mark with `TODO:`. Pre-commit norm: the primary metric and threshold in §6.3 must match the decision rule in §3.

**Section 7 — Sanity Checks.** Each sanity check in §7.1 and §7.2 has an expected value under the null or known truth, stated *before* showing results. A sanity check without a pre-stated expected value is not a sanity check. The tolerances around expected values are the same thresholds referenced in §3.3 success criteria — keep these numerically consistent.

**Section 8 — Reproducibility Appendix.** §8.3 is critical for simulations specifically: state the master seed list (e.g., `seeds = [0, 1, ..., 19]`) and which subsystems consume which seeds. Mis-seeded simulations are the most common irreproducibility bug.

## Style and conventions

- **Voice:** "we." The memo speaks for the project, not for you (the agent) and not for any single author.
- **Bold for headline claims.** Use `**bold**` for the load-bearing claim of a bullet or the first sentence of a section. Don't bold for emphasis-of-tone.
- **Short paragraphs.** Three to five sentences max.
- **Define on first use.** Spell out every acronym (DGP, SE, MSE, etc.) the first time it appears.
- **No marketing words:** avoid "novel," "powerful," "state-of-the-art," "robust" (unless used technically).
- **Math:** inline `$...$`, display `$$...$$`. Never `\(...\)`. Define every symbol on the line following the equation. Use the symbol table in §2.3 as the canonical reference.
- **Notation discipline.** Audit every memo for symbol consistency before handoff. No symbol may carry two meanings in the same memo. Common collisions to check: $T$ (threshold vs. time horizon), $n$ (sample size vs. iteration index), $\sigma$ (noise scale vs. policy/strategy), $\theta$ (parameter vs. angle), $\pi$ (policy vs. constant), $D$ (direction vs. dataset $\mathcal{D}$), $M$ (metric vs. number of rollouts/replicates). When a collision is unavoidable, disambiguate by font (italic vs. calligraphic vs. blackboard-bold) and call out the collision in §2.3.
- **Tables:** every table has a caption. Numeric precision: 4 decimals for metric values, 2 decimals for percentages, integers for counts and seeds.
- **File paths:** always full and inline-coded. Never vague ("in the configs folder").

## Figures and diagrams

**Every simulation memo must include at least two visuals: a DGP diagram (§4) and at least one figure summarizing planned or observed metrics.** If the analysis is non-trivial and you can't justify a visual, you don't yet understand the simulation well enough to write the memo.

Two kinds of visuals:

- **Plots** — generated in code cells via matplotlib/seaborn/plotly. For design memos these can be schematic (e.g., a hypothetical curve showing the shape of the expected result with a `TODO:` caption noting it's illustrative). For pilot/main memos these are real results. Reference every plot in prose within a sentence of where it appears.
- **Diagrams** — DAGs, pipelines, architectures. Use Mermaid (Quarto renders it natively). Drop the Mermaid block directly into a Markdown cell:

    ````
    ```{mermaid}
    flowchart LR
        X["X ~ P_X (observed)"] --> M[mechanism]
        U["U (latent)"] --> Y[outcome Y]
        M --> A[action / decision]
        A --> Y
        Y --> Metric[primary metric M]
    ```
    ````

    Use a consistent visual convention: solid nodes for observed variables, dashed nodes for latent variables, and a separate legend if multiple components have different observability.

Standard diagram locations:
- **§4** — DGP / dependency diagram (**mandatory**).
- **§5** — architecture or computation-graph diagram for non-trivial mechanisms.
- **§7** — failure-mode probe diagram when multiple modes interact.

Every diagram has a caption and is referenced in prose.

## Honesty conventions

- **`TODO:` inline.** Whenever a parameter, threshold, path, or result is not finalized, write `TODO: <specific item the user needs to provide>` in the prose. Don't hide TODOs in comments; they must be visible in the rendered HTML.
- **Stage-appropriate placeholders.** For design memos, results in §6.5 are not TODOs — they are correctly absent. Write the sentence "*Results will populate this section after the run; no result is shown here.*" rather than a TODO.
- **Section-level incomplete markers.** Add `[incomplete, ignore for now]` to any section heading whose conclusions can't yet be trusted.
- **Explicit scope.** When the simulation's conclusion is narrower than a reader might assume (e.g., valid only under the synthetic DGP, only at small $n$, only for the planted parameter region), state the scope in **bold** in §1 and again in §7.4.
- **No invented numbers.** Especially: do not invent DGP parameters, threshold values, seed counts, sample sizes, or compute budgets. Use `TODO:` instead.

## Anti-patterns

Do not:

- Bury the decision rule past Section 3.
- Conflate the decision rule (what the simulation concludes) with the success criteria (whether the simulation is trustworthy).
- Apply the decision rule when the success criteria failed.
- State success criteria without pre-stated thresholds.
- Show results before stating the decision rule.
- Specify the DGP only verbally; every distribution must be in display math.
- Omit the symbol table in §2.3.
- Ship a memo with no DGP diagram in §4.
- Conflate the primary metric with secondary metrics.
- Skip sanity checks because the result "looks right."
- State a sanity check without a pre-stated expected value.
- Rename or reorder the eight sections.
- Omit the status header or leave the `Last updated` field stale.
- Invent a GitHub issue link not referenced by the user.
- Let a single symbol carry two meanings within the memo.
- Invent numbers, paths, or results. Use `TODO:` instead.
- Write tables without captions or figures without prose references.
- Use `\(...\)` for inline math.
- Render, execute, or commit the notebook. Those are the user's actions.

## Pre-handoff self-check

Run this against your draft before handing back to the user. Every item must pass or be explicitly noted in the handoff.

- [ ] Status header is present immediately after the YAML, with stage, last-updated date, and currently-working-on line.
- [ ] GitHub issue link is included in the status header iff the user referenced one; not invented if absent.
- [ ] Section 1 ends with a concrete headline (for pilot/main stages) or a clearly marked `TODO:` (for design stage).
- [ ] All eight sections present, in order, with the correct names; all subsections present.
- [ ] §2.3 symbol table is populated with definition, type/space, and source columns; every symbol used downstream is in it.
- [ ] Notation discipline check: no symbol carries two meanings; common collisions ($T$, $n$, $\sigma$, $\theta$, $\pi$, $D$, $M$) audited.
- [ ] §3.2 decision rule is in the $(M, T, D, S, C)$ form with each component defined, with TODOs marked for any unfinalized component.
- [ ] §3.3 success criteria are stated with pre-committed thresholds for sanity checks, failure-mode flags, budget/coverage, and numerical integrity.
- [ ] Success-criteria thresholds in §3.3 numerically match the expected values and tolerances used in §7.
- [ ] §4 specifies every random variable's distribution in display math.
- [ ] §4.2 observability matrix specifies which components see which variables.
- [ ] §4 includes a DGP diagram with caption and prose reference.
- [ ] §5.4 hyperparameter table lists every knob actually used by the code.
- [ ] §6.3 primary metric matches §3.2 decision rule's metric $M$.
- [ ] §7.1 and §7.2 sanity checks each have a pre-stated expected value under null / known truth.
- [ ] §8.3 lists the master seed list and which subsystems consume which seeds.
- [ ] Pre-commit norm held: the metric, threshold, and decision criteria in §3 match what is evaluated in §6 and validated in §7.
- [ ] At least two visuals are included (DGP diagram + one metric figure).
- [ ] Every table, figure, and diagram has a caption and is referenced in prose.
- [ ] Every symbol in every equation is defined on the following line or in §2.3.
- [ ] Inline math uses `$...$`, display math uses `$$...$$`. No `\(...\)`.
- [ ] All file paths are full and inline-coded; nothing vague.
- [ ] Every uncertain item is marked `TODO:` with a specific ask of the user.
- [ ] §8.6 contains a live `watermark` cell, not a paste.
- [ ] No invented DGP parameters, thresholds, seeds, or results.

## Handoff template

When you finish drafting, return the notebook and a structured handoff message to the user. Use this template:

---

> **Simulation memo drafted: `<filename>.ipynb`** (`<stage>` memo)
>
> **Structure:** status header at top (stage, last updated, GitHub issue if any, current focus), then eight sections per the simulation-memo convention. Setup and notation in §2; question, decision rule, and success criteria in §3; DGP in §4 (the load-bearing technical section); mechanism and estimators in §5; conditions, metrics, and (planned/observed) results in §6; sanity-check protocol in §7; reproducibility appendix in §8.
>
> **TODOs for you and coauthors to resolve before rendering:**
>
> 1. <Specific TODO 1 — e.g., "§3 decision rule: confirm the threshold $T$ on the primary metric">
> 2. <Specific TODO 2 — e.g., "§4.1: specify the distribution of $X^*$ — currently marked TODO">
> 3. <Specific TODO 3 — e.g., "§5.4: confirm KL coefficient $\beta$ range">
> 4. <...>
>
> **Next steps:**
>
> 1. Open `<filename>.ipynb` in JupyterLab/VS Code/Cursor.
> 2. Fill in each `TODO:` listed above.
> 3. Confirm the pre-committed decision rule in §3 with all coauthors *before* any runs are launched.
> 4. Run **Kernel → Restart and Run All** to execute every cell top-to-bottom.
> 5. Render with `quarto render <filename>.ipynb`.
> 6. Open the resulting HTML in a real browser (not the IDE preview) and run the post-render sanity check below.
>
> **Post-render sanity check (you run this, after Quarto renders):**
>
> - **Status header:** rendered as a visible callout at the top with current stage, last-updated date, and current-focus line; GitHub issue link clickable if present.
> - **Equations:** every `$...$` rendered as math, not raw text; no stray dollar signs; no `\theta`-style backslashes leaking through; long display equations in §4 and §5 don't overflow at 1280-px width.
> - **DGP diagram:** §4 diagram rendered (no raw Mermaid source visible); dependency arrows are correct; hidden vs. observed variables are visually distinguishable.
> - **Symbol table:** §2.3 table renders with all three columns (definition, type/space, source); every symbol used in §4–§7 appears in it.
> - **Notation collisions:** no symbol carries two meanings; the common collision shortlist ($T$, $n$, $\sigma$, $\theta$, $\pi$, $D$, $M$) has been audited.
> - **Hyperparameter table:** §5.4 lists every knob the code actually uses; no orphan rows; no missing rows.
> - **Decision rule and success criteria:** §3.2 decision rule, §6.3 primary metric, and §1.1 second-bullet all reference the same metric and threshold; §3.3 success-criteria thresholds match the expected values and tolerances in §7.1/§7.2/§7.3.
> - **Sanity-check expected values:** §7.1 and §7.2 each show a pre-stated expected value before any observed value.
> - **Tables:** captions present; numeric precision correct (4 dp for metrics, 2 dp for %, integers for counts).
> - **Rendering integrity:** floating TOC present and clickable; section numbering applied; no raw YAML or cell metadata visible.
> - **Cross-references:** §1.2 roadmap matches actual section content; any "see Section N" references resolve correctly.
> - **Honesty markers:** every `TODO:` and `[incomplete, ignore for now]` is visible in the rendered HTML, not hidden by the toolchain.
> - **§8.6:** contains real `watermark` output, not a placeholder.

---

Stop after the handoff message. Wait for the user's response before doing anything else.
