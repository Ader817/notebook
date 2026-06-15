## tmux 速查与指南

tmux 是一个终端复用器，允许你在一个终端中管理多个会话（session）、窗口（window）和窗格（pane），并可随时分离/接入以保持任务在后台运行。

- Prefix 键：所有 tmux 快捷键的“前缀”，默认是 Ctrl+b。使用方式是先按 Ctrl+b，松开，再按功能键。
- 基本对象：
  - 会话 Session：一个独立工作区，包含多个窗口。
  - 窗口 Window：类似浏览器标签页，每个窗口可再分割为多个窗格。
  - 窗格 Pane：窗口内的子区域，每个都是一个独立终端。

### 快速上手

- 启动 tmux：在终端执行 `tmux`
- 启动并命名：`tmux new -s <会话名>`
- 分离当前会话：Prefix + d
- 接入最近会话：`tmux a`
- 接入指定会话：`tmux a -t <会话名>`
- 查看会话列表：`tmux ls`

注：`<会话名>` 可以是会话名称或编号。

## 会话管理（Session）

| 操作 | 命令（终端中执行） | 快捷键（tmux 中） |
| --- | --- | --- |
| 启动新会话 | `tmux` | — |
| 启动新会话并命名 | `tmux new -s <会话名>` | — |
| 查看所有会话 | `tmux ls` | — |
| 接入上一个会话 | `tmux a` | — |
| 接入指定会话 | `tmux a -t <会话名>` | — |
| 分离当前会话 | — | Prefix + d |
| 杀死指定会话 | `tmux kill-session -t <会话名>` | — |
| 切换会话列表 | — | Prefix + s |

## 窗口管理（Window）

| 操作 | 快捷键（tmux 中） |
| --- | --- |
| 创建新窗口 | Prefix + c |
| 关闭当前窗口 | Prefix + & |
| 重命名当前窗口 | Prefix + , |
| 切换到下一个窗口 | Prefix + n |
| 切换到上一个窗口 | Prefix + p |
| 切换到指定编号窗口 | Prefix + <编号>（如 1） |
| 列出所有窗口并切换 | Prefix + w |

## 窗格管理（Pane）

| 操作 | 快捷键（tmux 中） |
| --- | --- |
| 垂直分割（左右） | Prefix + % |
| 水平分割（上下） | Prefix + " |
| 在窗格间切换 | Prefix + 方向键（↑↓←→） |
| 显示窗格编号 | Prefix + q |
| 关闭当前窗格 | Prefix + x |
| 切换上/下一个窗格 | Prefix + o |
| 最大化/恢复当前窗格 | Prefix + z |

## 复制模式（Copy Mode）

用于滚动查看历史输出、选择并复制文本。

| 操作 | 快捷键（tmux 中） |
| --- | --- |
| 进入复制模式 | Prefix + [ |
| 退出复制模式 | q |
| 开始选择（vi 模式） | Space |
| 复制选中（vi 模式） | Enter |
| 粘贴复制内容 | Prefix + ] |
| 向上/下翻页 | PageUp / PageDown |

## 常见工作流示例

- 远程长任务不断线：在 tmux 中启动任务 → Prefix + d 分离 → 断线重连后用 `tmux a` 接回。
- 多任务并行：在一个会话内用多个窗口；单窗口内再用窗格分割查看日志与运行指令。

## 详细笔记

## 1. Basic Concepts

**tmux** 是一个终端复用器（terminal multiplexer），允许在单个终端窗口中创建和管理多个终端会话。

### 1.1 Core Concepts
- `Session` - 会话，最高层级的工作单位
- `Window` - 窗口，会话中的标签页
- `Pane` - 面板，窗口中的分割区域

> 默认前缀键为 `Ctrl+b`，下文简写为 `prefix`

## 2. Session Management

### 2.1 Creating and Managing Sessions

| 快捷键 | 功能描述 |
| :--- | :--- |
| `tmux new -s name` | 创建名为 `name` 的新会话 |
| `tmux ls` | 列出所有会话 |
| `tmux attach -t name` | 连接到名为 `name` 的会话 |
| `prefix $` | 重命名当前会话 |
| `prefix d` | 分离当前会话 |
| `tmux kill-session -t name` | 删除指定会话 |

### 2.2 Session Switching
- `prefix s` - 显示会话列表并选择切换
- `prefix (` - 切换到上一个会话
- `prefix )` - 切换到下一个会话

## 3. Window Operations

### 3.1 Creating and Switching Windows

