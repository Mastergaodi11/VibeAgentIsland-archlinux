# 示例配置包

英文版：[`README.md`](./README.md)

维护说明：以后只要灵动岛要求的 hooks、statusLine 或 provider 集成字段发生变化，就必须同步更新本目录中的示例文件以及中英文两份 README。

这个目录放的是 `Claude Code`、`Codex` 和 `Gemini CLI` 的示例配置文件，方便用户在另一台机器上照着填写。

## 这些示例文件已经包含什么

- 灵动岛所需的 hooks
- Claude 配额 HUD 需要的 `statusLine`
- Codex 的 `notify` 与 `approval_policy = "never"`
- Gemini 的 hooks 基础结构

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

## 推荐配置流程

1. 先把对应示例文件复制到真实配置路径。
2. 填好占位的鉴权字段，或者先完成 CLI 登录。
3. 确认 hook 命令指向你本机真实的 `vibeisland.py` 绝对路径。
4. 重启对应 provider CLI。
5. 再启动灵动岛，验证会话和审批是否能被识别。

## 路径占位提醒

这些示例文件里默认使用的是：

`/path/to/vibeisland-linux/tools/vibeisland.py`

用户在真实机器上必须把它替换成实际的绝对路径。
