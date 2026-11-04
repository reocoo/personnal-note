---
status: 占用
updated: 2026-11-04
ttl: 365d
tags: [方法论, AI使用, Felix, Anthropic]
---

# Felix · 野心四维度

> 原文：**How to be more ambitious: A tutorial for humans with models**，2026-10-13
> 作者：Felix Rieseberg，Anthropic 工程经理（带过 Claude.ai、Cowork、Claude Code Desktop），Electron 共同维护者，之前在 Notion / Stripe / Slack / Microsoft
> 出处：https://felixrieseberg.com/how-to-be-more-ambitious-a-tutorial-for-humans-with-models/
> **操作版在 [Anthropic工程心法](Anthropic工程心法.md)，这篇是完整原文提炼，需要细节时读。**

## 核心

他身边做 AI 的人有个口号：「我们在使用 AI 上还不够有野心」。他自己说这口号好喊难做，所以写了这篇教程。

野心拆四类，**明确跳过第一类**：

| 维度 | 意思 | 态度 |
|---|---|---|
| **Speed 速度** | 同样的事用更少时间 | **跳过**。路子成熟，不需要创意 |
| **Altitude 高度** | 往抽象层升，盯结果不盯任务 | 重点 |
| **Depth 深度** | 用顶尖专家的做法，往下摸到原始数据 | 重点 |
| **Range 广度** | 从多个领域借方法和人 | 重点 |

**跳过 Speed 这个选择本身就是态度**：「更快」是 AI 给的最低档价值。

## Altitude 高度

### 原理

你交给模型的任务，只是**你对「怎么达成目标」的一次猜测**。写「帮我修这个崩溃」，等于把你对问题范围的理解一起锁死了。

他自己的路径：前年让 Claude 修一个个具体崩溃，后来直接要「让我们的应用永远不崩溃」。

### 三步递进（QBR 季度业务评审那个例子）

**第一步**：抵抗住「只把模板里的数字更新一下」的冲动，把能拿到的数据全给它。

> Tell me what changed this quarter, why, and what we should do about it.

**第二步**：让会议以决定收尾。

> Work out which three decisions are most urgent in the face of everything you've learned, and start working on potential directions with for/against arguments.

**第三步**：把整个机制 redesign。

> The review exists so leadership catches problems early and moves money and people to the right places. Design a better way to do that than a quarterly deck, show from our own history that it would have caught things sooner, and run it beside the current one for a quarter.

### 什么时候抬（判据）

**这件事之后要开会讨论吗？要，就值得再升一层。**

不适合创作类任务：写小说、写文案，抬到「让读者哭」不如给具体写法要求。

### 最容易漏的：抬高度要求先改工作流

他要 Claude 保证应用不崩溃，先后做了三件事：崩溃报告能轻松拿到、每个版本对应具体 git commit、给它三台机器能自己复现和验证。做完这三件事，那句话才不是空话。

**不做这些，模型照样等你喂下一个任务。**

## Depth 深度

### 原理，全篇最扎的一句

> When prompted to act like a professional, models often **switch vocabulary without actually switching methods**.

判断是不是真进到深度：看它拿什么工具。它画 Markdown 表格，你就坚持用数据库和统计模型。

另一句让人安心：**你不必事先知道专家用什么，Claude 通常知道。**（这句跟上面那句有矛盾，见「哪里存疑」）

### 五条做法

**1. 坚持用业内最好的器械。** 见上。

**2. 设不合理目标，然后告诉模型这很合理。** 网站 10 秒要降到 5 秒？直接要 200 毫秒。模型说做不到，再强调这问题可以解，继续试。

**3. 挑战你已经确定的事。**

> We believe churn is an onboarding problem. Try to reproduce that from the raw data.

**4. 产出新知识。** 接着上一条：

> Build a simulation of our customers that reproduces the last year from our logs. Then find the earliest signal of churn we don't know about yet, and show it predicts on data it has not seen.

**5. 用外部裁判。** 原文很直接：**Do not ask the model to judge itself.** 他预期这毛病将来会被模型智能提升解决，但 2026 年 10 月，他请求你坚持外部验证。最理想是模型自己能跑、能立刻拿到反馈的工具。

### 四步提示词（可直接抄）

