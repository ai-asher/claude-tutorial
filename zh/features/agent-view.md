# Agent View 智能体视图

会话一多就乱：这个终端在跑重构，那个后台在等 CI，还有一个昨天没关。**Agent View**（`claude agents`，研究预览）把它们汇成一张表——每个 Claude Code 会话，无论运行中、等你回复、还是已完成，都一目了然。

## 什么是 Agent View

Agent View 是一个统一的会话总览界面。运行 `claude agents`，你会看到一个列表，每一行是一个会话，标注它当前的状态：

```bash
claude agents
```

```
┌─────────────────────────────────────────────────┐
│  Claude Code Sessions                            │
├─────────────────────────────────────────────────┤
│ ● 重构 auth 模块        running    · 2m 14s      │
│ ◇ 修复分页 bug          blocked    · 等你确认      │
│ ✓ 补单元测试            done       · 8m 02s       │
│ ● CI 监控 (bg)          running    · 后台          │
└─────────────────────────────────────────────────┘
```

| 状态 | 含义 |
|------|------|
| **running** | 正在工作 |
| **blocked** | 卡在等你回复 / 确认 |
| **done** | 已完成 |

后台会话（用 `claude --bg` 或 `←←` 启动的）也会出现在这个列表里，标记为 `bg`。

## 为什么需要它

以前管理多个会话只能靠在不同终端窗口/tmux pane 之间切。会话一多就容易：
- 忘了哪个还在跑
- 漏掉某个卡住等你回复的会话
- 找不到昨天那个会话的产出

Agent View 把这些集中到一个地方，尤其适合**重度并行使用**的人。

## 常用操作

### 按目录过滤

```bash
# 只看当前目录相关的会话
claude agents --cwd .

# 看指定项目的会话
claude agents --cwd ~/asher/git/eat-what
```

### 脚本化（--json）

```bash
# 以 JSON 输出所有会话，方便脚本处理
claude agents --json
```

这对接 tmux-resurrect、状态栏、自定义会话选择器很有用。例如配合 `jq`：

```bash
# 列出所有"卡住等你"的会话
claude agents --json | jq '.[] | select(.status=="blocked") | .name'
```

### 配置派发的后台会话

`claude agents` 派发新会话时，支持和主命令一样的配置参数：

```bash
claude agents --add-dir ../shared --model claude-opus-4-8 \
  --effort high --permission-mode plan
```

支持的参数包括 `--add-dir`、`--settings`、`--mcp-config`、`--plugin-dir`、`--permission-mode`、`--model`、`--effort`、`--dangerously-skip-permissions`。

## 与后台会话配合

Agent View 和后台会话（background sessions）是天生一对：

```bash
# 启动一个后台会话跑长任务
claude --bg -p "监控 CI，失败就分析日志"

# 稍后用 agent view 查看它的状态
claude agents
```

后台会话完成时还会带上耗时（如 "Agent completed · 3h 2m 5s"）。

::: tip 配合动态工作流
跑[动态工作流](/zh/features/workflows)时会有几十上百个 agent 在后台，Agent View 能让你看到整体的会话层级和状态，不至于"失联"。
:::

::: info 研究预览
Agent View 目前是 **Research Preview** 阶段，行为和界面可能会随反馈调整。官方文档：https://code.claude.com/docs/en/agent-view
:::

## 小结

| 要点 | 记忆点 |
|------|-------|
| 一句话定义 | 所有 Claude Code 会话的统一总览 |
| 启动 | `claude agents` |
| 三种状态 | running / blocked / done |
| 按目录过滤 | `--cwd <path>` |
| 脚本化 | `--json`（配合 jq） |
| 最佳搭档 | 后台会话 `--bg`、动态工作流 |

---

上一篇：[动态工作流 ←](/zh/features/workflows) | 下一篇：[目标模式 →](/zh/features/goal-mode)
