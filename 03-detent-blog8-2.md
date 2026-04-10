# Blog 8-2: WebUI翻车、控制论重构、和"框架应该是工具而不是项目"

> 4/6 - 4/9 | Detent Phase 6-8 | 3天，137 commits

---

## 项目进度

```
Detent — 控制论Agent编排框架

[■■■■■■■■□□] Phase 1-8完成，Web监控面板 + 控制论重构 + NPX打包
```

## 核心认知（动态更新）

> 从 Blog 1 积累至今

1. ~~工具和插件装够了就能vibe coding~~ → 工具不是瓶颈（Blog 1）
2. 动手之前先想清楚要做什么（Blog 1）
3. 多Agent之间的上下文要隔离（Blog 2）
4. Pipeline的可观测性很重要（Blog 2）
5. ~~QA环节放在最后兜底就行~~ → 质量约束必须嵌入每个阶段（Blog 4）
6. 上下文管理是直接影响产出质量的第一因素（Blog 3）
7. ~~编译能过不代表应用能用~~ → 进一步：Playwright自测通过也不代表功能可用（Blog 8-2）
8. 认知没有变成硬约束，执行时一定会丢（Blog 3）
9. 补丁式修复解决不了架构问题，要敢停下来重构（Blog 4）
10. AI不会主动质疑你的方向，批判性思考得自己来（Blog 4）
11. 让AI研究一个项目要验证它真的读了源码（Blog 4）
12. ~~"胶水编程"——搭建优先，零自研~~ → 嘴上说搭建优先，实际还是会从头造（Blog 7）
13. session上下文管理是AI Coding的第一质量杠杆（Blog 5）
14. 工具的价值不在数量（Blog 5）
15. 需求必须包含约束条件，不只是期望（Blog 6）
16. "需求→功能"的转换才是最大杠杆点（Blog 6）
17. 第三方工具的配置schema会变，文档会过时（Blog 6）
18. 参考实现不能只借概念，要深入研究架构（Blog 7）
19. 做交互设计前要验证平台API的实际能力（Blog 7）
20. 自治和控制是一个频谱，不是开关（Blog 8-1）
21. 约束嵌入不等于约束执行——prompt里必须强制（Blog 8-1）
22. 从内往外建的东西对开发者OK，对用户不OK（Blog 8-1）
23. 82个结构测试通过不等于功能可用（Blog 8-1）
24. 🆕 WebUI在AI Agent框架里只能做监控，不能做控制——Claude headless不能被HTTP触发
25. 🆕 真值面不应该让agent直接写——父模型统一写入，agent只提proposal
26. 🆕 框架应该是一个工具，不是一个项目——用户目录里只留运行时数据

---

## 4/6 下午 — Phase 6 WebUI：按钮点了没反应

Phase 6做WebUI。设计上是一个控制面板——用户可以从浏览器看到pipeline状态、启动新的pipeline、查看每个阶段的产出。

技术栈选了Fastify后端 + Svelte 5前端 + WebSocket实时推送。执行很快，当天晚上就标记完成了。Playwright自动化测试也通过了。

然后我自己打开浏览器试了试。

Pipeline页面打开了，看到了启动按钮。点了。什么都没发生。

这个瞬间很眼熟。mosaicat时代的"编译通过但应用不能用"（Blog 3），到Detent的"82个结构测试通过但没测核心行为"（Blog 8-1），现在又来了"Playwright通过但按钮不能用"。自动化测试给你的信心，和功能真正可用之间，总是隔着一道鸿沟。

