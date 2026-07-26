# Member roster — verified command templates

Templates below were live-verified 2026-07-26 with codex-cli 0.145.0, agy 1.1.7 (Antigravity), and kimi-code 0.29.1 (cursor-agent installed but unverified — needs login). If a flag errors after an upgrade, re-check `<cli> --help` and update this file.

Conventions:
- `$SP` = the scratchpad created in SKILL.md Step 2, with two subdirectories whose separation is load-bearing: `$SP/ask/prompt.md` is the canonical prompt (the only location a member may be pointed at) and `$SP/out/` collects every answer.
- **Expand `$SP` to its absolute path before running anything here.** Each Bash call is a fresh shell with no memory of earlier variables; a bare `$SP` expands to nothing and the command writes to `/`.
- Every member reads the canonical prompt, runs read-only, writes its answer to `$SP/out/<member>.out` and stderr to `$SP/out/<member>.log`, wrapped in `timeout`.
- **Omit the model flag to use the CLI's default** (usually right — defaults tend to track the vendor's current best) **unless a member's section says otherwise** — the Google member (agy) pins its pro model explicitly rather than trusting a router default. Pin other models only when the user asks for specific versions, and verify IDs first; each pinned model is its own member.
- Default roster (7 voices): codex ×2 (`gpt-5.6-sol`, `gpt-5.6-luna`) + agy ×2 (`gemini-3.1-pro-high`, `gemini-3.6-flash-high`) + kimi (`k3`) + the two native Claude members (§ claude). cursor-agent and everything after it is opt-in.

## codex — OpenAI (GPT family) — VERIFIED WORKING

- Probe: `command -v codex` · Auth check: `codex login status` · Login: `codex login`
- **Two default members** (prompt via stdin; the final answer lands clean in each `-o` file):

  ```bash
  # member: codex-sol (config-default model, effort max)
  timeout 600 codex exec --sandbox read-only --ephemeral --skip-git-repo-check \
    -o "$SP/out/codex-sol.out" - < "$SP/ask/prompt.md" > "$SP/out/codex-sol.log" 2>&1

  # member: codex-luna
  timeout 600 codex exec --sandbox read-only --ephemeral --skip-git-repo-check \
    -m gpt-5.6-luna \
    -o "$SP/out/codex-luna.out" - < "$SP/ask/prompt.md" > "$SP/out/codex-luna.log" 2>&1
  ```

- Model status (probed 2026-07-26 on this ChatGPT-plan account):
  - ✅ `gpt-5.6-sol` — default member `codex-sol`; the config default (`~/.codex/config.toml` pins it with `model_reasoning_effort = "max"`)
  - ✅ `gpt-5.6-luna` — default member `codex-luna` (added at user request 2026-07-26)
  - ❌ `gpt-5.6-tera` — rejected: "not supported when using Codex with a ChatGPT account" (exists, but needs API-key billing — retry after `codex login --api-key`)
  - ❌ `gpt-5.6-high` / `gpt-5.6-max` — not model IDs. High/max are **reasoning-effort levels**: make an effort variant with `-c 'model_reasoning_effort="high"'` (verified working). Each (model, effort) combo can be its own member.
- Read-only: `--sandbox read-only` (required). Never `--dangerously-bypass-approvals-and-sandbox`.
- For repo-reading tasks, run from the repo root (drop nothing; `--skip-git-repo-check` is harmless inside a repo). For self-contained questions, add `-C "$SP/ask"` so it doesn't wander the repo — **the prompt directory, never `$SP` itself**: `-C` sets codex's working root and `--sandbox read-only` still permits reads, so rooting it one level up would let it read the other members' answers as they land.
- Bonus for code review: `codex exec review` is a purpose-built reviewer for the current repo — usable as this member's invocation, but it ignores the output contract, so map its findings into the consensus clusters manually.

## agy (Antigravity) — Google — VERIFIED WORKING

- Google's successor to the legacy gemini-cli (free-tier OAuth is cut off on gemini-cli ≥ 0.46 with `IneligibleTierError: UNSUPPORTED_CLIENT`). Binary: `agy` (`~/.local/bin/agy`, v1.1.7 here; install fallback: `brew install --cask antigravity-cli`).
- Probe: `command -v agy` · Auth: Antigravity account login (cached on this machine — no prompt needed). If auth fails, run `agy` once interactively to sign in.
- **Two default members** (both live-verified 2026-07-26, ~3–6s each):

  ```bash
  # member: agy-pro (Google's strongest tier)
  timeout 600 agy --mode plan --model gemini-3.1-pro-high \
    -p "$(cat "$SP/ask/prompt.md")" > "$SP/out/agy-pro.out" 2> "$SP/out/agy-pro.log"

  # member: agy-flash (newer generation, flash tier — the fast wildcard)
  timeout 600 agy --mode plan --model gemini-3.6-flash-high \
    -p "$(cat "$SP/ask/prompt.md")" > "$SP/out/agy-flash.out" 2> "$SP/out/agy-flash.log"
  ```

