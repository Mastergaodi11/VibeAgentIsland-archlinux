# Vibe Island 集成配置说明

英文版：`docs/INTEGRATION_SETUP.md`

维护说明：以后只要本文件或英文版发生变更，必须同步更新中英文两个版本。

本文档说明：在另一台 Linux 机器上，要让 `Vibe Island` 正常与 `Claude Code`、`Codex`、`Gemini CLI`、`Cursor CLI` 和 `OpenCode` 联动，需要哪些本地 CLI 配置。

## 目标

要让灵动岛正常工作，至少需要两类集成：

- `Claude Code` 需要通过 `~/.claude/settings.json` 把生命周期和审批事件转发出来
- 如果希望灵动岛显示 Claude 的 5 小时 / 7 天剩余额度，还需要通过 `statusLine` 暴露 `rate_limits`
- `Codex` 需要通过 `~/.codex/config.toml` 和 `~/.codex/hooks.json` 把生命周期事件转发出来
- `Gemini CLI` 需要通过 `~/.gemini/settings.json` 把生命周期 / 工具 / agent 事件转发出来
- `Gemini CLI` 目前没有稳定的本地 5 小时 / 7 天配额窗口来源，所以灵动岛只能显示 Gemini 的 transcript / session token 总量
- `Gemini CLI` 还应该通过 `python tools/vibeisland.py install gemini` 安装到 `~/.local/bin/gemini` 的本地 wrapper 启动，不然 Gemini 自己的原生审批模式会和灵动岛的审批托管互相打架
- `Cursor CLI` 需要同时保持 `~/.cursor/cli-config.json` 与 `~/.cursor/hooks.json` 和灵动岛安装器生成的内容一致
- `Cursor CLI` 还需要保留 `approvalMode = "default"`，这样 Cursor 会继续发出审批检查点，而灵动岛再统一承接这些交互
- `OpenCode` 需要在 `~/.config/opencode/opencode.json` 里加载本地 `node_modules/vibeisland-opencode-plugin`
- `Cursor` 与 `OpenCode` 当前都没有稳定可信的本地 5 小时 / 7 天配额窗口来源，所以界面只显示运行次数或会话活动
- `OpenCode` 的审批交互已经在一等支持环境下实测过，而 `Cursor` 的审批交互这一版仍按 beta / 尚未完全实测确认来说明

本项目已经提供了一键安装器：

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py install all
```

如果你更想手动配置，可以按下面的方式做。

如果你不想从零手写配置，也可以直接参考仓库里的示例配置包：

- `../settings_exp/README.md`
- `../settings_exp/README.zh-CN.md`

这个示例包会把认证相关字段留空或保留占位，但会预先把灵动岛需要的 hooks 和 `statusLine` 配好。

## Claude Code

配置文件：

- `~/.claude/settings.json`

### 浏览器 OAuth 登录

如果你希望 `Claude Code` 使用官方浏览器登录，而不是 API key 或自定义 token 模式：

- 不要在 `settings.json` 中设置 `ANTHROPIC_AUTH_TOKEN`
- 不要在 `settings.json` 中设置 `ANTHROPIC_BASE_URL`
- 建议设置：

```json
"forceLoginMethod": "claudeai"
```

推荐的认证相关片段如下：

```json
{
  "env": {
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"
  },
  "forceLoginMethod": "claudeai"
}
```

完成后，重新打开 `claude`，按照浏览器流程登录即可。

### Vibe Island 必需的 hooks

下面这些 hook 事件都应该调用：

```bash
/usr/bin/python "/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py" claude-hook
```

当前项目会安装这些 Claude 事件：

- `SessionStart`
- `UserPromptSubmit`
- `PreToolUse`
- `PermissionRequest`
- `Notification`
- `Elicitation`
- `Stop`
- `PostToolUse`
- `PostToolUseFailure`

示例结构：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume",
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python '/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py' claude-hook",
            "timeout": 8
          }
        ]
      }
    ]
  }
}
```

重要说明：

- 即便切换 Claude 的登录方式，也必须保留 `hooks` 段
- 浏览器 OAuth 与 hooks 是两套独立能力，切换登录方式不应该删除 hooks

### Claude 配额 HUD 必需的 `statusLine`

如果你希望灵动岛显示 Claude OAuth 的剩余额度百分比，还需要让 `Claude Code` 调用：

```bash
/usr/bin/python "/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py" claude-statusline
```

示例结构：

```json
{
  "statusLine": {
    "type": "command",
    "command": "/usr/bin/python '/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py' claude-statusline"
  }
}
```

说明：

