# Example Integration Settings

Chinese version: [`README.zh-CN.md`](./README.zh-CN.md)

Maintenance note: whenever Vibe Island changes the required hooks or provider integration fields, update these example files and both README editions together.

This folder contains starter configuration files for `Claude Code`, `Codex`, and `Gemini CLI`.

Use them as copy-and-edit examples when setting up another machine.

## What These Files Include

- prewired Vibe Island hooks
- Claude `statusLine` for quota HUD
- Codex `notify` and `approval_policy = "never"`
- Gemini hook scaffolding

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

## Recommended Setup Flow

1. Copy the relevant example file to the real config path.
2. Replace placeholder auth values or complete CLI login.
3. Keep the hook commands pointing at your local `vibeisland.py`.
4. Restart the provider CLI.
5. Launch Vibe Island and verify the island sees sessions and approvals.

## Placeholder Path Reminder

These example files use:

`/path/to/vibeisland-linux/tools/vibeisland.py`

Replace it with the real absolute path on the target machine.
