# OpenAI / Anthropic Agent 方法论主要文章阅读导览

日期：2026-06-23

## 一、阅读结论先行

这组材料不要按公司读，而要按问题读。

如果目标是理解“Agent 到底怎么做”，先读 Anthropic；如果目标是理解“如何用 API / SDK / 工具把 Agent 做出来”，再读 OpenAI。

最重要的顺序是：

```text
1. Anthropic - Building effective agents
2. Anthropic - Equipping agents for the real world with Agent Skills
3. Claude Code - Extend Claude with skills
4. Agent Skills open standard
5. OpenAI - Responses API
6. OpenAI - Agents SDK
7. OpenAI - Skills
8. OpenAI - Using tools
```

一句话概括：

- Anthropic 的文章解决“workflow / agent / skill 这些概念怎么分”。
- OpenAI 的文档解决“在工程上如何调用工具、保持状态、编排 agent、挂载 skills”。
- 对法律工作来说，最关键不是追求完全自主 agent，而是先把 workflow 和 skill 做好。

---

## 二、第一篇：Anthropic - Building effective agents

来源：
https://www.anthropic.com/engineering/building-effective-agents

### 这篇文章讲什么

这是目前最值得先读的一篇，因为它把 agentic system 拆成了两类：

- Workflow：LLM 和工具沿着预定义代码路径运行。
- Agent：LLM 动态决定流程和工具使用方式，自主控制完成任务的路径。

它还提出一个很重要的原则：先从最简单方案开始，只有在简单方案不够时才增加复杂度。很多任务其实不需要 agent，只需要单次 LLM 调用、检索增强或少量固定步骤。

### 重点概念

1. Augmented LLM

基础能力不是 agent，而是“增强后的 LLM”：LLM + retrieval + tools + memory。也就是说，工具、检索、记忆是 agentic system 的底层积木。

2. Prompt chaining

把任务拆成顺序步骤，每一步只处理一个明确问题，中间可以加 gate / check。

法律映射：

```text
抽取合同条款 → 识别风险 → 生成修改建议 → 形成报告 → 校验输出
```

3. Routing

先分类，再进入不同路径。

法律映射：

```text
合同审查 / 法规检索 / 案例分析 / 合规资讯 / 法律问答
```

4. Parallelization

并行处理不同维度，再聚合。

法律映射：

```text
文本校对、交易结构、付款条款、违约责任、解除条款、合规条款并行审查
```

5. Orchestrator-workers

一个主模型拆任务，多个 worker 处理，再综合。

法律映射：

```text
大型研究报告、复杂合同包、跨法域合规问题
```

6. Evaluator-optimizer

一个模型生成，一个模型评价，再迭代优化。

法律映射：

```text
合同审查报告质检、条款修改稿二次校验、法律研究报告润色
```

7. Agent

当任务路径不能预先固定、需要多轮工具反馈、需要根据环境结果调整路径时，才进入 agent。

法律映射：

```text
开放式法律研究、复杂证据链构建、跨资料源事实核验、代码化工具修复
```

### 对你的项目有什么用

你的合同审查 skill executor，本质上更接近 workflow，不是完全 agent。它应该优先做：

```text
固定步骤 + 每步结构化输出 + 中间校验 + 失败重试 + 人工复核
```

只有在“系统不知道下一步该查什么、需要自主探索文件/网页/代码”时，才应该让 agent 接管。

---

## 三、第二篇：Anthropic - Equipping agents for the real world with Agent Skills

来源：
https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

### 这篇文章讲什么

这篇文章解释 Skills 为什么重要。它把 skill 类比成给新员工的 onboarding guide：不是重新训练模型，而是把组织流程、专业知识、脚本和资源打包，让 agent 在需要时加载。

### 重点概念

1. Skill 不是 prompt

Prompt 通常是一次性的。Skill 是一个可复用能力包，可以包含说明、脚本、模板、参考资料和资源。

2. Progressive disclosure

这是 Skills 最核心的设计原则。

```text
第一层：只暴露 name / description，让模型知道什么时候可能用。
第二层：触发后读取完整 SKILL.md。
第三层：需要时再读取 references / scripts / templates 等附属文件。
```

这样做的好处是：不把所有材料一股脑塞进上下文，而是在需要时逐步加载。

3. Skills 可以和 MCP 互补

MCP 解决“连接外部系统和工具”的问题；Skill 解决“怎么完成一个复杂流程”的问题。

法律行业里可以这样理解：

```text
MCP = 连接合同库 / 案例库 / 邮件 / GitHub / 内部系统
Skill = 告诉 agent 如何审合同、如何写案例分析、如何做合规资讯 JSON
```

### 对你的项目有什么用

Nomos / 合同审查 skill 很适合按 progressive disclosure 重构：

