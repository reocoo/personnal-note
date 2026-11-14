---
status: 占用
updated: 2026-11-14
ttl: 90d
tags: [skills, 上下文, 优化]
---

# skills 清单瘦身方案

## 一句话结论

本机每轮常驻的 99 条 skill 目录里，最近 30 场会话真正调用过 8 条，21,085 个字符的 description 中有 19,275 个字符对应的 skill 一次都没被用过。按下面三档执行，常驻 description 会从 21,085 字符降到 4,633 字符，省 16,452 字符，约 4,113 token，占基线的 78.0%。

先动哪三个：`cel-glow-atelier` 代表的纯代码画画组（12 条，5,447 字符）、5 条重复的 code-review 变体（408 字符）、4 条 agent 编排与评估重叠条目（357 字符）。

## 数据来源与证据等级

| 项 | 数值 | 怎么得到的 |
|---|---|---|
| 会话窗口 | 最近 30 个 `session.v4.jsonl.zstd`，覆盖 2026-11-10 12:34 至 2026-11-14 16:06（本地时间） | `Get-ChildItem ... \sessions -Recurse -Filter session.v4.jsonl.zstd \| Sort-Object LastWriteTime -Descending \| Select -First 30`，跳过 >20 MB 的（本批没有超限文件） |
| 压缩文件总量 | 22,984,322 字节 | 同上，逐个 `Length` 求和 |
| zstd 帧数 | 13,990 帧，全部解压成功（失败 0） | 脚本 `sk_stats.js`，按 zstd magic `28 B5 2F FD` 切帧后逐帧 `zlib.zstdDecompressSync` |
| 解压后文本 | 71,613,244 字节 | 同上 |
| JSON 事件 | 25,792 条 | 同上，逐行 `JSON.parse` 成功的行 |
| `tool/call` 事件 | 3,890 条 | 同上 |
| 其中的 skill 调用 | 12 条事件，按 `callId` 去重后 11 次，涉及 8 个不同 skill | 脚本 `sk_final.js` |
| 注入的 skill 目录 | 99 条，description 合计 21,085 字符 | 脚本 `sk_catalog2.js`，从会话里 `<available_skills>` 块抽，每条取 `- \`名\`: ` 之后的文本长度（JS `String.length`，中文按 1 算） |

**测量时点漂移**：同一批 30 个文件，几分钟后重跑得到 14,089 帧、72,128,814 字节。原因是父会话 `b9df9ee7` 当时仍在写入（它就是这次调研所在的那场会话）。上面表里的 13,990 帧 / 71,613,244 字节是第一次一致测量的结果，后面所有结论都基于这一次。

**证据等级**：调用统计属**强证据**（原始事件逐条可重放，脚本可重跑，抽了 3 条原始记录人眼核对）。三档归类属**判断**，不是测量结果，理由在每档里写清了。下面这两条属于**未验证**：

- `/quick-mode`、`/prompt-generator` 这类由 Host 指令触发的 skill，不走 `skill` 工具，因此不在 `tool/call` 统计里。它们的实际触发次数没有算进来。
- 30 场只覆盖 5 天。更早的使用情况没统计（要查得用 `obelisk --search "<skill名>"`）。

### 三个抽样核对

从原始事件里抽了 3 条 `tool/call` 与对应 `tool/result`，逐字看过：

```
{"type":"tool/call","seq":39,"time":1794527210982,"data":{"turn":1,"step":3,
 "callId":"call_00_axidgw0yne92kdwchtsqcbye","name":"skill",
 "arguments":"{\"name\":\"zh-ai-tells\"}"}}
→ result isError=false，正文 3,557 字符，开头 <skill_content name="zh-ai-tells">

{"type":"tool/call","seq":24,"time":1794448004736,"data":{...,"name":"skill",
 "arguments":"{\"name\":\"deai\"}"}}
→ result isError=false，正文 3,137 字符

{"type":"tool/call","seq":461,"time":1794231300527,"data":{"turn":3,...,"name":"skill",
 "arguments":"{\"name\": \"doubao-image-poster\"}"}}
→ result isError=false，正文 4,982 字符
```

13 条匹配到结果的 skill 调用里，`isError` 为 true 的是 0 条，全部是真实的 `skill_content` 加载。

## 目录对账

注入的 99 条不等于磁盘上的目录数，两个口径要分开看。

