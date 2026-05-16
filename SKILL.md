---
name: InterruptingCow
description: Send Aaron an async ping on Telegram via the `moo` / `interruptingcow` CLI. Use whenever a long-running task finishes / fails / crosses a threshold and Aaron may not be at the keyboard. Triggers include "ping me on telegram", "notify me when X done", "let me know when this finishes", "interrupt me when Y", "tell me when the training run completes", and any "alert me about Z" framing where the alert needs to reach Aaron's phone, not just print to the terminal.
---

# InterruptingCow — Telegram async-ping skill

When Aaron has stepped away from the keyboard (going to sleep, away from desk, in a meeting) and something Claude is watching reaches a state worth surfacing, this skill is how to tell him without relying on him noticing terminal output. Calls the `moo` CLI from the `interruptingcow` Python package.

## When to fire

**Fire on async events that change state Aaron cares about:**

- Long-running task completion (training run, deploy, backup, multi-hour build)
- Threshold crossings on resources Aaron is paying for (RunPod balance, AWS spend, disk full)
- Unrecoverable errors during overnight / scheduled work
- "Almost done" milestones if a task is hours long and Aaron asked to be kept in the loop
- A scheduled cron or `loop` task that fired and needs his attention

**Do NOT fire on:**

- Anything during normal conversational back-and-forth — Aaron is reading the terminal already
- Routine in-progress updates (`pytest is running...`) — only fire on state changes
- "Sycophantic" updates ("just wanted to let you know I'm thinking about this") — only useful information
- Anything below a real "this changes what Aaron should do next" threshold

If unsure: ask yourself "is Aaron likely already looking at the terminal?" If yes, terminal output is enough; don't ping.

## How to invoke

```bash
moo "<message>"                            # info, audible notification
moo --severity warning "<message>"          # ⚠️ prefix
moo --severity critical "<message>"         # 🚨 prefix
moo --silent "<message>"                    # delivered without phone notification sound
```

Severity guide:

- **info** (default): a thing completed normally. "Training run done."
- **warning**: a thing needs eyes soon but isn't on fire. "RunPod balance crossed $10."
- **critical**: act now or lose work. "RunPod balance $0.20, pod terminating, adapter UNSAVED."

Always include in the message:
1. **What happened** (one short clause)
2. **What Aaron's next action would be**, if non-obvious

Example bodies that work:

- `moo "training done, adapter at /workspace/dave_adapter, pod still up — run hydra publish then teardown"`
- `moo --severity warning "RunPod $9.74 — under 6h at A100 rates. Plan teardown or top up."`
- `moo --severity critical "pod $0.32 left. terminating now to avoid stranding artifacts."`

Example bodies to avoid:

- `moo "FYI"` (no information)
- `moo "training is running"` (state didn't change)
- `moo "thinking about how to approach this"` (conversational, terminal output is the right channel)

## Prereqs operator-side

If `command -v moo` returns nothing, the skill can't fire. Prompt Aaron to install:

```bash
pip install interruptingcow
# Then one-time Telegram setup:
# 1. Create bot via @BotFather, get token
# 2. Send the bot any message, then visit https://api.telegram.org/bot<TOKEN>/getUpdates → grab chat_id
# 3. export TELEGRAM_BOT_TOKEN=... TELEGRAM_CHAT_ID=...  (or write ~/.config/interruptingcow/config.json)
```

Full install + setup docs at https://github.com/CryptoJones/InterruptingCow.

## Failure modes the skill must handle

| Failure | Detection | Response |
|---|---|---|
| `moo` not installed | `command -v moo` returns nothing | Don't try; print the install command + the message that *would* have been sent, so Aaron can act once he's back at the terminal. |
| Missing credentials (exit 2) | `moo` exited 2 with "bot_token not set" / "chat_id not set" | Same — print the message + a one-line "set TELEGRAM_BOT_TOKEN + TELEGRAM_CHAT_ID then re-run." |
| Telegram API error (exit 3) | `moo` exited 3 with a TelegramError | Print the error AND the message body, so the ping is preserved in the transcript even if the phone-side delivery failed. |

A failed ping should never silently swallow the message. The text was important enough to write — make sure it lands somewhere Aaron will see, even if Telegram is down.

## The underlying CLI

`moo` and `interruptingcow` are both entry points to the same code in the [`interruptingcow`](https://github.com/CryptoJones/InterruptingCow) Python package. Stdlib-only. Truncates at Telegram's 4096-char limit with an `…(truncated)` marker.

## Why this lives as a skill (not just a memory rule)

Memory tells the model "remember to do X." A skill is `frontmatter.description` matched against turn context, so the *decision to fire* happens automatically when the trigger phrases appear. Async pings need that automatic-fire behavior — relying on memory recall to remember to ping is the same failure pattern that caused the original RunPod credit incidents.