- 这和 `hooks` 是分开的
- `hooks` 负责生命周期与审批事件
- `statusLine` 负责把 Claude 的 `rate_limits` 提供给灵动岛，以显示 5 小时和 7 天剩余额度
- 新增或修改 `statusLine` 后，必须重启一次 `claude`

## Codex

配置文件：

- `~/.codex/config.toml`
- `~/.codex/hooks.json`

### 必需的 `config.toml`

当前 `Vibe Island` 依赖这些键：

```toml
approval_policy = "never"
notify = ["/usr/bin/python", "/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py", "codex-notify"]

[features]
codex_hooks = true
```

说明：

- `approval_policy = "never"` 是因为审批体验由灵动岛接管
- `notify` 让灵动岛能观察完成和通知类事件
- `codex_hooks = true` 用来开启 hook 系统

### 必需的 `hooks.json`

下面这些事件应该调用：

```bash
/usr/bin/python "/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py" codex-hook
```

当前项目会安装这些 Codex 事件：

- `SessionStart`
- `UserPromptSubmit`
- `PreToolUse`
- `PermissionRequest`
- `PermissionDenied`
- `PostToolUse`
- `PostToolUseFailure`
- `Stop`
- `StopFailure`

示例结构：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume",
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python '/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py' codex-hook"
          }
        ]
      }
    ]
  }
}
```

## Gemini CLI

配置文件：

- `~/.gemini/settings.json`

### Gemini 必需的 hooks

下面这些 Gemini 事件应该调用：

```bash
/usr/bin/python "/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py" gemini-hook
```

当前项目会安装这些 Gemini 事件：

- `SessionStart`
- `SessionEnd`
- `BeforeTool`
- `AfterTool`
- `BeforeAgent`
- `AfterAgent`

示例结构：

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "/usr/bin/python '/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py' gemini-hook",
            "timeout": 600000
          }
        ]
      }
    ]
  }
}
```

说明：

- 当前公开版里的 Gemini 支持是“最小可用”路线
- 目标覆盖 live 会话可见、审批 / 提问、jump、Telegram 与 peek
- `BeforeTool` 在等待人工审批选择时必须持续存活，所以它的 timeout 会故意配得比其它 Gemini hook 长很多
- 安装器还会额外写一个本地 `~/.local/bin/gemini` wrapper；如果你没有显式传入审批参数，它会自动以 `--approval-mode=yolo` 启动 Gemini
- 如果没有这个 wrapper，灵动岛已经处理完审批后，Gemini 仍可能回退到自己的原生 `Action Required` 界面
- 如果当前 Gemini 是通过 NVM 安装的，安装器还会顺手接管对应的 NVM `bin/gemini` 入口，这样即使某些终端直接命中 NVM 路径，也会继续走灵动岛的 wrapper 行为
- 请确保 `PATH` 里 `~/.local/bin` 的优先级高于 NVM 或系统自带的 Gemini 二进制；如果某个终端表现得还像旧版本，请把那个终端彻底关掉再重新开
- Gemini 的额度 HUD 目前不在公开版承诺范围里
- 当前 Gemini CLI 的本地状态里还没有找到稳定的 `5H / 7D` 配额窗口来源，所以灵动岛会诚实显示 `Unavailable`，而不是伪造百分比

## Cursor CLI

配置文件：

- `~/.cursor/cli-config.json`
- `~/.cursor/hooks.json`

执行下面这条命令时，安装器会一起写入这两份文件：

```bash
python tools/vibeisland.py install cursor
```

必须保留：

- `approvalMode = "default"`
- 指向 `cursor-statusline` 的 `statusLine`
- 生成出来的 `hooks.json`

原因：

- 灵动岛依赖 Cursor hooks 来感知生命周期、shell 审批、MCP 活动和 subagent 状态
- Cursor 的 shell 审批应该由灵动岛统一托管，而不是再弹一层 Cursor 自己的原生审批
- Cursor 目前没有稳定可信的本地 `5H / 7D` 配额来源，所以分组标题只显示 `RUN`/会话活动
- 当前 beta 里，Cursor 的审批完成链路仍按“已经接线，但还需要继续现场实测”来说明，暂时不要把它写成完全稳定
- Cursor 的专项说明在 `../settings_exp/cursor/README.zh-CN.md`

## OpenCode

配置文件：

- `~/.config/opencode/opencode.json`

执行下面这条命令时，安装器会同时写入配置和本地 plugin：

```bash
python tools/vibeisland.py install opencode
```

必须保留：

- `plugin` 数组里的 `node_modules/vibeisland-opencode-plugin`

原因：

