---
counter: True
---

# keyboard shortcuts

## VsCode

- **shift** + command + b: 打开/关闭侧边栏
- **shift** + command + j：打开/关闭终端
- command + 1/2/3/4..: 切换 tab
- command + ,: 打开设置
- command + \：水平分屏
- command + w：关闭分屏/tab
- shift + command + e: 打开资源管理器（切换编辑器和资源管理器）
- ctrl + 1/2/3/4..：切换分屏
- command + p：搜索文件
    - **\>** 等价于 command + shift + p：搜索命令
    - **\:** 跳转到指定行号
- command + j: 往下接受一行提示补全

## Vim

> [💻【Linux】Vim 入门笔记](https://imageslr.com/2021/vim.html#-%E8%A7%86%E9%A2%91%E5%88%86%E4%BA%ABvs-code--vim): 非常详细的参考文档

在 vim 中，一个 word 的定义是连续的【数字+字母+下划线】，或者连续的【特殊字符】（不包含空白字符），例如 `hello, world!!!` 里包含 `hello`、`,`、`world`、!!! 四个单词；按 w 跳转时，会跳过单词和后面的所有空白字符，落在下一个单词的开头，示例：

```markdown
↓ 光标在这里
Hello, world!
     ↑ 按下 w
    ↑ 按下 e
Hello, world!
       ↑ 按两下 w
     ↑ 按两下 e
```

记录一些比较有用且不好记住的快捷键：

- :w <filename> : 保存到一个新的文件
- gg：前往第一行
- G：前往最后一行
- nG：前往第 n 行
- f<target>: 移动到下一个指定字符
- D：删除光标位置到行尾
- A：前往行尾并进入插入模式
- v/V：进行字符/行可视化模式
    - n< 向左缩进 n 个 tab
    - n> 向右缩进 n 个 tab
    - 在 normal 下，可用 << 或 >> 对当前行进行缩进
    - gv 重新选中上一次选择的区域

### Vscode-vim

我的 Vscode-vim 配置：

```json
// 1. 与系统共用剪贴板（用 y 复制的东西，可以在外面 Ctrl+V 粘贴）
"vim.useSystemClipboard": true,
// 2. 开启光标全屏跳转
"vim.easymotion": true,
"vim.sneak": true,
"vim.camelCaseMotion.enable": true,
// 3. 用 jj 快速退出插入模式
"vim.insertModeKeyBindings": [
  {
    "before": ["j", "j"],
    "after": ["<Esc>"]
  }
],
// 4. 回到 normal mode 时自动切换回英文输入法（需要配合系统输入法设置）
"vim.autoSwitchInputMethod.enable": true,
"vim.autoSwitchInputMethod.defaultIM": "com.apple.keylayout.ABC",
"vim.autoSwitchInputMethod.obtainIMCmd": "/usr/local/bin/im-select",
"vim.autoSwitchInputMethod.switchIMCmd": "/usr/local/bin/im-select {im}",
// 5. 让 cw 和 dw 等命令也删除后面的空格（更符合直觉）
"vim.changeWordIncludesWhitespace": true,
"github.copilot.nextEditSuggestions.eagerness": "auto",
```

- EasyMotion：<leader><leader>s+一个字符：搜索这个字符并跳过去，我的 <leader> 默认设置为 \
- Sneak: s+两个字符：跳到这个字符，相当于 f 的增强版
- camelCaseMotion：按驼峰结构移动单词：e.g. myVariableName -> my | Variable | Name

## Ghostty

> [Ghostty + yazi + lazygit 配置](https://juejin.cn/post/7612501079290740771)

- command + d：左右分屏
- command + shift + d：上下分屏

### Yazi

输入 y 后

- 基本操作

    - y/yazi: 启动

    - j/k 或方向键：上下移动

    - h/l：返回上级/进入目录
    - ENTER：进入目录/打开文件
    - .：显示隐藏文件
    - q：退出

- 文件猜错：

    - Space：选中

    - y：复制
    - d：删除

    - x：剪切

    - p：粘贴

    - r：重命名

    - a：新建文件; a/: 新建目录

- 搜索 & 跳转

    - /：搜索
    - f: 过滤
    - gg/G：跳到顶部/底部
    - z：zoxide 智能跳转（需安装插件）
    - z proj      # 跳到最近访问过的含 "proj" 的目录
    - z foo bar   # 模糊匹配包含 foo 和 bar 的目录

### Lazygit

输入 lazygit 启动，q 退出

- 面板切换（数字键）
    - 1：文件变更
    - 2：分支
    - 3：提交历史
    - 4：Stash
- 核心操作
    - Space 暂存/取消缓存文件
    - a：暂存所有文件
    - c：提交
    - P：Push
    - p：Pull
    - b：切换/新建分支
    - e：用 nvim 编辑文件
    - ？：查看所有快捷键
    - q：退出
