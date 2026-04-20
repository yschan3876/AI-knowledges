# Harness Design for Long-Running Apps — 学习笔记

## 内容概览

- **作者**：Prithvi Rajasekaran（Anthropic Labs）
- **来源**：[Anthropic Engineering Blog](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- **类型**：技术实践文章（一手工程经验）
- **学习日期**：2026-04-16 ~ 2026-04-17
- **学习方式**：苏格拉底式深度阅读（socratic-reader skill）

### 核心主旨

围绕 LLM 自主编程的两大结构性瓶颈（**Context 退化** + **自评估失效**），设计出 GAN 启发的多 Agent Harness 架构，并随模型能力演进**持续迭代简化**。核心信念：Harness 组合空间不会随模型改进而缩小，只是移动了——AI 工程师的价值在于不断寻找下一个新颖组合。

---

## 单元 1：为什么 naive 实现会失败

### 核心论点

朴素的单 Agent 长任务实现必然崩塌，存在两个**结构性瓶颈**——不是 prompt 写得不好，而是任务性质本身决定的。

### 瓶颈一：Context Window 退化

- **现象**：随着上下文窗口填满，模型的**一致性开始崩塌**
- **Context Anxiety（上下文焦虑）**：模型感知到上下文快满时会**主动提前收尾**——不是因为任务完成，而是因为它"紧张了"
- **两种解法**：

| | Compaction（压缩） | Context Reset（重置） |
|---|---|---|
| **做法** | 原地总结早期对话，继续在同一会话 | 彻底清空，新 Agent 通过结构化文件接手 |
| **优点** | 简单、连续性好 | 彻底解决焦虑，clean slate |
| **缺点** | 焦虑严重时不够 | 编排复杂、交接成本高 |

- **实测结论**：Sonnet 4.5 的 context anxiety 严重到 compaction 不够，**必须 context reset**

### 瓶颈二：Self-Evaluation 失效

- **现象**：Agent 评估自己时**倾向于自信地称赞自己的作品**——即使质量平庸
- **主观任务尤其致命**（UI 设计等没有二元 pass/fail）
- **解法**：分离 Generator 和 Evaluator
- **关键洞察**：**分离本身不消除 LLM 的宽容倾向**，但**调教一个独立 Evaluator 变严格，比让 Generator 自省严格容易得多**

### 深度讨论产出

**"分离"改变了什么？** 不止是上下文，至少有三层：
1. **上下文层**：去掉决策痕迹，消除自我一致性偏差
2. **Prompt 层**：可以给 Evaluator 专门为"挑剔"优化的 system prompt
3. **角色层**：创造了**对抗结构**——单 Agent 自评估是"法官审自己的案子"，双 Agent 是"控辩分离"

**Context Anxiety 与 DDL 效应的类比**：
- **行为表现相似**：都在"时间/资源快耗尽"时降低标准、草草收尾
- **机制差异**：
  - 人的 DDL 是**有意识的取舍**——知道时间不够主动降标准
  - Context Anxiety 是**涌现行为**——从训练数据中长对话末尾的统计模式学到

### 深层启示

**针对模型弱点做重度工程 vs 保持简单等模型变强**——倾向于"带保质期的工程"：
1. "等模型变强"不是策略，是赌博——产品不能停
2. 工程本身是认知工具——解决过程中想清楚的架构会沉淀
3. **但必须带拆除计划**——每个组件要能回答"补偿什么弱点？还在吗？"

---

## 单元 2：GAN 启发的双 Agent 架构 + 让主观质量可评分

### GAN 思想迁移

```
Generator Agent  →  产出  →  Evaluator Agent
      ↑                            │
      └──── 具体可操作的反馈 ────────┘
                 (5-15 轮迭代)
```

**关键差异**：原始 GAN 在训练阶段更新权重；Agent GAN 在**推理阶段**通过自然语言反馈更新下一轮 prompt 输入。价值相同：**把"创造"和"评判"两种认知活动在组织层面分离**。

### 主观变可评分——核心方法论

- **问题重新框架**：
  - "这个设计美吗？" ❌（难回答，无参照）
  - "这个设计符合这 4 条标准吗？" ✅（可回答，每条可打分）
- **同一套标准同时给 Generator 和 Evaluator**——两者用同一套"语言"对话，反馈才可操作

### 四项评分标准（Frontend Design）

| 标准 | 考察 | 权重 |
|------|------|------|
| **Design quality** | 整体连贯性、独特情绪/身份 | 高 |
| **Originality** | 刻意创意选择；**显式惩罚 AI slop**（如白卡片紫渐变） | **最高** ⭐ |
| **Craft** | 字体层次、间距、色彩和谐——技术执行 | 中 |
| **Functionality** | 独立于美学的可用性 | 中 |

**权重设计哲学**：**重点杠杆在模型的弱项上**。Claude 在 Craft 和 Functionality 上天然够好，真正需要推动冒险的是美学决策和原创性。

### 荷兰艺术博物馆案例

- **第 9 轮**：干净的深色主题登陆页，精致但平庸
- **第 10 轮**：**彻底推翻重来**——重构为 3D 空间体验（CSS perspective 房间、棋盘格地板、艺术品墙挂、门道导航）
- **真正启示**：**当反馈系统允许"战略 pivot"时，模型会做出创造性跳跃**——前提是前面轮次没强制收敛

### 设计模式产出

**把"主观判断"拆解为可评分标准**的通用模式：
- 代码 review → Coding style guide
- 面试评估 → STAR 法 rubric
- 产品决策 → OKR/PRD 模板

**这个方法的阴暗面**：
- 评分标准本身会**塑造结果**（作者承认："museum quality" 会把设计推向特定收敛）
- 一旦标准固化，模型会**过拟合**
- 创新空间被**标准本身的边界**限定

**破解**：加元层面矫正机制（如 Originality 对抗"被标准驯化"）

### 代码评审场景的 Criteria 推演

| 标准 | 考察 | 权重 |
|------|------|------|
| **Correctness** | 逻辑、边界、并发、错误处理 | 中 |
| **Craft** | 命名、分层、函数长度 | 低 |
| **Change Minimality** | 只触达必要范围？过度重构？防御性代码？ | **高** ⭐ |
| **Future Cost** | 6 个月后修改代价？过度抽象？ | **高** ⭐ |

**对应的 AI slop**：过度工程、YAGNI 违反、把 5 行改动写成 200 行的"清理"。

### 评估语言本身的训练污染

模型对代码的 AI slop 评价（如"代码完美、没有问题、全部执行通过"）源于**训练数据本身就扭曲**：
- Code review 评论大多是 "LGTM"、"Nice!"
- PR merge 很少写"这个方案的代价是……"
- Stack Overflow 高赞回答很少说"这个方法的失败场景是……"

**诚实表达是带边界的，套路话是无边界的绝对判断**。Evaluator 可以用一条规则抓住：**任何不带"但是 / 除了 / 在 X 假设下"的质量声明都视为可疑**。

---

## 单元 3：三 Agent 全栈架构 + Sprint Contract

### 三 Agent 架构

```
┌─────────────┐  product spec   ┌─────────────┐  sprint contract   ┌─────────────┐
│   Planner   │ ───────────────→│  Generator  │ ←─────────────────→│  Evaluator  │
└─────────────┘                  └─────────────┘                    └─────────────┘
  1-4 句 prompt                  一次一个 feature                    Playwright 交互
  → 完整 spec                     sprint by sprint                   打分 + 阈值
```

### Planner 的反直觉设计

- **关键坑**：Planner 过度指定技术细节时，错误会**级联**到下游每一行代码
- **约束**：只规划"要什么"，不规划"怎么做"
- **价值**：**主动雄心扩展 scope**——Generator 没 Planner 时会欠规划，直接开工产出单薄

### Sprint Contract ⭐（本单元核心）

每个 sprint 开始**前**，Generator 和 Evaluator 先协商 **"done 长什么样"**——**写任何代码之前**达成一致。

**机制价值**：在**模糊的 Planner spec** 和 **明确的 Evaluator 评分** 之间架桥。

**双向协商**：Generator 参与定义自己的成功标准，对齐两个 Agent 视角。

### 文件通信机制

Agent 间不对话，通过**文件**通信：
- **异步**：不需要同步在线
- **可审计**：人类可 inspect 每一步状态
- **克制**：逼迫 Agent 把模糊想法**落到文档上**

### 2D 游戏制作器：Solo vs Harness

| | Solo | Harness |
|---|---|---|
| 时长 | 20 分钟 | 6 小时 |
| 成本 | $9 | $200（22 倍） |
| 初看 UI | 符合预期 | 画布用满视口 |
| 深挖 | entity 无响应输入，**游戏实际坏了** | play 模式**真能玩** |

**Harness 贵 22 倍换到什么**：
1. spec 从一句话 → 10 sprints / 16 features
2. **功能真实可用**（抓到"表面看不出"的 runtime 断线 bug）
3. 特性丰富度（AI 辅助、可分享导出等）

### Evaluator 调教反模式

"Out of the box，Claude 是糟糕的 QA agent"：
1. 先识别真 bug，再说服自己"这不是大问题"，最后批准（最阴险）
2. 测试浅尝辄止
3. 细微 bug 漏掉

**调教循环**：读日志 → 找判断分歧 → 更新 prompt → 重复。

### Sprint Contract vs 接口协议（IDL）

**接口协议是 Sprint Contract 的不完整子集**：

| | **接口协议** | **Sprint Contract** |
|---|---|---|
| 覆盖 | 数据结构 | 可测试**行为** |
| 层次 | "返回 UserInfo" | "点击 X，300ms 内看到 toast" |

**跨团队协作翻车的隐性失误**：
1. **字段对齐，语义没对齐**（"status: 1" 两边理解不同）
2. **成功路径定义了，异常路径没定义**（timeout 怎么办？失败文案？）
3. **时序和性能假设隐式**（P99？QPS？）

**升级版 Sprint Contract 应包含**：

| 维度 | 描述 |
|------|------|
| 数据结构 | 字段、类型、返回码 |
| **场景化语义** | 同一接口在不同 use case 下的入参/输出含义 |
| **时序** | 调用时机、前置依赖、幂等性 |
| **性能假设** | P99 延迟、最大 QPS、超时策略 |
| **异常路径** | 失败时调用方怎么做、降级方案、兜底文案 |

---

## 单元 4：Harness 迭代哲学 + DAW 案例

### 核心原则

> **"Find the simplest solution possible, and only increase complexity when needed."**
>
> — Anthropic "Building Effective Agents"

**可执行推论**：
> 每个 Harness 组件都编码了一个关于"模型独自做不到什么"的假设。这些假设值得压力测试，因为：(1) 可能本身就不对；(2) 随着模型变强，会快速过时。

**Harness 组件 = 对模型弱点的补偿机制**。弱点消失，组件变死重。

### Load-Bearing 分析方法论

| 方法 | 结果 |
|------|------|
| 激进削减：一次去掉多个组件 | ❌ 性能崩了，不知锅在哪 |
| **一次移除一个组件（控制变量法）** | ✅ 能分辨每个组件是否 load-bearing |

### Opus 4.6 带来的具体简化

| 组件 | 之前为什么存在 | 4.6 之后 |
|------|-------------|---------|
| **Context Reset** | 对抗 context anxiety | ❌ 去掉，改连续 session + 自动 compaction |
| **Sprint 结构** | 把工作分 chunks 让模型连贯 | ❌ 去掉，模型原生能处理长任务 |
| **Planner** | 防止 Generator 欠规划 | ✅ **保留** |
| **每 sprint Evaluator** | 每个 sprint 打分挂掉 | 🔄 改为 run 末尾一次性 pass |

### 最重要的一句话

> **"The evaluator is not a fixed yes-or-no decision. It is worth the cost when the task sits beyond what the current model does reliably solo."**

**Harness 价值 = 任务难度 - 模型能力边界**。边界随版本移动，Harness 的经济性也随之变化。

### DAW 案例数据

V2 Harness 总计：**3h 50min / $124**（比 V1 的 6h/$200 更便宜更快）。

- **Builder 连续跑 2h 7min 不需要 sprint 分解**（证明 4.6 连贯性提升）
- **QA 第一轮仍抓到真问题**："core interactions, not edge cases"
- **Evaluator 的能力边界**：Claude 听不到 → 音乐品味上 QA 反馈循环失效

### 两类组件的微妙区分 ⭐

作者保留 Planner 的理由引出一个洞察：

- **补偿能力弱点的组件**（Context Reset、Sprint）→ 随模型升级**会消失**
- **对抗行为倾向的组件**（Planner）→ 即使模型能力强了也**不自发做**

**有些 Harness 组件不是为了补偿能力弱点，而是为了对抗行为倾向。前者会消失，后者长期存在。**

### Key Takeaways（作者原话）

1. 始终在**目标模型**上实验，读它的 traces，针对真实问题调教
2. 复杂任务上，分解任务 + 专门 agents 仍有提升空间
3. 新模型落地时**重新审视 Harness**：
   - 剥离不再 load-bearing 的部件
   - 添加新部件达成原来不可能的更高能力

### 作者核心信念

> **"The space of interesting harness combinations doesn't shrink as models improve. Instead, it moves, and the interesting work for AI engineers is to keep finding the next novel combination."**

---

## AI 工程师的长期价值

### 模型能力会吃掉的工作

1. 大部分**编排式 Harness**（prompt chaining、固定 pipeline、手写 retry）
2. 一部分**状态管理**（context 切换、handoff 协议）
3. 简单场景的**一次性代码生成**

### 模型能力吃不掉的工作

1. **Criteria 设计** ⭐ — 定义"好"长什么样（偏好 ≠ 能力）
2. **任务分解的判断** — 哪些任务值得 Harness、哪些不值得
3. **评估基础设施** — trace 观察、失败模式识别、调教反馈循环

### 核心认知

```
Criteria（what / why）  ←  人类定义，不可替代
     ↓
Model   （how）         ←  能力会越来越强
```

**AI 工程师的长期价值在于"偏好工程师"而不是"流程工程师"。**

**Harness 具体配方会过期，但"怎么思考配方"不会过期。**

---

## 概念关联图

```
Context 退化 + Self-Eval 失效
        ↓ 解法
GAN 启发的分离架构（Generator ⇆ Evaluator）
        ↓ 方法论提炼
主观变可评分（Criteria 设计 + 权重杠杆）
        ↓ 扩展到全栈
三 Agent 架构（加入 Planner 提高天花板）
        ↓ 关键机制
Sprint Contract（在 spec 和实现之间架桥）
        ↓ 随模型演进
Load-Bearing 分析（区分补偿能力 vs 对抗倾向的组件）
        ↓ 长期价值
Criteria 不会被吃掉（AI 工程师 = 偏好工程师）
```

---

## 核心收获 Top 3

1. **Harness 组件 = 补偿机制 + 死期**
   每段"绕过模型弱点"的代码应该注释 `[LOAD-BEARING | 补偿的弱点 | 移除条件]`。workaround 必须带拆除计划，否则变技术债。

2. **主观判断可评分化 + 权重杠杆**
   把"好不好"拆成可编码的 criteria，同时传给 generator 和 evaluator。权重**不均匀**——重点压在"天然薄弱的维度"上推动对抗默认值。适用于代码评审、Design Doc、PRD 等所有主观任务。

3. **Criteria 是偏好，不是能力**
   长期来看，AI 工程师的价值在"定义什么是好"而不是"实现 Harness 流程"。模型能力演进会吃掉执行层，偏好层必须由人定义。

---

## 待探索问题

1. **Sprint Contract 在非代码场景如何落地？**
   写作、设计、产品决策等场景，contract 应该怎么表达？"可测试行为"在这些场景对应什么？

2. **Evaluator 的能力边界如何识别？**
   作者提到"Claude 听不到"导致音乐品味失效。我们怎么在设计 Evaluator 时提前识别它的盲区？

3. **"对抗行为倾向"型 Harness 组件的识别方法**
   怎么提前判断一个组件是补偿能力还是对抗倾向？前者会随升级消失，后者长期存在——这个区分对投资决策很关键。

4. **Criteria 的版本化和持续演进**
   criteria 会"驯化结果"，需要像代码一样持续演进。组织应该如何管理 criteria 的变更历史和 A/B 测试？

---

## 延伸阅读

- [Anthropic Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) — 本文引用的核心原则来源
- [Agentic Engineering Patterns 学习笔记](../AI-Agent/agentic-engineering-patterns.md) — 模式层面的总览，与本文形成互补

---

## 苏格拉底式学习元反思

本次学习过程本身也产出了一条方法论收获，已写入 socratic-reader skill：

> **不迎合用户**：先给出自己独立的思考，再评价用户的回答。迎合会掏空讨论价值——用户想要的是碰撞而非点头。当用户的回答方向对但不完整时，明确指出"对但不够"的部分；当用户的类比有机制上的偏差时，指出差异而非强化相似。

这条原则本身就是本文"Evaluator 要严格、不能说服自己批准"的**元层应用**——在教学场景里，AI 导师如果自评估过宽，学生的思考深度就会停在表层。
