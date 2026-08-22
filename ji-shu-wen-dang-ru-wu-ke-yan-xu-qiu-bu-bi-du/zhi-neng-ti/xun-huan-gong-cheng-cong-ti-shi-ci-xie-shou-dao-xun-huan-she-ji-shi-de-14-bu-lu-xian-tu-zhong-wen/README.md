# 循环工程：从提示词写手到循环设计师的 14 步路线图（中文）

**原文来源：** [X 帖子](https://x.com/0xCodez/status/2064374643729773029)　**作者：** [@0xCodez](https://x.com/0xCodez)　**发布日期：** 2026-06-09

本文为上述原文的中文译文。文中的数据、引语与判断均归属于原作者；核对原意时请参阅本页下方的“英文原文”。

![图像](https://pbs.twimg.com/media/HKYhsj-XsAAd3fr?format=jpg\&name=large)

大多数开发者仍在手动给编码智能体写提示词。他们敲下提示词，等待回复，阅读 diff，再敲下一轮。**10** 个开发者里，**9** 个从未写过哪怕一个能替自己提示智能体的循环。

没有**自动化机制**，没有**状态文件**，没有**验证器**，没有**定时调度**。杠杆点已经转移——从亲手敲提示词，到设计一个能自己敲提示词的系统。这就是从提示词写手到循环设计师的 14 步路线图。

关注我的 LinkedIn，获取最新 AI 前沿动态：[linkedin.com/in/lev-deviatkin](https://linkedin.com/in/lev-deviatkin)

这份路线图帮你完成这一转变——综合了 Anthropic 工程文档、Addy Osmani 关于循环工程的长文，以及近期测量研究的观点。

三个层级：先判断你是否真的需要循环，再学习五个构建模块，最后搭建一个最小可用、不会反噬你的循环。

![图像](https://pbs.twimg.com/media/HKYTPjUX0AAJooq?format=png\&name=large)

**14 步。3 个层级。停止写提示词。开始设计循环。**

第一部分 **· 为什么做 & 要不要做**

## 01. 循环工程：把自己从提示词工程师的位置上替换下来

两年来，你从编码智能体那里获得产出的方式一成不变：写提示词、提供上下文、阅读回复、再写下一条提示词。智能体只是工具，全程由你操控。**这一幕正在落幕。**

循环工程（loop engineering）是搭建一个小系统：自己发现任务、交给智能体执行、检查结果、记录过程、决定下一步——全程无需人工干预。你只需设计一次，之后系统自己替你去提示智能体。

Addy Osmani 把它拆成六个部分：

![图像](https://pbs.twimg.com/media/HKYT-XBXEAI0yYt?format=png\&name=large)

Anthropic 的工程师现在每天合并的代码量是 2024 年的八倍——Anthropic 自己称这个数字"几乎肯定夸大了真实的生产力提升"。

数字有争议。机制没有：**杠杆点从亲手敲提示词转移到了设计那个敲提示词的循环。**

## 02. 动手之前，先跑一遍四条件测试

循环只有在满足四个条件时才值回票价。漏掉一个，成本就会超过收益。这是 AlphaSignal 分析中的实话，也是大多数 X 帖子避而不谈的部分：

![图像](https://pbs.twimg.com/media/HKYVILFW0AAmsCm?format=png\&name=large)

四个条件，用大白话说：

* **任务重复出现。** 循环通过多次运行摊薄搭建成本。一次性任务？一条好提示词更快更便宜。如果工作不是每周都重复，你就没有循环——你只有一个跑过一次的脚本。
* **验证可以自动化。** 循环需要某种机制，能在你不在场的情况下判定工作失败。测试套件、类型检查器、代码风格检查器、构建脚本。没有自动化检查，你就得坐回椅子上逐条阅读 diff——这正是循环本该替你干掉的活。
* **你的 token 预算扛得住损耗。** 循环会重复读取上下文、重试、探索。这些都要烧 token，不管最终有没有产出。这项技术花得起才玩得起，所以对那些 token 近乎免费的人来说理所当然，对那些按量计费的人来说就是 reckless（鲁莽）。
* **智能体拥有资深工程师的工具。** 日志、bug 复现环境、能运行自己写的代码并看到哪里报错的能力。没有这些，循环就是盲迭代。

## 03. 谁受益，谁吃亏。循环向舍得花钱的人倾斜

这套经济学并非普适。那些觉得循环工程理所当然的人，通常拥有不计费的 token。

那些觉得它 reckless 的人，往往用着 20 美元的消费级套餐，试图运行重度验证循环，却担心撞上限额或收到意外账单。

实践中真正受益的人：

* **有重复性、机器可检查的工作，且有预算运行的团队**——持续测试分类、依赖升级、代码规范检查与自动修复（lint-and-fix）、在测试覆盖率强的代码库上将 issue 转为 PR 草稿。
* **已有强测试套件的代码库。** 如果一个初级工程师拿着检查清单就能做这件事，且测试套件能抓住他们的错误，那循环就适合。
* **已经使用多智能体模式的异步优先团队。** 对这些团队来说，Routines 正是缺失的编排层。

今天应该跳过循环的人：

* **消费级套餐的独立开发者**——token 账单比生产力提升先到。
* **代码没有任何自动化验证的人。** 没有真实检查的循环，不过是智能体在自说自话。
* **真正瓶颈是评审容量而非打字速度的团队。** 循环会生成更多代码；如果评审本来就是瓶颈，它只会让队列更长。

对于一次性任务、探索性工作，或任何"完成"需要主观判断的事，**一条精准的手动提示词仍然是更好的选择**。这篇文章的诚实版本是：循环工程是真的，但大多数开发者暂时还不需要它。

## 04. 30 秒循环检查

第二步的四条件测试是战略决策。这是战术层面的——在把某个具体任务变成循环之前，你跑的检查清单。

**漏掉一项，就继续用手动提示词。**

* **1. 任务每周至少发生一次。** 低于每周 → 搭建成本永远摊不回来。
* **2. 测试、类型检查、构建或代码风格检查器能拒绝坏输出。** 没有自动化门禁 → 智能体自己给自己打分。
* **3. 智能体能运行它修改的代码。** 没有复现环境 → 迭代是盲的。
* **4. 循环有硬停止。** Token 预算、迭代次数或时间限制。没有之一，循环就会一直跑到有人发现账单为止。
* **5. 合并、部署或变更依赖前有人类评审。** 任何不可逆的操作都需要人类审批门禁。

![图像](https://pbs.twimg.com/media/HKYYGruWIAAWmLC?format=png\&name=large)

适合作为第一个循环的场景：

* **CI 故障分类**——每晚扫描失败、分类根因、为简单问题起草修复 PR。
* **依赖升级 PR**——每周扫描更新、测试兼容性、开启 PR。
* **代码规范检查与自动修复**——每次 PR 开启事件时自动应用风格修复。
* **Flaky test（不稳定测试）复现**——循环直到某个假设通过测试检验。
* **Issue 转 PR 草稿**——在测试强的代码上，坏输出会被测试套件拒绝。

不适合作为第一个循环的场景——这些需要人类坐在椅子上：

* 架构重写
* 认证或支付代码
* 生产部署
* 模糊的产品工作
* 任何"完成"是主观判断的事

**第二部分 · 五个构建模块**

## 05. 自动化：循环的心跳

自动化让循环成为真正的循环，而不是你手动跑过一次就忘的东西。它们按时间表、事件或触发条件启动。它们是心跳——循环里其他一切都挂在它上面。

在两个关键工具里长这样：

* **Codex。** Automations 标签页——选一个项目、设一条提示词、设定节奏、选择本地 checkout 或后台 worktree。发现问题的运行会进入 Triage inbox（待审队列）；没发现问题的运行会自动归档。
* **Claude Code。** 三个原语组合出同样的形状：`/loop` 用于会话级节奏，Desktop 定时任务用于重启后仍然存活，Routines 用于电脑关机后的云端运行。配合生命周期事件的 hook（钩子）使用。

自动化内部有两个原语，区分能工作的循环和烧钱的循环：

* **`/loop`** 按节奏重复运行。想要定期检查、不管状态如何时都用它。
* **`/goal`** 持续运行，直到你写的某个条件真正成立。一个独立的小模型检查完成状态，所以写代码的智能体和打分的不是同一个。

![图像](https://pbs.twimg.com/media/HKYY3V-WkAAiB-j?format=jpg\&name=large)

这就是\*\*制作者-检查者分离（maker-checker split）\*\*在停止条件上的应用。

```
/loop 30m /goal All tests in test/auth pass and lint is clean.
  Scan src/auth for new failures, propose fixes in claude/auth-fixes,
  open draft PR when goal condition holds.

▲ Claude
  CronCreate(*/30 * * * * : auth quality loop)
  Stop condition: tests pass + lint clean (verified by checker)
✓ Scheduled. Will continue past intermediate completions
  until /goal condition is met by independent checker.
```

## 06. Worktree（工作树）：并行而不混乱

一旦你运行超过一个智能体，文件就开始冲突。两个智能体写同一个文件，跟两个工程师不沟通就提交到同一行代码一样头疼。

**Git worktree 解决了这个问题**——一个独立的工作目录，位于自己的分支上，共享同一个仓库历史，所以一个智能体的编辑根本碰不到另一个的 checkout。

![](https://pbs.twimg.com/tweet_video_thumb/HKYZaJqXQAAIF96.jpg?name=large)

GIF

在两个工具中如何呈现：

* **Codex** 内置 worktree 支持——多个线程同时访问同一个仓库而不会互相碰撞。
* **Claude Code** 直接暴露 git worktree，`--worktree` 标志可以在独立 checkout 中开启会话，子智能体（sub-agent）的 `isolation: worktree` 设置让每个助手获得一个干净的 checkout，用完后自动清理。

**Worktree 消除了机械层面的冲突，但你仍然是天花板。** 你的评审带宽决定了你能实际并行运行多少智能体——不是工具。

## 07. Skill（技能）：项目知识写一次。每次运行都读。

Skill 是你停止像金鱼一样每次会话都重新解释同一套项目上下文的方式。两个工具使用相同格式：一个文件夹，里面放 SKILL.md，包含指令和元数据，外加可选的脚本、引用和资产。

为什么这对循环特别重要：没有 skill 的循环每次周期都从零重新推导整个项目上下文。**有了 skill，意图才能累积。**

编码规范、构建步骤、"我们之所以不这么干是因为那次事故"——写一次在外面，每次运行都读。

```
name: ci-triage
description: Classify CI failures by root cause (env, flake, real bug,
  dependency, infra), draft fixes for the easy ones, escalate the rest.
  Trigger whenever a workflow run fails or on the morning triage loop.
---

# CI triage skill

## Classification rules
- env: missing secret, wrong env var, infra not provisioned. # human
- flake: passes on retry without code change. # retry once, then file
- bug: deterministic failure tied to recent commit. # draft fix
- dependency: failure tied to a version bump. # draft rollback
- infra: timeout, OOM, runner issue. # escalate

## Fix patterns
- Auth tests → check src/auth/middleware first
- Database tests → verify migration applied in CI env
- E2E tests → check selectors against the latest UI snapshot

## Never do
- Disable failing tests — always file as escalation instead
- Modify CI config without human approval
- Touch src/payments/ or src/billing/ (in claude/permissions.md)

## State
Update STATE.md after each run: file paths checked, classifications,
PRs opened, items escalated.
```

## 08. 连接器：循环触碰你的真实工具。通过 MCP。

只能看到文件系统的循环是个小循环。**连接器（Connectors）**，基于模型上下文协议（Model Context Protocol, MCP），让智能体读取你的 issue 追踪器、查询数据库、调用 staging API、在 Slack 里发消息。

![图像](https://pbs.twimg.com/media/HKYZ6H9XEAA54ks?format=jpg\&name=large)

Codex 和 Claude Code 都支持 MCP，所以你为一个写的连接器通常直接在另一个里也能用。

这就是"给你修复方案"的智能体和"开启 PR、关联 Linear 工单、CI 变绿后在频道里 ping 人"的循环之间的区别。

连接器是循环能在你的真实环境中行动的原因，而不只是告诉你"如果它能的话它会怎么做"。

对循环工作回报最快的连接器，按顺序：

* **GitHub**——读取仓库、创建分支、开启 PR、在 issue 下评论、响应 webhook 事件。任何代码循环最大的首日收益。
* **Linear 或 Jira**——随着循环进展更新工单、把 PR 关联回 issue、验证通过后自动关闭条目。
* **Slack**——发布分类结果、升级时 ping 人类、早上总结夜间运行。
* **Sentry / 你的错误追踪器**——让循环调查线上告警、为高频率问题起草修复。

## 09. 子智能体：让制作者远离检查者

循环中最有用的结构性设计，毫无疑问，是把写代码的智能体和检查代码的智能体分开。Osmani 的表述很精确：写代码的模型"给自己打分的时候太宽容了"。第二个拥有不同指令、有时甚至不同模型的智能体，能抓住第一个智能体说服自己忽略的东西。

![图像](https://pbs.twimg.com/media/HKYaufLXEAAgTCz?format=png\&name=large)

这正是 Anthropic 2024 年 12 月工程文章里记录的**评估器-优化器模式（evaluator-optimizer pattern）**，只是换了个名字。一个模型生成，另一个批判，重复。2026 年火起来的说法，十八个月前就写进了文档。

子智能体在两个工具中的落地方式：

* **Codex** 只在被要求时 spawn 子智能体，同时运行它们，然后把结果折叠回一个答案。你在 `.codex/agents/` 里用 TOML 文件定义自己的智能体——名称、描述、指令、可选的模型和推理强度。你的安全评审员可以是高推理强度上的强模型，而你的探索者可以是某个快速的只读模型。
* **Claude Code** 在 `.claude/agents/` 里用子智能体和智能体团队做同样的事，在它们之间传递工作。通常的分工：一个探索、一个实现、一个对照 spec 验证。

**为什么这在循环里特别重要：** 循环在你离席时照样运转，所以只有当你拥有一个真正信得过的验证器时，你才敢放心离席。子智能体烧更多 token，因为每个都做自己的模型和工具调用——在值得买第二意见的地方花这个钱。

第三部分 **· 要么搭对，要么别搭**

## 10. 状态文件。智能体会忘。文件不会。

这块听起来太简单以至于不重要，实际上是每个能工作的循环的脊柱。一个 markdown 文件、一个 Linear 看板、一个 JSON 状态——任何活在单次对话之外、记录已完成和待办的东西。

为什么重要：智能体默认记忆很短。这次会话学到的东西，除非写下来，明天就没了。

**Osmani 的法则：智能体会忘，仓库不会。** 没有持久状态的循环每次运行都重启；有状态的循环能恢复。

```json
# Loop state · ci-triage

## Last run
2026-06-09 03:30 UTC · 7 failures classified, 3 fixes drafted, 4 escalated

## In progress
- claude/fix-auth-token-refresh — tests passing locally, awaiting CI
- claude/fix-flaky-payment-webhook — retry pattern applied, monitoring

## Completed today
- claude/bump-axios-1.7.4 → merged (CI green, deps loop verified)
- claude/lint-fix-pass-june-9 → merged

## Escalated to humans
- src/billing/refund.ts — tests failing in 3 ways, root cause unclear
- ci/staging-runner — infra timeouts, not a code issue

## Lessons learned (write here, not in chat)
- 2026-06-08: PowerShell hits TLS 1.2 issue on this Windows runner. Use bash.
- 2026-06-07: tests/e2e/checkout requires Stripe webhook secret in env. Skip if missing.

## Stop conditions met since last review
- /goal "all tests pass + lint clean" achieved on commit 3a7b8c1 at 02:14 UTC
```

状态文件存放位置的两种模式：

* **仓库内的 Markdown**——根目录或 `.claude/` 内的 STATE.md。版本控制。简单。可读 diff。最适合个人或小团队。
* **外部系统（Linear、GitHub Issues、数据库）**——跨仓库存活、可查询、支持团队级可见性。最适合生产级循环，多个需要看到循环在做什么的人类。

对于可能偏离目标的长期运行循环，把状态文件配上**一个常驻的高层级 spec**——VISION.md 或 AGENTS.md——让智能体每次运行都重读。状态告诉智能体它在哪里。Spec 告诉它要去哪里。

## 11. 最小可用循环

如果你通过了第二步的四条件测试，先搭一个能工作的最小循环，再上花哨的东西。**四个部分，没有蜂群。**

![图像](https://pbs.twimg.com/media/HKYbR9NXcAAfivV?format=png\&name=large)

四个部分，用大白话说：

* **一个自动化。** 按节奏启动、在明确条件下停止的定时运行。Claude Code 里用 `/loop`，Codex 里用 automation。想要它跑到某个声明条件成立时，配上 `/goal`。
* **一个 skill。** 一个 SKILL.md，存储智能体否则每次运行都要从零重新推导的项目上下文。
* **一个状态文件。** 一个 markdown 文件或 Linear 看板，记录已完成和待办。明天的运行恢复，而不是重启。
* **一个门禁。** 测试、类型检查或构建，自动让坏工作失败。**这是决定循环是在帮你省钱还是在白白烧钱的部件。**

**顺序很重要：** 先让一次手动运行可靠。把它变成 skill。包进循环。然后调度它。跳过步骤，循环就会在生产环境里栽跟头。

关键指标是**每次被接受变更的成本**——不是花了多少 token，不是尝试了多少任务，不是调度了多少循环。如果你的接受率低于 50%，你就在做循环本该替你省掉的评审工作，循环得不偿失。

## 12. Ralph Wiggum 式循环：那些悄无声息失败的循环

工程师 Geoffrey Huntley 记录了这种失败模式，并用《辛普森一家》里那个总是后知后觉的角色给它命名：Ralph Wiggum 式循环。一个本该只在完成时发出完成 token 的智能体提前发出了，循环在半完工的状态下退出。没有硬门禁，循环悄无声息地失败，继续烧钱。

![图像](https://pbs.twimg.com/media/HKYcBV9WwAApV5v?format=jpg\&name=large)

Ralph Wiggum 式循环发生在：

* **没有真实验证器。** 只是让第二个智能体"评审"一下，没有客观信号。两个乐观主义者在互相捧场。
* **软完成条件。** "完成"由智能体自己判断，而不是由测试、构建或类型检查定义。
* **没有硬停止。** 循环继续跑到某个外部因素杀死它（速率限制、你发现）为止，而不是在成功被验证后停止。

修复方案就是第 11 步的门禁——**某种客观到能让工作失败的东西**。测试通过或失败。构建编译或不编译。代码风格检查器返回零或非零。不是靠"主观判断"的验证器。

其他值得了解的实测失败模式：

* **长会话中的目标漂移。** 每次总结步骤都有损；"不要做 X"的约束在第 47 轮消失。缓解：每次运行重读常驻的 VISION.md 或 AGENTS.md。
* **自我偏好偏差。** 写代码的智能体给自己打分太宽容。缓解：一个独立的验证子智能体，不接触制作者的推理过程。
* **智能体敷衍症（agentic laziness）。** 循环在部分完成时宣布"够好了"。缓解：`/goal` 配客观停止条件，由新模型检查。

## 13. 理解债务与认知投降

这是那种循环越好用起来越疼、而不是越差越疼的失败模式。两个被命名的风险，都来自 Osmani 的文章：

* **理解债务（comprehension debt）。** 循环越快地产出你没写过的代码，仓库里实际存在的东西和你理解的东西之间的距离就越大。最疼的账单不是 token 账单。是你不得不调试一个团队里没人读过的系统的那天。就像技术债务一样，它不会立刻发作，但利息会越滚越高。
* **认知投降（cognitive surrender）。** 一种让你放弃独立思考、对循环返回的结果照单全收的心理惯性。带着判断力设计循环是解药；为了逃避思考而设计循环是加速器。**同一个动作，相反的结果。**

缓解方案不是技术性的：

* **读 diff。** 如果你不读循环产出的东西，你就是在以复利租金租借理解债务。
* **抽查门禁。** 挑几个循环开启的 PR，验证批准它们的测试是否真的抓住了你在乎的失败模式。门禁会腐烂。
* **禁止循环碰架构工作。** 让它留在小的、机器可检查的变更上。一旦让它碰需要判断的事，理解债务就会加速。
* **和队友结对设计循环。** 设计循环时多一双眼睛，能抓住否则循环会永远利用的盲点。

## 14. 安全税。无人看管的循环就是无人看管的攻击面。

循环无人看管地运行，攻击面也在无人看管地运行。

你的循环需要防御的威胁模型：

* **生成的代码未经评审就上线。** 循环开启 PR 的速度比人类阅读还快。没有包含安全检查（SAST、依赖审计、密钥扫描）的门禁，不安全代码自动合并。
* **Skill 作为注入向量。** 自动安装 skill 的循环会继承藏在 skill 描述里的所有提示词注入。安装前审计 skill 来源。
* **日志中的凭证。** 长期运行循环中的调试日志把密钥撒在你不监控的日志里。生产循环禁用详细日志；对必须记录的做脱敏处理。
* **权限范围蔓延。** 一个以只读权限测试的循环为了图方便"就加一次"写权限，然后永远不再审计。每 30 天重新审计权限。

## § 让循环沦为烧钱黑洞的失误

* **没跑四条件测试就搭循环。** 第二步存在是有原因的。大多数开发者至少不满足一个条件。
* **没有客观门禁。** 让第二个智能体"评审"但没有测试、类型检查或构建，只是第二个乐观主义者。
* **一个智能体既写又验。** 自我偏好偏差。制作者给自己打分，永远是"A+"。
* **没有状态文件。** 明天的运行从零重启，而不是恢复。
* **模糊的停止条件。** "看起来好了就停"永远靠不住。用测试、类型通过或构建通过。
* **没有 token 预算上限。** 循环重复读取上下文和重试。没有上限，雄心勃勃的循环会烧掉你预期 5-10 倍的 token。
* **在消费级套餐上运行重度验证循环。** Token 账单或速率限制，总有一个会找上你。
* **自动安装社区 skill。** 17,022 个被审计的 skill 里有 520 个泄露凭证。安装前读源码。
* **把循环用在需要判断的工作上。** 架构、认证、支付、模糊的产品决策。让循环干代码规范检查与自动修复的活，别碰需要拍板的战略决策。
* **不读 diff。** 理解债务以复利累积。当你不得不调试一个团队里没人读过的系统时，代价远超所有 token 费用之和。

## 结论：

## 杠杆点转移了。你的工作也随之转移。

两年来，与编码智能体协作的杠杆点在提示词上。更好的提示词、更好的上下文、更好的一次性输出。

**那个阶段正在结束。** 智能体已经足够好，下一个杠杆点在上一层楼：决定它们做什么、什么时候做、用什么门禁、什么状态在运行之间存活。

但这个故事的诚实版本不是每个人都该急着去搭循环。**大多数开发者暂时还不需要**——直到任务重复出现、验证自动化、预算能承受浪费、智能体拥有资深工程师的工具。

漏掉一个条件，循环的成本就会超过收益。

如果你通过了测试，从小搭起。**一个自动化。一个 skill。一个状态文件。一个门禁。** 先让手动运行可靠。把它变成 skill。包进循环。然后调度它。顺序很重要。跳过步骤，你就是在为一个没人理解的系统买单。

Cherny 的观点不是说工作变简单了。而是说杠杆点转移了。**搭好循环。守住工程师的本分。**
