# claude-warmup

Claude's usage resets on a 5-hour rolling window. I wanted it to reset before I start work. This does that.

![Workflow status](https://github.com/kong-pd/claude-warmup/actions/workflows/warmup.yml/badge.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## How

[cron-job.org](https://cron-job.org) fires a POST to the GitHub API at 07:30, 12:30, 17:30, and 22:30. GitHub Actions runs `claude -p "hi" --model haiku`. That's it.

GitHub Actions has a built-in scheduler but it drifts by 1–3 hours under load. cron-job.org doesn't.

## Setup

**1. Claude OAuth token**

```bash
claude setup-token
```

Requires Claude Code CLI and a Pro/Max subscription. The token prints once, pls save it.

**2. Workflow**

`.github/workflows/warmup.yml`:

```yaml
name: claude-warmup
on:
  workflow_dispatch:
jobs:
  warmup:
    runs-on: ubuntu-latest
    steps:
      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code
      - name: Warm up
        run: claude -p "hi" --model haiku
        env:
          CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**3. GitHub secret**

Repo → Settings → Secrets → `CLAUDE_CODE_OAUTH_TOKEN` → paste token from step 1.

**4. GitHub PAT**

Settings → Developer settings → Personal access tokens → `workflow` scope only.

**5. cron-job.org**

Four jobs, one per time slot:

| Field | Value |
|---|---|
| URL | `https://api.github.com/repos/YOUR_USERNAME/claude-warmup/actions/workflows/warmup.yml/dispatches` |
| Method | POST |
| `Authorization` | `Bearer YOUR_GITHUB_PAT` |
| `Content-Type` | `application/json` |
| Body | `{"ref":"main"}` |

## Caveats

As of June 15, 2026, Anthropic moved programmatic usage (`claude -p`) to a separate credit pool from interactive chat. It's unclear whether this still anchors the chat window. Test it before relying on it: run the workflow manually, then check Settings → Usage for a changed reset time.

Don't commit tokens. If one appears in a screenshot, revoke it.

## License

MIT
