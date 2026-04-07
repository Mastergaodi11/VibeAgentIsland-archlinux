# Vibe Island 跨平台移植路线图

英文版：[`PORTABILITY_ROADMAP.md`](./PORTABILITY_ROADMAP.md)

维护说明：以后只要本文件或英文版发生变更，必须同步更新中英文两个版本。

这份文档是下一阶段跨平台扩展的交接地图。它并不表示 Ubuntu 24.04 或 Windows 11 已经在当前版本中正式支持，而是说明：哪些层已经具备可复用基础，哪些部分仍然强依赖当前的 Linux/KDE 环境，以及后续移植应该从哪里开始接手。

## 当前基线

目前的一等支持环境仍然是：

- Arch Linux / EndeavourOS
- KDE Plasma
- Wayland
- Konsole

但代码结构已经整理到一定程度，后续在 Ubuntu 24.04 和 Windows 11 上的移植，理论上都可以直接复用大部分 provider bridge、事件归一化、Telegram bridge 和 shell 状态模型。

## 已经具备可移植性的层

下面这些部分在 Ubuntu 24.04 和 Windows 11 上都应尽量复用：

- `tools/vibeisland.py` 里的 provider hook adapter
- `crates/common` 里的统一事件模型
- `crates/daemon` 里的 daemon 快照模型
- `apps/shell/main.py` 里的 shell view model
- `apps/shell/ui/Main.qml` 里的分组式 UI 结构
- `settings_exp/` 示例配置包
- Telegram bridge 与远程审批链路
- replay timeline、自动缩放、provider 分组展示
- shell 里的 live 进程裁剪逻辑。现在 provider 卡片会更偏向真实运行态，而不是长期残留的旧 session 快照

## 仍然偏 Linux/KDE 的层

下面这些部分目前仍然明显偏向当前机器环境，后续移植时应把它们视作“平台适配器”去替换：

- 基于 KWin 的窗口聚焦与提权逻辑
- Konsole 专属的 DBus / session 假设
- KDE 的托盘与窗口管理行为
- Wayland 下 best-effort 的 always-on-top 行为
- 当前进程发现逻辑里对 KDE/Konsole 进程树的偏置
- 当前基于 Linux `/proc` 的 live 进程存活裁剪逻辑。后续到 Windows 时需要用原生进程探测方式替换，不能直接照搬 `/proc`

## Ubuntu 24.04 路线

Ubuntu 24.04 应该被视作第一个扩展目标，因为它可以最大化复用现有 Linux daemon、Python shell、Unix socket 和 provider 集成逻辑。

### 推荐推进顺序

1. 先在以下组合上验证基线：
   - Ubuntu 24.04
   - GNOME on Wayland
   - GNOME Terminal 或 Ptyxis
2. 把现有 Jump 里过于依赖 KDE/KWin 的路径，重构为更中性的 Linux 终端适配器。
3. 回归测试：
   - Claude 审批
   - Codex 审批
   - Gemini 审批
   - Telegram bridge
   - tray summon
4. 补 Ubuntu 依赖安装和打包说明。

### 最可能先撞到的问题

- GNOME 下托盘行为
- GNOME Wayland 下的聚焦/前置逻辑
- 没有 Konsole DBus 时的终端识别
- Qt / Python 依赖在 Ubuntu 上的包名差异

## Windows 11 路线

Windows 11 建议作为第二阶段目标。

### 可以直接复用的部分

- provider adapters 与 hook 安装器
- 统一事件模型与 daemon 核心逻辑
- shell 的分组式 view model 和大部分 QML 结构
- Telegram bridge
- 任务命名、replay、审批状态机

### 需要新写的平台适配器

- 进程发现 helpers
- jump / focus / raise 行为
- launcher 与桌面入口安装逻辑
- IPC 默认路径和 transport 假设
- 对以下终端的适配：
  - Windows Terminal
  - PowerShell
  - Command Prompt
  - VS Code 集成终端

### Windows 交接建议顺序

1. 先把硬编码的 Unix 路径和 shell 假设收敛到平台 helper。
2. 增加 Windows 的路径与配置发现层。
3. 为以下目标补 jump provider：
   - Windows Terminal
   - VS Code terminal
4. 如果 Unix socket 在 Windows 上不合适，就换成更适合 Windows 的 IPC，但尽量不破坏 daemon / session 模型。

## 推荐的适配层拆分

下一位接手 Ubuntu 或 Windows 移植的 Codex，建议按下面三层来推进：

- `provider adapters`
  尽量跨平台共享。
- `platform runtime adapters`
  各平台各自实现：
  - 进程扫描
  - jump/focus
  - tray/launcher
  - 文件系统 / 配置默认路径
- `shell UI`
  尽量保持共享，只有平台确实强制要求时再分支。

## 交接前检查清单

在真正开始 Ubuntu 24.04 或 Windows 11 工作前，先确认：

- `settings_exp/` 仍然与当前 hooks 保持一致
- Claude、Codex、Gemini 的示例配置没有落后
- `python tools/vibeisland.py launch` 在当前 Linux 源环境上可正常工作
- 公开版导出仍然不包含协作运行时文件
- 文档已经明确写清 Linux-first 与未来目标的边界

## 当前诚实边界

截至当前 beta：

- Linux-first 支持是真正可用的
- Ubuntu 24.04 是计划中的近期移植目标
- Windows 11 是计划中的后续目标
- macOS 明确不在本项目路线图内
