# How We Work on VACUUM_TOOLKIT

Simple guide for Chris + Uncle. No coding needed.

## The loop (every time you want a change)

1. **Fetch first** — Open GitHub Desktop → click **"Fetch origin"** (top bar). Gets the latest so you two never clash.
2. **Tell Claude the change** — In Claude Cowork, say in plain English what you want (e.g. "add a reset button", "the total should multiply by 2"). Claude edits the file for you. You don't touch code.
3. **Push when happy** — Back in GitHub Desktop: type a short note (e.g. "added reset button") → **"Commit to main"** → **"Push origin"**. Now it's live on GitHub.

That's it: **Fetch → tell Claude → Commit & Push.**

## The one rule (Chris + Uncle)

Only ONE person edits at a time.
- Whoever is working: say so, do your changes, push when done.
- The other person: click **"Fetch origin"** BEFORE you start, so you have their latest.

This avoids the #1 beginner headache (two people changing the same file).

## Which Claude model to use

- **Sonnet 5** — default. Smart + cheap. Use for almost everything.
- **Opus 4.8** — only when Sonnet gets stuck on something hard. Smartest, most expensive.
- **Haiku 4.5** — tiny stuff (typo, color change). Cheapest.

Switch models with the picker in the chat box.

## Where to talk to Claude

- Always use **Claude Cowork** (the desktop app) — it's the only place Claude can edit these files directly.
- The folder stays connected; no re-setup each time.
- Start a **new chat for each new task** — cheaper on tokens, and Claude reads this file if it needs a reminder.

## Token-saving tips

- Describe the change clearly in one message instead of many small ones.
- New chat per task (don't let one chat run forever).
- Stay on Sonnet unless you truly need Opus.
