---
status: 占用
updated: 2026-11-04
ttl: 365d
tags: [方法论, AI使用, Boris, Anthropic, ClaudeCode]
---

# Boris · Claude Code 创作者 13 条

> 作者：**Boris Cherny**，Claude Code 的创作者
> 原始出处：X（Twitter）长贴，2026-01-02
> **本笔记的来源等级：二手整理。** 我读的是 [tim-kaa-py/ai-wiki](https://github.com/tim-kaa-py/ai-wiki) 的归档版本（经 GitHub API 取到全文）。归档方自己在开头标注：**原始长贴里第 3、4、8 条的截图当时没抓到**，是照作者自己的编号整理的。所以这篇的细节比前三篇弱一档，我按可信度分开写。
> ⚠️ `raw.githubusercontent.com` 在本机被重置，走 GitHub API 取的内容。

## 一条主线

13 条背后只有一句话：

> 把 Claude Code 当成**专业的工程协作者**，而不是一个自动补全。

具体是：给它最好的模型、真工具（MCP）、结构化的权限、按工作流定制的 agent，以及最要紧的，**一个能验证自己工作的反馈回路**。

## 最值钱的那条（归档方标注为第 9 条）

**给模型一个验证自己工作的反馈回路，成品质量 2~3 倍。**

原文的分档很明确：这条是 13 条里**杠杆最高**的一条。他说的反馈回路是具体东西：跑测试、跑类型检查、跑 lint。

这条跟 Felix 的「别让模型评判自己」、Thariq 的「事后考人」撞在一起。**三个人不同时间独立指向同一处，这条是四篇里最可信的一条。**

**对本机怎么落**：`pwsh` 真跑一遍、`wiki_index.py` 的退出码、图片的 `naturalWidth`、网页的横向溢出读数。**能读出数字的才算回路，只在嘴上说「应该行」的不算。**

## 归为高可信的几条

这几条归档里给了完整的「怎么做」（How to apply），说明当时抓到了原文：

**1. 用最强最大的模型。** Opus 单次调用更慢更贵，但你写的代码更少、它处理任务更好，**净结果是整体更快**。

**2. 并行跑 5~10 个 Claude。** 在 `claude.ai/code` 开多个浏览器标签，加上本地终端会话，把评审或后台工作派出去，你在终端继续干自己的。跨会话用 `/compact` 管上下文。

**3. 每个会话都从 Plan mode 开始。** 连按两次 `shift+tab` 进入。先写完整蓝图再执行，好计划能大幅提高一次成功率。

**4. 用自定义 agent 处理重复工作流。** 放在 `.claude/agents/<名字>.md`。他自己那套是：`build-validator` / `code-architect` / `code-simplifier` / `oncall-guide` / `verify-app`。做法是**找出你最常做的 3~5 件「写完代码之后要做的事」，各写一个 agent 文件**。

**5. PostToolUse hook 做自动格式化。** 每次 Write/Edit 之后跑格式化命令。他说这能静默处理掉最后 10% 的格式问题，避免 CI 挂。

**6. 用 `/permissions` 而不是 `--dangerously-skip-permissions`。** 按模式预先放行安全的 bash 命令（build、test、lint、typecheck），存进 `.claude/settings.json` 并提交到仓库，全组共享同一份白名单。后者是一刀切绕过。

**7. 通过 MCP 给真工具。** 接 Slack、BigQuery、Sentry（或你的对应物），配置在 `.mcp.json` 里提交进仓库。

## 弱一档：归档里只给了简述的

**8. 长时间无人值守的任务要有确定性验证。** 三种做法：(a) 让它做完时用后台 agent 验一遍 (b) 用 agent 的 Stop hook (c) 用 `ralph-wiggum` 插件。

**（第 3、4、8 条编号在原始截图里缺，归档方按作者自己的编号排的，位置对不对我没法独立核实。）**

## 可直接抄的配置片段（归档里有原文）

**自动格式化 hook：**

```json
"PostToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [{
        "type": "command",
        "command": "<你的格式化命令> || true"
    }]
}]
```

**MCP 配置：**

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://slack.mcp.anthropic.com/mcp"
    }
  }
}
```

**预放行权限的例子：**

```
Bash(bun run build:*)
Bash(bun run test:*)
Bash(bun run lint:file:*)
Bash(bun run typecheck:*)
```

## 这套东西对本机有多少能直接用

> 我的判断。

| 那条建议 | 本机情况 |
|---|---|
| 验证回路 2~3 倍 | ✅ **最能用的就是这条**，而且成本最低 |
| 用最强模型 | ⚠️ 本机按量计费（千问、AI97），得看任务值不值 |
| 并行跑 5~10 个 | ⚠️ DSH 的 teammate 名册上限 8 个，且会话级只增不减。想大规模并行得开新会话 |
| Plan mode | ⚠️ DSH 有 `PROGRESS.md` 那套（预计 5 次以上调用才写），精神一样：先落盘目标再动手 |
| 自定义 agent | ✅ DSH 有 skill 机制，比 `.claude/agents/` 更适合本机。**这批 Anthropic 心法已经做成了一个 skill** |
| PostToolUse hook | ❌ 本机 DSH 没有这个钩子机制，跳过 |
| `/permissions` 白名单 | ❌ 本机是 `danger-full-access` + 审批关闭，机制不同 |
| MCP 真工具 | ✅ 本机已挂 4 个 MCP（agentmemory / jev / cloudflare-docs / obsidian 等） |
| 无人值守确定性验证 | ✅ 本机的一堆 verify 脚本就是干这个的（49 号 44/44、68 号 21 PASS 0 FAIL） |

**结论：13 条里真正能搬的是一条（验证回路）加半个（规划先行）。** 其余是 Claude Code 专属，别硬套。

## 这篇的短板

**它是唯一一篇二手来源。** 前三条（Felix×2、Thariq）我都拿到了原作者发布页的全文，这篇只有归档整理版，且归档方自己承认有几条缺原文截图。

所以：**拿它佐证「验证回路」这条没问题**（因为另外两位独立说过同样的话），**别拿它引任何具体数字或配置细节当权威**。

## 关联

- 合卡 → [Anthropic工程心法](Anthropic工程心法.md)
- 同团队 Thariq → [Thariq-未知地图](Thariq-未知地图.md)
- 索引 → [学习笔记索引](../学习笔记索引.md)
