# Cursor CLI 示例说明

英文版：[`README.md`](./README.md)

这个目录说明的是 `VibeAgentIsland-archlinux` 在当前 beta 里对 `Cursor CLI` 的真实预期。

## 当前已经实测过的部分

- live 会话识别
- 分组 section 渲染
- status line 采集
- jump
- mini terminal peek

## 当前仍按 beta / 尚未完全实测确认处理的部分

- 审批交互的端到端完成链路

虽然审批链路已经通过 `approvalMode = "default"` 和安装进去的 `hooks.json` 接线了，但这一版仍然不把 Cursor 审批写成“完全稳定”，请先按实验性能力对待。

## 必需文件

- `~/.cursor/cli-config.json`
- `~/.cursor/hooks.json`

## 必需设置

- 保留 `"approvalMode": "default"`
- 保留生成出来的 `statusLine`
- 保留生成出来的 `hooks.json`

## 实际建议

1. 执行：

```bash
python tools/vibeisland.py install cursor
```

2. 把所有已经开着的 `cursor-agent` 终端全部彻底关掉。
3. 开一个全新的终端。
4. 再重新启动 `cursor-agent`。
5. 先测试 live 会话和分组显示。
6. 审批链路先按 beta 能力测试，不要直接假定它已经百分百稳定。

## 路径占位提醒

这些示例 JSON 默认使用：

`/path/to/VibeAgentIsland-archlinux/tools/vibeisland.py`

请替换为目标机器上的真实绝对路径。
