# claude-skill-InterruptingCow

A [Claude Code](https://claude.com/claude-code) skill that pings the
operator on Telegram when a long-running task finishes / fails /
crosses a threshold. Wraps the `moo` CLI from the
[InterruptingCow](https://github.com/CryptoJones/InterruptingCow)
Python package.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?logo=apache)](LICENSE)
[![Codeberg](https://img.shields.io/badge/Codeberg-CryptoJones%2Fclaude--skill--InterruptingCow-2185D0?logo=codeberg&logoColor=white)](https://codeberg.org/CryptoJones/claude-skill-InterruptingCow)
[![GitHub](https://img.shields.io/badge/GitHub-CryptoJones%2Fclaude--skill--InterruptingCow-181717?logo=github&logoColor=white)](https://github.com/CryptoJones/claude-skill-InterruptingCow)
[![Upstream](https://img.shields.io/badge/Upstream-InterruptingCow-007ec6)](https://github.com/CryptoJones/InterruptingCow)

> Mirrored on both [GitHub](https://github.com/CryptoJones/claude-skill-InterruptingCow) and
> [Codeberg](https://codeberg.org/CryptoJones/claude-skill-InterruptingCow). Issues filed on
> either are welcome; commits are pushed to both.

---

## What it does

When the operator's turn contains phrasing like:

- "ping me on telegram when X finishes"
- "notify me when this training run is done"
- "let me know when the deploy completes"
- "interrupt me when balance crosses $5"
- "tell me when the backup is done"

…this skill loads (via frontmatter `description` matching) and shells out to `moo` (the short alias for `interruptingcow`) at the appropriate moment — usually after a long-running task ends, or when a threshold-watching loop crosses a configured boundary.

It does **not** re-implement Telegram bot mechanics; it delegates to the canonical CLI in the upstream package so the HTTP layer + credential resolution + error handling stay in one place.

See [`SKILL.md`](SKILL.md) for the full skill definition, including the trigger-phrase list, fire-vs-don't-fire heuristics, severity guide, and the failure-mode table.

## Install

Two pieces:

1. **The skill itself** (this repo):
   ```bash
   git clone https://github.com/CryptoJones/claude-skill-InterruptingCow \
     ~/.claude/skills/InterruptingCow
   ```
2. **The underlying CLI** (the upstream package):
   ```bash
   pip install interruptingcow
   ```

3. **Telegram credentials** — one-time, ~3 minutes:
   - Create a bot via [@BotFather](https://t.me/BotFather); get the token.
   - Send the bot any message, then visit `https://api.telegram.org/bot<TOKEN>/getUpdates` to read your chat ID.
   - Export `TELEGRAM_BOT_TOKEN` + `TELEGRAM_CHAT_ID`, or write them to `~/.config/interruptingcow/config.json`.

Restart Claude Code (or open `/hooks` once) for the skill to be picked up.

Full setup walkthrough: see the upstream [README](https://github.com/CryptoJones/InterruptingCow#quick-start).

## Why two repos?

Same two-layer-mirror pattern as [claude_skill-correcthorsebatterystaple](https://github.com/CryptoJones/claude_skill-correcthorsebatterystaple) →
[correcthorsebatterystaple](https://github.com/CryptoJones/correcthorsebatterystaple):

- **Upstream** ([InterruptingCow](https://github.com/CryptoJones/InterruptingCow)) — the Python package: HTTP client, credential resolution, CLI, tests, the eventual PyPI wheel.
- **This repo** — only `SKILL.md` + install glue. Independently installable into Claude Code without dragging in the entire upstream package source.

Updating the SKILL.md trigger phrases doesn't require re-cutting the Python release. Updating the CLI doesn't require touching this repo.

## Customization

The `frontmatter.description` in `SKILL.md` is what Claude matches against to decide when to fire. Fork + edit if your trigger vocabulary or the upstream CLI binary name differs (e.g. you renamed `moo` to `tg` in your shell).

The severity emojis (✅ / ⚠️ / 🚨) and `--silent` semantics live in the upstream CLI, not in the skill. Edit there if you want different glyphs or different notification behavior.

## License

Apache 2.0. See [LICENSE](LICENSE).

Proudly Made in Nebraska. Go Big Red! 🌽 https://xkcd.com/2347/