```text
contract-review-skill/
  SKILL.md                  # 总流程，短而稳定
  playbooks/
    lease.md                # 租赁合同审查规则
    cooperation.md          # 高校合作协议审查规则
    power-grid.md           # 并网调度协议审查规则
  templates/
    review_report.md        # 输出模板
  scripts/
    validate_output.py      # 校验 JSON / Markdown
  examples/
    sample_review.md        # 示例输出
```

不要把所有合同类型、所有条款规则都写进一个超长 prompt。这样既贵，也不稳定。

---

## 四、第三篇：Claude Code - Extend Claude with skills

来源：
https://code.claude.com/docs/en/skills

### 这篇文章讲什么

这是最接近“怎么写一个真实 skill”的文档。它说明 Claude Code 的 skill 就是一个目录，里面必须有 `SKILL.md`，还可以带模板、examples、scripts、references。

### 重点概念

1. Skill 什么时候该创建

当你反复粘贴同一组指令、checklist、多步骤流程时，就应该把它做成 skill。

这句话对法律工作特别关键。合同审查、案例分析、法规检索、资讯 JSON 化，都是反复粘贴流程的典型任务。

2. Skill body 只有在使用时才加载

这意味着长参考材料不会一开始就消耗上下文，只有触发 skill 后才进入上下文。

3. 动态上下文注入

Claude Code 支持在 skill 中嵌入命令，例如读取 git diff，然后把当前真实 diff 注入提示词。

这对代码类 agent 很关键，也能映射到你的 executor：

```text
读取合同 diff / semantic_analysis_input.md / semantic_issues.json
→ 注入到当前审查任务
→ 让模型只基于真实输入输出结果
```

4. allowed-tools / disallowed-tools

Skill 可以限制或授权工具。对企业场景非常重要，因为 skill 不只是文本，还可能调用命令、读文件、改文件。

### 对你的项目有什么用

你的 skill 不应只是一个 prompt，而应该形成“任务包”：

```text
说明文件：这个 skill 什么时候用
输入规范：需要哪些文件
执行步骤：先做什么、再做什么
输出契约：必须输出哪些字段
校验脚本：结果不合格就失败
权限边界：能读什么、能写什么、不能碰什么
示例输出：帮助模型稳定格式
```

---

## 五、第四篇：Agent Skills open standard

来源：
https://agentskills.io/

### 这篇文章讲什么

这是 Agent Skills 的开放标准首页。它把 skill 定义为一个包含 `SKILL.md` 的文件夹，可以附带 scripts、references、assets 等。

它明确了三阶段机制：

```text
Discovery：启动时只加载 name 和 description。
Activation：任务匹配时读取完整 SKILL.md。
Execution：按说明执行，必要时运行脚本或读取资源。
```

### 对你的项目有什么用

这说明 Skill 正在变成一个跨产品的轻量标准。未来你的合同审查 skill 如果按这个格式写，就不只适配 Claude，也可能更容易迁移到 Codex、OpenAI Agent Skills、其他 agent 客户端。

---

## 六、第五篇：OpenAI - Migrate to the Responses API

来源：
https://developers.openai.com/api/docs/guides/migrate-to-responses

### 这篇文章讲什么

这不是单纯迁移文档，而是 OpenAI 现在 agent-like 应用的入口文档。

Responses API 被定义为新的 API primitive，是 Chat Completions 的演进版本；OpenAI 建议新项目使用 Responses API。它把内置工具、多轮交互、图文多模态、状态、reasoning、function calling 等统一进一个 response 对象。

### 重点概念

1. Responses API 是 agentic loop

OpenAI 文档说 Responses API 是 agentic by default：模型可以在一个 API 请求中多次调用 web_search、file_search、code_interpreter、remote MCP、自定义函数等。

2. Item 取代 message 的中心地位

Chat Completions 主要是 messages；Responses API 里则把 reasoning、message、function_call、function_call_output 等都作为 output items。

这更贴近 agent 的真实运行：一次响应不只是“说一段话”，还包括思考片段、工具调用、工具结果、最终输出。

3. Statefulness

Responses API 支持通过 previous_response_id 或 conversation state 延续上下文。

### 对你的项目有什么用

如果你要做一个“上传合同 → 自动审查 → 产物下载”的在线 agent，OpenAI 路线大概是：

```text
Responses API
+ file_search / retrieval
+ function calling
+ code_interpreter / shell
+ structured outputs
+ stateful run
```

当流程还不复杂时，不一定需要 Agents SDK。

---

## 七、第六篇：OpenAI - Agents SDK

来源：
https://developers.openai.com/api/docs/guides/agents

### 这篇文章讲什么

OpenAI 把 Agents SDK 定位为 code-first agent app 的框架。它适合应用自己掌控 orchestration、tool execution、approvals、state。

### 重点概念

1. Agents 是什么

OpenAI 定义下的 agent 应用会计划、调用工具、跨 specialist 协作，并保持足够状态来完成多步骤工作。

2. 什么时候用 Responses API

