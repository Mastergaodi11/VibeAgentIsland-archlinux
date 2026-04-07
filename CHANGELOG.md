# Changelog

All notable changes to `Vibe Island for Linux` should be documented in this file.

The format is intentionally lightweight for the beta stage.

## Unreleased

Post-beta fixes currently staged for the next public update:

- collaboration cards are no longer injected into the island unless collaboration is explicitly re-enabled
- pending approval requests are now overlaid into the shell session model so blocked prompts remain visible in the island even when Telegram already sees them
- session dedupe now also merges on stable session id aliases, which reduces duplicate cards for the same live terminal
- Codex `approval_policy = "never"` is now respected by the local bridge so normal implementation work no longer produces false approval cards in the island
- the shell now backfills live Claude / Codex / Gemini sessions from local process and transcript artifacts, which makes reopen detection more reliable even when daemon snapshots lag behind
- Gemini approvals now flow through the same provider-aware island + Telegram path as Claude and Codex when a managed approval request is present
- grouped provider headers now own their own `5H / 7D / RUN` strip instead of the old mixed top quota row
- session cards are moving to a compact three-line recent-dialogue layout with explicit `You` / provider labels
- a new `settings_exp/` package now carries ready-to-copy example configs for Claude, Codex, and Gemini
- Gemini island approvals now carry an explicit managed approval request key so in-island button clicks resolve the exact pending request instead of guessing by tty/pid
- Gemini `BeforeTool` hooks now use a long-lived timeout so the CLI can keep waiting while the island holds the approval UI open
- Gemini managed approvals now resolve against the exact live request first, and dead/stale Gemini approval request files are pruned instead of being reused accidentally
- the shell now drops live Claude / Codex / Gemini cards as soon as their backing process disappears, which keeps killed terminals from lingering in grouped sections
- Gemini installation now also writes a local `~/.local/bin/gemini` wrapper that launches Gemini CLI with `--approval-mode=yolo` unless you explicitly override approval flags, which prevents Gemini's native approval UI from fighting the island-managed approval flow
- the shell now treats provider cards as live-only surface by default, so dead Claude / Codex / Gemini processes no longer linger in the grouped session list
- the shell now stores automatic inactivity collapse settings as persistent prefs, and the settings panel exposes both the enable toggle and the delay in seconds
- the expanded panel now trims the empty gap above replay/session content when the settings panel is closed
- a dedicated portability roadmap now documents the Ubuntu 24.04 and Windows 11 follow-up path
- Gemini still does not expose a stable local `5H / 7D` quota window source, so the shell intentionally keeps Gemini quota output marked as unavailable instead of fabricating percentages

## 0.1.0-beta.1 - 2026-04-06

First public GitHub-ready beta release of the Linux island.

Highlights:

- one-command launcher through `python tools/vibeisland.py launch`
- desktop launcher installation through `python tools/vibeisland.py install-desktop`
- Claude Code and Codex CLI hook integration
- in-island approval and reply UX for core flows
- terminal jump-back and mini terminal peek
- Telegram remote approval bridge
- provider usage HUD, replay timeline, idle/sleep collapsed state
- public export flow through `python tools/vibeisland.py export-public --output <dir>`

Known limits:

- first-class environment is still Arch Linux / KDE Plasma / Wayland / Konsole
- always-on-top / pin behavior remains best-effort under Wayland compositor policy
- the private Claude+Codex collaboration runtime is intentionally excluded from this public release
