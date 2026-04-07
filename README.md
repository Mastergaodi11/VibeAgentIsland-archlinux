# Vibe Island for Linux

中文说明：[`README.zh-CN.md`](./README.zh-CN.md)

Vibe Island for Linux is a local-first floating “agent island” for Linux desktops. It watches live Claude Code, Codex, and Gemini CLI sessions, pulls urgent approvals back into view, lets you jump to the right terminal, and keeps the core workflow on your own machine.

## Release Status

Current public release target: `0.1.0-beta.1`

This first GitHub release is a usable beta for real local workflows, but it is still opinionated toward the environment used during development and testing.

## Screenshots

Expanded grouped view:

![Expanded Vibe Island grouped view](./media/screenshots/beta1-expanded.png)

Collapsed notch view:

![Collapsed Vibe Island notch view](./media/screenshots/beta1-collapsed.png)

## What It Does

- monitors multiple Claude Code / Codex / Gemini CLI sessions in one floating shell
- groups expanded sessions by provider so Claude, Codex, and Gemini work stay visually separated
- places `5H / 7D / RUN` usage directly in each provider section header instead of one mixed strip
- expands for real approvals and user questions
- supports in-island approval, reply, mini terminal peek, and jump-back actions
- shows provider usage HUD, replay timeline, idle/sleep states, and Telegram remote approval as optional extras
- keeps event flow local through a Rust daemon, Unix socket, SQLite state, and a PyQt6/QML shell
- keeps Claude / Codex / Gemini rendered as provider-grouped sections instead of one mixed wall of sessions
- shows compact three-line dialogue cards with recent `You` / `Agent` context instead of a single generic summary line

## First-Class Environment

Current first-class support is:

- Arch Linux / EndeavourOS
- KDE Plasma
- Wayland
- Konsole

Other Linux desktops and terminals are still best-effort.

## Requirements

- Python 3.12 or newer
- Rust toolchain with `cargo`
- PyQt6 available in your Python environment
- Claude Code and/or Codex CLI and/or Gemini CLI installed locally
- Arch Linux / KDE Plasma / Wayland / Konsole if you want the most tested path

## Quick Start

1. Install the Claude Code / Codex / Gemini bridge hooks:

```bash
cd /path/to/vibeisland-linux
python tools/vibeisland.py install all
```

2. Start the island with one command:

```bash
cd /path/to/vibeisland-linux
python tools/vibeisland.py launch
```

3. Optional: install the desktop launcher:

```bash
cd /path/to/vibeisland-linux
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
git clone <your-repo-url> vibeisland-linux
cd vibeisland-linux
```

3. Log in to the providers you actually use:

- `claude` for Claude Code
- `codex` for Codex CLI
- `gemini` for Gemini CLI

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
- Claude Code + Codex + Gemini hooks bridge
- provider-grouped expanded view for Claude / Codex / Gemini
- in-island approvals and replies for core flows
- jump-back and mini terminal peek
- Telegram remote approval bridge
- provider usage HUD, replay timeline, and idle/sleep presentation

## Configuration Notes

- Claude Code config lives in `~/.claude/settings.json`
- Codex config lives in `~/.codex/config.toml` and `~/.codex/hooks.json`
- Gemini config lives in `~/.gemini/settings.json`
- ready-to-copy example configs live under [`settings_exp/`](./settings_exp/)
- Switching Claude between OAuth and API key modes must not remove the `hooks` or `statusLine` sections
- Codex must keep `approval_policy = "never"`, `notify`, and `features.codex_hooks = true`
- Gemini must keep the installed `hooks` block in `~/.gemini/settings.json`
- Gemini should also be launched through the local `~/.local/bin/gemini` wrapper written by `python tools/vibeisland.py install gemini`
- that wrapper injects `--approval-mode=yolo` unless you explicitly pass your own approval flags, which prevents Gemini's native approval UI from re-appearing after the island already handled the decision
- make sure `~/.local/bin` is ahead of any NVM or system Gemini binary in `PATH`
- Telegram is optional and should never be required for the island to run

Detailed setup instructions:

- [`docs/INTEGRATION_SETUP.md`](./docs/INTEGRATION_SETUP.md)
- [`docs/INTEGRATION_SETUP.zh-CN.md`](./docs/INTEGRATION_SETUP.zh-CN.md)
- [`settings_exp/README.md`](./settings_exp/README.md)
- [`settings_exp/README.zh-CN.md`](./settings_exp/README.zh-CN.md)

## Setup Gotchas

- After `python tools/vibeisland.py install all`, fully restart every already-open `claude`, `codex`, and `gemini` terminal. Old sessions keep old hooks until they are relaunched.
- Gemini must resolve to the wrapper written by `python tools/vibeisland.py install gemini`. If a terminal still behaves like the old binary, close that terminal completely and open a fresh one, or run `hash -r`.
- Keep `~/.local/bin` ahead of NVM or system Gemini paths in `PATH`, otherwise Gemini can bypass the wrapper and fall back to its own native approval UI.
- Switching Claude between browser OAuth and API key mode must not remove the `hooks` or `statusLine` blocks from `~/.claude/settings.json`.
- Codex integration depends on `approval_policy = "never"`, `notify`, and `features.codex_hooks = true`; removing any of them will break island-managed approvals or status reporting.
- Gemini currently does not expose a stable local `5H / 7D` quota window source, so `5H N/A / 7D N/A` is expected and honest, not a rendering bug.
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

This repository is being prepared as the public `vibeisland-linux` project.

The local Claude+Codex collaboration runtime remains a separate private/local project and is not part of the public release promise for the island itself. The island must keep working even when that collaboration runtime is not installed.

## Known Beta Limitations

- first-class behavior is still tuned for KDE Plasma + Wayland + Konsole
- Gemini support is intentionally minimal in `beta1`: live state, approvals/questions, jump, Telegram, and peek are the target surface
- Gemini 5h / 7d quota windows are still not exposed by a stable local CLI state source, so the island currently shows `Unavailable` instead of fabricating percentages
- the grouped header layout now keeps per-provider quota text local to each section, while the top notch stays focused on global state
- the grouped session list is now treated as a live-only surface; once a Claude / Codex / Gemini process disappears, its main card should drop out and only history remains in replay
- Ubuntu 24.04 and Windows 11 are future targets; the current codebase is being cleaned up for portability, but this beta still only promises Linux-first behavior
- pin / always-on-top remains best-effort under Wayland compositor policy
- terminal jump precision outside the tested environment is still improving
- UI polish is already strong enough for daily use, but not final

## License

MIT, see [`LICENSE`](./LICENSE).
