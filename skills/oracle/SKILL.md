---
name: oracle
description: Use @steipete/oracle CLI to bundle a prompt plus files for a second-model review. Use for Oracle CLI, GPT-5.5 Pro browser-mode ChatGPT reviews, API or multi-model cross-checks, dry-run/file-bundle prep, session recovery, partial browser output, duplicate-run avoidance, and Claude Code/Codex skill workflows.
---

# Oracle

Use this skill when a task needs `@steipete/oracle`: preparing a high-signal prompt/file bundle, previewing what will be sent, running an API or browser-mode second opinion, recovering a stored session, or documenting the canonical result.

Oracle is a second-pass reviewer over curated context. It is not the source of truth by itself.

## Core Rules

- Do local source review and context curation first.
- For substantive reviews, always pass both a prompt and explicit `--file` attachments. Prompt-only runs are for smoke/preflight checks only.
- Treat Oracle output as advisory. Verify every conclusion against local source before making a decision or publishing a finding.
- Track a review by topic, prompt fingerprint, attached files/export, and output path, not by slug alone.
- Run `oracle --version` and `oracle --help` when exact flags, models, or defaults matter.
- Prefer `--slug`, not `--name`. Prefer `-p/--prompt`, not `--prompt-file`.
- Check existing sessions before starting a long run. Do not duplicate a run because a prior one is still thinking.
- If `oracle` is not on PATH, use `npx -y @steipete/oracle` as the command prefix.
- Read `references/troubleshooting.md` for browser/session recovery.
- Read `references/compatibility.md` for Codex/Claude Code install and publishing details.

Verified baseline for the published package `@steipete/oracle@0.10.0`:

- npm package requires Node `>=24`.
- Default model is `gpt-5.5-pro`.
- `--engine` is `api` or `browser`; if omitted, Oracle picks API when `OPENAI_API_KEY` is set, otherwise browser.
- Browser engine supports GPT models through ChatGPT and Gemini models through `gemini.google.com` cookies.
- Use API engine for Claude, Grok, Codex, `gemini-3.1-pro`, and multi-model runs.
- API mode may cost money; get explicit user consent before starting API runs when spend matters.
- `--file` accepts files, directories, globs, repeated flags, comma-separated entries, and `!pattern` excludes.
- Default per-file cap is 1 MB unless `ORACLE_MAX_FILE_SIZE_BYTES` or `maxFileSizeBytes` in `~/.oracle/config.json` raises it.
- Sessions live under `~/.oracle/sessions`; `ORACLE_HOME_DIR` can override the Oracle home.

## When To Use

Use Oracle for:

- final adversarial review of a narrowed candidate, patch, finding, or design decision
- second opinion on subtle multi-file logic after the relevant files are known
- cross-model API checks with `--models`
- ChatGPT browser-mode review when API access/spend is undesirable
- assembling a manual paste bundle when browser automation is unhealthy
- inspecting, reattaching, restarting, or classifying stored Oracle sessions

Do not use Oracle for:

- first-pass repository exploration
- whole-repo bug hunting without a curated bundle
- questions answerable by reading one local file
- launching duplicate sessions for the same topic/prompt/context
- treating model output as evidence without local verification

## Prepare The Bundle

1. Write a 6-30 sentence prompt that states:
   - project/repo and platform
   - exact review question
   - what each important attached file/export represents
   - constraints, attacker model, or decision criteria
   - what would falsify the candidate
2. Attach the smallest complete context:
   - prefer a curated export for broad or multi-file questions
   - attach exact files for narrow comparisons
   - exclude generated/vendor/test paths when irrelevant
   - redact `.env`, keys, tokens, cookies, and credentials unless absolutely required
3. Preview before costly or long runs:

```bash
oracle --dry-run summary \
  --files-report \
  -p "<review prompt>" \
  --file "<curated-export-or-files>"
```

Use `--engine api --dry-run summary --files-report` when you need a per-file token breakdown; dry-run does not call the API. Browser dry-run is still useful for checking total tokens and inline/upload delivery. Use `--dry-run json` when another tool will inspect the preview. Use `--render --copy` when automation is unhealthy and you want a manual paste bundle.

## Attachment Details

Useful `--file` behavior:

- Include literals, directories, and globs: `--file src --file README.md --file "src/**/*.ts"`.
- Exclude with `!`: `--file "src/**" --file "!src/**/*.test.ts" --file "!**/*.snap"`.
- Default-ignored directories include `node_modules`, `dist`, `coverage`, `.git`, `.turbo`, `.next`, `build`, and `tmp`, unless explicitly passed as literal paths.
- Glob expansion honors `.gitignore`.
- Symlinks are not followed during glob expansion.
- Dotfiles are filtered unless the pattern explicitly includes a dot-segment, for example `--file ".github/**"`.
- Use API dry-run plus `--files-report` to find token-heavy files before spending API or browser time.