- Models — `agy models` enumerates them (this account, 2026-07-26): `gemini-3.1-pro-{high,low}` (`agy-pro` uses -high), `gemini-3.6-flash-{high,medium,low}` (`agy-flash` uses -high), `gemini-3.5-flash-{high,medium,low}`, plus non-Google models `claude-sonnet-4-6`, `claude-opus-4-6-thinking`, `gpt-oss-120b-medium`. Reasoning effort is baked into the model-ID suffix.
- **Multi-vendor caveat** (same rule as cursor-agent): use its `gemini-*` models for the Google voice. Its `claude-*` models would double-count Anthropic (the ensemble already has fable + opus); `gpt-oss-120b` overlaps OpenAI lineage with codex.
- Read-only: `--mode plan` (verified working). Never pass `--dangerously-skip-permissions`; `--sandbox` adds terminal restrictions if a task ever needs a non-plan mode.
- Long tasks: agy's internal print wait defaults to 5m — pass `--print-timeout 9m` and raise the outer `timeout` to 1200 for big reviews.
- Legacy fallback (borrowed time — the old client still passes server-side): `timeout 600 npx -y @google/gemini-cli@0.26.0 -m gemini-2.5-pro -o text -p "$(cat "$SP/ask/prompt.md")" > "$SP/out/gemini.out" 2> "$SP/out/gemini.log"`

## cursor-agent — Cursor (multi-vendor: GPT, Claude, Gemini, …) — OPT-IN; INSTALLED, NEEDS LOGIN

- Probe: `command -v cursor-agent` · Auth check: `cursor-agent status` · Login: `cursor-agent login` (or `CURSOR_API_KEY`)
- Status on this machine 2026-07-26: not authenticated — run `cursor-agent login` once to activate this member.
- Template:

  ```bash
  timeout 600 cursor-agent -p --output-format text --sandbox enabled \
    "$(cat "$SP/ask/prompt.md")" > "$SP/out/cursor.out" 2> "$SP/out/cursor.log"
  ```

- Models: `cursor-agent --list-models`, pin with `--model <id>`. Because it fronts multiple vendors, pick a model from a vendor **not** already in the roster — otherwise one vendor gets two votes.
- Read-only: `--sandbox enabled`, and never `-f/--force` — print mode has write+bash access by default.

## kimi — Moonshot (kimi-code) — VERIFIED WORKING

- Binary is **not on PATH** here: it lives at `~/.kimi-code/bin/kimi` (installed via the kimi-plugin-cc Claude Code plugin, kimi-code 0.29.1).
- Auth: `kimi login` (device-code flow) — credentials verified cached on this machine 2026-07-26.
- Template — the probe and the call must be **one command**, since a variable set in one Bash call is gone by the next:

  ```bash
  KIMI_BIN="$(command -v kimi || echo "$HOME/.kimi-code/bin/kimi")"; \
  timeout 600 "$KIMI_BIN" -p "$(cat "$SP/ask/prompt.md")" --output-format text \
    > "$SP/out/kimi.out" 2> "$SP/out/kimi.log"
  ```

- Version variant: add `-m <model-alias>` (aliases come from `~/.kimi-code/config.toml` / `kimi provider`).
- Read-only: never pass `-y/--yolo` or `--auto` — in `-p` mode, tool calls needing approval are not auto-approved. `--plan` is a stricter plan-mode option if needed.
- Quirks: output lines carry a leading `• ` marker (strip when parsing); stderr ends with a session-resume hint and may note a cwd reset to the registered workspace — run from the repo root for repo tasks.
- API fallback (no CLI available, e.g. CI; OpenAI-compatible, needs `MOONSHOT_API_KEY`, verify endpoint + model ID against Moonshot docs; no repo access — self-contained prompts only):

  ```bash
  jq -n --rawfile p "$SP/ask/prompt.md" \
    '{model: "<kimi-model-id>", messages: [{role: "user", content: $p}]}' \
  | timeout 600 curl -s https://api.moonshot.ai/v1/chat/completions \
      -H "Authorization: Bearer $MOONSHOT_API_KEY" -H "Content-Type: application/json" -d @- \
  | jq -r '.choices[0].message.content' > "$SP/out/kimi.out" 2> "$SP/out/kimi.log"
  ```