这其实是测试理论里一个反复出现的主题。Blog 8提到了Martin Fowler的测试金字塔（[Fowler, 2012](https://martinfowler.com/bliki/TestPyramid.html)）——我的Playwright测试看起来像是金字塔顶部的E2E测试，但它验证的只是"页面能渲染、按钮能点击"，没有验证"点了之后真的启动了pipeline"。本质上还是在测结构，不是在测行为。**测试的层次不是由工具决定的（Playwright vs Jest），而是由验证的对象决定的（结构 vs 行为）。**

排查之后发现根因：WebUI设计的"启动pipeline"功能，需要通过REST API触发Claude Code进程。但Claude Code的headless模式有严格限制——**它只能在CLI会话里运行，不能被HTTP请求触发。** WebUI里点"启动"等于在空气中喊话，没有人接。

这是一个架构预设错误。设计的时候默认"WebUI可以控制Agent"，但没有验证Claude Code是否允许这种调用方式。跟Blog 6飞书卡片的问题一样——交互设计建在了一个不成立的平台假设上。

从产品设计的角度看，这属于Blog 7提过的 **platform risk** 的又一个实例。但这次更隐蔽：飞书卡片的限制好歹还需要查API文档才能发现，Claude Code的headless限制是一个更底层的运行时约束——不是API不支持某个功能，而是整个调用模型不成立。**平台风险不只存在于API能力层面，还存在于运行时架构层面。** 这类风险更难通过文档发现，往往要到集成测试时才暴露。

---

## 4/7 凌晨 — 重新定位：监控面板，不是控制面板

睡了一觉想清楚了。

WebUI不应该做执行入口。Agent只能从CLI和技能调用。WebUI的价值是让用户在浏览器里实时看到pipeline在干什么——哪个阶段、哪个agent、产出了什么、有没有报错。

方案很简单：把WebUI的执行代码全部删掉，server端只做文件监听——watch state.json和JSONL事件日志，有变化就通过WebSocket推给前端。

从"控制面板"退回到"监控面板"。功能砍了一大半，但剩下的部分是实实在在能用的。

这个决策在产品管理里有个对应的概念叫 **scope hammering**——当发现原始需求不可行时，不是放弃整个feature，而是削减到一个可行的子集。关键是分清哪些是 must-have（看到pipeline状态）、哪些是 nice-to-have（从浏览器控制pipeline）。控制功能不可行，但监控功能完全可行，而且对用户的核心价值——"知道pipeline在干什么"——是满足的。

---

## 4/7 下午 — 用真实项目跑pipeline，constraint-ledger出问题了

WebUI修好之后，拿一个真实项目跑了一次完整pipeline。

pipeline本身跑通了。但跑完之后我去看constraint-ledger——就是truth surface，整个框架的核心——发现完全看不懂。

一堆约束条目，分不清哪个agent在哪个阶段写进去的。有5条最初的约束从PROPOSED状态就没动过，因为没有人challenge过它们，而freeze gate要求`challenged_by`不为空才能freeze。这5条约束卡在那里，既不能freeze也不会被清理。G-Blue提出的修订版约束形成了孤岛，没有agent负责把它们整合回去。

我问Claude：你是不是也觉得这个结构有问题？Claude说是的，多个agent同时写入，没有清晰的所有权。

这是一个比Blog 8里"truth surface拆成三个文件"更深层的问题。那次是文件组织的问题，改个结构就行。这次是**写入模型本身有问题**——谁有权写truth surface？

从分布式系统的角度看，这就是经典的 **concurrent write conflict** 问题。数据库领域早就有成熟的解决方案：要么用锁（pessimistic concurrency），要么用版本号冲突检测（optimistic concurrency），要么指定单一写入者（single writer principle）。我的constraint-ledger允许任何agent随时写入，等于是一个没有任何并发控制的共享状态——在分布式系统里这是第一天就会被毙掉的设计。

---

## 4/7 晚上 — Phase 7：控制论重构

从控制论的角度重新想了一下。

在控制系统里，参考信号（reference signal）只由上级设定。下级单元只能提出建议，最终写入由上级决定。这个原则来自Stafford Beer的 **可行系统模型 (VSM)**（[Beer, 1972](https://en.wikipedia.org/wiki/Viable_system_model)）中System 3（内部监管）和System 1（执行单元）的关系——System 1不能直接修改System 3设定的规则，只能向上反馈。映射到Detent：

- Agent不应该直接写truth surface
- Agent在自己的输出文件里包含结构化的proposal（"我建议增加这条约束"）
- 由父模型（orchestrator skill）统一读取proposal、做冲突检测、执行写入

这就是"parent-as-sole-writer"模式。真值面从条目日志改成认知快照——constraint-ledger.md是当前有效状态，constraint-history.jsonl是审计追踪。

这个模式在分布式系统里也有对应：**Event Sourcing + CQRS**（[Fowler, 2005](https://martinfowler.com/eaaDev/EventSourcing.html)）。Agent提proposal相当于发event，父模型做projection生成当前状态。读写分离、单一写入者、完整审计追踪——这些都是Event Sourcing的标准特征。只不过这里的"event"不是用户操作，而是agent的约束建议。

同时把约束的状态机从二元（PROPOSED/FROZEN）扩展成四元：PROPOSED → CHALLENGED → FROZEN → SUPERSEDED。

还补了Discovery阶段的完整链路：A（需求澄清）→ B（方案发散）→ C（收敛编译）→ D（挑刺）。我最初的构想就是这样的——需求先发散再收敛，不是一步到位写进约束。但之前的实现跳过了A、B、C，直接从用户输入到D-Critique。

这个"先发散再收敛"的设计，跟产品设计领域的 **Double Diamond** 模型（[Design Council, 2005](https://www.designcouncil.org.uk/our-resources/the-double-diamond/)）异曲同工：第一个菱形是Discover→Define（发散→收敛找到正确的问题），第二个菱形是Develop→Deliver（发散→收敛找到正确的解法）。我的A/B阶段对应Discover/Develop（发散），C/D阶段对应Define/Deliver（收敛）。只不过传统设计思维里这是人做的，我把它变成了agent pipeline。

Phase 7从4/8凌晨开始执行，上午完成。改动量很大但方向很清楚。

---

## 4/8 中午 — 跑完发现新增的stage没有agent定义

Phase 7加了A、B、C三个新stage，但实际上只改了skill模板，忘了给这三个stage写对应的agent定义文件。另外之前有个Playbooks目录不知道什么时候变空了。

这种"改了一半"的问题在快速迭代中特别容易出现。创建了一个quick task把blocking issues全部修掉。

---

## 4/8 下午 — "Detent对用户项目的侵入性太大了"

修完之后又拿真实项目跑了一次。这次pipeline运行正常了，但我注意到另一个问题：用户项目里到处都是Detent的文件。

detent-tools.cjs在项目根目录。package.json和node_modules是Detent的依赖。locale/目录、web/目录、test/目录全都在用户项目里。CLAUDE.md也被Detent的内容覆盖了。

用户安装一个工具，结果自己的项目被塞了一堆不相干的文件。这不像是在用一个工具，像是在改造一个项目。

这个问题让我想到了Unix哲学里一个经典原则：**"Make each program do one thing well"**（[McIlroy, 1978](https://en.wikipedia.org/wiki/Unix_philosophy)）。一个好的工具应该是非侵入性的——你安装它、使用它、它帮你做事，但它不应该改变你的项目结构。想象一下你装了个ESLint，结果它在你项目里创建了20个目录——没人会接受这种工具。

从用户体验角度看，这违反了Jakob Nielsen的**最小惊讶原则 (Principle of Least Astonishment)**：用户对"安装一个框架工具"的心理预期是"配置文件+命令行入口"，不是"项目目录被重组"。

想清楚之后目标很明确：用户项目里只应该有两样东西——`.detent/`运行时目录和`.claude/`下的技能入口。CLI、依赖、测试、WebUI、locale这些全都应该在Detent自己的安装目录里。

GSD已经验证了这个模式：框架代码全部放在`~/.claude/get-shit-done/`下面，用户项目里只有`.planning/`。

---

## 4/8 晚上 — Phase 8 NPX Packaging

按照GSD的模式做了一次彻底的重构：

- 所有框架代码迁移到`$HOME`绝对路径引用
- 创建npm可发布的repo结构
- 写了`bin/detent.cjs`安装入口
- 清理了用户项目里的所有框架文件

参考了npm生态里成熟的CLI工具分发模式——[create-react-app](https://github.com/facebook/create-react-app)、[degit](https://github.com/Rich-Harris/degit)这类工具，安装后只在用户项目里留最小化的配置，框架代码全在全局或缓存目录。

大概2小时完成。这个phase跟Phase 5一样很快，因为目标清晰且自包含。

凌晨0点完成Phase 8，紧接着做了一次全局的废文件清理。

---

## 4/9 — 回头验证Discovery的效果

拿一个新项目跑了一次完整的pipeline。这次重点看Phase 7重构后的Discovery阶段——A/B/C三个stage是否真的做到了"先发散再收敛"。

跑完之后对比了一下我最初的构想：

- A阶段：需求即假设，反问模糊不清的点，质疑不符合客观事实的东西
- B阶段：发散畅想，补全用户的盲区
- C阶段：将畅想内容收敛拆解成具体需求
- D阶段：挑刺，引入对抗agent做正交过滤

基本对应上了，但还有差距。实际跑下来A阶段的澄清深度不够，B阶段的发散广度不够。Discovery要真正有实质性内容，还需要继续迭代。

触发了Phase 9的规划。

---

## 这几天做得好的 / 做得不好的

**做得好的：**
- WebUI翻车后没有硬修执行功能，而是重新定义了WebUI的边界——监控而非控制
- constraint-ledger不可读的问题追到了根因（写入模型），不是简单格式化一下
- "parent-as-sole-writer"的设计有控制论和分布式系统的双重理论支撑
- 侵入性问题发现后当天就完成了NPX重构

**做得不好的：**
- WebUI的执行功能在设计阶段就应该验证Claude headless能不能被HTTP触发
- Phase 7新增了三个stage但忘了写agent定义——快速迭代中的遗漏
- Playwright自测通过了但功能不可用——又一次"自动化测试的虚假信心"

---

## 反思

这三天最有意思的是constraint-ledger的演进。

Blog 8里把三个文件合成一个，解决的是"用文件边界表达状态"的冗余问题。这次发现的是更深一层的问题：**谁来写真值面？** 让每个agent都能写，结果是状态混乱、约束卡死、改进建议形成孤岛。

从控制论拿到了答案：参考信号只由上级写入。agent只提proposal，父模型统一执行。这个模式一旦确立，很多之前的问题自然消解了——不需要考虑多个agent并发写入的冲突，不需要担心谁的proposal覆盖了谁的。

有趣的是，这个"parent-as-sole-writer"模式跟软件架构里的很多经典模式是同构的。Redux的单向数据流（action → reducer → state）、数据库的WAL（Write-Ahead Log）、甚至Git的commit模型——都是"提议→单一权威处理→生成新状态"的变体。**好的架构模式是跨领域通用的，只是在不同场景下穿了不同的外衣。**

另一个有意思的点是Phase 8。侵入性问题不是技术问题，是产品问题——在自己的开发目录里跑pipeline的时候注意不到，换到一个真实用户项目里立刻就觉得不对。这跟Blog 8里setup返回`{"ok":true}`但用户看到空目录是同一类问题：**开发者视角和用户视角之间总是有落差，而你只有在用户视角下才能发现这种落差。**

这让我联想到产品设计里的一个基本功：**dogfooding**——自己用自己的产品。但只是在开发环境里dogfooding是不够的，你需要在用户的真实环境里dogfooding。Google的 [Chromium](https://www.chromium.org/getting-involved/dev-channel/) 项目有dev/beta/stable三个频道，每个频道代表越来越接近真实用户的环境。我的"在自己开发目录里跑"相当于dev频道，"在真实用户项目里跑"才是beta频道。很多产品问题只在beta频道才暴露。

到这里，Detent从一个"能跑的框架"变成了一个"可以安装的工具"。还差得远，但方向对了。

---

*接下来：Blog 8-3，review代码后发现偏离过大，决定弃置Detent。*