> I want a deeper version of this task, not a bigger one. Before rewriting it:
> (1) say what a top professional in this field would do here that I'm not asking for;
> (2) name the rawest material they would start from instead of what has already been digested for me;
> (3) name the specific methods and tools they would use;
> (4) say what, other than you or me, would show the result is right.
> Then rewrite my prompt to ask for exactly that.

中文：

> 我要这个任务更深，不是更大。改写之前先做四件事：
> (1) 说这领域顶尖的人在这里会做什么、而我没要求；
> (2) 说出他们会从哪些最原始的材料入手，而不是已经被消化好的东西；
> (3) 说出会用哪些具体方法和工具；
> (4) 说出除了你我之外，还有什么能证明结果是对的。
> 然后把提示词改写，准确要求上面这些。

第 (2) 条最有用：「已经被消化好的东西」指别人给的摘要、图表、二手结论。

## Range 广度

### 原理

过去跨领域不划算，请不起那么多专业的人。现在多 agent 便宜了（多线程、编排器、subagent），成本掉下来。

他的例子：看到同事 Alex Albert 用 Blender 渲染室内设计，他才想到给自己的主页做 3D 环境，而他自己以前做游戏的。

### 两个做法

**1. 名词换动词。** 出自 Tom Kelley 和 Jonathan Littman《The Art of Innovation》。

季度评审不是「一个 PPT」，是「30 分钟内让一群很忙的人相信一个关于因果的故事，并在信息不全时做出决定」。

**2. 组队。** 他的玩笑：这份 PPT 必须惊艳，否则外星人要炸地球。人类会组什么队？生产级配置是分析师 + 记者 + 编辑；要上台再加动效和音频设计师；再来一支红队：做空者和法务会计。

### 最容易做歪的一条

**别搞成角色扮演。** 原文：很容易滑进角色扮演，那只会让对话记录更怪，不会让结果更好。

正确用法是拿队伍**找领域和技术**：不要让它「以做空者的口吻写作」，要让它「像做空者研究一个仓位那样做研究」。

## 四维度的共同前提

他明说：每种野心都要求模型有一台**能装软件、能拿数据**的电脑（走 MCP 或其他方式）。**没这个前提，前面三条都停在问答层面。**

## 后半段：为什么这件事难（别跳过）

心理学里的词：**自我效能（self-efficacy）**。自我效能高的人更相信自己的能力，结果真的做得更多。有证据可以被撬动。

- **相信难事可能，你更可能做成。** 1939 年 George Dantzig 迟到伯克利统计课，从黑板抄了两道题当作业交。那是两道未解的统计难题。
- **看别人做成，你更可能做成。** 四分钟一英里被追了几十年，Roger Bannister 1954 年 5 月破掉，独享纪录只有 46 天。
- **给自己失败的许可。** 《The Inner Game of Tennis》核心论点：非常非常用力反而打不出最好的球。Linux 当年公告「只是个爱好，不会像 GNU 那样做大做专业」。

还有 Locke 和 Latham 对 35 年目标设定研究的综述：**被给到具体困难目标的人，表现好过只被告知「尽力就好」的人。**

米开朗基罗不愿画西斯廷礼拜堂，因为他是雕塑家不是画家。这节落点：**人一直被「自己相信什么是可能的」限制住。**

## 哪里存疑

> 我的判断。

**1. 「模型只换词汇不换方法」和「Claude 通常知道专家用什么」两句打架。** 前者说它不懂方法，后者说它懂。合理读法是：它知道**名词**，不一定知道**方法**。所以四步提示词的第 (3) 步之后得追一句「现在就按这个做」。

**2. 「继续试」没有停止条件。** 本机模型按量计费，照原文裸用会烧钱。用前先定：试几轮停、停下来交什么。

**3. 两个案例未核实。** 「可能发现一颗新行星」「Sawe 伦敦首个官方破两小时马拉松」都是作者单方面陈述，bubbles-brain 的摘要页自己标了 not checked again here。

## 关联

- 压缩成操作流程 → [Anthropic工程心法](Anthropic工程心法.md)
- 同作者第二篇 → [Felix-饥饿胜过资历](Felix-饥饿胜过资历.md)
- 索引 → [学习笔记索引](../学习笔记索引.md)
