# Blog 8: 把所有教训做进一个框架里

> 4/5 - 4/6 | Detent Phase 1-5 | 2天，全新项目

---

## 项目进度

```
Detent — 控制论Agent编排框架

[■■■■■□□□□□] CLI + Pipeline + 对抗性规划 + Coding Loop + 可观测性
```

## 核心认知（动态更新）

> 从 Blog 1 积累至今，划掉的是后来发现不对的

1. ~~工具和插件装够了就能vibe coding~~ → 工具不是瓶颈（Blog 1）
2. 动手之前先想清楚要做什么（Blog 1）
3. 多Agent之间的上下文要隔离（Blog 2）
4. Pipeline的可观测性很重要（Blog 2）
5. ~~QA环节放在最后兜底就行~~ → 质量约束必须嵌入每个阶段（Blog 4）
6. 上下文管理是直接影响产出质量的第一因素（Blog 3）
7. 编译能过不代表应用能用（Blog 3）
8. 认知没有变成硬约束，执行时一定会丢（Blog 3）
9. 补丁式修复解决不了架构问题，要敢停下来重构（Blog 4）
10. AI不会主动质疑你的方向，批判性思考得自己来（Blog 4）
11. 让AI研究一个项目要验证它真的读了源码（Blog 4）
12. ~~"胶水编程"——搭建优先，零自研~~ → 进一步修正：嘴上说搭建优先，实际还是会从头造，需要刻意约束（Blog 7）
13. session上下文管理是AI Coding的第一质量杠杆（Blog 5）
14. 工具的价值不在数量（Blog 5）
15. 需求必须包含约束条件，不只是期望（Blog 6）
16. "需求→功能"的转换才是最大杠杆点（Blog 6）
17. 第三方工具的配置schema会变，文档会过时（Blog 6）
18. 参考实现不能只借概念，要深入研究架构（Blog 7）
19. 做交互设计前要验证平台API的实际能力（Blog 7）
20. 🆕 自治和控制是一个频谱，不是开关——需要根据阶段重要性动态调节
21. 🆕 约束嵌入不等于约束执行——agent有工具权限不代表它会用，prompt里必须强制
22. 🆕 从内往外建的东西对开发者OK，对用户不OK——UX要从用户视角验证
23. 🆕 82个结构测试通过不等于功能可用——LLM系统必须有真实E2E测试

---

## 这几天想干什么

新项目。名字叫Detent。

Detent是机械术语，就是那种"定位卡扣"——旋转旋钮的时候，转到某个位置咔哒一声卡住的那个东西。pipeline里的每个门控就像卡扣，条件满足才放行。

设计上融合了三个东西：Beer的可行系统模型做组织架构、GSD的执行范式做流程引擎、ECL的约束账本做质量控制。核心是一个叫"truth surface"的东西——本质上是一个约束账本，所有agent的决策都必须跟frozen的约束对齐。

说白了就是把前面几个项目踩的坑系统化地解决。mosaicat的"质量不能后置"——truth surface来管。"AI不会主动质疑方向"——对抗性规划来管。

ROADMAP写好了。开干。

---

## 4/5 凌晨 — autonomous模式，快但不放心

带着完整ROADMAP，直接用GSD的autonomous模式跑。

Phase 1是CLI工具，没什么好纠结的，让它跑。Phase 2是基础pipeline，也还好。autonomous跑这种"明确知道要什么"的阶段确实快。

但跑到Phase 2快结束的时候开始不对劲了。Phase 3是对抗性规划——D-Critique、G-Red（红队攻击）、G-Blue（蓝队防御）、H-Review、J-Compile。这是整个框架最核心的创新点。autonomous意味着AI自己决定怎么实现，我来不及审视每一个设计决策。

比如红蓝对抗的边界怎么定、truth surface的读写权限怎么分配、哪些约束frozen哪些proposed——这些得我自己想清楚。让AI自己决定，万一方向歪了到后面才发现，又是mosaicat时代的"虚假进展感"。

果断停了。从Phase 3开始切手动：discuss → plan → review → execute，每个阶段我都过一遍。

---

## 4/5 下午 — 第一次UAT，全线翻车

在`/tmp/detent-uat`目录做用户验收测试。

`detent setup`——跑完了，返回`{"ok":true}`。打开目录。什么都没有。ls一下，发现有几个隐藏的JSON文件。就这？用户跑完setup看到空文件夹，你告诉我成功了？

典型的"从开发者视角设计的UX"。对写代码的人来说JSON状态文件写进去了就是成功了。对用户来说，什么可读的文件都没看到。

接着想跑skill。报"Unknown skill: detent:plan"。路径写死了`./detent-tools.cjs`，换了目录就找不到。skill的分发机制根本没想过。

