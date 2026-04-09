# Cursor CLI Example Notes

Chinese version: [`README.zh-CN.md`](./README.zh-CN.md)

This folder documents the current `Cursor CLI` beta expectations for `VibeAgentIsland-archlinux`.

## What is currently verified

- live session visibility
- grouped section rendering
- status line capture
- jump
- mini terminal peek

## What is still treated as beta-only / not fully field-verified

- approval interaction end-to-end

The approval path is wired through `approvalMode = "default"` plus the installed `hooks.json`, but this release still documents Cursor approvals as needing more field verification before they are promoted to "fully reliable."

## Required files

- `~/.cursor/cli-config.json`
- `~/.cursor/hooks.json`

## Required settings

- keep `"approvalMode": "default"`
- keep the generated `statusLine` command
- keep the generated hooks file

## Practical advice

1. Run:

```bash
python tools/vibeisland.py install cursor
```

2. Fully close every already-open `cursor-agent` session.
3. Open a completely fresh terminal.
4. Start `cursor-agent` again.
5. Test live state and grouped rendering first.
6. Treat approval testing as beta-only until you confirm it on your own machine.

## Placeholder path reminder

The example JSON files use:

`/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py`

Replace that with the real absolute path on the target machine.