当“一次模型调用 + 工具 + 应用自己的逻辑”就够时，用 Responses API。

3. 什么时候用 Agents SDK

当你的服务端要控制：

```text
orchestration
工具执行
审批
状态
多 agent / handoff
观测与 eval
```

才进入 Agents SDK。

### 对你的项目有什么用

Nomos 的复杂版本会越来越接近 Agents SDK：

```text
用户上传合同
→ 文档解析 agent
→ 条款抽取 agent
→ 风险识别 agent
→ 修改建议 agent
→ 质检 agent
→ 导出 agent
→ 人工复核
```

但这不意味着一开始就要上多 agent。前期可以先按 workflow 实现，后期再把每个稳定步骤升级为 specialist agent。

---

## 八、第七篇：OpenAI - Skills

来源：
https://developers.openai.com/api/docs/guides/tools-skills

### 这篇文章讲什么

OpenAI 把 Skill 定义为一个带有 `SKILL.md` manifest 的 versioned bundle。Skill 用于 codify processes and conventions，比如公司风格指南、多步骤工作流等。

### 重点概念

1. Skill 是 versioned bundle

这意味着它不是随手写的一段提示词，而是可以版本管理、上传、挂载、复用的任务包。

2. Hosted shell / local shell

OpenAI Skills 可以挂载到 hosted shell 或 local shell 环境，和执行环境结合。

3. 安全风险

OpenAI 特别提示：skills 是 privileged code and instructions，应当审查。也就是说，skill 有可能不只是“提示词”，还可能带有可执行脚本和权限影响。

### 对你的项目有什么用

合同审查 skill 以后要加版本号，例如：

```text
contract-review-skill@0.1.0
contract-review-skill@0.2.0
contract-review-skill@1.0.0
```

每次改 playbook、改输出 schema、改校验脚本，都应该能回溯版本。否则很难评测“到底是哪次规则改动导致审查结果变差”。

---

## 九、第八篇：OpenAI - Using tools

来源：
https://developers.openai.com/api/docs/guides/tools

### 这篇文章讲什么

这篇是 OpenAI 的工具总览。它说明模型可以通过 built-in tools、function calling、tool search、remote MCP servers 来扩展能力。

### 重点概念

1. Built-in tools

例如：web search、file search、computer use、code interpreter、image generation、shell 等。

2. Function calling

让模型调用你自己定义的函数。法律场景里，这可以是：

```text
查合同库
查条款模板
读取审查规则
写入审查结果
校验 JSON schema
导出 Word / PDF
```

3. Remote MCP

让模型连接外部系统。它更像“工具接入层”。

4. Tool search

模型不必一开始加载全部工具定义，而是运行时再动态加载相关工具定义。这个思路和 skill 的 progressive disclosure 很像。

### 对你的项目有什么用

对于法律 agent，工具设计和 prompt 一样重要，甚至更重要。一个糟糕的工具接口会让模型反复犯错。

例如，不要让模型随便写文件路径，而是提供：

```text
read_contract(run_id)
write_issue(run_id, issue_json)
validate_semantic_issues(run_id)
export_review_report(run_id)
```

这比让模型自由操作文件系统稳定得多。

---

## 十、这几篇文章的关系图

```text
Anthropic - Building effective agents
    ↓ 解决：workflow 和 agent 怎么分

Anthropic - Agent Skills article
    ↓ 解决：skill 为什么能封装组织流程

Claude Code Skills docs / Agent Skills standard
    ↓ 解决：skill 具体怎么写成文件夹、SKILL.md、scripts、references

OpenAI Responses API
    ↓ 解决：一次 agent-like 调用如何统一工具、状态、多模态和输出

OpenAI Agents SDK
    ↓ 解决：复杂 agent app 如何编排、handoff、审批、观测、评测

OpenAI Skills / Tools docs
    ↓ 解决：怎么把可复用流程和外部动作接入 agent
```

---

## 十一、推荐精读顺序

### 只读 2 篇

1. Anthropic - Building effective agents
2. Anthropic - Equipping agents for the real world with Agent Skills

这两篇足够建立 agent / workflow / skill 的基本心智模型。

### 读 4 篇

加上：

3. Claude Code - Extend Claude with skills
4. OpenAI - Responses API

这样就能从方法论走到工程实现。

### 读 8 篇

完整读完本文列出的 8 篇/组，就能形成比较完整的公开 agent 方法论框架。

---

## 十二、下一步建议

下一步应该围绕 Skills 单独写一篇：

《Agent Skills 专题：SKILL.md、progressive disclosure 与法律工作流封装》

重点回答：

1. 一个 skill 的最小结构是什么？
2. description 应该怎么写，才能让模型正确触发？
3. SKILL.md 里该放什么，不该放什么？
4. scripts / references / examples / templates 怎么分工？
5. 合同审查 skill 应该如何从一个超长 prompt 拆成可维护目录？
6. skill 的安全边界和版本管理怎么设计？
