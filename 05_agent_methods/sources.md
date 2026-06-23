# Sources: OpenAI / Anthropic Agent Methods

日期：2026-06-23

## OpenAI 官方来源

### 1. OpenAI Agents SDK

URL: https://developers.openai.com/api/docs/guides/agents

关注点：

- Agents 是可以规划、调用工具、跨 specialist 协作、维持状态以完成多步骤工作的应用。
- 当一次模型调用加工具和应用自有逻辑足够时，使用 Responses API。
- 当应用需要自己掌控 orchestration、tool execution、approvals、state 时，使用 Agents SDK。
- 重点页面：Agent definitions、Running agents、Sandbox agents、Orchestration and handoffs、Guardrails and human review、Results and state、Integrations and observability、Evaluate agent workflows。

### 2. OpenAI Responses API migration / overview

URL: https://developers.openai.com/api/docs/guides/migrate-to-responses

关注点：

- Responses API 是 OpenAI 推荐给新项目的统一接口。
- 其定位是 unified interface for building powerful, agent-like applications。
- 支持 web search、file search、computer use、code interpreter、remote MCPs 等内置工具。
- 支持 multi-turn、stateful context、structured outputs、function calling 等。

### 3. OpenAI Using tools

URL: https://developers.openai.com/api/docs/guides/tools

关注点：

- 工具扩展模型能力。
- 支持 built-in tools、function calling、tool search、remote MCP servers。
- 典型工具包括 web search、file search、function calling、remote MCP。

### 4. OpenAI Skills

URL: https://developers.openai.com/api/docs/guides/tools-skills

关注点：

- Agent Skills 是 versioned bundle of files + SKILL.md manifest。
- Skills 可以 codify processes and conventions，例如 company style guides、multi-step workflows。
- 支持 hosted shell 和 local shell 两种形态。
- 模型根据 skill name、description、path 判断是否使用 skill；触发后读取 SKILL.md。
- Skills 有安全风险，应作为 privileged code and instructions 审查。

## Anthropic 官方来源

### 1. Building effective agents

URL: https://www.anthropic.com/engineering/building-effective-agents

关注点：

- 成功的 agentic system 往往不是复杂框架，而是 simple, composable patterns。
- 明确区分 workflow 与 agent：workflow 是预定义代码路径；agent 是 LLM 动态控制流程和工具使用。
- 常见 workflow：prompt chaining、routing、parallelization、orchestrator-workers、evaluator-optimizer。
- Agent 适合开放式、难以预判步骤数量的问题，但成本更高，错误会累积，需要 sandbox、guardrails 和 human feedback。

### 2. Anthropic Tool use with Claude

URL: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview

关注点：

- Claude 可以调用用户定义工具或 Anthropic 提供的工具。
- Claude 根据用户请求和工具描述决定是否调用工具。
- Client tools 由应用执行，Claude 返回 tool_use block，应用执行后回传 tool_result。
- Server tools 由 Anthropic 基础设施执行，例如 web_search、code_execution、web_fetch、tool_search。
- tool_choice 可用于更强制地控制工具调用。

### 3. Introducing Agent Skills

URL: https://claude.com/blog/skills

关注点：

- Skills 是包含 instructions、scripts、resources 的文件夹，Claude 可在需要时加载。
- Skills 可组合、可迁移、按需加载，并可包含可执行代码。
- 支持 Claude apps、Claude Code、API、Claude Agent SDK。
- 需要关注安全，因为 Skills 可能执行代码。

### 4. Equipping agents for the real world with Agent Skills

URL: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

关注点：

- Skills 把程序性知识和组织上下文封装为可组合、可移植能力。
- Progressive disclosure：先加载 skill 的 name / description，触发后读取 SKILL.md，必要时再读取附属文件。
- Skills 可包含脚本，适合把确定性操作交给代码完成。
- 开发 skill 应从 eval 出发，观察 agent 在代表性任务中的失败点，再增量构建。

### 5. Claude Code Skills

URL: https://code.claude.com/docs/en/skills

关注点：

- Claude Code 通过 SKILL.md 扩展能力。
- Skills 适合把重复粘贴的指令、checklist、多步骤流程封装起来。
- 技能正文只有在使用时才加载，长参考材料在触发前几乎不消耗上下文。
- 支持 project / personal / enterprise / plugin 等不同层级。
- 支持 dynamic context injection、allowed-tools、disable-model-invocation、subagent 等扩展。

## 开放标准

### Agent Skills

URL: https://agentskills.io/

关注点：

- Agent Skills 是一种轻量开放格式，用于用专业知识和 workflow 扩展 AI agent。
- 核心是一个包含 SKILL.md 的文件夹，可包含 scripts、references、assets 等。
- 工作机制：Discovery → Activation → Execution。
- 目标是跨 agent 产品复用。