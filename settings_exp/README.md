# Example Integration Settings

Chinese version: [`README.zh-CN.md`](./README.zh-CN.md)

Maintenance note: whenever Vibe Island changes the required hooks or provider integration fields, update these example files and both README editions together.

This folder contains starter configuration files for `Claude Code`, `Codex`, `Gemini CLI`, `Cursor CLI`, and `OpenCode`.

Use them as copy-and-edit examples when setting up another machine.

## What These Files Include

- prewired Vibe Island hooks
- Claude `statusLine` for quota HUD
- Codex `notify` and `approval_policy = "never"`
- Gemini hook scaffolding
- Cursor `statusLine`, hooks, and default approval mode
- OpenCode plugin registration

## What These Files Do Not Include

- real OAuth credentials
- real API keys
- private trust lists or project-specific paths

You must fill or adjust the login/authentication part yourself.

## Included Files

- `claude/settings.oauth.json`
  Example `~/.claude/settings.json` for browser OAuth login.
- `claude/settings.apikey.json`
  Example `~/.claude/settings.json` for API key / compatible endpoint mode.
- `codex/config.toml`
  Example `~/.codex/config.toml`.
- `codex/hooks.json`
  Example `~/.codex/hooks.json`.
- `gemini/settings.json`
  Example `~/.gemini/settings.json`.
- `cursor/cli-config.json`
  Example `~/.cursor/cli-config.json`.
- `cursor/hooks.json`
  Example `~/.cursor/hooks.json`.
- `cursor/README.md`
  Cursor-specific notes, known caveats, and the current beta verification status.
- `cursor/README.zh-CN.md`
  Chinese Cursor-specific notes.
- `opencode/opencode.json`
  Example `~/.config/opencode/opencode.json`.
- `opencode/README.md`
  OpenCode-specific setup notes, known pitfalls, and the verified approval flow.
- `opencode/README.zh-CN.md`
  Chinese OpenCode-specific notes.

## Important Notes

### Claude Code

- OAuth login and API key login are separate examples.
- Keep the `hooks` section.
- Keep the `statusLine` section.
- Switching auth mode must not remove the Vibe Island integration pieces.

### Codex

- Codex login is usually handled by the CLI itself, not by these files.
- `approval_policy = "never"` is intentional because the island manages approval UX.
- `notify` and `features.codex_hooks = true` must stay enabled.

### Gemini CLI

- Gemini login is usually completed through the CLI itself.
- The example file focuses on the `hooks` block.
- If your Gemini installation stores auth outside `settings.json`, that is expected.
- `BeforeTool` intentionally uses a much longer timeout than the other Gemini hook events because it needs to stay alive while the island waits for a human approval choice.
- Public installs should also use the local `gemini` wrapper written to `~/.local/bin/gemini` by `python tools/vibeisland.py install gemini`.
- That wrapper starts Gemini with `--approval-mode=yolo` unless you explicitly pass your own approval flags, which is required to prevent Gemini's native approval UI from re-appearing after the island already handled the decision.
- When Gemini is installed through NVM, the installer also shims the active NVM `bin/gemini` entrypoint so older shells do not bypass the wrapper accidentally.
- Make sure `~/.local/bin` is ahead of any NVM or system Gemini binary in `PATH`. If a terminal still behaves like the old binary, close that terminal completely and open a fresh one.

### Cursor CLI

- Cursor login is handled by the CLI itself.
- Keep `approvalMode = "default"` so Cursor still emits approval checkpoints that Vibe Island can surface and manage.
- Keep the `statusLine` block and the `hooks.json` file together; both are required.
- Cursor does not currently expose a reliable local 5h / 7d quota window, so Vibe Island only shows run/session activity for Cursor.
- Cursor live sessions, grouping, jump, peek, and status capture are in the supported beta surface.
- Cursor approval interaction is wired, but it is still not documented as fully field-verified in this beta. Treat it as experimental until you confirm it on your own machine.
- Read `cursor/README.md` for the provider-specific notes before declaring Cursor approval flow "done."

### OpenCode

- OpenCode login is handled by the CLI itself.
- The integration works by loading the local `node_modules/vibeisland-opencode-plugin` plugin path, so the `plugin` entry must stay present exactly as shown.
- OpenCode does not currently expose a reliable local 5h / 7d quota window, so Vibe Island only shows run/session activity for OpenCode.
- If OpenCode is upgraded and its plugin API changes, update both this example file and the generated plugin from `python tools/vibeisland.py install opencode`.
- OpenCode approval interaction has been field-verified on the primary Arch Linux / KDE Plasma 6 / Wayland / Konsole environment.
- Do not treat `opencode.json` as the whole setup. You must also let `python tools/vibeisland.py install opencode` generate the local plugin tree and then restart every already-open `opencode` terminal.
- Read `opencode/README.md` before testing OpenCode approvals. That file documents the real failure modes we hit while bringing the plugin bridge online.

## Recommended Setup Flow

1. Copy the relevant example file to the real config path.
2. Replace placeholder auth values or complete CLI login.
3. Keep the hook commands pointing at your local `vibeisland.py`.
4. Restart the provider CLI.
5. Launch Vibe Island and verify the island sees sessions and approvals.

## Placeholder Path Reminder

These example files use:

`/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py`

Replace it with the real absolute path on the target machine.
