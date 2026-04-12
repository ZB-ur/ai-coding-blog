# Blog 9: bonfire——81次提交，两天，核心先行

> 4/10 - 4/12 | bonfire从零搭建 | 9个Plan、81次提交、114个测试

---

## 项目进度

```
bonfire — 基于控制论的AI Coding闭环系统（Claude Code原生plugin）

[■■■■■■■■□□] 核心框架完成，README重构中
```

## 核心认知（动态更新）

> 从 Blog 1 积累至今

1. ~~工具和插件装够了就能vibe coding~~ → 工具不是瓶颈（Blog 1）
2. 动手之前先想清楚要做什么（Blog 1）
3. 多Agent之间的上下文要隔离（Blog 2）
4. Pipeline的可观测性很重要（Blog 2）
5. ~~QA环节放在最后兜底就行~~ → 质量约束必须嵌入每个阶段（Blog 2→4）
6. 上下文管理是直接影响产出质量的第一因素（Blog 3）
7. ~~编译能过不代表应用能用~~ → Playwright自测通过也不代表功能可用（Blog 3→8-2）
8. 认知没有变成硬约束，执行时一定会丢（Blog 3）
9. 补丁式修复解决不了架构问题，要敢停下来重构（Blog 4）
10. AI不会主动质疑你的方向，批判性思考得自己来（Blog 4）
11. 让AI研究一个项目要验证它真的读了源码（Blog 4）
12. ~~"胶水编程"——搭建优先，零自研~~ → 嘴上说搭建优先，实际还是会从头造（Blog 5→7）
13. session上下文管理是AI Coding的第一质量杠杆（Blog 5）
14. 工具的价值不在数量（Blog 5）
15. 需求必须包含约束条件，不只是期望（Blog 6）
16. **"需求→功能"的转换才是最大杠杆点**（Blog 6）
17. 第三方工具的配置schema会变，文档会过时（Blog 6）
18. 参考实现不能只借概念，要深入研究架构（Blog 7）
19. 做交互设计前要验证平台API的实际能力（Blog 7）
20. 自治和控制是一个频谱，不是开关（Blog 8-1）
21. 约束嵌入不等于约束执行——prompt里必须强制（Blog 8-1）
22. 从内往外建的东西对开发者OK，对用户不OK（Blog 8-1）
23. 结构测试通过不等于功能可用（Blog 8-1）
24. WebUI在AI Agent框架里只能做监控，不能做控制（Blog 8-2）
25. 真值面不应该让agent直接写——父模型统一写入，agent只提proposal（Blog 8-2）
26. 框架应该是工具，不是项目——用户目录里只留运行时数据（Blog 8-2）
27. Roadmap本身会编码优先级错误——核心功能如果不在Phase 1出现，说明规划时就没把核心放在核心位置（Blog 8-3）
28. Phase膨胀是scope creep的结构化表现——AI会把每个想法做得很完整，让膨胀看起来像进展（Blog 8-3）
29. 理念偏离在早期最容易修复，一旦"先做完再fix"，后面每一步都在加固错误方向（Blog 8-3）
30. **用独立会话做review——让另一个agent审查主会话的产出，是对抗自动化偏差的低成本实践**（Blog 9）
31. 核心先行不是口号，是可验证的：如果两天就能搭完核心，说明之前在非核心的事情上花的时间都是方向债（Blog 9）

---

## 从弃置到重建：48小时内发生了什么

4月10日凌晨决定弃置Detent，当天上午就开始了bonfire。

心态上没有太多纠结。Detent的201次提交不是浪费——真值面双层系统、父模型写入模式、对抗式Agent循环、reentry路由表，这些设计在Detent里已经验证过了。要弃的不是这些设计，而是承载这些设计的那个"从WebUI开始做"的工程。bonfire要做的是：**把验证过的设计用对的顺序重新实现一遍**。

这次的约束很明确：
- 用Claude Code原生原语（skills、agents、hooks）实现，不造自己的运行时
- 先做核心pipeline（真值面→状态机→渲染器→Agent定义→Skills编排），后做文档和体验优化
- 作为plugin分发，`bash install.sh`一键安装到任意项目

两天，81次提交，9个Plan，114个测试全部通过。

---

## 一个关键的工作方式：双会话Review

这次做bonfire有一个新的实践，事后看效果很好：**主会话做开发，另开一个独立会话做review。**

具体流程是这样的：

主会话里用superpowers（一个brainstorming工具）做设计——它会生成设计Spec、实现Plan、然后逐Plan执行编码。但superpowers的产出是否合理，主会话里的Claude不会质疑自己（#10：AI不会主动质疑你的方向）。

所以我在另一个会话里，把superpowers输出的设计和Plan贴过去，让另一个Claude独立review。这个review会话不继承主会话的上下文，它只看到设计文档本身，所以能从一个"第一次看到这份设计"的视角给出反馈。

几个具体的例子：

**设计阶段**：主会话产出了bonfire的Plugin结构设计，我把Section 1贴到review会话，review会话立刻指出了两个硬性遗漏——templates目录和hooks目录在设计中被漏掉了。主会话里的Claude因为参与了讨论过程，对这些遗漏"视而不见"；review会话因为只看结果文档，反而更容易发现缺口。

**实现阶段**：Plan 6（E2E修复）之后，我把测试结果和剩余问题列表贴到review会话做triage。review会话对5个问题按severity排序，判断哪些需要立即修、哪些可以defer。主会话在连续执行多个Plan之后已经"沉浸"在细节里，review会话提供了一个更冷静的优先级视角。

