# AI Coding 反思复盘

> 后端工程师 × Claude Code × 4个项目 × 1700+会话 × 726+ commits
这个系列记录的是认知变化的完整过程 —— 包括后来发现走错了的部分。

---

## 认知迭代记录

> 被划掉的认知是后来被推翻的，括号里标注了在哪篇blog被推翻/修正

| # | 认知 | 来源 |
|---|------|------|
| 1 | ~~工具和插件装够了就能vibe coding~~ → 工具不是瓶颈 | Blog 1 |
| 2 | 动手之前先想清楚要做什么 | Blog 1 |
| 3 | 多Agent之间的上下文要隔离 | Blog 2 |
| 4 | Pipeline的可观测性很重要 | Blog 2 |
| 5 | ~~QA环节放在最后兜底就行~~ → 质量约束必须嵌入每个阶段 | Blog 2→4 |
| 6 | 上下文管理是直接影响产出质量的第一因素 | Blog 3 |
| 7 | ~~编译能过不代表应用能用~~ → Playwright自测通过也不代表功能可用 | Blog 3→8-2 |
| 8 | 认知没有变成硬约束，执行时一定会丢 | Blog 3 |
| 9 | 补丁式修复解决不了架构问题，要敢停下来重构 | Blog 4 |
| 10 | AI不会主动质疑你的方向，批判性思考得自己来 | Blog 4 |
| 11 | 让AI研究一个项目要验证它真的读了源码 | Blog 4 |
| 12 | ~~"胶水编程"——搭建优先，零自研~~ → 嘴上说搭建优先，实际还是会从头造 | Blog 5→7 |
| 13 | session上下文管理是AI Coding的第一质量杠杆 | Blog 5 |
| 14 | 工具的价值不在数量 | Blog 5 |
| 15 | 需求必须包含约束条件，不只是期望 | Blog 6 |
| 16 | **"需求→功能"的转换才是最大杠杆点** | Blog 6 |
| 17 | 第三方工具的配置schema会变，文档会过时 | Blog 6 |
| 18 | 参考实现不能只借概念，要深入研究架构 | Blog 7 |
| 19 | 做交互设计前要验证平台API的实际能力 | Blog 7 |
| 20 | 自治和控制是一个频谱，不是开关 | Blog 8-1 |
| 21 | 约束嵌入不等于约束执行——prompt里必须强制 | Blog 8-1 |
| 22 | 从内往外建的东西对开发者OK，对用户不OK | Blog 8-1 |
| 23 | 结构测试通过不等于功能可用 | Blog 8-1 |
| 24 | WebUI在AI Agent框架里只能做监控，不能做控制 | Blog 8-2 |
| 25 | 真值面不应该让agent直接写——父模型统一写入，agent只提proposal | Blog 8-2 |
| 26 | 框架应该是工具，不是项目——用户目录里只留运行时数据 | Blog 8-2 |
| 27 | Roadmap本身会编码优先级错误——核心功能如果不在Phase 1出现，说明规划时就没把核心放在核心位置 | Blog 8-3 |
| 28 | Phase膨胀是scope creep的结构化表现——AI会把每个想法做得很完整，让膨胀看起来像进展 | Blog 8-3 |
| 29 | 理念偏离在早期最容易修复，一旦"先做完再fix"，后面每一步都在加固错误方向 | Blog 8-3 |
| 30 | **用独立会话做review——让另一个agent审查主会话的产出，是对抗自动化偏差的低成本实践** | Blog 9 |
| 31 | 核心先行不是口号，是可验证的：如果两天就能搭完核心，说明之前在非核心的事情上花的时间都是方向债 | Blog 9 |

如果只留一条：**#16 "需求→功能"的转换才是最大杠杆点。**

---

## 项目进展

### mosaicat（2/13 — 3/30）

一句话需求 → 13个Agent → 交付可运行Web应用