还有setup之后pipeline_stage停在"idle"，要手动改成"discovery"才能开始。为什么不能setup完直接开始？

三个问题，全是基础UX。Phase 1-2该解决的，但那两个phase是autonomous跑的。

---

## 4/5 下午 — 拆truth surface

回来看Phase 3的架构。

truth surface被拆成了三个文件：frozen-decisions.md、constraint-ledger.md、domain-model.md。

不对。constraint-ledger里已经有PROPOSED和FROZEN两个状态了。文件边界不需要再表达一遍状态。而且freeze操作要从一个文件删再写另一个文件，有原子性问题——删了没写成功怎么办？

两个文件合成一个。truth surface就是constraint-ledger.md。状态用字段表达，不要用文件边界表达。

---

## 4/5 下午 — G-Blue的角色纯洁主义

G-Blue是蓝队防御agent。红队攻击完之后它负责防御和反驳。过程中G-Blue经常产生很好的reformulation建议——"这个约束应该这样表述更严谨"。

但G-Blue没有truth-propose权限。它觉得propose不是自己的职责，这些建议产生了，写在prose文本里，然后没有人执行。

从控制论角度想，G-Blue是唯一在对抗上下文中产生这些洞察的agent。让别的agent来做propose，它得先读G-Blue的输出、理解对抗上下文、再做propose——信息转手一次，必然有损耗。

直接给G-Blue加truth-propose权限。职责边界不应该卡死到妨碍信息流转。

---

## 4/5 晚上 — 手动UAT，4个bug

一个一个手动跑了一遍。

1. **truth-update没被agent调用。** 有工具权限但prompt里没强制要求。LLM不会"主动"使用工具——你给它锤子，它不会自己找钉子
2. **spawn不加载agent配置**
3. **--verbose参数缺失**
4. **reformulation建议丢在prose里**，没有结构化输出

这些问题有E2E测试应该能提前发现。给自己立了规则：每个phase做完都要跑一次E2E。

---

## 4/6 凌晨 — Phase 4 Coding Loop

Phase 4实现了Coder/Evaluator对抗循环。凌晨3:50完成。

完成之后看了下测试结果：82个测试全部通过。挺好的，对吧？

不对。仔细一看，82个测试全是结构性测试——检查文件是否存在、CLI参数是否正确、字符串是否包含特定内容。**没有任何一个测试真正启动了Claude Code子进程。** 整个框架的核心行为是调LLM生成代码、LLM评估代码，结果测试里一次LLM都没调过。

这跟mosaicat的Validator假通过是同一个问题的变体。测试的有效性取决于它验证的层次。结构测试只能保证文件在正确的位置，保证不了pipeline能真正跑通。

当即追加了一次真实的E2E测试——实际启动Coder和Evaluator agent，用真实的LLM调用跑一遍。

---

## 4/6 上午 — Phase 5 Observability

Phase 5做可观测性：JSONL事件日志、locale基础设施、generate-log命令。

这个phase大概2小时就完成了，几乎没什么返工。整个项目最平滑的一个phase。

可能因为可观测性作为横切关注点，加JSONL tee对现有代码的侵入很小。也说明如果需求足够清晰且自包含，AI Coding的效率其实非常高。

---

## 这几天做得好的 / 做得不好的

**做得好的：**
- autonomous跑到一半觉得不对，果断停了切手动
- truth surface三文件合一的决策干净利落
- G-Blue的权限问题用控制论的信息流框架来分析，不是拍脑袋
- 发现82个测试全是结构测试后立刻补了真实E2E

**做得不好的：**
- Phase 1-2用autonomous跑没经过充分测试，UAT一堆基础问题
- setup的UX完全是开发者视角
- skill分发没想清楚，只能在Detent仓库内用
- 82个测试"通过"给了我虚假信心——跟Blog 3里Validator的假通过是同一类问题

---

## 反思

Detent是前面所有教训的集大成。truth surface解决mosaicat的"质量不能后置"。对抗性规划解决"AI不会主动质疑方向"。constraint-ledger解决"认知没有变成硬约束就会丢"。

但即使带着这些认知来做，还是踩了新坑。setup的UX、skill分发、82个假测试。

认知不会一劳永逸地解决问题。只会让你踩更高级的坑。以前是"编译通过但应用不能用"，现在是"agent有权限但不主动使用工具""82个测试通过但没有一个测到核心行为"。坑的层次在升级，但坑不会消失。

不过有一个变化是实在的：从mosaicat时候的"被问题推着走"，到现在的"主动设计规则来预防问题"。

---

*接下来：WebUI翻车和控制论重构。*
