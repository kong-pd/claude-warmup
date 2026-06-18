# claude-warmup

> A serverless GitHub Actions workflow that keeps your Claude usage window aligned with your working hours — no server, no local machine required.

![Workflow status](https://github.com/kong-pd/claude-warmup/actions/workflows/warmup.yml/badge.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

## Why

Claude's subscription plans meter usage in a rolling **5-hour window** that is anchored to the *first* message of a session. If that window happens to reset at an inconvenient time, you can lose productive hours waiting for it to roll over — or end up setting a midnight alarm to "warm up" the window by hand. (Yes, really. That's the itch this scratches.)

`claude-warmup` automates the warm-up. It fires a tiny throwaway prompt on a schedule you control, so a fresh 5-hour window is always anchored to the times you actually work — fully hands-off, even while your computer is asleep or powered off.

## How it works

```
GitHub Actions (cron, UTC)
      │
      ├─ installs the Claude Code CLI
      │
      └─ runs:  claude -p "hi" --model haiku   ← cheapest possible call
                      │
                      └─ anchors a fresh 5-hour usage window
```

- **Scheduler** — GitHub Actions `schedule` triggers (cron). Runs in the cloud, so it never depends on a local machine being on.
- **Runtime** — Headless Claude Code (`claude -p`), the official non-interactive mode.
- **Cost control** — Uses Haiku (the lightest model) with a one-word prompt, so token usage is negligible.
- **Auth** — A long-lived OAuth token stored as an encrypted GitHub Actions secret; never committed to the repo.

## Schedule

Warm-ups fire **4× per day, spaced 5 hours apart** to give continuous fresh windows across working hours.
| Local (UTC+8) | UTC   | Cron          |
|:-------------:|:-----:|:--------------|
| 07:30         | 23:30 | `30 23 * * *` |
| 12:30         | 04:30 | `30 4 * * *`  |
| 17:30         | 09:30 | `30 9 * * *`  |
| 22:30         | 14:30 | `30 14 * * *` |

> GitHub Actions cron **always runs in UTC** — adjust the offset for your own timezone.

## Setup

1. **Use this repo as a template** (or fork it).
2. Generate a long-lived token locally — requires the Claude Code CLI and a Pro/Max subscription:
   ```bash
   claude setup-token
   ```
3. In your repo, go to **Settings → Secrets and variables → Actions → New repository secret**:
   - **Name:** `CLAUDE_CODE_OAUTH_TOKEN`
   - **Value:** the `sk-ant-oat01-…` token from step 2
4. *(Optional)* Edit the cron times in `.github/workflows/warmup.yml` to match your hours.
5. Test it: **Actions → claude-warmup → Run workflow**. A green check means you're live.

## The workflow

```yaml
name: claude-warmup
on:
  schedule:
    - cron: '30 23 * * *'   # 07:30 UTC+8
    - cron: '30 4 * * *'    # 12:30 UTC+8
    - cron: '30 9 * * *'    # 17:30 UTC+8
    - cron: '30 14 * * *'   # 22:30 UTC+8
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

## Notes & limitations

- **Verify it still anchors your window.** As of **June 15, 2026**, Anthropic split programmatic usage (`claude -p`, Agent SDK, CI) into a metered credit pool billed at API rates, separate from interactive chat limits. Depending on your account configuration, a headless warm-up may or may not still anchor your interactive chat window. After the first run, check **Settings → Usage** to confirm the reset time actually shifted.
- **Intended for personal, individual use** — a convenience utility for a single user's own account, consistent with ordinary individual usage.
- GitHub **disables scheduled workflows after 60 days** of repo inactivity; an occasional commit keeps them alive.
- Scheduled runs can be **delayed a few minutes** during peak load — perfectly fine for warm-ups.

## Tech stack

GitHub Actions · cron · Claude Code CLI (headless) · OAuth · encrypted secrets

## License

MIT — see [`LICENSE`](./LICENSE).
