---
status: 占用
updated: 2026-11-03
ttl: 365d
tags: [方法论, AI使用, Claude, 提示词]
---

# AI 野心四维度（Felix Rieseberg）

> 一句话：大家在用 AI 时都太保守了，这篇文章把「怎么更有野心」拆成四个可以分别动手的维度。
> 什么时候该读它：你发现自己在让 AI「帮我改一下这个」「帮我把这段润色一下」，活儿干得挺快但没什么长进的时候。

## 原文是谁写的、在哪

| 项 | 内容 |
|---|---|
| 作者 | **Felix Rieseberg**，Anthropic 的 EM（工程经理），带过 Claude.ai、Cowork、Claude Code Desktop 的工程；Electron 的共同维护者，之前在 Notion、Stripe、Slack、Microsoft |
| 标题 | How to be more ambitious: A tutorial for humans with models |
| 发布 | 2026-10-13，作者个人站 [felixrieseberg.com](https://felixrieseberg.com/how-to-be-more-ambitious-a-tutorial-for-humans-with-models/) |
| 中文转载 | 36氪、量子位、掘金等多处有译稿，但**多篇译稿缺后半段**（"We have been here before" 那一节把「为什么需要这样做」讲透了，别只读译文） |

作者自己在文末写明职位，不是二手转述。文中的公司 Acme Inc 是他虚构的例子。

## 核心主张

他身边做 AI 的人有个口号：「我们在使用 AI 上还不够有野心」。他自己说这口号好喊难做，所以写了这篇教程。

他把「野心」拆成四类，明确跳过第一类：

| 维度 | 意思 | 他的态度 |
|---|---|---|
| **Speed 速度** | 同样的事，用更少时间 | **跳过**。路子已经很成熟，不需要创意 |
| **Altitude 高度** | 往抽象层升，盯结果而不是盯任务 | 重点讲 |
| **Depth 深度** | 用这个领域顶尖专家的做法，往下摸到原始数据和工具 | 重点讲 |
| **Range 广度** | 从多个领域借方法和人 | 重点讲 |

跳过 Speed 这个选择本身就是这篇文章的态度：**「更快」是 AI 给的最低档价值，别再把它当目标。**

## 一、Altitude：先问「你想让什么发生」

### 原理

你交给模型的那个任务，只是**你对「怎么达成目标」的一次猜测**。写「帮我修这个崩溃」，等于把你对问题范围的理解也一起锁死了。往上升一层，交出目标，让模型自己去找方法。

他自己的例子：前年还让 Claude 修一个个具体的崩溃，后来直接要求「让我们的应用永远不崩溃」。

### 三步递进（拿季度业务评审 QBR 举例，原文原样搬）

**第一步：让它搞懂业务发生了什么**

> Tell me what changed this quarter, why, and what we should do about it.

他特别点了一条：**抵抗住「只把模板里的数字更新一下」的冲动**。要把能拿到的数据全给它。

**第二步：让它指出必须做哪些决定**

> Work out which three decisions are most urgent in the face of everything you've learned, and start working on potential directions with for/against arguments.

**第三步：让它把整个工作机制 redesign**

> The review exists so leadership catches problems early and moves money and people to the right places. Design a better way to do that than a quarterly deck, show from our own history that it would have caught things sooner, and run it beside the current one for a quarter.

### 什么时候该抬高度（判据）

作者给的信号很好用：**如果这件事之后要开会讨论，就值得再往上升一层。**

不适合的情况：创作类任务。写小说、写文案这种，抬到「让读者哭」反而不如给具体的写法要求。

### 最容易漏的一条

抬高度**通常要求你先改工作流，不是改措辞**。

他要 Claude 保证应用不崩溃，先后做了三件事：让崩溃报告能轻松拿到、把每个版本对应到具体 git commit、给它 Windows/macOS/Linux 三台机器能自己复现和验证。这三件事做完，「让应用不崩溃」这句话才不是空话。

**不做这些，模型照样只能等你喂下一个任务。**

## 二、Depth：用顶尖专家用的器械

### 原理

你让它「像专家一样」，**它通常只换词汇不换方法**。原文这句是全篇最扎人的：

> When prompted to act like a professional, models often switch vocabulary without actually switching methods.

判断是不是真的进到深度：看它拿的工具。它伸手去画 Markdown 表格，你就坚持让它用数据库和统计模型。

一句让人安心的话：**你不必事先知道专家用什么，Claude 通常知道。**

### 四条具体做法

**1. 设一个不合理的目标，然后告诉模型这非常合理**

原文例子：网站加载要从 10 秒降到 5 秒？直接要 200 毫秒。模型说做不到？再强调一遍这问题完全可以解，继续试。

**2. 挑战你已经确定的事**

> We believe churn is an onboarding problem. Try to reproduce that from the raw data.

商业里很多「我们以为是真的」的东西只是对现实的粗略近似。给它原始数据，让它用怀疑态度去验。

**3. 产出新知识**

接着上一条往下钻：

> Build a simulation of our customers that reproduces the last year from our logs. Then find the earliest signal of churn we don't know about yet, and show it predicts on data it has not seen.

**4. 用外部裁判，别让模型自己给自己打分**

原文写得很直接：**Do not ask the model to judge itself.** 他预期这个毛病将来会被模型智能提升解决，但在 2026 年 10 月，他请求你坚持用外部验证。最理想的验证方式是一个模型自己能跑、能立刻拿到反馈的工具。

### 那段可以直接抄的四步提示词

他常用来把任务「变深而不是变大」：

> I want a deeper version of this task, not a bigger one. Before rewriting it:
> (1) say what a top professional in this field would do here that I'm not asking for;
> (2) name the rawest material they would start from instead of what has already been digested for me;
> (3) name the specific methods and tools they would use;
> (4) say what, other than you or me, would show the result is right.
> Then rewrite my prompt to ask for exactly that.

中文版：

> 我要这个任务更深，不是更大。在改写之前，先做四件事：
> (1) 说这个领域顶尖的人在这里会做什么、而我没要求；
> (2) 说出他们会从哪些最原始的材料入手，而不是已经被消化好的东西；
> (3) 说出他们会用哪些具体方法和工具；
> (4) 说出除了你我之外，还有什么能证明结果是对的。
> 然后把我的提示词改写，准确要求上面这些。

注意 (2) 那条：**「已经被消化好的东西」**指的是别人给你的摘要、图表、二手结论。这条对查资料特别有用。

## 三、Range：从别的领域借方法和人

### 原理

过去跨领域不划算，请不起那么多专业的人。现在多 agent 便宜了（多线程、编排器、或者干脆就是 subagent），跨领域的成本掉下来了。

他的例子：看到同事 Alex Albert 用 Blender 渲染室内设计，他才想到用 Claude 给自己的主页做 3D 环境，而他自己以前做游戏的。

### 两个做法

**1. 把名词换成动词**

出自 Tom Kelley 和 Jonathan Littman 的《The Art of Innovation》。季度评审不是「一个 PPT」，是：

> 30 分钟内让一群很忙的人相信一个关于因果的故事，并在信息不全的情况下做出决定。

换成动词之后，你和模型都能看出哪些领域可能带来有用的视角。

**2. 组队**

他的玩笑：这份 PPT 必须惊艳，否则外星人要炸地球。那人类会组一支什么队伍？

生产级的配置：分析师、记者、编辑。要上台讲：再加动效设计师和音频设计师。再来一支对抗性的红队：做空者和法务会计。

### 这条最容易做歪

**别搞成角色扮演。** 原文原话：很容易不小心滑进角色扮演，那只会让你的对话记录更怪，不会让结果更好。

正确用法是拿这支队伍**找出领域和技术**：不要让它「以做空者的口吻写作」，要让它「像做空者研究一个仓位那样去做研究」。

## 四、四维度共同的前提

他明说：上面每一种野心，都要求模型有一台**能装软件、能拿到相关数据**的电脑（走 MCP 连接器或其他方式）。市面上有不少工具能做到这一点，他建议你用一个。

**没有这个前提，前面三条都停在「问答」层面。**

## 五、后半段：为什么这件事难（别跳过这节）

他引用心理学里的一个词：**自我效能（self-efficacy）**。自我效能高的人更相信自己的能力，结果真的做得更多。而且有证据表明它可以被撬动。

几条有据可查的撬法：

- **相信一件难事可能，你更可能做成。** 1939 年 George Dantzig 迟到了伯克利的统计课，从黑板上抄了两道题，当成作业交上去。那是两道当时未解的统计学难题。
- **看别人做成，你更可能做成。** 四分钟一英里被人类追了几十年，Roger Bannister 在 1954 年 5 月破掉，而他独享这个纪录只有 46 天。同样的戏码正在两小时马拉松上重演。
- **给自己失败的许可。** 《The Inner Game of Tennis》的核心论点：你如果非常非常用力，反而打不出最好的球。搞工程的人可能记得 Linux 当年的公告：「只是个爱好，不会像 GNU 那样做大做专业」。

他还引了 Locke 和 Latham 对 35 年目标设定研究的综述：**被给到具体困难目标的人，表现好过只是被告知「尽力就好」的人。**

米开朗基罗当年不愿画西斯廷礼拜堂，因为他是雕塑家不是画家。这节的落点是：**人一直被「自己相信什么是可能的」限制住。**

## 六、对本机的判断：哪几条现在能用

> 这一节是我的判断，不在原文里。

| 原文主张 | 在本机的情况 |
|---|---|
| 「模型要有能装软件、能拿数据的电脑」 | ✅ 已经是。DSH + pwsh + 文件读写，本机就满足 |
| 「别让模型自己给自己打分」 | ✅ 这条**工作区已经在做了**，就是 `prompt-inject.md` 里「默认未验证，除非有正面证据」那条的另一种说法。两边独立得出同一个结论，可信度更高 |
| Altitude（抬高度） | ✅ 最值得练。但门槛在工作流不在措辞。他那个「让应用不崩溃」的例子对照本机的 67 号轨迹审计、60 号 trace.py，就是同一个思路：先让模型拿得到原始数据，再谈抬高度 |
| Depth（不合理目标） | ⚠️ 能用，**但必须配停止条件**。见下一节 |
| Range（组队） | ⚠️ 本机 agent team 位子有限（名册满 8 个要开新会话），大规模组队不现实。但「名词换动词」这条零成本，随手能用 |
| Speed（跳过） | 认同。用 AI 只图快，等于只用了它最便宜的那一档 |

### 名词换动词，拿你自己的事举例

| 原来的说法（名词） | 换成动词之后 |
|---|---|
| 「写一篇指针算术的笔记」 | 「两周后不看笔记也能讲清 p+1 为什么跳 4 字节，并当场举一个会出错的例子」 |
| 「做个 C 语言作业」 | 「让这个程序在老师可能给的边界输入下都不出内存错误，并且我能解释每一处为什么安全」 |
| 「整理一份文学社活动方案」 | 「让三个意见不合的人在一小时内对宣传口径达成一致，且谁也不觉得自己被压了」 |

第三行那个换法，会直接暴露真正的难点在哪。

## 七、哪里会翻车

> 同样是我的判断。

**1. 「设不合理目标」没有停止条件会烧钱。** 原文说模型说做不到就再强调一遍继续试。本机的模型是按量计费的（千问、AI97 都有过余额/风控记录），一条「继续试」可能跑掉几十次调用。**用之前先定好：试到第几次停、停下来的时候要它交什么（比如「列出你试过的方案和各自卡在哪」）。** 没有这个，这条会从「有野心」变成「没效率」。

**2. 「Claude 通常知道专家用什么」这句存疑。** 文章自己前半句刚说「模型被要求像专家时只换词不换方法」，后半句又说它知道专家用什么。这两句在逻辑上是紧的：它知道**名词**，不一定知道**方法**。我用下来，让它列「专家会用什么工具」它答得像模像样，但真到执行时还是滑回方便的路子。所以那段四步提示词的第 (3) 步，**得追一句「现在就按这个工具做，不要用 Markdown 表格替代」**。

**3. 原文几个鼓动性的案例我没核实。** 「可能帮助发现了一颗新行星」「Sabastian Sawe 在伦敦跑出首个官方破两小时马拉松」这两条是作者单方面陈述，bubbles-brain 的摘要页也明确标注了「not checked again here」。当故事听可以，别当事实引用。

**4. 这篇文章的隐含前提是模型能力足够强。** 它是给 Claude 写的。同一套提示词换成本机这些模型，「抬高度」容易变成空转：它答应得挺好，具体到执行又滑回原来那层。所以**先用 Depth 那套外部验证卡住，再抬高度**，顺序反了会得到一堆听起来很高级的废话。

## 还不懂

- 「抬高度」和「任务说不清楚」怎么区分？我现在的判据是「抬完之后它还能给出可验证的产物吗」，但这只是我自己想的，没验证过。
- 原文说创作类任务不适合抬高度。那写小说、写文案这类，深度和广度还能用吗？作者只说了 altitude 不适合，没说另外两个。这条他没展开。

## 关联

- 「别让模型自己给自己打分」在本机的落地版本 → [Harness² 双层递归](harness双层递归（杂）.md)
- 「默认未验证，除非有正面证据」的完整说法 → `~/.dsh/prompt-inject.md` 对应小节
- 怎么记这类方法论笔记 → [怎么记笔记](../怎么记笔记.md)
- 索引 → [学习笔记索引](../学习笔记索引.md)
