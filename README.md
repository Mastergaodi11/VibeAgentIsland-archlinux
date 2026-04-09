# VibeAgentIsland-archlinux

中文说明：[`README.zh-CN.md`](./README.zh-CN.md)

`VibeAgentIsland-archlinux` is the public Arch Linux / KDE Plasma / Wayland / Konsole release repository for the local-first Vibe Island desktop shell. It watches live Claude Code, Codex, Gemini CLI, Cursor CLI, and OpenCode sessions, pulls urgent approvals back into view, lets you jump to the right terminal, and keeps the core workflow on your own machine.

## Release Status

Current public release target: `0.1.0-beta.1`

This first GitHub release is a usable beta for real local workflows, but it is still opinionated toward the environment used during development and testing.

## Screenshots

Expanded grouped view:

![Expanded Vibe Island grouped view](./media/screenshots/beta1-expanded.png)

Collapsed notch view:

![Collapsed Vibe Island notch view](./media/screenshots/beta1-collapsed.png)

Cursor + OpenCode support in the current beta:

![Cursor and OpenCode provider support](./media/screenshots/beta1-cursor-opencode.png)

## Provider Status in This Beta

- `Claude Code`: supported and daily-driver ready on the primary environment
- `Codex CLI`: supported and daily-driver ready on the primary environment
- `Gemini CLI`: minimum-viable support is in place for live sessions, grouped cards, jump, peek, island approvals, and Telegram approvals
- `OpenCode`: live state, grouped cards, jump, peek, Telegram prompts, and in-island approval interaction have been field-verified on the primary environment
- `Cursor CLI`: live state, grouped cards, jump, peek, and status capture are integrated; approval interaction is wired, but it is still marked as not fully field-verified in this beta

## What It Does

- monitors multiple Claude Code / Codex / Gemini CLI / Cursor CLI / OpenCode sessions in one floating shell
- groups expanded sessions by provider so Claude, Codex, Gemini, Cursor, and OpenCode work stay visually separated
- places `5H / 7D / RUN` usage directly in each provider section header for providers that expose quota windows locally, and keeps `RUN`-only headers for Cursor/OpenCode
- expands for real approvals and user questions
- supports in-island approval, reply, mini terminal peek, and jump-back actions
- shows provider usage HUD, replay timeline, idle/sleep states, and Telegram remote approval as optional extras
- includes an optional `Aero Glass` appearance mode with a saved transparency slider for a softer frosted shell
- can auto-front not only the island itself, but also the exact approval card that needs attention so urgent prompts are easier to answer quickly
- keeps event flow local through a Rust daemon, Unix socket, SQLite state, and a PyQt6/QML shell
- keeps Claude / Codex / Gemini / Cursor / OpenCode rendered as provider-grouped sections instead of one mixed wall of sessions
- shows compact three-line dialogue cards with recent `You` / `Agent` context instead of a single generic summary line

## First-Class Environment

Current first-class support is:

- Arch Linux / EndeavourOS
- KDE Plasma 6
- Wayland
- Konsole

Other Linux desktops and terminals are still best-effort.

## Requirements

- Python 3.12 or newer
- Rust toolchain with `cargo`
- PyQt6 available in your Python environment
- Claude Code and/or Codex CLI and/or Gemini CLI and/or Cursor CLI and/or OpenCode installed locally
- Arch Linux / KDE Plasma / Wayland / Konsole if you want the most tested path

## Quick Start

1. Install the Claude Code / Codex / Gemini / Cursor / OpenCode bridge integrations:

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py install all
```

2. Start the island with one command:

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py launch
```