| 快捷键 | 功能描述 |
| :--- | :--- |
| `prefix c` | 创建新窗口 |
| `prefix ,` | 重命名当前窗口 |
| `prefix &` | 关闭当前窗口 |
| `prefix 0-9` | 切换到指定编号的窗口 |
| `prefix p` | 切换到上一个窗口 |
| `prefix n` | 切换到下一个窗口 |
| `prefix w` | 以菜单方式显示所有窗口 |

### 3.2 Window Navigation
- `prefix l` - 在最近使用的两个窗口间切换

## 4. Pane Operations

### 4.1 Splitting Panes

| 快捷键 | 功能描述 |
| :--- | :--- |
| `prefix %` | ==垂直分割== 当前面板 |
| `prefix "` | ==水平分割== 当前面板 |
| `prefix x` | 关闭当前面板 |

### 4.2 Switching and Moving Panes

| 快捷键 | 功能描述 |
| :--- | :--- |
| `prefix 方向键` | 在面板间切换 |
| `prefix o` | 顺时针切换到下一个面板 |
| `prefix ;` | 切换到上一个使用的面板 |
| `prefix {` | 向前交换当前面板与上一个面板 |
| `prefix }` | 向后交换当前面板与下一个面板 |

### 4.3 Pane Adjustments
- `prefix z` - 缩放当前面板（全屏显示/还原）
- `prefix space` - 循环切换面板布局

## 5. Copy and Paste Mode

### 5.1 Entering and Exiting Copy Mode
- `prefix [` - 进入复制模式
- `q` 或 `ESC` - 退出复制模式

### 5.2 Operations in Copy Mode
- `space` - 开始选择
- `enter` - 复制选中文本
- `vi 方向键` 或 `方向键` - 移动光标
- `pageup/pagedown` - 翻页
- `g` - 跳到顶部
- `G` - 跳到底部

### 5.3 Paste Operations
- `prefix ]` - 粘贴最近复制的文本
- `prefix =` - 显示缓冲区列表选择粘贴

## 6. Other Useful Features

### 6.1 Viewing Help
- `prefix ?` - 显示所有快捷键帮助
- `man tmux` - 查看完整手册

### 6.2 Displaying Information
- `prefix t` - 显示时钟
- `prefix Ctrl+space` - 显示消息历史

### 6.3 Reloading Configuration
- `prefix :` - 进入命令模式
  ```shell
  # 在命令模式下
  source-file ~/.tmux.conf
  ```

## 7. Common Command Examples

### 7.1 Batch Operations
```shell
# 查看所有会话
tmux ls

# 创建新会话并指定名称
tmux new -s work

# 连接到指定会话
tmux attach -t work

# 在后台创建会话
tmux new -s work -d

# 杀死指定会话
tmux kill-session -t work

# 杀死所有 tmux 会话
tmux kill-server
```

### 7.2 Command Line Shortcuts
```shell
# 为常用操作创建别名
alias tml="tmux list-sessions"
alias tma="tmux attach-session -t"
alias tmn="tmux new-session -s"
```

## 8. Configuration Suggestions

### 8.1 Basic Configuration File (`~/.tmux.conf`)
```shell
# 修改前缀键为 Ctrl+a
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# 启用鼠标支持
set -g mouse on

# 设置默认终端模式为 256 色
set -g default-terminal "screen-256color"

# 设置窗口和面板索引从 1 开始
set -g base-index 1
set -g pane-base-index 1

# 设置状态栏
set -g status-right "%H:%M %d-%b-%y"
```

> 重新加载配置：在 tmux 中按 `prefix :` 然后输入 `source-file ~/.tmux.conf`

## 9. Troubleshooting

### 9.1 Common Issues
1. **无法滚屏**：进入复制模式 (`prefix [`) 然后使用 `pageup/pagedown`
2. **鼠标不工作**：确认配置文件中有 `set -g mouse on`
3. **前缀键冲突**：考虑修改前缀键，如改为 `Ctrl+a`

### 9.2 Emergency Exit
- `prefix &` - 关闭当前窗口
- `prefix x` - 关闭当前面板
- `exit` - 退出当前 shell 或面板
- `tmux kill-server` - 强制终止所有 tmux 会话

## 10. Advanced Tips

### 10.1 Multi-Client Connections
- 同一个会话可以被多个客户端同时连接
- 不同客户端可以查看同一会话的不同窗口

### 10.2 Session Recovery
- 断开 SSH 连接后，tmux 会话会继续在后台运行
- 使用 `tmux attach` 可以恢复之前的会话状态

### 10.3 Automation Scripts
```shell
#!/bin/bash
# 创建开发环境会话
tmux new-session -d -s dev
tmux send-keys -t dev 'vim' C-m
tmux split-window -h -t dev
tmux send-keys -t dev 'git status' C-m
tmux attach-session -t dev
```
