---
status: 草稿
updated: 2026-10-23
ttl: 180d
tags: [方法论, agent, harness, 可观测性]
---
//
ComfyUI，一个ai视觉创作工具，最近才进入我的视野。但是相较于视觉创作，我更感兴趣的是这个工具内的工作流。
## 有向无环图
这是它吸引我的地方，ai的工作被限制在节点和连线之间，还可以通过*loRA*进行微调
## loRA

- 核心思想：插件化挂载，极小，可训练
- 优点：减少微调成本，可以避免全量微调的“冲刷”效应同时保留所有技能



# 我想做什么
当然，我无论如何都用不上本地部署大模型和视觉创作工作流，但是这种掌控感让我非常着迷。我在使用agent的过程中，ai对我来说基本上如同黑箱，时常超出我的控制，结果也不符合预期。我尝试个性化定制，搭自己的harness，但是面对一个黑箱，盲目操作也只是徒劳。于是，如标题所说，我的目标~~吹牛~~是结合现有破除ai迷雾的手段，真正地了解AI的运作方式，已达到个人化改进harness的目的

**验收标准**（写给自己，防止这个目标无限延伸）：能回答「这次结果不对，是哪一步偏的」，就算入门；能在不重跑整条流程的前提下改那一处并验证，就算达成。

**补充事实（2026-10-23 实测）**：这条路线不用从零开始 —— 本机 `~\.dsh\sessions` 里已经有 **486 个会话文件 / 131.5 MB**（解压 477 MB），全量扫描 **3.98 秒**，**不装任何插件就能读**。里面已有工具调用、reasoning 全过程；缺的是 token 用量与延迟（`usage`/`duration`/`ttft` 字段完全不存在）。详细拆解见 `60-AI黑箱观测\PLAN.md`。



# 1.可观测性
打开黑箱的第一步是观测，除了原生日志可用的插件有
- dsh-sseye（就像DevTool），展示调用内容，保留turn/step归因 → **已核实**：真包 `dsh-sseye@0.4.4`（MIT），README 明示支持 DSH `0.2.0-rc.2`。**desktop 版不能在命令行装**（CLI 拒绝 `--profile desktop`），要走应用内侧边栏 Plugins 页
- ~~dsh-insider~~ → **真包名 `@howardchan/dsh-insider`**（0.1.8），展示插件、服务等之间关系图谱。**风险**：开发基线是 DSH `0.1.5-rc.1`，未验证 `0.2.0-rc.2` —— 跟 25 号 context-lens 失败是同一类原因（上游 API 拆名 → `failed to import`）
-  @loongsuite/dsh-plugin 把事件转换标准格式otel，接入专业监控系统 → **已核实**：真包 `@loongsuite/dsh-plugin@0.1.3`
- langfuse：（待补：它跟上面三个的分工是什么，从哪看到的）→ **已核实**：npm 上的 `langfuse` 是 v3 SDK，**官方已标废弃**；且需要外部后端服务，**本机没装 Docker**，装不了

# 2.可视化
我毕竟还是新手，对于大量的信息，可视化是让我快速理解的有效途径。像剥洋葱般，从外到内，~~第一步应该是展示agent的工作流~~（先有观测数据才画得出图，所以可视化排在观测之后）。也许可以利用canvas？
 可用的插件有：
 - ~~workflow-studio：结构图，依赖的连线，时间线泳道，常驻进度条~~
   → **核实有误**：`workflow-studio` 是 Vercel Workflow DevKit 的 devtools，**不是 DSH 插件**，装不上。这条原记录错误，保留划线存档
# 3.可复用化
可视化的最终目的是让工作流从“一次性技巧”变成可版本化、可分享的工程资产。
可用的插件有
- ~~dsh-dag-workflow~~ → **真名 `@dsh-external/workflow`**（GitHub 149⭐），坚持==一个真源==，把工作流，模版化，文件化。**注意**：npm 上没有叫 `dsh-dag-workflow` 的包；且它的 `compatibility.json` 钉死 DSH `0.0.1-rc.2` 快照，本机是 `0.2.0-rc.2`，差一个大版本，装前必须先验兼容
- dsh-external/workflow，将工作流升级为具备命名、暂停/恢复、按快照重跑、成本记录和持久化证据的完整产品层，让多 Agent 流程变成可审计、可分享的团队资产。




//万博士言：
1. 理论深潜：若想真正看清AI内部逻辑，不能仅停留在输出端，需深入模型底层参数。重点调研 J Space 与 Z Space 概念，理解大模型内部的潜空间与特征表达机制。

2. 工程对标：在着手开发类似 Canvas 的可视化插件前，强烈建议研究开源项目 Langfuse。重点看它如何实现 Tracing（链路追踪）、Prompt 管理及工作流可视化。




## 对「万博士言」的核实（2026-10-23）

**第 1 条：J Space 是真的，但 Z Space 不能跟它并列。**

| 项 | 核实结果 |
|---|---|
| **J Space** | ✅ 真实且重要。全称 **Jacobian space**，出自 Anthropic 2026-07 的 [book-length 论文](https://transformer-circuits.pub/2026/workspace/index.html)。用 Jacobian 矩阵把模型中间层的向量状态，换算成「此刻哪些 token 最可能被说出来」→ 相当于模型的**工作记忆**。配套工具叫 **J-lens**，能读到模型「想了但没说」的内容。[IBM 的解读](https://www.ibm.com/think/news/what-anthropic-j-space-research-means-future-ai)讲得最清楚。Anthropic 明确说这**不证明模型有意识**，纯粹是功能层面的可解释性工具 |
| **Z Space** | ⚠️ **不是同一层的东西**。在生成模型语境里 `z-space` 就是**潜空间（latent space）**的通称 —— 自编码器/扩散模型里那个压缩表示的 `z`。它是通用叫法，不是某个具体理论；把它和 J Space 并列写，会让人以为两者是配套概念，实际不是 |
| **结论** | J Space 值得深挖（它确实是「看清 AI 内部」的主流前沿）；Z Space 写上去价值很低，建议改成「潜空间（latent space，通用概念）」并说明它跟 J Space 不是一回事 |

**第 2 条：Langfuse 的方向对，但本机装不了。**

- Langfuse 确实是开源的 LLM 可观测平台，Tracing / Prompt 管理都有。**拿它当「工程对标」学习它的设计是对的**。
- 但 npm 上的 `langfuse` 是 **v3 版 SDK，官方已标废弃**（`NOT the latest Langfuse JS/TS SDK`），新版要装 `@langfuse/tracing` + `@langfuse/otel`。
- 更要紧的是：**它需要外部后端服务，本机没装 Docker**（`docker`/`podman` 都没有）。所以「装一个 langfuse 来看」这条路本机走不通，只能读它的设计文档。
- 想在本机真跑起来，得先解决「往哪存」的问题 → 记进 `60-AI黑箱观测\PLAN.md` 阶段 4。



==以上内容均未经作者实践验证，仅供参考，待作者学成归来再调整==
