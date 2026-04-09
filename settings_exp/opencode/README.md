# OpenCode Example Notes

Chinese version: [`README.zh-CN.md`](./README.zh-CN.md)

This folder documents the real setup path for `OpenCode` in `VibeAgentIsland-archlinux`.

## What is currently field-verified

- live session visibility
- grouped section rendering
- jump
- mini terminal peek
- Telegram prompts
- in-island approval / question interaction

These were field-verified on the primary environment:

- Arch Linux
- KDE Plasma 6
- Wayland
- Konsole

## Required config file

- `~/.config/opencode/opencode.json`

## Required generated plugin

The config file alone is not enough.

You must also let the installer generate the local plugin tree:

```bash
python tools/vibeisland.py install opencode
```

After that, confirm these exist:

- `~/.config/opencode/opencode.json`
- `~/.config/opencode/node_modules/vibeisland-opencode-plugin/`

And make sure `opencode.json` still contains:

```json
{
  "plugin": [
    "node_modules/vibeisland-opencode-plugin"
  ]
}
```

## Real pitfalls we hit

- Editing only `opencode.json` is not enough; the generated local plugin must exist too.
- Already-open `opencode` terminals can keep stale plugin code loaded.
- If approvals never arrive in the island, reinstall and relaunch `opencode` from a completely fresh terminal.
- If OpenCode changes its plugin API in a future release, both this example file and the generated local plugin may need updates.

## Recommended setup flow

1. Run:

```bash
python tools/vibeisland.py install opencode
```

2. Confirm `~/.config/opencode/opencode.json` contains the plugin entry.
3. Confirm `~/.config/opencode/node_modules/vibeisland-opencode-plugin/` exists.
4. Fully close every already-open `opencode` terminal.
5. Open a completely fresh terminal and start `opencode` again.
6. Trigger a real approval prompt and verify the island sees it.

## Placeholder path reminder

The generated plugin itself points to:

`/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py`

If you copy examples manually, replace that placeholder with the real absolute path on the target machine.