- OpenCode 的事件、权限请求和提问是通过本地 plugin 桥接到灵动岛里的
- OpenCode 目前同样没有稳定可信的本地 `5H / 7D` 配额来源，所以分组标题只显示 `RUN`/会话活动
- OpenCode 的审批交互已经在当前一等支持环境（Arch Linux / KDE Plasma 6 / Wayland / Konsole）下实测通过
- 不要把 `opencode.json` 当作完整配置；本地生成出来的 plugin 目录同样是必需的
- OpenCode 的专项说明在 `../settings_exp/opencode/README.zh-CN.md`

## 在新机器上的推荐配置流程

1. 克隆或复制 `VibeAgentIsland-archlinux`
2. 把文档示例中的绝对路径替换为本机真实路径
3. 执行：

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py install all
```

4. 如果使用 Claude 浏览器登录，确认 `~/.claude/settings.json` 中不包含：

- `ANTHROPIC_AUTH_TOKEN`
- `ANTHROPIC_BASE_URL`

## 可移植性说明

当前一等支持环境仍然是：

- Arch Linux / EndeavourOS
- KDE Plasma
- Wayland
- Konsole

同时，当前代码结构也正在清理成更利于后续移植的形态，后续目标包括：

- Ubuntu 24.04
- Windows 11

5. 增加或保留：

```json
"forceLoginMethod": "claudeai"
```

6. 如果本机安装了这些 provider，也一并重启 `claude`、`codex`、`gemini`、`cursor-agent` 和 `opencode`
7. 通过统一启动器启动 Vibe Island：

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py launch
```

## 常见安装与配置雷点

- 执行 `python tools/vibeisland.py install all` 之后，请把已经开着的 `claude`、`codex`、`gemini`、`cursor-agent`、`opencode` 终端全部彻底关掉，再开新的终端会话。旧 provider 进程会继续沿用旧 hooks 和旧启动参数。
- Gemini 的审批链路只有在当前 `gemini` 命令命中了 Vibe Island 安装到 `~/.local/bin/gemini` 的 wrapper，或者命中了已经被接管的 NVM 入口时才会稳定工作。如果审批又退回 Gemini 自己的 `Action Required`，请彻底关掉那个终端，再开一个全新终端，并确认 `PATH` 里 `~/.local/bin` 的优先级最高。
- 如果某个 shell 还记着旧的 Gemini 二进制路径，执行一次 `hash -r`，或者直接换一个新终端后再测审批。
- Cursor 的审批链路依赖 `approvalMode = "default"` 和安装进去的 `hooks.json`。如果灵动岛里仍然看不到 Cursor 的审批卡，请执行 `python tools/vibeisland.py install cursor` 后重开 CLI。当前仍应把 Cursor 审批写成 beta / 尚未完全实测确认。
- OpenCode 的审批链路依赖本地 `node_modules/vibeisland-opencode-plugin`。如果 OpenCode 的权限请求或提问完全不进灵动岛，请执行 `python tools/vibeisland.py install opencode`，确认本地 plugin 目录仍存在后，再从一个全新终端里重开 `opencode`。
- Claude 不管是浏览器 OAuth 还是 API key 模式，都必须继续保留同一套 `hooks` 和 `statusLine`。切换登录方式时不要把这些集成段删掉。
- Codex 的审批和生命周期同步依赖三件事同时保留：`approval_policy = "never"`、`notify`、`features.codex_hooks = true`。
- Gemini 目前没有稳定公开的本地 `5H / 7D` 配额窗口来源，所以界面上显示 `Unavailable` 属于预期行为，不是渲染问题。
- Cursor 和 OpenCode 也一样，当前不会显示 `5H / 7D` 配额，这属于预期行为，不是渲染问题。
- 在 KDE Plasma + Wayland 下，`pin` / 始终置顶仍然只能算 best-effort。如果还是被其它窗口盖住，请使用托盘召回作为官方兜底方式。
- 如果你已经改过 hooks 或 wrapper 路径，但界面看起来仍然像旧逻辑，顺手把灵动岛 shell 也重启一次：

```bash
vibeisland stop
vibeisland
```

## 详细安装路径：Arch Linux + KDE Plasma + Konsole

这是当前最稳、测试最多的安装路径。

1. 安装系统依赖：

```bash
sudo pacman -S --needed git python python-pip rustup base-devel qt6-base qt6-declarative qt6-multimedia
rustup default stable
python -m pip install --user PyQt6
```

2. 克隆项目：

```bash
git clone <你的仓库地址> VibeAgentIsland-archlinux
cd VibeAgentIsland-archlinux
```

3. 先完成 provider 登录：

- 使用 Claude Code 的话先运行 `claude`
- 使用 Codex CLI 的话先运行 `codex`
- 使用 Gemini CLI 的话先运行 `gemini`
- 使用 Cursor CLI 的话先运行 `cursor-agent`
- 使用 OpenCode 的话先运行 `opencode`

