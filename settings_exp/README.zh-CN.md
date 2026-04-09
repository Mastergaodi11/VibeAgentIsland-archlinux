# 示例配置包

英文版：[`README.md`](./README.md)

维护说明：以后只要灵动岛要求的 hooks、statusLine 或 provider 集成字段发生变化，就必须同步更新本目录中的示例文件以及中英文两份 README。

这个目录放的是 `Claude Code`、`Codex`、`Gemini CLI`、`Cursor CLI` 和 `OpenCode` 的示例配置文件，方便用户在另一台机器上照着填写。

## 这些示例文件已经包含什么

- 灵动岛所需的 hooks
- Claude 配额 HUD 需要的 `statusLine`
- Codex 的 `notify` 与 `approval_policy = "never"`
- Gemini 的 hooks 基础结构
- Cursor 的 `statusLine`、hooks 与 default 审批模式
- OpenCode 的本地 plugin 注册

## 这些示例文件不会包含什么

- 真实 OAuth 凭据
- 真实 API key
- 私有 trust 列表
- 你本机的项目路径

也就是说，登录或鉴权相关部分需要用户自己填写或自己完成 CLI 登录。

## 目录内容

- `claude/settings.oauth.json`
  用于 `~/.claude/settings.json` 的浏览器 OAuth 示例。
- `claude/settings.apikey.json`
  用于 `~/.claude/settings.json` 的 API key / 兼容端点模式示例。
- `codex/config.toml`
  用于 `~/.codex/config.toml` 的示例。
- `codex/hooks.json`
  用于 `~/.codex/hooks.json` 的示例。
- `gemini/settings.json`
  用于 `~/.gemini/settings.json` 的示例。
- `cursor/cli-config.json`
  用于 `~/.cursor/cli-config.json` 的示例。
- `cursor/hooks.json`
  用于 `~/.cursor/hooks.json` 的示例。
- `cursor/README.md`
  Cursor 的专项说明、当前 beta 已知边界与配置注意点。
- `cursor/README.zh-CN.md`
  Cursor 专项说明中文版。
- `opencode/opencode.json`
  用于 `~/.config/opencode/opencode.json` 的示例。
- `opencode/README.md`
  OpenCode 的专项说明、踩坑记录与正确配置流程。
- `opencode/README.zh-CN.md`
  OpenCode 专项说明中文版。

## 重要说明

### Claude Code

- OAuth 登录和 API key 登录是两份不同示例。
- `hooks` 段必须保留。
- `statusLine` 段必须保留。
- 以后切换登录方式时，不能把灵动岛相关段落删掉。

### Codex

- Codex 的登录通常不是靠这些文件，而是靠 CLI 自己的登录流程。
- `approval_policy = "never"` 是故意这样配置的，因为审批交互由灵动岛接管。
- `notify` 和 `features.codex_hooks = true` 必须保留。

### Gemini CLI

- Gemini 的登录通常也是在 CLI 自己那里完成。
- 这个示例文件主要关注 `hooks`。
- 如果你的 Gemini 安装把登录信息放在别处，而不是 `settings.json`，这属于正常情况。
- `BeforeTool` 会故意使用比其它 Gemini hook 更长的 timeout，因为它必须在灵动岛等待人工审批选择的这段时间里一直存活。
- 公开版安装时，还应该使用 `python tools/vibeisland.py install gemini` 写入本地 `~/.local/bin/gemini` wrapper。
- 这个 wrapper 会在你没有显式传入审批参数时自动以 `--approval-mode=yolo` 启动 Gemini，避免灵动岛已经处理完审批后，Gemini 自己又弹出一层原生审批界面。
- 如果 Gemini 是通过 NVM 安装的，安装器还会顺手接管当前 NVM 里的 `bin/gemini` 入口，这样旧终端也不会意外绕开 wrapper。
- 请确保 `PATH` 里 `~/.local/bin` 的优先级高于 NVM 或系统自带的 Gemini 二进制。如果某个终端表现得还像旧版本，请彻底关掉那个终端再重新开。

### Cursor CLI

- Cursor 的登录通常也是由 CLI 自己完成。
- `approvalMode = "default"` 必须保留，这样 Cursor 仍会发出审批检查点，而灵动岛可以接管并展示这些审批交互。
- `statusLine` 和 `hooks.json` 要一起保留，少一个都会影响灵动岛能力。
- Cursor 目前没有稳定可信的本地 `5h / 7d` 配额窗口来源，所以灵动岛只会显示运行次数或会话活动，不会伪造窗口百分比。
- Cursor 的 live 会话、分组卡片、jump、peek 和状态采集已经在 beta 支持范围里。
- Cursor 的审批交互虽然已经接线，但这一版仍然不把它写成“完全实测通过”，请先按实验性能力来对待，并阅读 `cursor/README.zh-CN.md`。

### OpenCode

- OpenCode 的登录同样通常由 CLI 自己完成。
- 这个集成是通过加载本地 `node_modules/vibeisland-opencode-plugin` 插件路径实现的，所以 `plugin` 里必须按示例保留这一条。
- OpenCode 目前也没有稳定可信的本地 `5h / 7d` 配额窗口来源，所以灵动岛只会显示运行次数或会话活动，不会伪造窗口百分比。
- 如果 OpenCode 将来升级了 plugin API，记得同时更新这个示例文件以及 `python tools/vibeisland.py install opencode` 生成出来的本地插件。
- OpenCode 的审批交互已经在当前一等支持环境下做过实测验证。
- OpenCode 的正确配置不止是修改 `opencode.json`，还必须让 `python tools/vibeisland.py install opencode` 生成本地 plugin，并且把所有旧的 `opencode` 终端全部关掉再重开。
- 真正的踩坑点和正确流程都写在 `opencode/README.zh-CN.md` 里，配置 OpenCode 时请优先看那份说明。

## 推荐配置流程

1. 先把对应示例文件复制到真实配置路径。
2. 填好占位的鉴权字段，或者先完成 CLI 登录。
3. 确认 hook 命令指向你本机真实的 `vibeisland.py` 绝对路径。
4. 重启对应 provider CLI。
5. 再启动灵动岛，验证会话和审批是否能被识别。

## 路径占位提醒

这些示例文件里默认使用的是：

`/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py`

用户在真实机器上必须把它替换成实际的绝对路径。
