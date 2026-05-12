# Codex And Claude Code Compatibility

This package keeps the shared skill portable across Codex, Claude Code, and other Agent Skills-compatible tools.

## Shared Skill Shape

```text
oracle/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── troubleshooting.md
    └── compatibility.md
```

Keep the shared `SKILL.md` portable:

- `name` must match the directory name: `oracle`.
- `description` should include both what the skill does and when to use it.
- Keep `SKILL.md` under 500 lines.
- Keep detailed recovery and product-specific installation guidance in one-level `references/` files.
- Keep scripts optional; Oracle itself is the executable.

## Codex

Codex local authoring paths:

- Repo skill: `.agents/skills/oracle/SKILL.md`
- User skill: `$HOME/.agents/skills/oracle/SKILL.md`

For reusable GitHub distribution, prefer a Codex plugin:

```text
oracle-skill/
├── .codex-plugin/
│   └── plugin.json
└── skills/
    └── oracle/
        ├── SKILL.md
        ├── agents/openai.yaml
        └── references/
```

This package sets `policy.allow_implicit_invocation: false` in `agents/openai.yaml` so Codex only uses Oracle when explicitly requested. Set it to `true` only if you want Codex to choose Oracle based on prompt matching.

## Claude Code

Claude Code can load the same skill from:

- Personal skill: `~/.claude/skills/oracle/SKILL.md`
- Project skill: `.claude/skills/oracle/SKILL.md`
- Plugin skill: `<plugin>/skills/oracle/SKILL.md`

Direct invocation:

```text
/oracle
```

Plugin invocation is namespaced by plugin name:

```text
/oracle-review:oracle
```

Claude Code can use GPT-5.5 Pro through Oracle by running:

```bash
oracle --engine browser --model gpt-5.5-pro --browser-research off --browser-archive never ...
```

or, when API spend is intended:

```bash
oracle --engine api --model gpt-5.5-pro ...
```

Do not set this in shared Claude frontmatter:

```yaml
model: gpt-5.5-pro
```

Claude Code's `model` frontmatter selects Claude's own execution model, not Oracle's external target model.

Use Claude-specific frontmatter only in a Claude-only fork or local copy:

```yaml
disable-model-invocation: true
allowed-tools:
  - Bash(oracle *)
  - Read
  - Grep
  - Glob
```

Guidance:

- `disable-model-invocation: true` is appropriate if you want Oracle only when a user explicitly invokes `/oracle`.
- `allowed-tools` pre-approves tool calls while the skill is active. Do not include it in the public default unless you intentionally want those permissions.
- Prefer a locally installed `oracle` binary over pre-approving `npx`; `npx -y @steipete/oracle` can download and execute package code.
- Avoid Claude dynamic shell injection in the shared skill because it is Claude-specific and can run commands before the model sees the prompt.

## Dual-Plugin Layout

This repository supports both Codex and Claude Code with one shared `skills/` directory:

```text
oracle-skill/
├── .codex-plugin/
│   └── plugin.json
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── oracle/
        ├── SKILL.md
        ├── agents/openai.yaml
        └── references/
```

## Validation Checklist

Before publishing:

- Validate YAML frontmatter in `skills/oracle/SKILL.md`.
- Confirm `name` equals the skill directory name.
- Confirm `description` is under 1024 characters and starts with strong trigger words.
- Confirm the skill works without Claude-specific frontmatter.
- Confirm `oracle --version` and `oracle --help` match the commands in `SKILL.md`.
- Run:

```bash
oracle --engine api --dry-run summary --files-report \
  -p "Preview this skill package for obvious missing attachment issues." \
  --file skills/oracle/SKILL.md \
  --file skills/oracle/references/troubleshooting.md \
  --file skills/oracle/references/compatibility.md
```