4. 如果你想用示例配置起步，先看：

- `../settings_exp/README.md`
- `../settings_exp/README.zh-CN.md`

5. 把灵动岛集成安装到本机 provider 配置里：

```bash
python tools/vibeisland.py install all
```

6. 把已经开着的 provider CLI 全部关掉后再重开，让新的 hooks 生效。

7. 启动灵动岛：

```bash
python tools/vibeisland.py launch
```

8. 可选：安装桌面入口：

```bash
python tools/vibeisland.py install-desktop
```

之后日常使用命令就是：

```bash
vibeisland
vibeisland status
vibeisland stop
```

Shell 行为说明：

- Telegram bot token 和自动学到的 `chat_id` 会保存在 `~/.config/vibeisland-shell/state.json`
- 展开态 shell 的宽高也会保存在这个文件里，并在下次启动时以被钳制后的合理尺寸恢复
- 在 KDE Wayland 下，`pin` 是“尽量保证”的行为；真正可靠的手动唤回路径是托盘里的 `Summon Island`
- 如果你还想安装桌面入口和 `vibeisland` 终端命令，可执行：

```bash
cd /path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py install-desktop
```

之后就可以直接使用：

```bash
vibeisland
vibeisland status
vibeisland stop
```

## 故障排查

### Claude 仍然要求 token / API 登录

检查 `~/.claude/settings.json`，删除：

- `ANTHROPIC_AUTH_TOKEN`
- `ANTHROPIC_BASE_URL`

然后重新打开 `claude`。

### Shell 重开后尺寸过大或位置异常

检查：

- `~/.config/vibeisland-shell/state.json`

shell 现在会把展开态尺寸和位置持久化到这里。如果这个文件里保留了更早版本遗留的旧几何尺寸，新的实现应该会自动钳制；如果仍有异常，可以先手动删除一次这个文件，再重新启动 shell。

### 灵动岛看不到 Claude 或 Codex 事件

检查：

- `~/.claude/settings.json` 里是否还保留 `hooks`
- 如果你希望看到 Claude 配额百分比，`~/.claude/settings.json` 里是否还保留 `statusLine`
- `~/.codex/config.toml` 是否仍有 `notify` 和 `features.codex_hooks = true`
- `~/.codex/hooks.json` 是否仍包含 Vibe Island 的命令

### Claude 事件正常，但额度仍显示 unavailable

请检查：

- `~/.claude/settings.json` 是否包含 `statusLine`
- 增加 `statusLine` 后是否重启过 `claude`
- 打开 Claude 时，`~/.local/state/vibeisland/claude_statusline.json` 是否会被更新

### 项目路径变化后 hooks 失效

hook 命令使用的是绝对路径。如果项目路径改变，请重新执行：

```bash
cd /new/path/to/VibeAgentIsland-archlinux
python tools/vibeisland.py install all
```


## 可选的 Telegram 远程审批配置

Telegram 是可选能力。即便完全不配置 Telegram，Vibe Island 也必须保持可正常使用。

1. 通过 BotFather 创建 bot，并复制 bot token。
2. 点击 shell 右上角的 settings 按钮打开设置面板。
3. 粘贴 bot token，开启 Telegram 开关并保存。
4. 在 Telegram 中打开这个 bot，先发送一次 `/start`，让 shell 学会你的 `chat_id`。
5. 使用内置的 `TEST` 按钮确认 bot 已能把消息发到你的手机。

bot token、自动学到的 `chat_id` 和最近一次处理过的 Telegram update id 都会写入 shell 偏好文件，因此下次启动时 bridge 会自动恢复连接。

启用后，审批既可以在桌面灵动岛上处理，也可以从 Telegram 处理。`reply / deny` 路径会提示你再发一条文本消息，这条文本会被转发成 agent 的选项 3 式 follow-up 回复。
当 Telegram 上的审批按钮成功送达 agent 后，shell 现在也会反向给 Telegram 一个简短成功回执，帮助用户确认选择已经生效。

Telegram 推送应保持克制：

- 只转发真正面向用户的 live 审批 / 提问
- 协作状态噪音和 bridge/runtime 诊断信息应保留在桌面，不应刷到手机
- 对于完全相同、仍未解决的审批，请求应去重，不应反复提醒

## 可选的前台提醒行为

shell 设置面板现在提供 `AUTO FRONT ON APPROVAL`。

- 开启后，新的审批 / 提问会自动展开灵动岛，并请求桌面把它带到前台。
- 普通的 live 会话波动不应触发这条路径；shell 现在会把它保留给真正可操作的审批 / 提问。
- 关闭后，灵动岛会更安静；用户可以改用系统托盘图标手动召回。