## claude — Anthropic (default members: Fable 5 + Opus 5) — NATIVE, NO CLI

This skill runs inside Claude Code, so Claude voices dispatch natively:

- **The orchestrator is one voice for free.** Whatever model the session runs (label it by model — the session may be on Fable 5, Opus 5, or something else entirely) writes its blind answer to `$SP/out/claude-<model>.out` before reading anything. Spawn subagents for the *other* default Claude model(s), so the pair is always covered without doubling a voice.
- **Each other requested Claude model is a background subagent** launched via the Agent tool with the matching `model` override — `"fable"` → Fable 5, `"opus"` → Opus 5 (also available: `"sonnet"`, `"haiku"`). Spawn it in the background alongside the CLI members. Its prompt must contain exactly, and only:
  1. the canonical prompt (`$SP/ask/prompt.md` contents, verbatim);
  2. the output contract;
  3. this closing instruction: "Work read-only — do not create or modify any files. Your final message is your complete answer, ending with the required json block."
  No conversation context, no mention of the ensemble, no other member's output — the subagent must be as blind as everyone else.
- Treat the subagent's returned text as that member's `.out` content (save it to `$SP/out/claude-<model>.out` for the record).
- Same-vendor caveat: Fable 5 and Opus 5 are two members but **one vendor** — when a verdict hinges on the count, report the vendor-distinct tally (SKILL.md Step 4).
- Fallback outside Claude Code (plain terminal): `timeout 600 claude -p --model <model-id> "$(cat "$SP/ask/prompt.md")" > "$SP/out/claude-x.out" 2> "$SP/out/claude-x.log"`.

## Bench — opt-in variants and when to field them

Verified-available alternates outside the default seven. Field one as an extra member or a substitution when the task fits:

| Variant | Via | Field it when |
|---|---|---|
| `k3-256k` | kimi `-m` | The diff/context is huge — swap in for `k3` on long-context reviews |
| `kimi-for-coding` (+ `-highspeed`) | kimi `-m` | Code-review tasks; `-highspeed` when latency matters |
| `gemini-3.5-flash-high` | agy `--model` | A third Google opinion, or speed |
| `gpt-oss-120b-medium` | agy `--model` | You want an open-weights voice — but it shares OpenAI ancestry with codex, so count vendors accordingly |
| `sonnet` / `haiku` | native subagent `model` override | Cheap extra Claude voices (same-vendor with fable/opus) |
| `gpt-5.6-tera` | codex `-m` | Only after switching codex to API-key billing (`codex login --api-key`) — blocked on ChatGPT plans |
| cursor-agent catalog | cursor-agent `--model` | After `cursor-agent login`; point it at a vendor not already fielded |
| Effort variants | codex `-c 'model_reasoning_effort="…"'`; agy `-high/-medium/-low` ID suffixes | Comparing effort tiers within one vendor is itself a useful mini-ensemble |

## Executor subagents (parallel dispatch vehicle)

Claude members are always subagents; CLI members may also be dispatched through **thin executor subagents** instead of direct background Bash — same parallelism (spawn them all in one message), with member outputs kept out of the orchestrator's context until aggregation. The contract keeps them pure plumbing:

- One background general-purpose subagent per member, all launched together.
- Its entire prompt: *"Run exactly this Bash command, unmodified: `<the member's template from this file, with the absolute scratchpad path substituted in — never a bare $SP>`. Do not read the prompt file or the output file, do not add flags or context. When it exits, reply with exactly one line: `<member> exit=<code> secs=<n>`, plus the last stderr line if the exit code is nonzero."*
- The executor must never rewrite the canonical prompt, retry with different flags, or summarize the member's answer — the orchestrator reads `$SP/out/<member>.out` itself at aggregation time.

## Adding a member

A member must provide: (1) an availability probe, (2) an auth check + login hint, (3) a non-interactive invocation that reads the canonical prompt, (4) a read-only guarantee, (5) output captured to `$SP/out/<member>.out`, (6) a model-pin syntax. Add a section here in the same shape.

Generic OpenAI-compatible API member (covers Grok, DeepSeek, Qwen, local Ollama endpoints, …): reuse the kimi curl template with the vendor's base URL, key env var, and model ID — same no-repo-access caveat.
