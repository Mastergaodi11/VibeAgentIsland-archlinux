# OpenCode 示例说明

英文版：[`README.md`](./README.md)

这个目录记录的是 `VibeAgentIsland-archlinux` 里 `OpenCode` 的真实配置路径和踩坑点。

## 当前已经实测验证过的能力

- live 会话识别
- 分组 section 渲染
- jump
- mini terminal peek
- Telegram 提示
- 岛内审批 / 提问交互

这些能力是在当前一等支持环境下实测通过的：

- Arch Linux
- KDE Plasma 6
- Wayland
- Konsole

## 必需配置文件

- `~/.config/opencode/opencode.json`

## 必需的本地生成 plugin

只改配置文件还不够。

你还必须执行安装器，让它生成本地 plugin：

```bash
python tools/vibeisland.py install opencode
```

执行后请确认下面两样都存在：

- `~/.config/opencode/opencode.json`
- `~/.config/opencode/node_modules/vibeisland-opencode-plugin/`

并确认 `opencode.json` 里仍然保留：

```json
{
  "plugin": [
    "node_modules/vibeisland-opencode-plugin"
  ]
}
```

## 我们实际踩过的坑

- 只改 `opencode.json` 是不够的，本地生成出来的 plugin 目录也必须存在。
- 已经开着的 `opencode` 终端可能会继续缓存旧 plugin 代码。
- 如果审批一直进不了灵动岛，重新安装之后还必须从一个全新终端里重开 `opencode`。
- 如果将来 OpenCode 更新了 plugin API，这份示例配置和本地生成 plugin 可能都要一起更新。

## 推荐正确流程

1. 执行：

```bash
python tools/vibeisland.py install opencode
```

2. 确认 `~/.config/opencode/opencode.json` 里有 plugin 条目。
3. 确认 `~/.config/opencode/node_modules/vibeisland-opencode-plugin/` 目录存在。
4. 把所有已经开着的 `opencode` 终端全部彻底关掉。
5. 开一个全新的终端，再启动 `opencode`。
6. 触发一次真实审批，确认灵动岛确实能看到并处理。

## 路径占位提醒

生成出来的 plugin 最终会指向：

`/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py`

如果你手动复制示例，请把这个占位路径替换成目标机器上的真实绝对路径。
