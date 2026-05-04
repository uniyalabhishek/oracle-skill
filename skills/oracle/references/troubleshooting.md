# Oracle Troubleshooting

Use this reference when Oracle browser mode, sessions, or saved output are confusing. Re-check `oracle --version`, `oracle --help`, and `oracle --debug-help` before relying on a remembered flag.

## Quick Diagnostics

```bash
oracle --version
oracle --help
oracle --debug-help
oracle status --hours 24 --limit 50
oracle session --help
oracle status --help
```

If `oracle` is missing:

```bash
npx -y @steipete/oracle --help
```

For stored files:

```bash
oracle session --path <slug-or-id>
```

Then inspect:

- `~/.oracle/sessions/<slug>/meta.json`
- `~/.oracle/sessions/<slug>/output.log`

## Failed Bootstrap

The prompt did not reach a real model/browser conversation.

Common signatures:

- `No ChatGPT cookies were applied from your Chrome profile; cannot proceed in browser mode`
- `connect ECONNREFUSED 127.0.0.1:<port>`
- unsupported option errors such as `--name` or `--prompt-file`
- missing local `oracle` binary when `npx -y @steipete/oracle` would work
- Node install/runtime errors; published `@steipete/oracle@0.10.0` reports Node `>=24`

Handling:

1. Mark the run `failed-bootstrap`.
2. Do not cite it as a review.
3. Fix the transport, installation, or flags.
4. Run a prompt-only smoke.
5. Retry the substantive prompt only after smoke succeeds.

## Real But Partial

The run reached the model, but local saved output is incomplete.

Symptoms:

- status says `completed`
- `output.log` contains only a short planning/opening paragraph
- the browser-visible response was longer or had a final conclusion

Handling:

1. Mark the run `partial`, not `final`.
2. Inspect `meta.json` for `conversationId`, `tabUrl`, `mode`, `model`, and browser runtime fields.
3. Recover the browser-visible answer if possible.
4. Save the recovered answer as the canonical output.
5. Verify the recovered answer is substantive before using it.

## Stale Running Session

The session appears running, but artifacts show a terminal loss.

Look for:

- `response.incompleteReason = chrome-disconnected`
- `errorMessage` indicating Chrome closed or the browser/session was lost
- no artifact growth plus a terminal-looking disconnect/error

Handling:

1. Mark it `stale`.
2. Do not relaunch blindly.
3. Reattach only if recovery is intentional.

## Wrong Topic

The slug says one topic, but the answer discusses another.

Handling:

1. Trust answer content over slug/session names.
2. Move or relabel the result under the topic it actually answered.
3. Leave the intended topic unresolved until a correct review exists.

## Browser Mode Playbook

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

Stable review shape:

```bash
oracle --engine browser \
  --model gpt-5.5-pro \
  --browser-manual-login \
  --browser-model-strategy select \
  --slug "<topic-slug>" \
  --files-report \
  --write-output "<output.md>" \
  -p "<review prompt>" \
  --file "<context>"
```

Long browser runs:

```bash
oracle --engine browser \
  --model gpt-5.5-pro \
  --browser-manual-login \
  --browser-auto-reattach-delay 30s \
  --browser-auto-reattach-interval 2m \
  --browser-auto-reattach-timeout 2m \
  -p "<review prompt>" \
  --file "<context>"
```

Remote signed-in browser host:

```bash
oracle serve --host 0.0.0.0 --port 9473 --token <secret>
oracle --engine browser \
  --remote-host <host:port> \
  --remote-token <secret> \
  -p "<review prompt>" \
  --file "<context>"
```

## `--browser-keep-browser`

Treat `--browser-keep-browser` as "keep the Chrome process/profile alive", not as a guarantee that the exact ChatGPT tab remains open after success.

Observed behavior:

- the Chrome process can remain alive
- the isolated ChatGPT tab can still close after successful completion
- an empty DevTools tab list after completion is not automatically a failed keep-browser run

If you need final text from the tab, capture it before completion or recover it from saved session metadata.

## Browser-Visible Recovery

When local output is partial but metadata has a browser conversation URL:

1. Locate the session directory:

```bash
oracle session --path <slug-or-id>
```

2. Read `meta.json` and find `tabUrl`, `conversationId`, and browser port/runtime fields.
3. If the same Chrome DevTools port is alive, inspect tabs:

```bash
curl http://127.0.0.1:<port>/json/list
```

4. If needed, open the saved conversation URL in that browser profile.
5. Use browser automation or manual copy to capture the visible assistant answer.
6. Save that recovered answer repo-locally and mark it canonical only after confirming it is final.

## Duplicate Runs

Before a new long run:

```bash
oracle status --hours 24 --limit 50
```

If the same topic/prompt/context is already running:

- do not start another run
- inspect artifacts passively
- reattach only when recovery is intentional
- use `--force` only when you deliberately want a second independent run

## Ledger Entry Template

```md
### <topic>

- Topic: <one-line scope>
- Prompt fingerprint: starts with `<opening words>`
- Attached context:
  - `<path-or-export>`
- Session:
  - slug: `<slug>`
  - model: `<model>`
  - engine: `<api|browser>`
- Output:
  - `<repo-local-output.md>`
- Classification:
  - `<failed-bootstrap|partial|final|stale|wrong-topic>`
- Canonical answer:
  - `<path or recovery note>`
- Source verification:
  - `<what was confirmed locally after Oracle>`
```
