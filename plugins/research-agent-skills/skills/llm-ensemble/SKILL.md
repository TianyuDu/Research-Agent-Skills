---
name: llm-ensemble
description: Dispatch the same question or task to several LLMs in parallel — external CLIs (Codex/GPT, Gemini, Kimi, …) plus native Claude voices (Fable 5, Opus 5) — each blind to the others, then aggregate their answers into a verified consensus report — agreements, majority views, adjudicated disputes, and rejected hallucinations. Use when the user wants a second opinion, cross-model code review, an ensemble answer, consensus across models, or to cross-check a claim, design, or diff to reduce single-model hallucination risk.
---

# LLM Ensemble

Ask several independent frontier models the same question, blind to each other, then synthesize their answers. You are the orchestrator **and the adjudicator**, not a vote counter: agreement raises confidence, but your own verification is what earns a claim its place in the final report.

Inputs: the task (a question, a claim to check, a design decision, or a code-review request) and, optionally, a roster spec ("only sol, agy-pro, and kimi", "add cursor", "swap agy-flash for 3.5-flash", "use k3-256k").

## Step 1 — Assemble the roster

1. Probe the external CLIs: `command -v codex agy kimi cursor-agent qwen grok copilot` (kimi may live at `~/.kimi-code/bin/kimi` instead of PATH — roster.md has the probe).
2. Default roster (7 voices): **codex ×2 (`gpt-5.6-sol`, `gpt-5.6-luna`), agy ×2 (`gemini-3.1-pro-high`, `gemini-3.6-flash-high` — Antigravity, Google), and kimi (`k3`)** per roster.md, **plus two native Claude members — Fable 5 and Opus 5 — dispatched without any CLI**, since this skill runs inside Claude Code (see Step 3). Other installed CLIs (cursor-agent, legacy gemini-cli, …) join on request.
3. Cross-vendor diversity decorrelates errors far better than multiple versions of one vendor's model. The user can still request version variants — each (CLI, model) pair is its own member: e.g. agy with two different `--model` values is two members.
4. If fewer than 2 non-Claude vendors are usable, see Failure handling before proceeding.
5. Read `roster.md` in this skill's directory for the exact dispatch template, read-only rule, and auth check per member **before invoking anything** — do not improvise flags.

## Step 2 — Build one canonical prompt

**First create the scratchpad**, once, and keep the path it prints:

```bash
SP="$(mktemp -d "${TMPDIR:-/tmp}/llm-ensemble-XXXXXX")"; mkdir -p "$SP/ask" "$SP/out"; echo "$SP"
```

Two rules that follow from it, and both are load-bearing:

- **Substitute that absolute path into every later command.** Each Bash call is a fresh shell that remembers nothing, so a bare `$SP` in a dispatch command expands to nothing and the command writes to `/` and dies. Paths in this skill and in roster.md are written as `$SP/…` for readability — expand them before running.
- **`ask/` and `out/` stay separate.** The prompt lives in `$SP/ask/prompt.md`; every member's answer lands in `$SP/out/`. A member may be pointed at `ask/`, never at `out/` or at `$SP` itself (Step 3).

One self-contained prompt, used **verbatim by every member**. Never tailor it per member (breaks comparability) and never include one member's answer in another's prompt (breaks independence).

Write it once to `$SP/ask/prompt.md`, containing:

1. **The task**, restated precisely.
2. **Context.** For code review: embed the `git diff` (or the relevant file contents) when it is ≤ ~2,000 lines; otherwise list the files/paths to read — CLI members run in the working directory and can read the repo themselves (API-fallback members cannot; see roster.md).
3. **The output contract**, appended verbatim. For questions / claims / design decisions:

   ~~~
   End your response with a fenced ```json block:
   {"answer": "<summary in at most 3 sentences>",
    "claims": [{"claim": "<one falsifiable statement>", "confidence": "high|medium|low"}],
    "caveats": ["<known limitations of your answer>"]}
   ~~~

   For code review:

   ~~~
   End your response with a fenced ```json block:
   {"overall": "<one-line verdict>",
    "findings": [{"file": "<path>", "line": <n>, "severity": "critical|major|minor",
                  "confidence": "high|medium|low", "issue": "<one sentence>"}]}
   ~~~

## Step 3 — Dispatch: parallel, blind, read-only