| 口径 | 数量 | 说明 |
|---|---|---|
| 注入目录（每轮常驻的那 99 条） | 99 | 会话里 `<available_skills>` 块的条目数 |
| 磁盘上含 `SKILL.md` 的目录 | 117（4 个根）+ 2（第 5 个根） | 根：`~/.dsh/skills`（59）、`工作区\.agents\skills`、`工作区\.claude\skills`、`工作区\agent\skills`、`~/.agents\skills` |
| 按 frontmatter `name` 去重 | 101（4 个根内） | 同一 skill 挂在多个根里会重复计入目录数 |
| 在注入目录里但不在四根里 | 2 | `dsh-tui-guide` 在 `%APPDATA%\npm\node_modules\@deepseek-harness-tui\dsh-tui\guide\`；`obelisk` 在 `~/.agents/skills/obelisk` |
| 在磁盘上但没进注入目录 | 4 | `quick-mode`、`prompt-generator`（设计上不进目录）、`i-have-adhd`、`grill-me`（原因未验证） |

`~/.dsh/skills` 下的 59 个条目里，除 `doubao-image-poster` 是真实目录，其余全是 junction（Windows 目录链接，删链接不会删源目录）。这一点决定了下面的「移出」操作只摘链接，不碰源文件。

description 长度用注入块量，因为那才是每轮真正付钱的数字。逐条比对过 frontmatter，只有 `check-understanding` 一条差 6 个字符（frontmatter 143、注入 149），其余全部一致，所以两种口径可以互认。

## 全量表

按 description 字符降序。调用次数是 30 场会话内的 `skill` 工具调用次数。最后使用时间已换成 +08:00。

| skill 名 | description 字符 | 调用次数 | 最后使用（+08:00） | 建议 |
|---|---:|---:|---|---|
| `ui-ux-pro-max` | 495 | 0 | 从无 | 收窄 |
| `interview-me` | 485 | 2 | 2026-11-11 16:53 | 保留 |
| `cel-glow-atelier` | 479 | 0 | 从无 | 移出目录 |
| `clay-cel-canvas` | 469 | 0 | 从无 | 移出目录 |
| `flat-painterly` | 467 | 0 | 从无 | 移出目录 |
| `dollhouse-maker` | 459 | 0 | 从无 | 移出目录 |
| `pastel-picturebook` | 459 | 0 | 从无 | 移出目录 |
| `vinyl-cel` | 456 | 0 | 从无 | 移出目录 |
| `soft-diorama` | 455 | 0 | 从无 | 移出目录 |
| `web-design-engineer` | 455 | 0 | 从无 | 收窄 |
| `maquette-painter` | 445 | 0 | 从无 | 移出目录 |
| `genoffice` | 443 | 0 | 从无 | 收窄 |
| `paper-stage` | 443 | 0 | 从无 | 移出目录 |
| `polygon-cel` | 443 | 0 | 从无 | 移出目录 |
| `geometric-illustration` | 437 | 0 | 从无 | 移出目录 |
| `tableau-vivant` | 435 | 0 | 从无 | 移出目录 |
| `spec-driven-development` | 431 | 0 | 从无 | 收窄 |
| `sw-dialogue` | 419 | 0 | 从无 | 收窄 |
| `avoid-ai-writing` | 403 | 0 | 从无 | 收窄 |
| `deai` | 365 | 1 | 2026-11-12 09:46 | 保留 |
| `security-audit` | 359 | 0 | 从无 | 收窄 |
| `incremental-implementation` | 343 | 0 | 从无 | 收窄 |
| `idea-refine` | 335 | 0 | 从无 | 收窄 |
| `debugging-and-error-recovery` | 323 | 0 | 从无 | 收窄 |
| `skill-creator` | 319 | 0 | 从无 | 收窄 |
| `find-skills` | 303 | 0 | 从无 | 收窄 |
| `reverse-engineer-anything` | 300 | 0 | 从无 | 收窄 |
| `morph-icon-button` | 298 | 0 | 从无 | 收窄 |
| `obelisk` | 293 | 0 | 从无 | 收窄 |
| `mcp-builder` | 277 | 0 | 从无 | 收窄 |
| `course-guide` | 269 | 0 | 从无 | 收窄 |
| `design-taste-frontend` | 269 | 0 | 从无 | 移出目录 |
| `goutoujunshi` | 249 | 0 | 从无 | 收窄 |
| `pixel-loom` | 248 | 0 | 从无 | 收窄 |
| `planning-and-task-breakdown` | 237 | 0 | 从无 | 收窄 |
| `miora-visual-fallback` | 235 | 0 | 从无 | 收窄 |
| `superagent` | 233 | 3 | 2026-11-13 17:25 | 保留 |
| `pixel-art` | 229 | 0 | 从无 | 收窄 |
| `lieflat-charts` | 226 | 0 | 从无 | 收窄 |
| `code-film-production` | 213 | 0 | 从无 | 收窄 |
| `start-learning` | 213 | 0 | 从无 | 收窄 |
| `claude-certification` | 210 | 0 | 从无 | 收窄 |
| `learn` | 206 | 0 | 从无 | 收窄 |
| `frontend-design` | 204 | 0 | 从无 | 移出目录 |
| `doubao-image-poster` | 203 | 1 | 2026-11-09 21:35 | 保留 |
| `find-your-level` | 201 | 0 | 从无 | 收窄 |
| `web-scrape-triple` | 197 | 1 | 2026-11-12 11:40 | 保留 |
| `static-delivery-check` | 194 | 0 | 从无 | 收窄 |
| `anthropic-engineering-mindset` | 172 | 0 | 从无 | 收窄 |
| `learn-mcp` | 156 | 0 | 从无 | 收窄 |
| `dsh-plugin-authoring` | 153 | 0 | 从无 | 收窄 |
| `dsh-tui-guide` | 152 | 0 | 从无 | 收窄 |
| `grilling` | 152 | 0 | 从无 | 收窄 |
| `check-understanding` | 149 | 0 | 从无 | 收窄 |
| `jev-dsh` | 146 | 0 | 从无 | 收窄 |
| `learn-agent-skills` | 143 | 0 | 从无 | 收窄 |
| `agent-team-playbook` | 141 | 1 | 2026-11-14 16:05 | 保留 |
| `code-review-ai-ai-review` | 141 | 0 | 从无 | 移出目录 |
| `egtool-proxy` | 140 | 0 | 从无 | 收窄 |
| `agent-desktop` | 139 | 0 | 从无 | 收窄 |
| `agent-squad` | 135 | 0 | 从无 | 移出目录 |
| `huashu-excel` | 123 | 0 | 从无 | 收窄 |
| `understand` | 122 | 0 | 从无 | 移出目录 |
| `diagnose-windows-sandbox-acl` | 121 | 0 | 从无 | 收窄 |
| `commit` | 117 | 0 | 从无 | 移出目录 |
| `math-tutor` | 114 | 0 | 从无 | 收窄 |
| `miora-cli-image` | 111 | 0 | 从无 | 收窄 |
| `local-ops-playbook` | 110 | 0 | 从无 | 收窄 |
| `data-audit` | 107 | 0 | 从无 | 收窄 |
| `windows-shell-contract` | 104 | 0 | 从无 | 收窄 |
| `zh-ai-tells` | 104 | 1 | 2026-11-13 07:46 | 保留 |
| `code-reviewer` | 102 | 0 | 从无 | 移出目录 |
| `wiki-index` | 101 | 0 | 从无 | 收窄 |
| `client-secret-exposure-audit` | 100 | 0 | 从无 | 移出目录 |
| `whale-widget-patch` | 100 | 0 | 从无 | 收窄 |
| `understand-explain` | 97 | 0 | 从无 | 移出目录 |
| `changelog-entry` | 95 | 0 | 从无 | 移出目录 |
| `agent-memory-systems` | 86 | 0 | 从无 | 收窄 |
| `humanizer-zh` | 82 | 1 | 2026-11-13 07:46 | 保留 |
| `code-review-checklist` | 81 | 0 | 从无 | 移出目录 |
| `changelog-automation` | 75 | 0 | 从无 | 移出目录 |
| `agent-evals` | 70 | 0 | 从无 | 收窄 |
| `architecture-review` | 68 | 0 | 从无 | 移出目录 |
| `clean-code` | 67 | 0 | 从无 | 移出目录 |
| `clean-code-guard` | 65 | 0 | 从无 | 移出目录 |
| `agent-evaluation` | 63 | 0 | 从无 | 移出目录 |
| `agent-orchestrator` | 62 | 0 | 从无 | 移出目录 |
| `example-kb` | 62 | 0 | 从无 | 移出目录 |
| `agent-evaluation-reporting` | 55 | 0 | 从无 | 移出目录 |
| `api-design-principles` | 55 | 0 | 从无 | 移出目录 |
| `code-review-and-quality` | 55 | 0 | 从无 | 收窄 |
| `agent-observability` | 53 | 0 | 从无 | 收窄 |
| `architecture-patterns` | 53 | 0 | 从无 | 移出目录 |
| `agent-memory-discipline` | 48 | 0 | 从无 | 收窄 |
| `code-review-sensei` | 44 | 0 | 从无 | 移出目录 |
| `agent-memory-mcp` | 42 | 0 | 从无 | 移出目录 |
| `architecture-decision-records` | 42 | 0 | 从无 | 移出目录 |
| `api-designer` | 41 | 0 | 从无 | 移出目录 |
| `code-review-excellence` | 40 | 0 | 从无 | 移出目录 |

## 三档汇总与算账

| 档位 | 条数 | 现在字符 | 执行后字符 | 省下字符 | 省下 token（4 字符≈1 token） |
|---|---:|---:|---:|---:|---:|
| 保留（不动） | 8 | 1,810 | 1,810 | 0 | 0 |
| 收窄（改 description） | 53 | 11,531 | 2,823 | 8,708 | 2,177 |
| 移出目录（摘链接） | 38 | 7,744 | 0 | 7,744 | 1,936 |
| 合计 | 99 | 21,085 | 4,633 | **16,452** | **约 4,113** |

对比基线：省下 16,452 / 21,085 = **78.0%**。每轮常驻 description 从 21,085 字符降到 4,633 字符。

拆开看，两档的性质不同：

- **移出档是硬收益**。7,744 字符全部来自 38 条一次没用过的条目，移出后不留替代品，省得干净。代价是这些东西从目录里消失，模型不会主动想起它们。
- **收窄档是估算上限**。8,708 字符省自把平均 217 字符的描述压到 53 字符。压得这么狠，前提是触发词保留在草稿里，而草稿命中率没有实测过。真实收益取决于重写质量，这一档建议分两批走，第一批只改字符数 ≥300 的 12 条，跑两周看触发有没有掉。

### 移出档的分组理由

| 组 | 条数 | 字符 |
|---|---:|---:|
| 纯代码画画风格库（12 条） | 12 | 5,447 |
| 前端与代码图谱（4 条） | 4 | 692 |
| 代码规范与发布流程（5 条） | 5 | 452 |
| 代码审查变体（5 条，另有 `code-review-and-quality` 留下） | 5 | 408 |
| agent 编排与评估重叠（5 条） | 5 | 357 |
| 企业架构与后端框架（6 条） | 6 | 326 |
| 示例条目（1 条） | 1 | 62 |

- **纯代码画画风格库**：`cel-glow-atelier`、`clay-cel-canvas`、`dollhouse-maker`、`flat-painterly`、`geometric-illustration`、`maquette-painter`、`paper-stage`、`pastel-picturebook`、`polygon-cel`、`soft-diorama`、`tableau-vivant`、`vinyl-cel`。12 条同一批「纯 Python 画图、不接生图模型」的风格库，单条 435 到 479 字符，合计 5,447 字符，30 场 0 调用。本机真正的出图路径是 `pixel-art`（字符网格管线）与 `miora-visual-fallback` / `doubao-image-poster`（生图模型），这三条都在保留或收窄里。要回头用，一条 `mklink /J` 就能挂回来，源目录不受影响。
- **代码审查变体**：`code-review-ai-ai-review`、`code-review-checklist`、`code-review-excellence`、`code-review-sensei`、`code-reviewer`。五个都是「审 diff 找 bug」，留 `code-review-and-quality` 一个够用，它是多维审查且描述只有 55 字符。
- **agent 编排与评估重叠**：`agent-squad`、`agent-orchestrator` 与本机实测过的 `agent-team-playbook` 重复；`agent-evaluation`、`agent-evaluation-reporting` 被 `agent-evals` 覆盖；`agent-memory-mcp` 与本机已装的 agentmemory 服务加 `local-ops-playbook` 重复。
- **企业架构与后端框架**：`api-design-principles`、`api-designer`、`architecture-decision-records`、`architecture-patterns`、`architecture-review`、`clean-code`。全部面向 Java/Spring 那类企业后端叙事，与老大的方向不沾边。
- **代码规范与发布流程**：`clean-code-guard`、`changelog-automation`、`changelog-entry`、`commit`、`client-secret-exposure-audit`。Sentry 的 commit 约定、Keep a Changelog 流程都属于团队协作产物，本机是单人工作区，且 `security-audit` 已覆盖安全审查那一块。
- **前端与代码图谱**：`frontend-design` 与收窄后的 `web-design-engineer`、`ui-ux-pro-max` 重复；`understand` / `understand-explain` 要配 Ontoly Software Graph 那套外部服务，本机没装。
- **示例条目**：`example-kb` 是演示写法的样例，不承担实际任务。

## 收窄档的 53 条草稿

草稿全部 ≤130 字符，保留了原描述里的触发词（「时读」「用户问」「当……时」这类句式原样留着，因为那是模型判断该不该加载的线索）。三条原描述本来就只有 48 到 55 字符的，标「维持原文」：重写省不出字，反而可能丢掉触发词。

| skill 名 | 原字符 | 新字符 | 收窄后的 description 草稿 |
|---|---:|---:|---|
| `ui-ux-pro-max` | 495 | 56 | 做页面、组件、设计系统、可访问性、响应式布局、排版配色时读。内含本地可搜的样式、配色、字体搭配与 UX 规则库。 |
| `web-design-engineer` | 455 | 49 | 做或重做浏览器里的视觉产物（页面、仪表盘、原型、幻灯片、动效、可视化）时读，含浏览器验收与 QA。 |
| `genoffice` | 443 | 56 | 本地建、转、读、改 Office 文件：pptx、带公式的 xlsx、docx、PDF，含格式互转与结构化改写。 |
| `spec-driven-development` | 431 | 38 | 开新项目或大改但还没有规格时读：先写 PRD 与能力地图，再把需求拆成模块。 |
| `sw-dialogue` | 419 | 60 | 写或改对白时读：麦基《对白》的动作性、潜台词、on-the-nose 诊断，合并 Walter 与 Hicks 的方法。 |
| `avoid-ai-writing` | 403 | 78 | 英文稿去 AI 味：审计并改写 README、changelog、release note、PR 描述、博客，消除机器腔。支持只检测、改写、原地改三种模式。 |
| `security-audit` | 359 | 42 | 代码库、API、服务、CLI、库的安全审查：安全问题、定向评审、漏洞研究、渗透测试。 |
| `incremental-implementation` | 343 | 30 | 改动跨多个文件时读：切成可验证的小片，逐个落地，配特性开关。 |
| `idea-refine` | 335 | 33 | 想法还模糊时读：结构化发散再收敛，压成可执行概念，含假设压力测试。 |
| `debugging-and-error-recovery` | 323 | 31 | 测试失败、构建崩、昨天还好今天坏了时读：系统化找根因，不靠猜。 |
| `skill-creator` | 319 | 49 | 新建或改写 skill、优化 description 触发率、跑 skill 评估与基准测试时读。 |
| `find-skills` | 303 | 42 | 用户问「有没有能干 X 的 skill」、想扩展能力、要装第三方 skill 时读。 |
| `reverse-engineer-anything` | 300 | 48 | 逆向原生、托管、Electron、打包、固件、浏览器目标：解释功能、对比版本、反编译、指导重建。 |
| `morph-icon-button` | 298 | 56 | 做会形变的图标按钮时读：汉堡与叉、太阳与月亮、播放与暂停、展开与收起。基于 morphicons，含四条实测坑。 |
| `obelisk` | 293 | 51 | 检索本机历史会话。用户问「上次怎么修的」「之前那个 session」，或需要旧上下文、要写入记忆时读。 |
| `mcp-builder` | 277 | 69 | 写 MCP 服务器时读：Python FastMCP 与 Node/TypeScript MCP SDK 两条路线，工具设计、传输、鉴权。 |
| `course-guide` | 269 | 38 | 课程选题路由：给它话题或正在卡的 bug，指出讲这个内容的课程与下一步命令。 |
| `goutoujunshi` | 249 | 52 | 恋爱与关系问题：心动、暧昧、聊天记录分析、约会、冲突、分手复合。分析信号、设计推进或退出策略、润色话术。 |
| `pixel-loom` | 248 | 53 | 通过 Pixel Loom MCP 编辑器画、改、看分层像素画与逐帧动画，导出 sprite sheet。 |
| `planning-and-task-breakdown` | 237 | 27 | 有规格但要拆成可执行任务时读：排序、估范围、找并行点。 |
| `miora-visual-fallback` | 235 | 55 | 代码画不出来的视觉（插画、海报、封面、立绘、质感）转妙境 Miora 生图，再做本地排版抠图，含积分成本核算。 |
| `pixel-art` | 229 | 58 | 画像素画、像素图标、sprite、像素动图时读：字符网格到放大渲染的可跑管线、分层绘画法、Saint11 技法提炼。 |
| `lieflat-charts` | 226 | 51 | 数据可视化与演示：模板驱动的 HTML 图表、整页报告、HTML/PDF 演示，按数据语义自动选色板。 |
| `code-film-production` | 213 | 56 | 用代码做带配音和字幕的短片时读（不接视频生成模型）。时间轴由语音实测反推，含 libass 字号换算与抽帧验收。 |
| `start-learning` | 213 | 37 | 课程一次上手：访谈、分班测验、写出 LEARNING.md 长期学习计划。 |
| `claude-certification` | 210 | 64 | Claude 认证四条线（CCAO-F、CCDV-F、CCAR-F、CCAR-P）：选线、备考、实操 lab、模拟考与弱点补强。 |
| `learn` | 206 | 54 | AI Engineering from Scratch 互动带学：取下一课、终端里逐节讲、出题小测、记进度。 |
| `find-your-level` | 201 | 35 | 分班测验：把水平映射到 523 课 20 个 phase 的合适起点。 |
| `static-delivery-check` | 194 | 50 | 交付静态网页前读：用可读数值验收对比度、渲染尺寸、内联图，附单文件 HTML 打包骨架与三条实测坑。 |
| `anthropic-engineering-mindset` | 172 | 75 | 接手任何要交出去的活（写码、写文档、做方案、查问题、跑分析）开工前读：先挖未知再开工、抬到结果层、验证回路、交付后考人。含本机适配与五处会翻车的地方。 |
| `learn-mcp` | 156 | 38 | MCP 路线带学：构建、加固、调试、验证 MCP 客户端与服务端，每次一课。 |
| `dsh-plugin-authoring` | 153 | 67 | 写或修自有 DSH 插件时读：webServer 静态伺服 + 注入 UI 的四件套结构、link 安装、三个能把 DSH 搞崩的坑。 |
| `dsh-tui-guide` | 152 | 42 | 问 dsh-tui 本身的事（用法、快捷键、配置、主题、迁移、VS Code）时读。 |
| `grilling` | 152 | 34 | 要拷问自己的方案、决定、想法时读：连续追问，直到站不住的地方露出来。 |
| `check-understanding` | 149 | 51 | 课程阶段小测：quiz me、test phase 3、check my understanding。 |
| `jev-dsh` | 146 | 70 | 把 Jev 决策模型接进 DSH，或 mcp__jev__* 工具消失、报认证错时读。含 insert 写法、key 存法、四个工具怎么选。 |
| `learn-agent-skills` | 143 | 49 | Agent Skills 工程路线带学：创建、发现、调用、加固、评估、打包 skill，每次一课。 |
| `egtool-proxy` | 140 | 77 | 境外网站打不开（raw.githubusercontent.com 被重置、TLS 握手失败、超时）时读：egtool 测速选节点、开系统代理、抓完停止。 |
| `agent-desktop` | 139 | 124 | 用 agent-desktop 操作 Windows 桌面应用（UIA 无障碍树 + @ref 语义点击，不走截图坐标）。要读窗口内容、点按钮、填表单、在某个 App 里找元素时读。含本机实测 6 个必知坑与和 windows-mcp 的分工判据。 |
| `huashu-excel` | 123 | 43 | Excel 与 CSV 数据分析全流程：体检脏表、清洗、定口径、算指标、对账、出报告。 |
| `diagnose-windows-sandbox-acl` | 121 | 52 | DSH 沙箱出现意外的文件访问拒绝、工作区写入或列目录失败时读：一条命令查该路径及每一级祖先的 ACL。 |
| `math-tutor` | 114 | 47 | 高数（同济第八版）辅导与诊断：第一章极限论结构、定理清单、八个实测误解。讲题、出题、批改时读。 |
| `miora-cli-image` | 111 | 50 | 用妙境 CLI 一条命令生图并自动归档记账时读。默认 draft 档 0 积分，付费档必须显式确认。 |
| `local-ops-playbook` | 110 | 80 | 本机运维手册：驱动与设备异常、EdgeGo 代理、Codex 接 DeepSeek、MinGW 工具链、agentmemory 与 cli-hub 出问题时读。 |
| `data-audit` | 107 | 43 | 拿到陌生数据表、判断数字能不能用、查两个数为什么对不上时读。含七条红线与脏表体检顺序。 |
| `windows-shell-contract` | 104 | 68 | Windows 上用 pwsh 扫描、删链接、改环境变量、查路径前读：junction 三态陷阱、安全删除法、持久 shell 不刷新。 |
| `wiki-index` | 101 | 46 | 工作区 Markdown 索引的生成与对账：查索引漂移、孤儿页、死链，验证编号目录是否登记。 |
| `whale-widget-patch` | 100 | 78 | 改 DSH 小鲸鱼挂件（dsh-whale-widget-plus）源码时读：apply_patch.py 用法、必须同时改两份正文、改完完全退出 DSH。 |
| `agent-memory-systems` | 86 | 54 | agent 记忆架构：短期（上下文窗口）与长期（向量库）怎么分工、怎么组织成认知结构。设计或改记忆系统时读。 |
| `agent-evals` | 70 | 61 | 为 agent 搭自动化评估套件：黄金数据集、评分量表、回归门禁。发布 agent 功能、验证提示词改动、卡质量门禁时用。 |
| `code-review-and-quality` | 55 | 55 | 维持原文（原描述已 ≤70 字符，重写不省字） |
| `agent-observability` | 53 | 53 | 维持原文（原描述已 ≤70 字符，重写不省字） |
| `agent-memory-discipline` | 48 | 48 | 维持原文（原描述已 ≤70 字符，重写不省字） |

## 怎么验证这个结论

三条路，成本从低到高。

1. **重跑这次统计**。解压与统计脚本的做法已经写在全量表上方的口径段里：按 `28 B5 2F FD` 切帧、逐帧 `zstdDecompressSync`、逐行 `JSON.parse`、只认 `type == "tool/call"` 且 `data.name == "skill"` 的事件，skill 名从 `JSON.parse(data.arguments).name` 取。同一批文件重跑，24 个 skill 名字和调用次数应当完全一致，只有帧数和字节数会随父会话增长。
2. **换口径交叉验一次**。用 `obelisk --search "superagent"` 查历史会话，看命中的会话数与统计里的 3 场是否对得上。这个口径覆盖全部 707 场，能立刻回答「30 场窗口之外的调用有多少」。
3. **执行后用下一轮的真实注入量对账**。移出与收窄做完后，新开一场会话，从 `session.v4.jsonl.zstd` 里抽 `<available_skills>` 块重新量一次字符总数，目标值 4,633。对不上说明哪条没改到。

## 执行顺序建议

1. **第一批（当天可做，省 7,744 字符 / 1,936 token）**：摘掉 38 条链接。只对 junction 做 `[System.IO.Directory]::Delete($p,$false)`，不动 `doubao-image-poster` 这类真实目录，也不进链接内部删任何东西。
2. **第二批（省 8,708 字符的约七成）**：只重写字符数 ≥300 的 12 条。改完量一次长度确认 ≤480（DSH 的 `catalogDescriptionMaxLength` 默认 500，超过会被静默截断）。
3. **第三批（两周后）**：看第二批的触发有没有掉，再改剩下的 41 条。

风险提示：12 条画画库和若干 art skill 的 description 曾经被 `42-pixel-loom像素画\artifacts\trim-art-descriptions.ps1` 打过收窄补丁。这次只是摘链接，源目录与补丁都不受影响，重新挂回后仍是打过补丁的版本。

## 临时文件清理声明

本次调研只在 `%TEMP%` 下建过脚本与中间结果，全部已删除：

- 脚本：`sk_decomp.js`、`sk_probe.js`、`sk_catalog.js`、`sk_catalog2.js`、`sk_find.js`、`sk_stats.js`、`sk_verify.js`、`sk_verify2.js`、`sk_dup.js`、`sk_final.js`、`sk_plan.js`、`sk_list.js`、`sk_md.js`
- 中间数据：`sk_catalog.json`、`sk_perfile.json`、`sk_final.json`、`sk_plan.json`、`sk_gen.json`

除本文件外没有修改任何文件，`~/.dsh/skills` 下的链接、各 `SKILL.md` 一字未动。