## Recipes

### Install / Preflight

```bash
oracle --version
oracle --help
```

If missing:

```bash
npx -y @steipete/oracle --version
npx -y @steipete/oracle --help
```

### API Review

Use API when API keys/spend are acceptable or when targeting non-browser models:

```bash
oracle --engine api \
  --model gpt-5.5-pro \
  --slug "<topic-slug>" \
  --files-report \
  --write-output "<output.md>" \
  -p "<review prompt>" \
  --file "<context-file-or-glob>"
```

For cross-checks:

```bash
oracle --engine api \
  --models gpt-5.5-pro,gemini-3-pro \
  --slug "<topic-crosscheck>" \
  --files-report \
  -p "<cross-check prompt>" \
  --file "<context-file-or-glob>"
```

### Browser Review

Use browser mode for ChatGPT GPT-5.5 Pro without an API key:

```bash
oracle --engine browser \
  --model gpt-5.5-pro \
  --browser-manual-login \
  --browser-model-strategy select \
  --slug "<topic-slug>" \
  --files-report \
  --write-output "<output.md>" \
  -p "<review prompt>" \
  --file "<context-file-or-glob>"
```

First login or post-failure smoke:

```bash
oracle --engine browser \
  --browser-manual-login \
  --browser-keep-browser \
  --browser-input-timeout 120000 \
  --browser-model-strategy select \
  --slug "oracle-browser-smoke" \
  -p "Reply with exactly: ORACLE_SMOKE_OK"
```

Useful browser flags:

- `--browser-manual-login`: skip cookie copy and reuse Oracle's persistent automation profile.
- `--browser-keep-browser`: leave Chrome open after completion; useful for first login/debugging.
- `--browser-model-strategy select|current|ignore`: choose whether to switch, keep, or skip ChatGPT model selection.
- `--browser-attachments auto|never|always`: choose inline paste vs uploaded files.
- `--browser-bundle-files`: upload all attachments as one bundle.
- `--browser-auto-reattach-delay`, `--browser-auto-reattach-interval`, `--browser-auto-reattach-timeout`: keep checking long browser runs after timeouts.
- `--browser-port <port>`: pin the Chrome DevTools port for repeatable debugging.
- `--remote-host <host:port> --remote-token <token>`: delegate browser automation to `oracle serve` on a signed-in machine.

### Manual Paste Fallback

When automation is blocked but the bundle is ready:

```bash
oracle --render --copy \
  -p "<review prompt>" \
  --file "<context-file-or-glob>"
```

Paste the rendered bundle into the target model manually, save the final answer repo-locally, and record that it came from manual paste/recovery.

## Monitor And Recover

Before launching:

```bash
oracle status --hours 24 --limit 50
```

Safe passive checks while a browser run is active:

- `oracle status --hours <n> --limit <n>` without a session id
- `oracle session --path <slug-or-id>` to locate stored files without attaching
- `~/.oracle/sessions/<slug>/meta.json`
- `~/.oracle/sessions/<slug>/output.log`

Do not use these as passive polls while the original run is active:

- `oracle session <slug-or-id>`
- `oracle status <slug-or-id>`

With an id, both attach to that session. Use them only when you intentionally want to reattach, recover, or render stored output.

Use these classifications:

- `failed-bootstrap`: prompt did not reach the model/browser conversation.
- `partial`: run reached the model, but saved output is only a planning paragraph, truncated answer, or incomplete capture.
- `final`: saved or recovered answer is the substantive final answer.
- `stale`: status appears active, but artifacts show a terminal disconnect/error.
- `wrong-topic`: slug/session metadata says one thing, but the answer addresses another topic.

Important: `completed` means Oracle captured a non-empty answer candidate. It does not prove the browser-visible answer was the true final answer.

## Writeback

For every substantive Oracle review, record:

- topic
- slug/session id
- model and engine
- prompt fingerprint
- attached files/export paths
- output path
- classification
- canonical final-answer location
- local source-verification result

Routine smoke/preflight runs do not need a ledger entry unless they reveal a durable operational issue.

## Prompt Skeleton

```text
Project: <repo/project>. Role: independent second-pass reviewer.

Read the attached context. Answer only this question: <narrow question>.

Important context:
- <file/export A> is <role>.
- <file/export B> is <role>.
- Current local hypothesis is <hypothesis>, but challenge it aggressively.

Distinguish code-backed behavior, realistic prerequisites, intentional trust
boundaries, and severity/impact. Call out false-positive risk. Do not infer
from missing context; say what extra source would be needed to decide.
```
