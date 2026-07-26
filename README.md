# Research-Agent-Skills

Claude Code skills that speed up research workflows, packaged as a plugin marketplace so they install anywhere with two commands.

## Install

Inside Claude Code:

```
/plugin marketplace add TianyuDu/Research-Agent-Skills
/plugin install research-agent-skills@research-agent-skills
```

Get updates later with `/plugin marketplace update`.

Alternatives: clone this repo and symlink a skill into `~/.claude/skills/` (personal, one machine), or copy a skill folder into a project's `.claude/skills/` and commit it (travels with that repo, works in cloud sessions).

## Skills

| Skill | What it does |
|---|---|
| **[llm-ensemble](plugins/research-agent-skills/skills/llm-ensemble/)** | Ask seven frontier models (OpenAI, Google, Moonshot, Anthropic) the same question in parallel, blind to each other, then verify their claims and return one adjudicated consensus report — agreements, adjudicated disputes, credited unique insights, and rejected hallucinations. Needs the external CLIs for full effect; degrades gracefully without them. |
| **[lab-research-memo](plugins/research-agent-skills/skills/lab-research-memo/)** | Build, edit, or review Jupyter-notebook research memos following a fixed eight-section lab convention — pre-committed decision rules, required figures, TODO-based honesty markers, and a post-render sanity check. |
| **[simulation-memo](plugins/research-agent-skills/skills/simulation-memo/)** | Specify a simulation study as a pre-registered notebook memo: full data-generating process, pre-committed decision rule, sanity-check protocol — detailed enough for a coauthor to replicate, audit, and interpret. |

## Repo layout

```
.claude-plugin/marketplace.json        the marketplace manifest
plugins/research-agent-skills/         the plugin
├── .claude-plugin/plugin.json
└── skills/
    ├── llm-ensemble/                  SKILL.md + roster.md + README.md
    ├── lab-research-memo/             SKILL.md
    └── simulation-memo/               SKILL.md
```

Each skill is a directory with a `SKILL.md` (instructions + frontmatter that drives auto-triggering) and optional supporting files. Skills follow the open [Agent Skills](https://code.claude.com/docs/en/skills) format, so they also work uploaded to claude.ai or via the Claude API's Skills endpoint.
