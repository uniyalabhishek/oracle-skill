# Oracle Skill

Agent Skill for using [`@steipete/oracle`](https://github.com/steipete/oracle) effectively from Codex, Claude Code, and other Agent Skills-compatible tools.

The skill focuses on the durable workflow:

- curate the right files before asking Oracle
- preview with `--dry-run summary --files-report`
- choose browser vs API intentionally
- verify live CLI flags before relying on version-specific recipes
- avoid duplicate long-running sessions
- wait efficiently with quiet heartbeats and browser-tab status checks
- recover partial browser output safely
- distinguish normal Pro/Extended Pro runs from ChatGPT Deep Research
- choose archive behavior explicitly when the ChatGPT conversation must remain visible
- verify Oracle conclusions against local source

## Layout

```text
oracle-skill/
├── .codex-plugin/plugin.json
├── .claude-plugin/plugin.json
└── skills/oracle/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
```

## Codex

Use as a plugin, or copy the skill folder into a Codex skill location:

```bash
mkdir -p "$HOME/.agents/skills"
cp -R skills/oracle "$HOME/.agents/skills/oracle"
```

Invoke explicitly:

```text
$oracle
```

## Claude Code

Use as a plugin, or copy the skill folder into a Claude Code skill location:

```bash
mkdir -p "$HOME/.claude/skills"
cp -R skills/oracle "$HOME/.claude/skills/oracle"
```

Invoke explicitly:

```text
/oracle
```

Claude Code can use GPT-5.5 Pro by following the skill's Oracle CLI recipe:

```bash
oracle --engine browser --model gpt-5.5-pro --browser-research off --browser-archive never ...
```

## Validate

```bash
oracle --engine api --dry-run summary --files-report \
  -p "Preview this skill package for obvious missing attachment issues." \
  --file skills/oracle/SKILL.md \
  --file skills/oracle/references/troubleshooting.md \
  --file skills/oracle/references/compatibility.md
```