| 阶段 | 关键事件 |
|------|----------|
| Blog 1（3/13-15） | 工具调研、Day 1推翻重来、从零搭pipeline |
| Blog 2（3/15-19） | 13 agent pipeline成型、Silent failure、可观测性缺失 |
| Blog 3（3/20-23） | 代码能编译但不能用、QA全线假通过、自动化偏见 |
| Blog 4（3/24-28） | 停下来重构、TDD + Constitution + 自验证机制 |
| Blog 5（3/29-30） | 两个方法论、转向ai-workstation |

### ai-workstation（3/30 — 4/4）

飞书集成的AI自治工作站

| 阶段 | 关键事件 |
|------|----------|
| Blog 6（3/30-4/3） | 模型切换、飞书API盲区、约束vs期望 |
| Blog 7（4/3-4） | ECL pipeline方向错误、NIH综合症、工具收敛 |

### Detent（4/5 — 4/10）

控制论Agent编排框架 → 弃置

| 阶段 | 关键事件 |
|------|----------|
| Blog 8-1（4/5-6） | Phase 1-5、CLI + 对抗规划 + Coding Loop |
| Blog 8-2（4/6-9） | WebUI翻车、parent-as-sole-writer、NPX打包 |
| Blog 8-3（4/9-10） | Review代码、发现优先级偏离、弃置决定 |

### bonfire（4/10 — ）

用Claude Code原生原语重新实现ECL pipeline——这次先做核心

| 阶段 | 关键事件 |
|------|----------|
| Blog 9（4/10-12） | 81次提交、9个Plan、114测试通过、双会话review实践 |

---

## 文档目录

| 文件 | 内容 |
|------|------|
| [00-序.md](00-序.md) | 系列背景、作者介绍、阅读指南 |
| [01-mosaicat-blog1.md](01-mosaicat-blog1.md) | 工具陷阱、Day 1推翻重来 |
| [01-mosaicat-blog2.md](01-mosaicat-blog2.md) | 13 Agent pipeline、Silent failure |
| [01-mosaicat-blog3.md](01-mosaicat-blog3.md) | 代码不能用、QA假通过、automation bias |
| [01-mosaicat-blog4.md](01-mosaicat-blog4.md) | 停下来重构、Constitution设计 |
| [01-mosaicat-blog5.md](01-mosaicat-blog5.md) | 方法论提炼、方向转换 |
| [02-ai-workstation-blog6.md](02-ai-workstation-blog6.md) | 模型切换、飞书盲区、JTBD |
| [02-ai-workstation-blog7.md](02-ai-workstation-blog7.md) | ECL方向错误、NIH综合症、VSM |
| [03-detent-blog8-1.md](03-detent-blog8-1.md) | Detent Phase 1-5、对抗性规划 |
| [03-detent-blog8-2.md](03-detent-blog8-2.md) | WebUI翻车、控制论重构、NPX打包 |
| [03-detent-blog8-3.md](03-detent-blog8-3.md) | Detent弃置、Roadmap优先级偏离 |
| [04-bonfire-blog9.md](04-bonfire-blog9.md) | bonfire搭建、双会话review、核心先行验证 |

---

## 引用索引

**开源项目/工具：**
[Claude Code](https://github.com/anthropics/claude-code) · [GSD](https://github.com/get-shit-done-ai/gsd) · [LangGraph](https://github.com/langchain-ai/langgraph) · [CrewAI](https://github.com/crewAIInc/crewAI) · [AutoGen](https://github.com/microsoft/autogen) · [OpenHands](https://github.com/All-Hands-AI/OpenHands) · [Context7 MCP](https://github.com/upstash/context7) · [Spec Kit](https://github.com/spec-kit/spec-kit) · [n8n](https://github.com/n8n-io/n8n)

**论文/著作：**
Liu et al. 2023 (*Lost in the Middle*) · Barr et al. 2014 (*Test Oracle Problem*) · Beer 1972 (*Brain of the Firm*) · Brooks 1975 (*Mythical Man-Month*) · Forsgren et al. 2018 (*Accelerate*) · Boehm 1981 (*Software Engineering Economics*) · Senge 1990 (*The Fifth Discipline*) · Parasuraman & Manzey 2010 (*Automation Bias*)
