# Vibe Island Portability Roadmap

Chinese version: [`PORTABILITY_ROADMAP.zh-CN.md`](./PORTABILITY_ROADMAP.zh-CN.md)

Maintenance note: keep the English and Chinese editions in sync whenever this document changes.

This document is the handoff map for the next platform-expansion stage after the Linux-first beta. It does not claim that Ubuntu 24.04 or Windows 11 are fully supported today. Instead, it explains what is already portable, what is still Linux/KDE-specific, and where a follow-up implementation should start.

## Current Baseline

The current first-class environment remains:

- Arch Linux / EndeavourOS
- KDE Plasma
- Wayland
- Konsole

The codebase is already structured well enough that the next ports should reuse most of the provider bridge logic, event normalization, Telegram bridge, and shell state model.

## Portable Layers Already In Place

These pieces should carry across Ubuntu 24.04 and Windows 11 with minimal or moderate change:

- provider hook adapters in `tools/vibeisland.py`
- normalized event model in `crates/common`
- daemon snapshot model in `crates/daemon`
- shell view-model shaping in `apps/shell/main.py`
- provider-grouped UI structure in `apps/shell/ui/Main.qml`
- `settings_exp/` example integration package
- Telegram bridge and remote approval flow
- replay timeline, inactivity collapse, and grouped provider sections
- live process pruning in the shell, which now treats provider cards as real runtime surfaces instead of long-lived stale session snapshots

## Linux/KDE-Specific Layers Still Needing Abstraction

These areas are still biased toward the current machine and should be treated as adapter surfaces when porting:

- KWin-based window focusing and raise logic
- Konsole-specific DBus / session assumptions
- KDE tray and window-manager behavior
- Wayland-specific always-on-top compromises
- local process discovery heuristics that currently assume KDE/Konsole-heavy process trees
- Linux `/proc`-based liveness pruning, which will need a Windows-native replacement instead of direct `/proc` inspection

## Ubuntu 24.04 Path

Ubuntu 24.04 should be treated as the first expansion target because it can reuse the Linux daemon, Python shell, Unix socket flow, and most provider integration logic.

### Recommended order

1. Confirm baseline on:
   - Ubuntu 24.04
   - GNOME on Wayland
   - GNOME Terminal or Ptyxis
2. Replace KDE/KWin-specific jump assumptions with provider-neutral Linux jump adapters.
3. Re-test:
   - Claude approvals
   - Codex approvals
   - Gemini approvals
   - Telegram bridge
   - tray summon
4. Add Ubuntu-specific packaging and dependency notes.

### Most likely problem areas

- GNOME tray behavior
- focus/raise behavior under GNOME Wayland
- terminal identification when Konsole DBus is not available
- distro package names for Qt and Python dependencies

## Windows 11 Path

Windows 11 should be treated as a second-stage port.

### Reusable pieces

- provider adapters and hook installers
- event normalization and daemon logic
- grouped shell model and most of the QML structure
- Telegram bridge
- task labeling / replay / approval state machines

### Parts that will need new platform adapters

- process discovery helpers
- jump / focus / raise behavior
- shell launcher and desktop entry installation
- socket/path handling defaults
- terminal integration assumptions for:
  - Windows Terminal
  - PowerShell
  - Command Prompt
  - VS Code integrated terminal

### Windows-first handoff tasks

1. Replace hard-coded Unix paths and shell assumptions with platform helpers.
2. Introduce Windows path and config discovery helpers.
3. Implement jump providers for:
   - Windows Terminal
   - VS Code terminal
4. Replace Unix socket assumptions if needed with a Windows-friendly IPC transport while keeping the daemon/session model stable.

## Recommended Adapter Split

When the next Codex session starts actual port work, use this split:

- `provider adapters`
  Keep shared across Linux / Ubuntu / Windows whenever possible.
- `platform runtime adapters`
  Implement per-platform logic for:
  - process scanning
  - jump/focus
  - tray/launcher
  - filesystem/config defaults
- `shell UI`
  Keep mostly shared; only branch when the platform truly forces it.

## Practical Handoff Checklist

Before starting Ubuntu 24.04 or Windows 11 work, verify:

- `settings_exp/` still matches current hooks
- Claude, Codex, and Gemini example configs are up to date
- `python tools/vibeisland.py launch` works on the source Linux machine
- the public export still excludes collaboration runtime files
- docs clearly label Linux-first guarantees versus future targets

## Current Honest Boundary

As of this beta:

- Linux-first support is real
- Ubuntu 24.04 is a planned near-term port target
- Windows 11 is a planned later target
- macOS is explicitly not part of the target roadmap for this project
