---
title: "Claude Code 32个实用技巧：从入门到进阶"
category: "AI工具"
layout: default
---

# Claude Code 32个实用技巧：从入门到进阶

## 简介

本文整理自 YK 的 [32 Claude Code Tips](https://agenticcoding.substack.com/p/32-claude-code-tips-from-basics-to)（发布于 2025-07-31），系统梳理了从基础配置到高级编排的 Claude Code 使用技巧。内容涵盖界面自定义、语音交互、上下文管理、自动化测试、容器化运行、多模型编排等主题，旨在帮助用户更高效地使用这款 AI 编程助手。

---

## 一、基础配置与效率提升

### Tip 0：自定义状态栏

Claude Code 底部状态栏可以自定义显示有用信息。YK 的配置包括：
- 当前使用的模型
- 当前目录
- Git 分支（如有）
- 未提交文件数
- 与远程的同步状态
- Token 使用量可视化进度条
- 上一条消息摘要

参考脚本：[context-bar.sh](https://github.com/ykdojo/claude-code-tips/blob/main/scripts/context-bar.sh)

### Tip 1：语音输入

使用语音转录系统与 Claude Code 交流比打字更快。Mac 上的可选方案：

| 工具 | 类型 | 特点 |
|------|------|------|
| superwhisper | 商业软件 | 本地运行 |
| MacWhisper | 商业软件 | 本地运行 |
| Super Voice Assistant | 开源 | YK 用 Claude Code 自建的语音助手 |

实际体验表明，即使转录有误（如将 "exclamation mark" 误听为 "ExcelElanishMark"），Claude 通常也能正确理解意图。

**场景技巧**：在公共场合可以佩戴耳机轻声耳语，不影响他人。

### Tip 2：问题分解

当 Claude Code 无法一次性解决复杂问题时，将其分解为多个子问题：

```
A → A1 → A2 → A3 → B
```

示例：构建语音转录系统时，YK 将其分解为：
1. 仅下载模型的可执行文件
2. 仅录制语音的功能
3. 仅转录预录音频的功能
4. 最后整合所有功能

核心原则：软件工程的解决问题能力在 Agentic Coding 时代依然高度相关。

### Tip 3：Git 与 GitHub CLI

让 Claude 处理常规的 Git 和 GitHub CLI 任务：
- 自动提交（无需手动写提交信息）
- 分支管理
- 拉取代码

**安全建议**：允许自动 `pull`，但谨慎开启自动 `push`（push 具有更高风险）。

对于 GitHub CLI（`gh`），建议多用 **Draft PR** 功能——让 Claude 创建低风险的草稿 PR，审查后再转为正式 PR。

### Tip 5：终端内容导出

| 方法 | 命令/方式 | 适用场景 |
|------|-----------|----------|
| 复制最后回复 | `/copy` | 快速复制 Claude 的上一条回复 |
| 直写剪贴板 | `pbcopy`（Mac/Linux） | 将输出直接发送到剪贴板 |
| 写入文件 | 让 Claude 写入文件后用 VS Code 打开 | 需要编辑或查看渲染效果 |
| 打开 URL | 用 `open` 命令在浏览器打开 | 需要自行查看网页 |
| GitHub Desktop | 打开当前仓库 | 在非根目录工作时特别有用 |

组合技巧：编辑 GitHub PR 描述时，可让 Claude 先写入本地文件，审查后再粘贴回 PR。

### Tip 6：终端别名

为常用命令设置短别名（添加到 `~/.zshrc` 或 `~/.bashrc`）：

```bash
alias c='claude'           # Claude Code
alias ch='claude --chrome' # 带 Chrome 集成
alias gb='github'          # GitHub Desktop
alias co='code'            # VS Code
alias q='cd ~/Desktop/projects' # 项目根目录
```

组合使用：
- `c -c`：继续上次对话
- `c -r`：列出最近对话
- `ch -c`、`ch -r`：对 Chrome 会话同样适用

---

## 二、上下文管理

### Tip 4：上下文保鲜

AI 上下文就像牛奶——越新鲜越好。Claude Code 在新对话中表现最佳，因为不需要处理历史上下文的复杂性。

**建议**：每个新话题都开启新对话；当性能下降时，也考虑开启新对话。

### Tip 7：主动压缩上下文

Claude Code 提供 `/compact` 命令来总结对话以释放上下文空间。自动压缩在上下文满时也会触发。

Opus 4.5 的上下文窗口为 200k tokens，其中 45k 预留给自动压缩。约 10% 被系统提示、工具、记忆和动态上下文占用。

**YK 的做法**：
1. 通过 `/config` 关闭自动压缩
2. 在需要时手动压缩
3. 让 Claude 写 **交接文档（handoff document）**，包含：
   - 已尝试的方法
   - 有效的方法
   - 无效的方法
   - 下一步计划

4. 新对话中只需提供文件路径即可继续

### Tip 12：搜索对话历史

所有对话历史存储在 `~/.claude/` 中：
- 项目特定对话：`~/.claude/projects/`
- 文件夹命名规则：项目路径中的斜杠替换为破折号

搜索命令示例：

```bash
# 查找提到 "Reddit" 的所有对话
grep -l -i "reddit" ~/.claude/projects/-Users-yk-Desktop-projects-*/*.jsonl

# 查找今天关于某话题的对话
find ~/.claude/projects/-Users-yk-Desktop-projects-*/*.jsonl -mtime 0 -exec grep -l -i "keyword" {} \;

# 提取对话中的用户消息（需要 jq）
cat ~/.claude/projects/.../conversation-id.jsonl | jq -r 'select(.type=="user") | .message.content'
```

也可以直接问 Claude："我们今天关于 X 聊了什么？"

---

## 三、自动化与测试

### Tip 8：完成写-测循环

让 Claude Code 自主运行任务的关键是完成 **写代码 → 运行 → 检查结果 → 重复** 的循环。

**示例：用 git bisect 定位问题**

对于交互式终端应用（如 Claude Code 自身），可用 tmux 模式：

```bash
tmux kill-session -t test-session 2>/dev/null
tmux new-session -d -s test-session
tmux send-keys -t test-session 'claude' Enter
sleep 2
tmux send-keys -t test-session '/context' Enter
sleep 1
tmux capture-pane -t test-session -p
```

有了测试脚本后，Claude 可以运行 `git bisect` 并自动测试每个提交。

### 创意测试策略

| 工具 | 特点 | 适用场景 |
|------|------|----------|
| Playwright MCP | 基于无障碍树（accessibility tree），不依赖截图 | 大多数非视觉密集型任务 |
| Chrome DevTools MCP | 直接操作浏览器 | 需要深度调试 |
| Claude 原生浏览器集成 | 基于截图和坐标点击 | 需要登录状态或视觉点击 |

YK 的默认配置：禁用原生浏览器集成（通过 `ch` 别名按需启用），日常用 Playwright 处理浏览器任务。

优化提示：在 `CLAUDE.md` 中要求 Claude 使用无障碍树引用（ref）而非坐标进行点击。

### Tip 16：指数退避等待长任务

等待 Docker 构建或 GitHub CI 等长任务时，让 Claude 执行手动指数退避检查：

```
检查 → 等待 1 分钟 → 检查 → 等待 2 分钟 → 检查 → 等待 4 分钟...
```

对于 GitHub CI，手动退避比 `gh run watch` 更省 token（后者持续输出大量日志）。推荐：

```bash
gh run view <run-id> | grep <job-name>
```

---

## 四、工作流优化

### Tip 9：Cmd+A / Ctrl+A 万能粘贴

当 Claude Code 无法直接访问某个 URL（如私有页面、Reddit 等），或需要复制终端输出时：

1. 在目标页面按 `Cmd+A`（Mac）或 `Ctrl+A`（其他平台）全选
2. 复制内容
3. 粘贴到 Claude Code

此方法适用于任何 AI 工具，不限于 Claude Code。

### Tip 10：用 Gemini CLI 访问被屏蔽网站

Claude Code 的 WebFetch 无法访问某些网站（如 Reddit），可创建 skill 让 Gemini CLI 作为备用方案。

实现方式：使用 tmux 模式——启动会话、发送命令、捕获输出。

Skill 文件路径：`~/.claude/skills/reddit-fetch/SKILL.md`

完整文件：[reddit-fetch skill](https://github.com/ykdojo/claude-code-tips/blob/main/skills/reddit-fetch/SKILL.md)

**注意**：skill 比 `CLAUDE.md` 更省 token，因为只在需要时加载；而 `CLAUDE.md` 会在每次对话开始时加载。

### Tip 11：投资个人工作流

花时间优化自己的工具链是值得的投资：
- 维护简洁的 `CLAUDE.md`
- 学习关键功能和工具
- 根据需求定制脚本和配置

不需要过度投入，但至少要关注常用工具的效率优化。

### Tip 13：多任务终端标签页

**YK 的"级联"方法**：
- 从左到右依次打开新标签页
- 按从左到右的顺序处理任务（从旧到新）
- 最多同时关注 3-4 个任务

示例布局：
1. 最左侧：持久运行的语音转录系统
2. 第二个：Docker 容器设置
3. 第三个：本地磁盘使用检查
4. 第四个：工程项目
5. 当前标签：撰写本文

### Tip 15：Git Worktrees 并行分支工作

在多个分支上工作而不冲突的方法：
- 让 Claude Code 创建 git worktree
- 在不同目录中处理不同分支
- 本质上是 "分支 + 目录" 的组合

可以与 Tip 13 的级联方法结合使用。

---

## 五、系统优化

### Tip 14：精简系统提示

Claude Code 的系统提示和工具定义约占 18k tokens（200k 上下文的 ~9%）。YK 创建了补丁系统将其缩减到约 10k tokens，节省 ~7,300 tokens（静态开销的 41%）。

**补丁效果**：
- TodoWrite 示例：从 6KB 缩减到 0.4KB
- Bash 工具描述：从 3.7KB 缩减到 0.6KB

实现方式：修剪 minified CLI bundle 中的冗余示例和文本，保留核心指令。

**保持补丁**：在 `~/.claude/settings.json` 中禁用自动更新：

```json
{
  "env": {
    "DISABLE_AUTOUPDATER": "1"
  }
}
```

### 懒加载 MCP 工具

MCP 服务器工具默认会在每次对话开始时全部加载，即使不使用。

在 `~/.claude/settings.json` 中启用懒加载：

```json
{
  "env": {
    "ENABLE_TOOL_SEARCH": "true"
  }
}
```

Claude 将在需要时按需搜索和加载 MCP 工具。从 2.1.7 版本起，当 MCP 工具描述超过上下文窗口的 10% 时会自动启用。

---

## 六、写作与研究

### Tip 17：Claude Code 作为写作助手

YK 的写作流程：
1. 给 Claude 充分的背景信息
2. 用语音给出详细指令，获取初稿
3. 逐行审查：指出喜欢的部分、需要调整的部分、需要移动的部分
4. 反复迭代

**布局建议**：终端在左，代码编辑器在右，形成并排工作流。

### Tip 18：Markdown 优先

Markdown 是最高效的文档格式，尤其在 AI 时代：
- 比 Google Docs/Notion 更高效
- 与 Claude Code 配合极佳
- 博客、LinkedIn 帖子都可以先用 Markdown 写

**跨平台粘贴技巧**：如果目标平台不支持 Markdown，先粘贴到 Notion，再从 Notion 复制到目标平台。

### Tip 19：用 Notion 保留链接

反向操作：从 Slack 等带有链接的文本，直接粘贴到 Claude Code 会丢失链接。先粘贴到 Notion，再从 Notion 复制，即可保留 Markdown 格式的链接。

### Tip 26：Claude Code 作为研究工具

Claude Code 可以作为 Google 或深度研究的替代方案，优势在于：
- 可以访问 `gh` 终端命令
- 可以通过容器访问各种环境
- 可以通过 Gemini CLI 访问 Reddit
- 可以通过 MCP 访问 Slack 等私有信息
- 可以通过 Cmd+A 方法获取私有页面

YK 曾通过 Claude Code 的研究能力节省了 $10,000。

### Tip 27：验证输出的多种方式

| 验证方式 | 适用场景 |
|----------|----------|
| 编写测试 | 代码输出 |
| 视觉审查 | Claude Code UI 中直接检查 |
| GitHub Desktop | 快速检查代码变更 |
| Draft PR | 生成草稿 PR 审查后再转正 |
| 自我检查 | "double check everything, every single claim..." |

YK 推荐的自我检查提示：
> "double check everything, every single claim in what you produced and at the end make a table of what you were able to verify"

---

## 七、容器与编排

### Tip 20：容器化运行高风险任务

常规会话适合需要仔细控制权限的 meticulous 工作；容器化环境适合 `--dangerously-skip-permissions` 会话。

**容器场景**：
- 研究或实验性任务
- 需要长时间无人值守运行的任务
- 可能具有风险的操作

YK 的容器包含：Claude Code、Gemini CLI、tmux 和所有自定义配置。

### 高级：编排容器中的 Claude Code

用外层 Claude Code 控制容器内的 Claude Code 实例：

1. 本地 Claude Code 启动 tmux 会话
2. tmux 会话中运行/连接容器
3. 容器内 Claude Code 以 `--dangerously-skip-permissions` 运行
4. 外层 Claude Code 通过 `tmux send-keys` 发送提示，`capture-pane` 读取输出

这样就获得了一个完全自主的"工作节点"Claude Code。

### 高级：多模型编排

不仅限于 Claude Code，可以在容器中运行不同 AI CLI：
- OpenAI Codex（用于代码审查）
- Gemini CLI
- 其他 AI CLI

Claude Code 成为中央协调界面，负责：启动不同模型、在容器和主机之间发送数据。

### Tip 22：克隆对话分支

当需要从对话的某个节点尝试不同方案而不丢失原线程时，可以使用克隆功能。

**原生支持**（较新版本）：
- `/fork`：在对话内分叉当前会话
- `--fork-session`：与 `--resume` 或 `--continue` 配合使用

为 `--fork-session` 添加短别名（添加到 `~/.zshrc`）：

```bash
claude() {
  local args=()
  for arg in "$@"; do
    if [[ "$arg" == "--fs" ]]; then
      args+=("--fork-session")
    else
      args+=("$arg")
    fi
  done
  command claude "${args[@]}"
}
```

**半克隆（half-clone）**：保留对话的后半部分，减少 token 使用。使用 [half-clone-conversation 脚本](https://github.com/ykdojo/claude-code-tips/blob/main/scripts/half-clone-conversation.sh)。

---

## 八、理解核心概念

### Tip 24：CLAUDE.md vs Skills vs Slash Commands vs Plugins

| 特性 | CLAUDE.md | Skills | Slash Commands | Plugins |
|------|-----------|--------|----------------|---------|
| 加载时机 | 每次对话开始 | 按需加载 | 手动调用 | 安装时 |
| 调用方式 | 自动 | 自动/手动（`/skill-name`） | 手动（`/command`） | 自动/手动 |
| 设计对象 | 用户 | Claude | 用户 | 两者 |
| Token 效率 | 低（始终加载） | 高 | 高 | 取决于内容 |
| 可包含内容 | 提示词 | 提示词 | 提示词 | Skills + Commands + Agents + Hooks + MCP |

**Notes**：
- Skills 和 Slash Commands 功能上已趋同合并
- Anthropic 官方的 `frontend-design` plugin 本质上就是一个 skill

### Tip 29：保持 CLAUDE.md 简洁

- 可以从没有 `CLAUDE.md` 开始
- 当发现反复告诉 Claude 同一件事时，再添加到 `CLAUDE.md`
- 可以让 Claude 自己编辑 `CLAUDE.md`
- 项目级 `./CLAUDE.md` 和全局 `~/.claude/CLAUDE.md` 配合使用

---

## 九、代码审查与 DevOps

### Tip 25：交互式 PR 审查

Claude Code 作为交互式 PR 审查员的优势：
- 可以请求 PR 信息（通过 `gh` 命令）
- 按文件逐步审查
- 用户控制审查节奏和深度
- 可以要求运行测试

与一次性 AI 审查的区别：可以进行对话式深入审查。

### Tip 28：Claude Code 作为 DevOps 工程师

处理 GitHub Actions CI 失败的流程：
1. 将失败信息给 Claude Code
2. 指令："dig into this issue, try to find the root cause"
3. 持续追问：是特定提交/PR 导致的？还是 flaky issue？
4. 定位问题后创建 Draft PR
5. 审查输出、自我验证
6. 转为正式 PR 修复问题

优势：AI 可以处理大量日志，比人工翻阅轻松得多。

---

## 十、最佳实践

### Tip 21：实践出真知

> "How do you get better at rock climbing?" "By rock climbing."

提升 Claude Code 使用能力的最佳方式就是使用它。

YK 提出的 **"十亿 Token 法则"**（替代一万小时法则）：大量消耗 token 是获得 AI 直觉的最佳方式。Opus 4.5 足够强大且价格合理，可以同时运行多个会话。

### Tip 23：使用 realpath 获取绝对路径

当需要告诉 Claude Code 其他文件夹中的文件时：

```bash
realpath some/relative/path
```

### Tip 30：Claude Code 作为通用接口

Claude Code 不仅是 "新 IDE"，更是计算机的**通用接口**：
- 视频编辑 → ffmpeg
- 音频/视频转录 → Whisper
- CSV 数据分析 → Python/JavaScript 可视化
- 互联网访问 → Reddit、GitHub、MCP

就像拥有第二大脑，而且可以同时打开第三、第四、第五...个大脑。

### Tip 31：选择合适的抽象层次

| 层次 | 描述 | 适用场景 |
|------|------|----------|
| Vibe Coding | 关注高层，不纠结每行代码 | 一次性项目、非关键代码 |
| 中等深度 | 查看文件结构和函数 | 需要理解架构时 |
| 深度审查 | 逐行检查代码和依赖 | 关键功能、故障排查 |

**核心洞察**：不是二元的"vibe coding 好/坏"，而是根据场景选择合适深度。Claude Code 是你的向导，可以带你从冰山表面潜到深处。

---

## 十一、其他实用技巧

### Tip 19：Notion 作为格式转换器
- 正向：Markdown → Notion → 其他平台
- 反向：其他平台链接 → Notion → Markdown（保留链接）
- 粘贴快捷键：`Cmd+Shift+V`（无格式粘贴）

### Tip 19 补充：容器化最佳实践
- 常规会话：meticulous work，严格控制权限
- 容器会话：`--dangerously-skip-permissions`，适合长时间无人值守任务
- 风险隔离：出问题也只会影响容器

---

## 总结

Claude Code 的使用可以归纳为几个核心原则：

1. **分解问题**：复杂任务拆分为可管理的子任务
2. **管理上下文**：保持上下文新鲜，主动压缩，善用新对话
3. **投资工作流**：定制别名、状态栏、`CLAUDE.md` 等个人配置
4. **完成循环**：写代码 → 测试 → 验证 → 迭代
5. **选择合适深度**：根据任务重要性调整审查粒度
6. **实践积累**：多用多试，培养对 AI 能力的直觉
