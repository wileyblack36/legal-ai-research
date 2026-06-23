# OpenAI / Anthropic 公开 Agent 方法图谱

日期：2026-06-23

## 一、先给一个总判断

如果把公开材料抽象成方法论，OpenAI 和 Anthropic 的侧重点不同：

- Anthropic 更像在讲“怎么设计 agentic system”：先区分 workflow 与 agent，再从简单模式逐步增加复杂度。
- OpenAI 更像在提供“怎么把 agent 做成产品/工程系统”：Responses API、Agents SDK、Tools、MCP、Skills、Shell、Computer Use、Guardrails、Tracing/Evals 等。

对法律行业应用来说，Anthropic 的框架更适合理解架构，OpenAI 的框架更适合理解工程落地。

## 二、Anthropic：Workflow 与 Agent 的区分

Anthropic 在《Building effective agents》中给出一个非常重要的区分：

- Workflow：LLM 与工具沿着预定义代码路径被编排。
- Agent：LLM 动态决定自己的流程和工具使用方式，并保持对任务完成路径的控制。

这个区分对法律场景非常有用。很多法律 AI 应用不应该一开始就做完全自主 Agent，而应先做 Workflow。

### Anthropic 的常见 workflow 模式

1. Prompt chaining

把任务拆成顺序步骤。前一步输出成为下一步输入，中间可以加入 gate / check。

适合法律场景：合同审查中的“抽取条款 → 风险识别 → 修改建议 → 输出报告”。

2. Routing

先分类，再分发到不同处理流程。

适合法律场景：把问题分成“合同审查 / 法规检索 / 案例分析 / 数据合规 / 劳动争议”等，再进入不同模板或工具。

3. Parallelization

多个 LLM 调用并行处理不同维度，最后合并。

适合法律场景：合同审查中同时检查“文本错误、交易结构、付款条款、违约责任、解除条款、合规义务”。

4. Orchestrator-workers

一个主 LLM 拆解任务并分配给多个 worker，再综合结果。

适合法律场景：大型法律研究报告、跨法域合规问题、复杂合同包审查。

5. Evaluator-optimizer

一个模型生成，另一个模型评价并提出修改，再迭代。

适合法律场景：报告润色、条款修改、合同审查报告的二次质检。

### Anthropic 对 Agent 的定位

Anthropic 认为 Agent 适合开放式、难以预判步骤数量的问题；Agent 会根据工具返回、代码执行、环境反馈等 ground truth 迭代，并在检查点请求人类反馈。缺点是成本更高、延迟更长、错误会累积，因此需要沙箱、停止条件、guardrails 和人工复核。

## 三、OpenAI：Responses API + Agents SDK + Tools

OpenAI 的路线可以理解为三层：

### 1. Responses API：一个 agent-like 的统一调用入口

Responses API 是 OpenAI 推荐的新项目接口。它把文本、图像、结构化输出、工具调用、文件检索、web search、computer use、code interpreter、MCP 等放进统一的响应对象中。

粗略理解：

- 简单 agent-like 应用，用 Responses API 就够。
- 如果只是“一次模型调用 + 工具 + 你自己的应用逻辑”，优先 Responses API。

### 2. Agents SDK：当应用自己负责复杂编排

OpenAI 文档说，如果你的应用需要自己掌控 orchestration、tool execution、approvals、state，就进入 Agents SDK 路线。

它适合：

- 多 agent / specialist handoff；
- 状态管理；
- 审批和 human review；
- sandbox agent；
- 观测、trace、eval；
- 与现有业务系统深度集成。

### 3. Tools：把模型从“说话”扩展到“行动”

OpenAI 的公开工具层包括：

- Web search
- File search / retrieval
- Tool search
- Function calling
- Remote MCP
- Shell
- Computer use
- Code interpreter
- Apply patch
- Image generation
- Skills

从法律行业角度，最值得关注的是 File search、Web search、Function calling、MCP、Code interpreter、Shell、Skills。

## 四、Skills：最值得重点学习的部分

Skills 可以理解成：把一类稳定流程、组织知识、脚本、模板、示例输出打包成一个可复用能力包。

它不是普通 prompt，也不是单个 tool。

- Prompt：一次性指导。
- Tool：一个可调用动作或函数。
- Skill：围绕某类任务组织起来的一组说明、资源、脚本和流程。
- MCP：连接外部系统和工具的协议。
- Knowledge base / RAG：检索知识来源，不一定规定怎么执行任务。

### Anthropic Skills 的核心思想

Anthropic 把 Skill 描述为包含 instructions、scripts、resources 的文件夹。Claude 会在需要时加载相关 Skill。它强调 progressive disclosure：先只暴露 name / description，触发后再加载 SKILL.md，必要时再读取附属文件。

这对法律行业很有启发：

- 一个“合同审查 Skill”不应把所有规则一次塞进上下文。
- 应该先用 description 触发；
- 再读核心步骤；
- 遇到特定合同类型或条款时，再读取对应 playbook / checklist / scripts。

### OpenAI Skills 的核心思想

OpenAI 也支持 Agent Skills。公开文档中，Skill 是带有 SKILL.md manifest 的 versioned bundle，可以在 hosted 或 local shell 环境中使用。OpenAI 强调版本管理、托管/本地执行、与 shell/container 环境结合，以及安全风险。

## 五、对法律行业的映射

### 更适合 Workflow 的任务

1. 合同审查：
   - 文本抽取
   - 条款分类
   - 风险点识别
   - 修改建议
   - 结构化报告
   - 人工复核

2. 合规资讯：
   - 检索
   - 来源核验
   - 分类打标
   - JSON 结构化
   - 人工确认

3. 案例分析：
   - 案件事实
   - 程序阶段
   - 请求权基础
   - 原告主张
   - 被告抗辩
   - 和解/裁判结果

这些任务路径大体稳定，适合 prompt chaining、routing、parallelization、evaluator-optimizer。

### 更适合 Agent 的任务

1. 开放式法律研究：不知道要查多少材料、多少轮核验。
2. 复杂项目资料整理：需要在多个文件夹、仓库、网页、数据库之间来回查找。
3. 代码化法律工具维护：需要读代码、改代码、运行测试、修复失败。
4. 多来源证据链构建：需要自主决定下一步查哪里、是否继续查。

这些任务路径不固定，才更适合 Agent。

## 六、我建议后续拆成 6 篇小研究

1. OpenAI Agent 方法：Responses API / Agents SDK / Tools / Skills
2. Anthropic Agent 方法：workflow vs agent / prompt chaining / routing / evaluator-optimizer
3. Skills 专题：SKILL.md、progressive disclosure、组织知识封装、安全风险
4. Claude Code 与 Codex：coding agent 为什么更容易落地
5. 法律行业 Agent 设计：合同审查、法律检索、案例分析、合规资讯
6. Agent 评测与风控：可控性、稳定性、权限、人工复核、审计轨迹

## 七、初步结论

对你的 Legal AI 研究项目来说，最应该先吃透的不是“完全自主 agent”，而是下面这条路径：

```text
单次模型调用
→ 带工具的模型调用
→ 固定 workflow
→ workflow + skill
→ workflow + evaluator
→ 半自主 agent
→ 多 agent / agent team
```

这条路线和法律工作的专业性更匹配：先保证可解释、可复核、可稳定输出，再逐步提升自动化程度。