3. Optional: install the desktop launcher:

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py install-desktop
```

After that you can use:

```bash
vibeisland
vibeisland status
vibeisland stop
```

## Detailed Install: Arch Linux + KDE Plasma + Konsole

If your machine is close to the primary test environment used for this project, this is the recommended step-by-step path:

1. Install the base runtime dependencies:

```bash
sudo pacman -S --needed git python python-pip rustup base-devel qt6-base qt6-declarative qt6-multimedia
rustup default stable
python -m pip install --user PyQt6
```

2. Clone the repository and enter it:

```bash
git clone <your-repo-url> VibeAgentIsland-archlinux
cd VibeAgentIsland-archlinux
```

3. Log in to the providers you actually use:

- `claude` for Claude Code
- `codex` for Codex CLI
- `gemini` for Gemini CLI
- `cursor-agent` for Cursor CLI
- `opencode` for OpenCode

4. If you want example config files instead of editing everything from scratch, start from:

- [`settings_exp/README.md`](./settings_exp/README.md)
- [`settings_exp/README.zh-CN.md`](./settings_exp/README.zh-CN.md)

5. Install the Vibe Island hooks into the local provider configs:

```bash
python tools/vibeisland.py install all
```

6. Restart every provider CLI that was already open. This matters, especially for Gemini and Claude, because old processes keep the previous hook configuration until they are relaunched.

7. Launch the island:

```bash
python tools/vibeisland.py launch
```

8. Optional: install the desktop launcher so KDE application search and the `vibeisland` command both work:

```bash
python tools/vibeisland.py install-desktop
```

After that, the normal daily commands are:

```bash
vibeisland
vibeisland status
vibeisland stop
```

## Beta 1 Highlights

- one-command launcher for daemon + shell
- desktop launcher installation through `install-desktop`
- Claude Code + Codex + Gemini + Cursor + OpenCode bridge layer
- provider-grouped expanded view for Claude / Codex / Gemini / Cursor / OpenCode
- in-island approvals and replies for core flows
- jump-back and mini terminal peek
- Telegram remote approval bridge
- provider usage HUD, replay timeline, and idle/sleep presentation
- optional `Aero Glass` appearance mode with saved transparency
- approval auto-front now scrolls directly to the card that triggered the prompt

## Configuration Notes

- Claude Code config lives in `~/.claude/settings.json`
- Codex config lives in `~/.codex/config.toml` and `~/.codex/hooks.json`
- Gemini config lives in `~/.gemini/settings.json`
- Cursor config lives in `~/.cursor/cli-config.json` and `~/.cursor/hooks.json`
- OpenCode config lives in `~/.config/opencode/opencode.json`
- ready-to-copy example configs live under [`settings_exp/`](./settings_exp/)
- OpenCode setup details and pitfalls are documented again in [`settings_exp/opencode/README.md`](./settings_exp/opencode/README.md)
- Cursor setup notes and the current approval-status caveat are documented in [`settings_exp/cursor/README.md`](./settings_exp/cursor/README.md)
- Switching Claude between OAuth and API key modes must not remove the `hooks` or `statusLine` sections
- Codex must keep `approval_policy = "never"`, `notify`, and `features.codex_hooks = true`
- Gemini must keep the installed `hooks` block in `~/.gemini/settings.json`
- Gemini should also be launched through the local `~/.local/bin/gemini` wrapper written by `python tools/vibeisland.py install gemini`
- that wrapper injects `--approval-mode=yolo` unless you explicitly pass your own approval flags, which prevents Gemini's native approval UI from re-appearing after the island already handled the decision
- make sure `~/.local/bin` is ahead of any NVM or system Gemini binary in `PATH`
- Cursor must keep both the `statusLine` block and `~/.cursor/hooks.json`, and should keep `approvalMode = "default"` so approval checkpoints still exist for Vibe Island to surface
- OpenCode must keep the local `node_modules/vibeisland-opencode-plugin` entry in `~/.config/opencode/opencode.json`
- Cursor and OpenCode do not currently advertise stable local `5H / 7D` quota windows, so the shell only shows `RUN` activity for those providers
- Telegram is optional and should never be required for the island to run

Detailed setup instructions:

- [`docs/INTEGRATION_SETUP.md`](./docs/INTEGRATION_SETUP.md)
- [`docs/INTEGRATION_SETUP.zh-CN.md`](./docs/INTEGRATION_SETUP.zh-CN.md)
- [`settings_exp/README.md`](./settings_exp/README.md)
- [`settings_exp/README.zh-CN.md`](./settings_exp/README.zh-CN.md)

## Setup Gotchas

- After `python tools/vibeisland.py install all`, fully restart every already-open `claude`, `codex`, `gemini`, `cursor-agent`, and `opencode` terminal. Old sessions keep old hooks until they are relaunched.
- Gemini must resolve to the wrapper written by `python tools/vibeisland.py install gemini`. If a terminal still behaves like the old binary, close that terminal completely and open a fresh one, or run `hash -r`.
- Keep `~/.local/bin` ahead of NVM or system Gemini paths in `PATH`, otherwise Gemini can bypass the wrapper and fall back to its own native approval UI.
- Cursor approvals rely on `approvalMode = "default"` plus the installed `hooks.json`; if Cursor does not surface approval checkpoints in the island, reinstall with `python tools/vibeisland.py install cursor` and restart the CLI. In this beta, Cursor approval completion is still treated as needing more field verification before it is documented as fully reliable.
- OpenCode approvals rely on the generated local plugin and have been field-verified on the primary environment. If approvals never reach the island, reinstall with `python tools/vibeisland.py install opencode`, confirm the generated plugin still exists, and relaunch `opencode` from a completely fresh terminal.
- OpenCode setup is intentionally more explicit than the other providers because it depends on a generated local plugin. Follow [`settings_exp/opencode/README.md`](./settings_exp/opencode/README.md) step by step instead of editing only `opencode.json` and guessing the rest.
- Switching Claude between browser OAuth and API key mode must not remove the `hooks` or `statusLine` blocks from `~/.claude/settings.json`.
- Codex integration depends on `approval_policy = "never"`, `notify`, and `features.codex_hooks = true`; removing any of them will break island-managed approvals or status reporting.
- Gemini currently does not expose a stable local `5H / 7D` quota window source, so `5H N/A / 7D N/A` is expected and honest, not a rendering bug.
- Cursor and OpenCode also intentionally skip `5H / 7D` quota output for now; this is expected behavior, not a missing render.
- Under KDE Plasma + Wayland, `pin` / always-on-top is still best-effort. The tray summon path is the reliable fallback if another window still covers the island.
- The grouped session list is live-only. If a provider process was closed while the shell was already running and a stale card remains for a moment, restart the shell with `vibeisland stop && vibeisland`.
- `settings_exp/` files are starter examples only. Users still need to fill auth values or complete CLI login on their own machine.

## Documentation

- [`docs/README.md`](./docs/README.md)
- [`docs/README.zh-CN.md`](./docs/README.zh-CN.md)
- [`docs/PRD.md`](./docs/PRD.md)
- [`docs/TECHNICAL_DESIGN.md`](./docs/TECHNICAL_DESIGN.md)
- [`docs/PORTABILITY_ROADMAP.md`](./docs/PORTABILITY_ROADMAP.md)
- [`CHANGELOG.md`](./CHANGELOG.md)
- [`CONTRIBUTING.md`](./CONTRIBUTING.md)
- [`SECURITY.md`](./SECURITY.md)

## Media

- Screenshots and release/demo assets should live under [`media/`](./media/)

## Public Open-Source Boundary

This repository is being prepared as the public `VibeAgentIsland-archlinux` project.

The local Claude+Codex collaboration runtime remains a separate private/local project and is not part of the public release promise for the island itself. The island must keep working even when that collaboration runtime is not installed.

## Known Beta Limitations

- first-class behavior is still tuned for KDE Plasma + Wayland + Konsole
- Gemini support is intentionally minimal in `beta1`: live state, approvals/questions, jump, Telegram, and peek are the target surface
- OpenCode support is intentionally local-first but field-verified on the primary environment for live state, approvals/questions, jump, Telegram, peek, and grouped UI
- Cursor support is intentionally local-first for live state, grouped cards, jump, peek, and status capture, but approval interaction still needs more field verification before it is treated as a guaranteed beta workflow
- Gemini 5h / 7d quota windows are still not exposed by a stable local CLI state source, so the island currently shows `Unavailable` instead of fabricating percentages
- the grouped header layout now keeps per-provider quota text local to each section, while the top notch stays focused on global state
- the grouped session list is now treated as a live-only surface; once a Claude / Codex / Gemini / Cursor / OpenCode process disappears, its main card should drop out and only history remains in replay
- Ubuntu 24.04 and Windows 11 are future targets; the current codebase is being cleaned up for portability, but this beta still only promises Linux-first behavior
- pin / always-on-top remains best-effort under Wayland compositor policy
- terminal jump precision outside the tested environment is still improving
- UI polish is already strong enough for daily use, but not final

## License

MIT, see [`LICENSE`](./LICENSE).
