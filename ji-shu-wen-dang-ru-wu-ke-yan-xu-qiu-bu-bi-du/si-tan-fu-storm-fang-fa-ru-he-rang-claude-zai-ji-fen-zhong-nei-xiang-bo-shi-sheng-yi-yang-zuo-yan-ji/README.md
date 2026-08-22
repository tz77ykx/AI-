# 斯坦福 STORM 方法：如何让 Claude 在几分钟内像博士生一样做研究（中文）

斯坦福开发了一套名为 STORM 的研究系统。在同行评审测试中，它产出的文章组织性比次优方法高出 **25%**。它是开源的。它是免费的。以下是完整方法。

![图像](https://pbs.twimg.com/media/HLASr6ObwAAG3do?format=jpg\&name=large)

## 第一阶段：STORM 到底是什么

STORM 的全称是 **Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking（通过检索与多视角提问合成主题大纲）**，由斯坦福 OVAL 实验室在 NAACL 2024 上发表。

你可以在这里体验在线版本：[storm.genie.stanford.edu](https://storm.genie.stanford.edu/)。免费。无需注册。输入一个主题，看着它在你面前生成一篇带引用来源的文章。

这里有一段 12 分钟的演示视频：YouTube 上的 STORM by Stanford。值得看一遍。

完整代码在 [github.com/stanford-oval/storm](https://github.com/stanford-oval/storm)。MIT 许可证。如果你想，也可以在自己的笔记本上运行。

但这里才是真正的宝藏。**你其实什么都不需要。** 斯坦福的方法本质上是一种思维方式。你可以用 4 个复制粘贴的提示词，在 Claude 中运行同样的思维过程。

这就是本文剩余部分要讲的内容。

![图像](https://pbs.twimg.com/media/HLATJkVbAAAp8aA?format=jpg\&name=large)

## 第二阶段：为什么一个提示词永远不够

当你问 Claude"告诉我关于 X 的事"，你得到的是主流观点。最常见的框架。最表面的东西。

你得不到的是每天与 X 打交道的从业者。认为这个领域有问题的怀疑者。追踪资金流向的经济学家。见过类似模式的历史学家。真正读过文献的学者。

这五种视角看到的东西截然不同。这正是博士生所做的。他们不会只问一个问题。他们会问五个。

斯坦福的论文用数据证明了这一点。基于多视角构建的文章，在组织性上比普通方法高出 **25%**，覆盖广度高出 **10%**。这就是整个突破所在。多视角提问能捕捉到单一提示词研究永远无法发现的盲点。

一个博士级别的研究工作需要 40 到 60 小时的人工阅读。大多数人抽不出这么多时间。STORM 将其压缩。下面的四个提示词进一步压缩。总共只需 5 分钟。

## 第三阶段：提示词 1，多视角扫描

这是整个方法的核心。把这个粘贴到 Claude 里。把第一行的主题替换成你的。

```
I need to research [YOUR TOPIC].
Simulate 5 different expert perspectives on this topic:
1. THE PRACTITIONER: works with this daily.
What do they know that academics miss?
What practical realities are usually ignored?
2. THE ACADEMIC: has studied this for years.
What does the peer reviewed evidence actually say?
Where does the evidence contradict popular belief?
3. THE SKEPTIC: thinks the mainstream view is wrong.
What is the strongest counterargument?
What evidence do proponents conveniently ignore?
4. THE ECONOMIST: follows the money.
Who profits from the current narrative?
What financial incentives shape the research?
5. THE HISTORIAN: has seen similar patterns before.
What historical parallels exist?
What can we learn from how those played out?
For each perspective give me:
- Their core position in 2 sentences
- The strongest evidence supporting their view
- The one thing they would tell me that no other perspective would
```

**你会得到什么：** 同一个主题的五种截然不同的解读。从业者看到了学者忽略的盲区，怀疑者戳破了从业者默认的成见，经济学家揭开了学者视而不见的利益链条，历史学家则提供了经济学家看不到的深层模式。

这是 60 秒的工作，却能捕捉到单一提示词永远找不到的东西。

## 第四阶段：提示词 2，矛盾映射图

现在让 Claude 找出这 5 种声音在哪里发生冲突。冲突之处，方见真章。

```
Based on the 5 perspectives above, map the contradictions:
1. Where do two or more perspectives directly contradict
   each other? List each conflict with the specific claims
   that clash.
2. Which perspective has the strongest evidence?
   Which has the weakest? Why?
3. What is the one question that, if answered, would
   resolve the biggest contradiction?
4. What does EVERY perspective agree on?
   (This is likely true. Even opponents confirm it.)
5. What topic did NONE of the perspectives address?
   (This is the blind spot in the whole field.
   Often the most valuable finding.)
```

**你会得到什么：** 一张专家们在何处、为何产生分歧的地图。大多数人跳过这一步。而这一步，正是表面理解与真正专业知识之间的分水岭。

如果 5 种视角都同意，那它很可能是真的。如果没人提到某个话题，你就发现了整个领域的盲点。

![图像](https://pbs.twimg.com/media/HLAUxmEbcAAeZF0?format=jpg\&name=large)

## 第五阶段：提示词 3，综合整合

现在让 Claude 把所有内容整合成一份研究简报。

```
Synthesize everything from the 5 perspectives and the
contradiction map into a research briefing:
1. THE ONE PARAGRAPH SUMMARY: explain this topic as if
   briefing a CEO who has 60 seconds and needs nuance,
   not just the headline.
2. THE 5 KEY FINDINGS: most important things I now know,
   ranked by reliability. For each, note which perspectives
   support it and which challenge it.
3. THE HIDDEN CONNECTION: one non obvious link between
   findings that only shows up when you look at all 5
   perspectives together.
4. THE ACTIONABLE INSIGHT: based on all the evidence,
   what should someone in [YOUR ROLE] actually DO
   differently? Be specific.
5. THE FRONTIER QUESTION: the one question that, if
   answered, would change everything about how we
   understand this topic.
```

**你会得到什么：** 一份任何单一专家都写不出的简报。它涵盖了每一个角度，标明了矛盾，排出了可靠性等级，并得出了具体的行动建议。这是博士生花 48 小时才能产出的东西。你只用了 90 秒。

## 第六阶段：提示词 4，同行评审

STORM 有一个已知的弱点。斯坦福的研究人员自己指出来了。系统不会自我批判。**来源偏见**和**事实错配**会悄悄潜入。这个提示词通过让 Claude 给自己的工作打分来解决这个问题。

```
Now peer review your own research briefing:
1. CONFIDENCE SCORES: rate each of the 5 key findings
   on a 1 to 10 scale for reliability. Explain each score.
2. WEAKEST LINK: which claim are you least confident in?
   What specific info would you need to verify it?
3. BIAS CHECK: which perspective might be overrepresented
   in your synthesis? Did one voice dominate?
4. MISSING PERSPECTIVE: is there a 6th angle I should
   have included that would change the conclusions?
5. OVERALL GRADE: if a Stanford professor reviewed this
   briefing, what grade would they give and why?
   What would they tell me to fix?
```

**你会得到什么：** 对自己研究的一次诚实审视。强主张、弱主张、偏见、缺失的角度。真正的同行评审需要几个月。你只用了 60 秒。

![图像](https://pbs.twimg.com/media/HLAVPjsaUAAO97i?format=jpg\&name=large)

## 第七阶段：5 分钟工作流

**第 1 分钟：** 提示词 1。你有了 5 个专家视角。

**第 2-3 分钟：** 提示词 2。你有了矛盾映射图。

**第 3-4 分钟：** 提示词 3。你有了研究简报。

**第 5 分钟：** 提示词 4。你知道什么是可靠的，什么不是。

总耗时：5 分钟。产出：一份包含多视角分析、矛盾分析、综合整合、具体行动建议和可靠性评分的简报。

一个博士生亲自完成这些需要 40 到 60 小时。不是因为他们慢。而是因为从一个视角阅读、绘制矛盾图、综合整合、自我批判，对人类大脑来说确实是 40 小时的工作量。

![图像](https://pbs.twimg.com/media/HLAVu_bacAA1ebw?format=jpg\&name=large)

## 第八阶段：今天就能开始用的 7 种场景

**1. 在撰写任何文章或报告之前。** 运行这 4 个提示词。你的文章将覆盖别人想不到的角度。

**2. 在做出重大商业决策之前。** 获取全部 5 个视角。从业者告诉你现实中什么有效。怀疑者告诉你什么可能出错。经济学家告诉你谁在获利。

**3. 在面试之前。** 用 5 分钟从 5 个角度研究公司。从业者视角给你内部语言。怀疑者视角给你尖锐的问题。你走进房间时，准备得比任何人都更充分。

**4. 在投资之前。** 看涨理由、看跌理由、历史类比、激励图谱、学术证据。5 分钟内搞定。矛盾映射图显示真正的风险在哪里。

**5. 在学习新技能之前。** 从 5 个角度绘制领域地图。从业者告诉你先学什么。学者告诉你理论。怀疑者告诉你什么被过度炒作。你跳过噪音。

**6. 在谈判之前。** 从 5 个角度研究对方。了解他们的激励、弱点、历史行为。你带着结构性优势走进房间。

**7. 在任何演讲之前。** 对主题运行 STORM。你的幻灯片会在观众提出异议之前就回答它们。你的问答环节将毫不费力。

## 读者画像

你是一个阅读者。你问尖锐的问题。你不想要那种听起来聪明但什么都没说的 200 字摘要。

你想真正理解事物。快速地。有来源地。像斯坦福研究生那样。不用交六年学费。

这就是为你准备的。

如果这就是你，收藏这篇文章。使用这 4 个提示词。看看差别。

![图像](https://pbs.twimg.com/media/HLAWJ4UagAA69p6?format=jpg\&name=large)

## 一个不愿被说出的真相

这里有些话没人愿意大声说出来。

斯坦福团队在 2024 年发表了这项研究。论文经过了同行评审。代码是开源的。在线工具是免费的。方法只有四个提示词。而几乎没人用它。

我们正处于一个 18 个月的窗口期。学会用 AI 正确做研究的人，将比不会的人思考得更深入。差距会很大。不是因为他们更聪明。而是因为当别人还在看第一个 Google 搜索结果时，他们已经运行了 5 个视角、一张矛盾映射图、一次综合整合和一次同行评审。

18 个月后，这种工作流将被内置到每一个工具中。先发优势将荡然无存。今天，它仍然是一个藏在明处的秘密。

选出你最需要研究的主题。打开 Claude。粘贴提示词 1。

5 分钟后，你将比那些花几天阅读的人知道得更多。

提示词在上面。斯坦福证明了方法。剩下的就看你了。

希望这对你有用。Nav