**prompt优化**：114个测试通过后，review会话独立检查了agent prompt的表述，发现了真值面冻结门控的描述不够精确——"for each entry meeting maturity gate"太模糊，改成了基于具体字段的显式检查。这些都是prompt-only的修改，不影响代码，但直接影响Agent的行为质量。

这个做法的本质就是**手动实现了bonfire自身设计中的"对抗式审查"**——D-Critique、G-Red、G-Blue、H-Review这4个Agent的存在意义，就是让不同视角独立审查同一份方案。在bonfire还没有搭好之前，我用"两个Claude会话"手动模拟了这个机制。

成本很低：review会话不需要完整的项目上下文，只需要看设计文档和Plan就能给出有价值的反馈。相当于用O(1)的成本获得了一个独立审查者。

---

## 9个Plan的执行节奏

bonfire的搭建过程被superpowers拆成了9个Plan，每个Plan对应一个明确的交付物：

| Plan | 交付物 | 关键决策 |
|------|--------|----------|
| 1 Foundation | CLI入口、Schema定义、目录脚手架 | schema先行：先定义bonfire-v1.json（note定义、路由表、delta schema），再写代码 |
| 2 Truth Surface + State | 约束账本（propose/freeze/supersede）、状态机（step转换、reentry、run管理） | 采用Detent验证过的JSONL history + regenerated snapshot双层模式 |
| 3 Delta Renderer + Hook | 模板引擎、JSON→Markdown渲染、dual-write hook | 关键决策：每次写入JSON时自动触发Markdown渲染，而非手动render |
| 4 References + Agents | 参考文档（playbook、quality bar等）、10个Agent定义 | Agent prompt的精确度直接决定pipeline质量——每个Agent的precondition/postcondition都是硬约束 |
| 5 Skills + Templates | 5个Skill编排器、22个Markdown模板 | Skill是用户的入口（/bonfire:pre等），模板是可观测性的载体 |
| 6 E2E Fixes | 集成修复、端到端测试 | 全量集成后发现的问题集中修复，而非边做边修 |
| 7 Pipeline-Aware Steps | 状态机增加pipeline字段、step顺序显式化 | 解决"状态机不知道自己在哪个pipeline里"的问题 |
| 8 Code-Bundle Rendering Fix | 渲染器数组处理、undefined校验、objectToArray兜底 | 边界case修复：模板引擎对复杂数据结构的处理不够健壮 |
| 9 README Restructure | README重写：读者优先排序（what→how→why） | 从"为什么"开头改为"怎么用"开头——从内往外建的东西对用户不OK（#22） |

9个Plan的节奏本身就是"核心先行"的实践：Plan 1-3是骨架（数据结构→状态管理→渲染），Plan 4-5是血肉（Agent定义→用户入口），Plan 6-7是修复和增强，Plan 8-9是体验优化。对比Detent在Phase 1就做了CLI框架+WebUI脚手架，bonfire的前3个Plan全部聚焦在数据层——因为数据结构对了，上面怎么建都不会歪。

---

## Detent vs bonfire：同一个设计理念，不同的实现顺序

两个项目的核心设计完全一致——真值面、对抗式审查、冻结交接、回流机制。区别只在实现顺序。

| 维度 | Detent | bonfire |
|------|--------|---------|
| 第一件事 | CLI框架 + WebUI脚手架 | schema定义 + 约束账本 |
| 核心功能（控制论编排） | Phase 7（后插入） | Plan 2（第二个就做） |
| WebUI | Phase 6，9/29 Plan | 没有，defer到v2 |
| 到达可用状态 | 5天，201次提交，未达到 | 2天，81次提交，114测试通过 |
| 弃置原因 | Roadmap编码了错误的优先级 | N/A（进行中） |

两天做完Detent五天没做完的核心，这件事本身就验证了Blog 8-3的判断：**如果两天就能搭完核心，说明之前在非核心的事情上花的时间都是方向债。**

---

## 新认知

### #30 用独立会话做review——对抗自动化偏差的低成本实践

从Blog 4开始我就知道"AI不会主动质疑你的方向"，但一直没找到低成本的解决方案。之前的做法是自己扮演reviewer的角色，但人的注意力有限，连续工作几小时后批判性思维会下降。

双会话review是一个实用的折中：主会话沉浸在实现细节里，review会话只看产出物，天然具备"局外人视角"。不需要复杂的工具或流程，只需要开一个新的Claude会话，把设计文档贴过去问"你觉得这个方案有什么问题"。

这和bonfire的对抗式审查是同一个逻辑——核心不在于"用几个Agent"，而在于**确保有一个不共享执行上下文的独立视角来审查方案**。工具是手段，独立视角是目的。

### #31 核心先行不是口号，是可验证的

怎么判断自己有没有做到"核心先行"？一个简单的检验方法：**如果把所有非核心的东西砍掉，核心部分多久能交付？**

Detent的回答是"5天做不完"——因为核心在Phase 7，前面6个Phase都是非核心。bonfire的回答是"2天"——因为核心在Plan 1-3。

如果这个数字让你意外地小，说明之前的规划里有大量方向债伪装成了"必要的前置工作"。

---

*接下来：继续迭代bonfire，完成README重构后开始在真实项目上试跑pipeline。*