- **Launch the whole roster in one turn, in parallel.** Issue every dispatch together in a single message — one background task per member — and only then start waiting. Never serialize (launching a member only after another finishes): wall-clock must be ≈ the slowest member, not the sum.
- **External members**: dispatch each via its roster.md template, wrapped in `timeout` (600s default; 1200s for large reviews), writing to its own files: answer to `$SP/out/<member>.out`, stderr to `$SP/out/<member>.log`. Two equivalent parallel vehicles: a **background Bash command** per member (default — zero token overhead), or a **thin executor subagent** per member (background Agent per member; prefer for big rosters, or to keep member outputs out of your context until aggregation). An executor subagent runs the roster.md command **unchanged** and returns only a one-line status — never rewriting the canonical prompt, adding context, or summarizing the output (contract in roster.md § Executor subagents).
- **Claude members need no CLI** — this skill runs inside Claude Code. The orchestrator's own blind answer is one Claude voice. For each *other* requested Claude model (default: whichever of Fable 5 / Opus 5 the session isn't already running), spawn a background **general-purpose subagent with the matching `model` override**, whose prompt is exactly the canonical prompt + the output contract + a read-only instruction — nothing else. Details and the exact prompt shape: roster.md § claude.
- **Read-only is mandatory.** Use exactly the roster.md read-only flag per CLI (codex `--sandbox read-only`, agy `--mode plan`, …); never pass yolo/force/bypass flags; instruct Claude subagents not to create or modify files. No member may modify the repo.
- **Blindness has to survive the fast members finishing first.** Answers accumulate in `$SP/out/` while slower members are still reasoning, so no member may be given a working root that is, or sits above, `$SP/out` — and no prompt may name that directory. Concretely: codex's `-C` points at `$SP/ask`, never at `$SP`. A member that reads another's answer manufactures agreement that Step 4 would then score as independent convergence.
- **You are a member too, and you must be equally blind.** Immediately after dispatching, write your own complete answer to the canonical prompt and commit it to `$SP/out/claude-<model>.out` — strictly **before** reading any member's output or any subagent's returned result (background results may arrive while you write; do not look until yours is committed).
- When all members finish or time out, collect the outputs. A member that errored, died, or timed out is reported in the ensemble log — never silently dropped.

## Step 4 — Aggregate into a consensus map

1. Parse each output — the trailing JSON block when present, the prose otherwise.
2. **Cluster** equivalent claims/findings across members (same file + same underlying issue = same finding, even if line numbers differ slightly).
3. **Vote** each cluster: `Unanimous` / `Majority k/n` / `Split` / `Unique (member)`. Same-vendor members correlate (sol+luna, the two agy models, and fable+opus each share a vendor) — when a verdict hinges on the count, give the vendor-distinct tally too ("5/7 members, 3/4 vendors"). An **intra-vendor split** (sol vs luna, agy-pro vs agy-flash, fable vs opus) is a strong ambiguity signal: the same lineage disagreeing with itself usually means the question is genuinely underdetermined or context-dependent — say so in the verdict rather than forcing a side.
4. **Adjudicate — the anti-hallucination step.** For every load-bearing claim that can be checked, check it yourself (read the code, run the command, consult the doc):
   - Consensus is a prior, not a verdict. Members share training data and can be confidently wrong together — even a unanimous claim gets verified when verification is cheap and the answer depends on it.
   - Unique claims are the most valuable or the most hallucinated — always verify before promoting or discarding one.
   - Where members disagree, rule on it and show the evidence for your ruling.
5. Label every cluster: **Verified ✓** / **Refuted ✗** / **Unverifiable — vote stands (k/n)**.

## Step 5 — Deliver the consensus report

~~~
## Bottom line
<2–3 sentences: the answer or verdict, and how strong the consensus behind it is>

## Consensus map
| # | Claim / finding | sol | luna | agy-pro | agy-flash | kimi | fable | opus | Verdict |
|---|-----------------|-----|------|---------|-----------|------|-------|------|---------|
| 1 | <claim>         | ✅ high | ✅ | ✅ med  | ✅        | ✅   | ✅    | ✅   | Unanimous · Verified ✓ |
| 2 | <claim>         | ✅  | ❌   | ✅      | ❌        | ✅   | ❌    | ❌   | Split 3/7 (2/4 vendors) · Refuted ✗ — <one-line evidence> |

## Agreed and verified — act on these
## Disputed — adjudicated
<each: the positions, your ruling, the evidence>

## Unique insights (verified)
<single-member catches that survived verification, credited to the member>

## Rejected
<claims that failed verification — one line each on why; this is the hallucination filter working>

## Ensemble log
| Member | Model | Time | Status |
<ok / timeout / auth error (with login hint) / not installed (with install hint)>
~~~

Credit members by name. Where you were outvoted and verification proved you right — or wrong — say so plainly.

## Failure handling

- CLI missing → skip; add the install hint from roster.md to the ensemble log.
- Auth error → surface the login command from roster.md; continue with the remaining members.
- Timeout → mark as no response; do not retry within the run unless the user asks.
- Fewer than 2 non-Claude vendors usable → tell the user the ensemble is degraded to Claude-only voices, still answer, and offer the setup steps for the missing CLIs.

## When not to ensemble

Skip the ensemble — and say why — when the question is cheaply verifiable by direct means: a grep, a doc lookup, running the test. Seven LLMs are not a substitute for one ground-truth check. Cost note: every member spends that provider's quota or subscription; wall-clock ≈ the slowest member.

## Calibrating the roster

When the user asks to tune the roster (or after a run on a task with known ground truth), score members from the report itself: a member earns its seat by contributing **Unique insights (verified)** or by dissenting correctly in **Disputed** rows; a member that only echoes majorities adds cost without signal, and one that mostly populates **Rejected** is actively noisy. Suggest demoting the latter to the bench and trying a bench variant (roster.md § Bench) in its place, then re-score on the next run. Record any roster change in roster.md so it persists.
