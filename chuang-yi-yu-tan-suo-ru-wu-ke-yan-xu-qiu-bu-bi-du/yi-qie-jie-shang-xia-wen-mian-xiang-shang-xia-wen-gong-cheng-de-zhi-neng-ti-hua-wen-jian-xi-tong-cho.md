# 一切皆上下文：面向上下文工程的智能体化文件系统抽象

作者：Xiwei Xu；Robert Mao；Quan Bai；Xuewu Gu；Yechao Li；Liming Zhu

### 摘要

生成式 AI（Generative AI, GenAI）正在重塑软件系统设计。它把基础模型引入系统，作为预训练子系统，进而改变架构与运行方式。当前出现的关键挑战，已经不再只是模型微调，而是上下文工程（Context Engineering）：系统如何捕获、结构化并治理外部知识、记忆、工具和人类输入，从而支撑可信推理。现有实践，如提示工程、检索增强生成（RAG）和工具集成，仍然是碎片化的，产生的上下文产物往往短暂存在，限制了可追溯性与问责性。

本文提出一种面向上下文工程的文件系统抽象，其灵感来自 Unix 中“一切皆文件”的思想。该抽象通过统一挂载、元数据和访问控制，为管理异构上下文产物提供持久、可治理的基础设施。本文在开源 AIGNE 框架中实现了这一架构，形成一条可验证的上下文工程流水线，包括上下文构造器（Context Constructor）、上下文加载器（Loader）和上下文评估器（Evaluator），用于在 token 约束下组装、交付并验证上下文。

随着 GenAI 在决策支持中成为主动协作者，人类也在其中承担策展者、验证者和共同推理者的核心角色。本文提出的架构为可问责、以人为中心的 AI 协作奠定了可复用基础，并通过两个示例加以说明：带记忆的智能体，以及基于模型上下文协议（MCP）的 GitHub 助手。AIGNE 框架中的实现展示了这一架构如何在开发者和工业场景中落地，支撑可验证、可维护、可用于产业实践的 GenAI 系统。

**索引词**：AI 工程、上下文工程、GenAI

### I. 引言

上下文工程正在成为生成式 AI（GenAI）和智能体系统软件架构中的核心议题 \[1]-\[3]。它指的是捕获、结构化并治理外部知识、记忆、工具和人类输入的过程，使大语言模型（LLM）和智能体的推理能够建立在正确的信息、约束与来源脉络之上。与关注单条指令编写的提示工程不同，上下文工程关注的是完整的信息生命周期：选择、检索、过滤、构建、压缩、评估与刷新。其目标是确保 GenAI 系统和智能体能够长期保持一致、高效且可验证。

LangChain 等近期工业框架已经开始把上下文工程表述为智能体架构中的结构化流程 \[3]。上下文工程通常包含四个关键阶段：智能体首先将上下文信息写入共享记忆或存储；随后为特定任务选择最相关的元素；再将所选上下文压缩到模型约束范围内；最后在不同智能体之间隔离最终子集，用于推理。这些工业实践说明，上下文管理正在被视为一个核心架构问题。AutoGen \[4] 以及其他支持工具使用和记忆增强的相关框架中，也能看到类似流水线 \[5]。然而，这些方案仍然偏临时、偏实现驱动。它们缺少一个统一的架构基础，难以保证可追溯性、治理能力，或系统性地处理持续演化的上下文。因此，在这些步骤中生成的上下文产物往往短暂、不透明且不可验证，在工业级 GenAI 系统中引发上下文腐化（context rot）\[6] 与知识漂移 \[7] 等挑战。

GenAI 为 AI 工程带来了新的架构约束 \[8]。基础模型以预训练子系统的形式存在，其有限的 token 窗口会约束推理能力。这种有界工作记忆会沿架构向上传导，影响运行时上下文如何被选择、模块化、压缩和加载。随着 GenAI 在教育、医疗、决策支持等领域成为主动协作者，人类也越来越多地与 AI 共同完成推理和决策任务 \[9], \[10]。但由于上下文感知有限、数据源不断变化，GenAI 系统可能产生不准确或误导性的输出。因此，系统需要架构机制，以可追溯、可验证、理解人类参与的方式，治理持久知识（长期记忆）如何进入有界上下文（短期窗口），并确保人类判断和隐性知识能够嵌入系统不断演化的上下文，用于推理与评估。

本文提出一种文件系统抽象，作为上下文工程的架构基础和阶梯。该抽象受 Unix “一切皆文件”思想启发 \[11]。它提供一个持久、分层、可治理的环境，使记忆、工具、外部知识和人类贡献等异构上下文源能够被统一挂载和访问。在这一基础设施之上，本文进一步把文件系统扩展为上下文工程流水线，在 token 窗口这一明确架构设计约束下，将上下文构建操作化。该流水线执行上下文的选择、压缩和增量流式加载，确保有限的上下文容量能够被高效、透明地使用。

第 II 节回顾 SE4AI 的相关发展，并说明为什么上下文工程需要架构基础。第 III 节介绍文件系统抽象及其作为持久上下文基础设施的角色。第 V 节提出设计约束，并展示基于这些约束构建的上下文工程流水线。第 VI 节详细介绍 AIGNE 中对所提文件系统抽象的实现，并通过两个示例说明。最后，第 VII 节讨论关键挑战、未来研究方向，并给出结论。

### II. 背景与相关工作

智能体化生成式 AI 系统的出现，催生了面向 LLM 的操作系统范式（LLM-as-OS）。这一范式把 LLM 视为一个内核，用于协调上下文、记忆、工具和智能体。AIOS 项目 \[12] 通过调度、资源分配和多智能体系统的记忆管理等类操作系统原语，将这一范式落地 \[13]。近期还有工作进一步扩展了这一视角，提出基于 LLM 的语义文件系统，使自然语言驱动的文件操作与语义索引成为可能 \[14]。MemGPT \[15] 则提出记忆层级，用于协调短期记忆（上下文窗口）和长期记忆（外部存储）。尽管 LLM-as-OS 范式提供了直观的高层概念模型，但它仍缺少一种软件架构抽象，用于说明上下文如何被结构化、共享和治理。尤其是，现有实现往往把记忆、检索和工具使用视为彼此独立的组件，而不是统一的基础设施。

与此同时，上下文工程 \[3], \[16] 已经成为生成式 AI 系统设计中的关键组成部分。传统提示工程把上下文视为一段固定文本，而上下文工程则将其视为一个有生命的、结构化的组合，包含指令、外部知识、工具定义、记忆、系统状态和用户查询。LangChain \[17]、AutoGen \[4] 等框架通过记忆和工具编排的模块化组件提供了部分支持，但它们缺少针对上下文产物的可追溯性、治理和生命周期管理的统一机制。新兴的基于链接的机制 \[1] 把上下文视为相互连接、可发现的资源，凸显出管理这种动态上下文需要统一且可验证的基础设施。近期学术综述 \[2] 也确认当前方法是碎片化的，并指出验证和生命周期支持方面存在缺口。另有研究提出一个用于衔接上下文构建与检索的集成框架 \[18]，但同样指出目前缺少可验证的架构基础。

工业界和开源社区已经逐渐把长期记忆视为智能体系统的关键能力。现有方案大致可以分为两类：一类是基于嵌入的方案，如 mem0 \[19] 和 Letta（原 MemGPT）\[20]；另一类是基于知识图谱（KG）的方案，如 Zep/Graphiti \[21] 和 Cognee \[22]。然而，在这些方案中，上下文治理、访问控制和多智能体共享大多仍然是临时处理。多数框架更关注存储与检索优化，而不是架构可组合性或可验证的可追溯性。

除架构范式外，人机协作的动态关系也受到越来越多关注。在这类场景中，人类和 AI 智能体共同执行推理、评估和决策任务。近期研究表明，当任务需要上下文理解、伦理推理或隐性领域知识时，将人类判断与 AI 推理结合起来可以提升表现 \[23]-\[25]。

本文提出的文件系统抽象与更广义的 LLM-as-OS 范式相一致。它为挂载和管理异构上下文资源提供持久且可治理的基础设施。通过对齐模块化、封装、关注点分离和可追溯性等软件架构核心原则，文件系统把上下文工程从临时实践转化为系统化、可验证、可复用的基础设施。人类角色也被直接嵌入上下文工程架构之中，确保隐性知识和伦理判断成为系统推理与评估不可分割的一部分。

### III. 文件系统作为上下文基础设施

文件系统为 GenAI 系统中的系统化上下文工程提供了基础设施。在这一环境中，智能体和人类专家类似操作系统进程，对挂载的上下文资源执行读、写、搜索等文件式操作。文件系统定义了统一命名空间和一组一致的基础操作，使自主行为体与人类行为体之间能够进行可扩展协调。图 1 展示了所提出基础设施的整体架构。

这一架构抽象与软件工程中成熟的第一性原则相一致。抽象、模块化、封装、关注点分离和可组合性等概念，决定了上下文资源如何被表示、访问和演化。通过应用这些原则，文件系统把异构上下文的复杂性转化为一个结构化、可验证、可扩展的人机协作环境。文件系统把“LLM 即操作系统”（LLM-as-Operating-System）这一范式从隐喻转化为具体的软件架构设计。

**1）抽象。** 文件系统实现了软件工程中的抽象原则，提供统一接口，隐藏底层上下文源的异构性。无论资源是知识图谱、记忆存储，还是由人类策展的笔记，它都通过标准化文件接口来表示。由于文件系统是模式驱动的，REST/OpenAPI 资源、GraphQL 类型、MCP 工具、记忆存储或外部 API 等异构结构都可以自动投射到命名空间中。这避免了集成代码，并把文件系统变成通用的语义接口。智能体因而可以围绕多种上下文类型进行推理，而无需知道其物理格式、存储机制或检索逻辑。

图 1. 文件系统作为上下文工程的统一抽象。

**2）模块化与封装。** 该架构通过把环境分解为可独立管理的上下文资源，实现了模块化。每个资源都作为一个带有明确边界和元数据的挂载组件被封装。这种封装隔离了每个资源的内部逻辑或后端实现，同时只暴露集成所需的最小操作集合。因此，一个组件的变化，例如把关系数据库替换为向量存储，不会传播到系统中的其他组件。这些能力消除了对大量硬编码工具描述的需求，而这些描述原本会挤占模型的 token 窗口。新的上下文源可以像 Unix 文件系统一样动态挂载，使智能体把外部服务、工具或数据库视为统一可寻址空间的一部分。

**3）关注点分离。** 遵循软件工程中的关注点分离原则，文件系统区分数据层、工具层和治理层。非可执行文件，例如 `config.yaml` 或 `experiment_results.csv`，作为数据或知识资源；可执行产物，例如 `analyser.py` 或 `simulate.sh`，则代表主动工具。这种清晰区分确保智能体和人类专家能够正确理解意图与行为，并采用适当的验证和执行策略。关注点分离也延伸到治理层面：访问控制、日志和元数据管理通过专门机制处理，并保持独立于检索或推理的功能逻辑。

**4）可追溯性与可验证性。** 对文件系统的每一次交互，无论由智能体还是人类发起，都会作为事务记录在持久化上下文仓库中。这种机制强制实现可追溯性，使系统能够重构上下文来源脉络，并追究行为责任。结合结构化元数据，这些日志还支持可验证性，使变更、推理步骤和工具调用都能在事后被审计。这确保上下文流水线不仅能正常运行，而且透明、可审计。

**5）可组合性与可演化性。** 文件系统通过为所有挂载资源定义一致的命名空间和可互操作的元数据模式，实现了可组合性。上下文元素可以被组合、查询，或集成到更高层次的推理过程中，而无需额外集成代码。可演化性则通过插件架构实现：全文索引器、向量数据库或知识图谱等新后端可以无缝挂载，而不需要修改其他组件。

除标准文件操作外，该抽象还可以为每个文件或目录关联由元数据定义的动作。这些动作指定智能体可发现的可调用行为，范围包括摘要、验证、同步等分析函数，也包括特定领域转换。动作把每个文件或目录提升为主动节点，使智能体能够直接通过文件系统接口执行工具、转换或服务调用。

### IV. 持久化上下文仓库：历史与记忆生命周期

大语言模型本质上是无状态的：一旦会话结束，所有上下文信息都会丢失。为了在跨会话场景中维持连贯推理，GenAI 系统需要一个外部的持久化记忆仓库，用于捕获、结构化并持续演化上下文。由文件系统支撑的持久化上下文仓库承担了这一角色。它把历史、记忆和暂存区统一到一个连续生命周期中，确保短期和长期上下文知识都保持可访问、可追溯且及时更新。

当交互发生时，原始数据首先被追加到历史（History）中。摘要、嵌入和索引把这些记录转换为适合检索与推理的记忆（Memory）表示。在推理过程中，临时信息会被写入暂存区（Scratchpad）；经过验证后，其中一部分可能被选择性插入记忆，或归档到历史中。这种分层设计确保所有上下文资源都能在智能体与会话之间保持可追溯和可复用。

这些组件在持久性上有所不同：历史是全局且永久的；记忆是智能体特定或会话特定的，持久但可变；暂存区则是临时的，但仍可审计。在模型 token 窗口内动态组装的短期上下文类似工作记忆；相对地，长期上下文必须保存在模型之外，并在需要时被选择性纳入。

图 2. 历史、记忆与暂存区的生命周期。

文件系统通过时间戳、版本控制和访问策略来支持来源脉络。每一次从历史到记忆、或从暂存区到记忆的转换，都会作为可验证的状态转移记录下来。每个产物都携带其创建上下文、所有权和谱系信息，从而可以可验证地重构推理过程。这使仓库不仅是数据存储，也是可追溯性基础设施，并使上下文管理与软件工程中的模块化和可追溯性原则对齐。

#### A. 历史：不可变的事实来源

历史记录用户、智能体和环境之间的所有原始交互。每个输入、输出和中间推理步骤都会以不可变形式记录，并用时间戳、来源和模型版本等元数据进行丰富。历史充当可验证的事实来源。它可以跨越多个智能体和会话，形成可通过文件系统命名空间访问的共享全局数据记录，例如 `/context/history/`。通过保留完整轨迹，系统保存了推理的来源脉络，并支持事后分析、调试和合规验证。

#### B. 记忆：结构化、索引化的视图

从上下文管理角度看，记忆可以沿时间、结构和表示三个维度分类。

* **时间维度**：记忆会持续多久。
* **结构维度**：所存内容的大小或抽象层级，例如 token 级、事实级或摘要级。
* **表示维度**：记忆在内部如何建模，例如原始文本、向量嵌入、结构化三元组或摘要。

虽然短期与长期的区分源自人类认知，但实际 GenAI 系统管理的是一系列记忆类型，它们在持久性与动态性之间取得平衡 \[26]。例如，情节记忆保存任务边界内的摘要，事实记忆保存持久的原子事实。语义记忆或诱导记忆则捕获由聚类或摘要生成的更高层嵌入。每种类型在 GenAI 推理过程中承担互补角色，从短暂推理支持到持久知识保存不等，并通过一致的命名空间层级暴露。表 I 展示了多种记忆类型如何共存。

记忆条目是智能体特定的，并通过共享元数据和访问控制规则治理。每个记忆项都会保留对其历史来源的引用，从而确保摘要数据与原始数据之间的可追溯性。索引日志和嵌入使系统不必重新扫描完整历史，也能进行选择性回忆，从而支持性能与可扩展性。在文件系统中，记忆以 `/context/memory/agentID` 的形式暴露，并可通过向量数据库或全文搜索引擎等插件扩展。

#### C. 暂存区：临时工作空间

暂存区是智能体在推理过程中组织中间假设、计算或草稿的临时工作空间。与记忆不同，暂存区是短暂的，并限定在特定任务或推理片段内。不过，一旦会话结束，相关产物可以被插入记忆或追加到历史中，从而闭合循环。暂存区在文件系统中表示为 `/context/pad/taskID`，并由与持久产物相同的元数据和访问控制模式治理。

#### D. 治理

历史、记忆和暂存区的生命周期由显式策略治理，包括版本控制、老化和保留策略。例如，过时暂存区可以被自动剪枝，而历史日志可以被压缩但不得删除。这些策略确保系统既可扩展又可审计。持久化上下文仓库作为文件系统中的一个挂载层运行，使用其分层命名空间和访问控制机制。它也是上下文工程流水线的主要数据源：选中的记忆与历史产物会被检索、压缩，并注入受 token 窗口约束的上下文中。所有状态转移和转换都以带时间戳和谱系元数据的文件级事件表示，从而支持重放、审计和可逆演化。

### V. 上下文工程流水线

#### A. 设计约束

GenAI 模型引入了一组独特的架构设计约束。这些约束共同构成上下文工程流水线设计的依据，并从根本上影响上下文如何被操作。它们内生于 GenAI 模型层，并沿软件架构向上传导，影响高层组件的结构与行为。识别并形式化这些设计约束，可以把上下文工程从临时提示实践转变为系统性的软件架构学科。

**1）Token 窗口。** GenAI 模型的 token 窗口是一项硬性架构约束，它定义了模型在一次推理过程中能够关注的最大 token 数。这个由模型架构决定的有界推理容量，为运行时可用的主动上下文数量设置了上限，例如 GPT-5 的 128K token、Claude Sonnet 4.5 的 200K token。此外，随着输入提示长度增加，由于自注意力机制具有二次复杂度，GenAI 模型的计算成本会显著上升 \[27]。

表 I. 上下文工程中的记忆类型分类

| 记忆类型  | 时间范围     | 结构单元        | 表示方式      |
| ----- | -------- | ----------- | --------- |
| 暂存区   | 临时、任务边界内 | 对话轮次、临时推理状态 | 纯文本或嵌入    |
| 情节记忆  | 中期、会话边界内 | 会话摘要、案例历史   | 纯文本摘要或嵌入  |
| 事实记忆  | 长期、细粒度   | 原子事实陈述      | 键值对或三元组   |
| 经验记忆  | 长期、跨任务   | 观察-行动轨迹     | 结构化日志或数据库 |
| 程序性记忆 | 长期、系统范围  | 函数、工具或函数定义  | API 或代码引用 |
| 用户记忆  | 长期、个性化   | 用户属性、偏好和历史  | 用户画像、嵌入   |
| 历史记录  | 不可变、完整轨迹 | 所有交互的原始日志   | 带元数据的纯文本  |

因此，上下文工程流水线必须从文件系统中策展、压缩并增量流式加载相关信息进入模型 token 窗口。记忆中的持久信息必须被模块化并以层级方式组织，以支持选择性检索和增量刷新。流水线还要管理主动窗口的时间一致性，确保推理在有界上下文限制内保持一致且可追溯。最简单的缓解方式是截断或摘要大文本，但这不可避免地带来信息损失风险 \[26]。

**2）无状态性。** GenAI 模型本质上是无状态的，不会跨会话保留对话历史或记忆。该约束要求系统具备外部持久化上下文仓库，用于跨交互记录、重构并治理相关信息。无状态性也推动了会话记忆机制的需求，以恢复连续性并避免重复计算。

然而，在外部持久化状态会引入与记忆增长和冗余相关的次级挑战。随着对话或任务历史不断积累，语义相似的条目或重复经验会迅速增加，降低检索精度并提高存储成本。为缓解这些影响，上下文工程流水线需要记忆去重和整合策略，以维护一个冗余最小的记忆库。

**3）非确定性与概率性输出。** LLM 会根据采样参数（如 temperature）产生概率性输出，因此相同提示也可能得到不同响应。从架构角度看，这种非确定性为可追溯性、测试和验证带来挑战。因此，上下文工程流水线必须在文件系统中保存输入-输出对、元数据和来源脉络，以支持审计、重放和事后评估。

图 3. 上下文工程流水线。

#### B. 上下文工程流水线设计

在文件系统所建立的统一架构基础之上，本节提出上下文工程流水线，作为协调各组件之间上下文演化的操作层。

通过长期和短期机制保留上下文的流水线，是自主 GenAI 智能体的关键组成部分 \[28]。这样的流水线保存未嵌入模型权重中的知识和上下文。上下文工程流水线将持久化上下文仓库中的上下文（历史、记忆、工具、人类输入）与有界推理（token 窗口）连接起来，确保在智能体的整个运行生命周期中，上下文能够被持续构建、刷新和评估。

从架构上看，如图 3 所示，该流水线由三个组件组成：上下文构造器、上下文更新器和上下文评估器。架构运行在第 V-A 节讨论的三个相互关联的设计约束之下。流水线执行选择、压缩、注入、刷新，以及人在回路中的评估和覆盖，从而形成上下文管理闭环。文件系统中的元数据被上下文工程流水线中的所有主要操作使用。

**1）上下文构造器。** 构造器定义如何从持久化上下文仓库中选择、排序并压缩相关上下文，为推理准备有界且面向任务的输入。这个过程把无界知识转化为经过策展的子集，使其适合模型的主动上下文窗口。上下文选择还必须满足隐私、访问控制和数据治理等非功能质量。由于文件系统是跨智能体和任务共享的全局基础设施，构造器会强制执行这些约束，确保每个推理会话都在其授权范围内运行，并且相应上下文被适当隔离。

从架构上看，构造器管理的是完整性与有界性之间的权衡：一方面要覆盖所有相关信息，另一方面要遵守 token 约束并保持成本效率。它依赖元数据中的新近性、来源脉络等信息，在检索和优先级排序时推断上下文元素的相关性。选中的上下文随后通过摘要、嵌入或聚类等技术压缩，以满足计算预算，然后再与模型的提示模式对齐 \[29], \[30]。提示模式是一种结构化输入格式，用于规定上下文元素在推理输入中如何组织。

构造器直接与文件系统挂载点交互，例如 `/context/memory/` 和 `/context/tool/`，查询元数据，并生成上下文清单。该清单记录哪些元素被选中、哪些被排除，以及原因。这为每个推理会话提供透明性、可复现性和可验证性，把上下文组装从临时操作转变为可追溯的架构过程。

**2）上下文更新器。** 上下文更新器负责把构建好的上下文传输并刷新到 GenAI 模型的有界推理空间中。由于模型 token 窗口有限，更新器必须持续同步 token 窗口、持久化上下文仓库状态和运行时对话，以维持连贯性与一致性。它确保主动上下文始终反映最相关且已授权的信息，同时不超过模型限制，也不违反访问和治理约束。这种同步要求持续监测上下文大小、相关性衰减，以及跨智能体和会话的时间依赖与结构依赖。

在流程开始时，可以先向单次推理任务注入一个静态上下文快照。对于扩展推理，增量流式加载允许在推理展开过程中逐步加载额外上下文片段。在动态或交互式会话中，自适应刷新机制会根据模型反馈或人类干预，替换过时或不太相关的片段。这些模式共同确保推理过程始终有上下文支撑。

所有上下文加载与替换动作都会作为元数据事件记录在文件系统中，包括时间戳、源路径和推理标识符，从而支持任意推理会话的完整可追溯性和重放。在多智能体场景中，上下文更新器还会执行资源隔离和访问分离，确保一个推理过程的上下文不会干扰或泄漏到另一个过程。

**3）上下文评估器。** 上下文评估器通过验证模型输出、更新持久化上下文仓库，以及治理不断演化的知识库来闭合循环。它确保新生成或被精炼的信息经过验证、被放回上下文，并以可追溯、可审计的方式重新集成进持久化上下文仓库。

模型输出会与其源上下文元素和来源元数据进行比对，以检测幻觉、矛盾或上下文漂移。这可能涉及自动语义比较、事实一致性检查，或与权威来源交叉核验。置信度分数、事实对齐程度、人类覆盖率等评估指标会作为结构化元数据记录在文件系统中，支持事后分析和可追溯性。

经过验证的输出会被转换为结构化记忆元素，用于更新或扩展持久化上下文仓库。长期记忆条目可以被追加、修订或摘要；情节记忆和暂存区则可以被剪枝或归档。每次更新都带有时间戳和谱系元数据的版本记录，确保上下文演化保持透明且可逆。

当置信度低于阈值或检测到矛盾时，评估器会触发人类审查。人类注释可以从事实修正到解释性洞见不等，它们会作为显式上下文元素存储，从而把隐性知识提升为知识库中的一等组成部分。

### VI. 实现平台：AIGNE 框架

本文提出的文件系统和上下文工程流水线在 AIGNE Framework 中实现。AIGNE 是一个函数式开发框架，旨在简化并加速 GenAI 智能体的创建。AIGNE 原生集成多种主流大语言模型（如 OpenAI、Gemini、Claude、DeepSeek、Ollama），并通过内置的模型上下文协议（MCP）接入外部服务，从而支持动态、具备上下文感知能力的应用行为。

在 AIGNE 框架中，AFS（Agentic File System，智能体化文件系统）模块作为主要文件系统接口。`SystemFS` 模块实现了一个虚拟文件系统，提供以下关键能力：

* 支持 `list`、`read`、`write` 和 `search` 命令，用于管理挂载目录中的文件。
* 支持在嵌套子目录中导航，并可配置深度限制。
* 集成 `ripgrep`，用于高效内容搜索。
* 访问文件时间戳、大小、类型，并支持用户自定义元数据。
* 将沙箱访问限制在挂载目录内，确保隔离与安全的文件操作。

所有挂载资源，包括 MCP 模块、记忆存储、数据库或外部 API，都会通过可编程解析器投射到文件系统中。

这些解析器实现声明式映射，类似 GraphQL/OpenAPI 模式，能够把内部结构转换为 AFS 节点，而无需改变底层存储格式，从而实现异构系统的无缝集成。

在 AIGNE 中，上下文元素以类型化资源的形式表示在 AFS 命名空间下。`SystemFS`、`FSMemory` 和 `UserProfileMemory` 等模块都通过标准异步方法暴露 `list`、`read`、`write`、`search` API。借助这种抽象，GenAI 智能体可以通过统一接口访问本地文件、聊天历史、结构化记忆条目等异构数据，而不必关心底层存储后端。

在 AIGNE 中，智能体执行推理，同时把执行委托给模块化函数（Functions）。这些函数实现为可执行文件，例如 Node.js 模块。每个函数都会导出一个默认异步函数，并带有元数据描述符（`description`、`input_schema` 和 `output_schema`），使智能体能够发现、校验并以结构化参数调用它们。函数是智能体执行具体行动的工具，可以在沙箱中运行代码，也可以调用外部 API。

上下文构造器被实现为一个流程。当收到新提示时，构造器会执行一系列工具调用，例如 `afs_list()` 和 `afs_read()`，收集候选产物，包括文档、历史记录或用户画像摘要。这些产物带有时间戳、来源脉络、访问范围等元数据。随后，构造器应用摘要和 token 预算估算函数，生成 JSON 格式的清单。该清单记录被选中产物、排列顺序，以及它们预计对模型提示的贡献。该清单随后传递给上下文更新器。

上下文更新器作为 AIGNE 智能体工作流引擎的一部分实现。更新器在对话过程中把上下文片段流式送入模型输入缓冲区。在单轮任务中，它一次性注入静态快照；在交互式会话中，它通过调用 AFS 的读操作增量刷新提示，替换或追加元素，使推理继续展开。

上下文评估器利用 AIGNE 的记忆模块来持久化新生成的信息。每次模型响应之后，经过验证的输出，例如被摘要的用户偏好或提取出的事实陈述，都会写回 AFS，并存储在 `/context/memory/fact/` 等目录下。每个条目都会增加谱系元数据，如 `createdAt`、`sourceId`、`confidence` 和 `revisionId`，以支持审计和回滚。当评估器检测到不确定性时，例如置信度低于阈值或信息不一致，它会触发人类验证阶段：注释会作为单独产物追加到 `/context/human/`。

#### A. 示例 1：具备记忆能力的上下文构建

AIGNE 使智能体能够在多轮对话中保持上下文连贯性。记忆在智能体构建时通过 `DefaultMemory` 模块以声明式方式启用，该模块把对话历史持久化为可检索上下文。存储位置被指定为文件路径，例如 `file:./memory.sqlite3`，使记忆数据能够跨会话保存并重新加载。每一轮对话都会追加到记忆中，并自动纳入后续推理，从而在无需显式状态管理的情况下支持长期、有状态的交互。

```ts
import { AIAgent } from "@aigne/core";
import { AFS } from "@aigne/afs";
import { AFSHistory } from "@aigne/afs-history";
import { UserProfileMemory } from "@aigne/afs-user-profile-memory";

const sharedStorage = { url: "file:./memory.sqlite3" }; // 使用 SQLite 数据库作为记忆存储提供者

const afs = new AFS()
  .mount(new AFSHistory({ storage: sharedStorage })) // 消息历史记忆
  .mount(new UserProfileMemory({ storage: sharedStorage, context: aigne.newContext() })); // 用户记忆

const agent = AIAgent.from({
  instructions: "You are a friendly chatbot",
  inputKey: "message",
  afs,
});
```

清单 1. 定义一个具备持久记忆的智能体。

#### B. 示例 2：基于 GitHub 的 MCP

第二个示例展示，任何 MCP（Model Context Protocol，模型上下文协议）服务器都可以作为 AFS 模块挂载，并通过统一文件系统接口暴露其能力。以 GitHub MCP 服务器这一真实场景为例，它展示了 AI 智能体如何像访问文件一样与 GitHub 交互。挂载之后，智能体可以通过 `/modules/github-mcp/search_repositories` 和 `/modules/github-mcp/list_issues`，使用 `afs_exec` 直接调用所有 GitHub MCP 工具。

```ts
import { AIAgent } from "@aigne/core";
import { AFS } from "@aigne/afs";
import { MCPAgent } from "@aigne/core";

const mcpAgent = await MCPAgent.from({ // 从 GitHub 官方 MCP Server 创建智能体
  command: "docker",
  args: [
    "run", "-i", "--rm",
    "-e", `GITHUB_PERSONAL_ACCESS_TOKEN=${process.env.GITHUB_PERSONAL_ACCESS_TOKEN}`,
    "ghcr.io/github/github-mcp",
  ],
});

const afs = new AFS()
  .mount(mcpAgent); // 挂载到 /modules/github-mcp

const agent = AIAgent.from({
  instructions: "Help users interact with GitHub via the github-mcp-server module.",
  inputKey: "message",
  afs, // 智能体可访问所有已挂载模块
});
```

清单 2. 附加一个 GitHub MCP 函数。

### VII. 结论与未来工作

本文立足于正在兴起的“LLM 即操作系统”（LLM-as-Operating-System）范式，提出了一种基于文件系统的上下文工程抽象。在这一基础之上，智能体和人类可以像操作系统进程一样交互，执行受元数据和事务日志治理的标准文件操作。AIGNE 框架中的实现及其配套示例，展示了所提方法的可行性和适应性。

把上下文视为文件，还可以使 GenAI 智能体变得可追溯、可审计，让上下文能够像 DevOps 和 DataOps 中的产物一样被版本化、审查和部署，而不是继续依赖临时提示管理。通过把文件系统视为通用上下文投射层，该架构为新兴 LLM-as-OS 范式提供了具体基底，使智能体能够以可验证、与人类目标对齐的方式，导航、组织并演化自己的世界模型。

未来扩展将探索 AFS 层级中的智能体化导航，使智能体能够在挂载空间中自主浏览、构建索引并演化数据结构。当智能体可以作为自组织过程观察并修改自身上下文时，该架构就能逐渐演化为一种活的知识织体。在其中，推理、记忆与行动汇聚于一个可验证、可扩展的文件系统基底。另一个重要方向是强化人机协作，使人类不仅能监督或纠正系统行为，也能作为上下文工程中的主动参与者，贡献、策展并语境化知识。

### 参考文献

\[1] M. Bleigh, Context engineering is sleeping on the humble hyperlink, [https://mbleigh.dev/posts/context-engineering-with-links/](https://mbleigh.dev/posts/context-engineering-with-links/), accessed: 2025-11-01 (October 2025).

\[2] L. Mei, J. Yao, Y. Ge, Y. Wang, B. Bi, Y. Cai, J. Liu, M. Li, Z.-Z. Li, D. Zhang, C. Zhou, J. Mao, T. Xia, J. Guo, S. Liu, A survey of context engineering for large language models (2025). arXiv:2507.13334. URL [https://arxiv.org/abs/2507.13334](https://arxiv.org/abs/2507.13334)

\[3] LangChain, Context engineering for agents, [https://blog.langchain.com/context-engineering-for-agents/](https://blog.langchain.com/context-engineering-for-agents/), accessed: 30 October 2025 (2024).

\[4] Microsoft, Memory and rag — autogen user guide, [https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/memory.html](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/memory.html), accessed: 30 October 2025 (2024).

\[5] Anthropic, Effective context engineering for ai agents, [https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), accessed: 30 October 2025 (2024).

\[6] T. Research, Context rot: How long-term ai memory goes stale, [https://research.trychroma.com/context-rot](https://research.trychroma.com/context-rot), accessed: 30 October 2025 (2024).

\[7] P. Laban, H. Hayashi, Y. Zhou, J. Neville, Llms get lost in multi-turn conversation (2025). arXiv:2505.06120. URL [https://arxiv.org/abs/2505.06120](https://arxiv.org/abs/2505.06120)

\[8] G. A. Lewis, I. Ozkaya, X. Xu, Software architecture challenges for ml systems, in: 2021 IEEE International Conference on Software Maintenance and Evolution (ICSME), 2021, pp. 634-638.

\[9] X. Xu, D. Zhang, Q. Liu, Q. Lu, L. Zhu, Agentic rag with human-in-the-retrieval, in: 2025 IEEE 22nd International Conference on Software Architecture Companion (ICSA-C), 2025, pp. 498-502.

\[10] X. Xu, H. Weytjens, D. Zhang, Q. Lu, I. Weber, L. Zhu, Ragops: Operating and managing retrieval-augmented generation pipelines (2025). arXiv:2506.03401. URL [https://arxiv.org/abs/2506.03401](https://arxiv.org/abs/2506.03401)

\[11] D. M. Ritchie, K. Thompson, The unix time-sharing system, Commun. ACM 17 (7) (1974) 365-375.

\[12] Y. Ge, Y. Ren, W. Hua, S. Xu, J. Tan, Y. Zhang, Llm as os, agents as apps: Envisioning aios, agents and the aios-agent ecosystem (2023). arXiv:2312.03815. URL [https://arxiv.org/abs/2312.03815](https://arxiv.org/abs/2312.03815)

\[13] K. Mei, X. Zhu, W. Xu, W. Hua, M. Jin, Z. Li, S. Xu, R. Ye, Y. Ge, Y. Zhang, Aios: Llm agent operating system (2025). arXiv:2403.16971. URL [https://arxiv.org/abs/2403.16971](https://arxiv.org/abs/2403.16971)

\[14] Z. Shi, K. Mei, M. Jin, Y. Su, C. Zuo, W. Hua, W. Xu, Y. Ren, Z. Liu, M. Du, D. Deng, Y. Zhang, From commands to prompts: Llm-based semantic file system for aios, in: Proceedings of the International Conference on Learning Representations (ICLR), 2025.

\[15] C. Packer, S. Wooders, K. Lin, V. Fang, S. G. Patil, I. Stoica, J. E. Gonzalez, Memgpt: Towards llms as operating systems (2024). arXiv:2310.08560. URL [https://arxiv.org/abs/2310.08560](https://arxiv.org/abs/2310.08560)

\[16] Q. Hua, L. Ye, D. Fu, Y. Xiao, X. Cai, Y. Wu, J. Lin, J. Wang, P. Liu, Context engineering 2.0: The context of context engineering (2025). arXiv:2510.26493. URL [https://arxiv.org/abs/2510.26493](https://arxiv.org/abs/2510.26493)

\[17] LangChain, Deep agents overview, [https://docs.langchain.com/labs/deep-agents/overview](https://docs.langchain.com/labs/deep-agents/overview), accessed: 30 October 2025 (2025).

\[18] S. Dai, J. Tang, J. Wu, K. Wang, Y. Zhu, B. Chen, B. Hong, Y. Zhao, C. Fu, K. Wu, Y. Ni, A. Zeng, W. Wang, X. Chen, J. Xu, S.-K. Ng, Onepiece: Bringing context engineering and reasoning to industrial cascade ranking system (2025). arXiv:2509.18091. URL [https://arxiv.org/abs/2509.18091](https://arxiv.org/abs/2509.18091)

\[19] P. Chhikara, D. Khant, S. Aryan, T. Singh, D. Yadav, Mem0: Building production-ready ai agents with scalable long-term memory, arXiv preprint arXiv:2504.19413 (2025).

\[20] L. AI, Letta (formerly memgpt), [https://github.com/letta-ai/letta](https://github.com/letta-ai/letta), accessed: 30 October 2025 (2025).

\[21] P. Rasmussen, P. Paliychuk, T. Beauvais, J. Ryan, D. Chalef, Zep: A temporal knowledge graph architecture for agent memory (2025). arXiv:2501.13956. URL [https://arxiv.org/abs/2501.13956](https://arxiv.org/abs/2501.13956)

\[22] V. Markovic, L. Obradovic, L. Hajdu, J. Pavlovic, Optimizing the interface between knowledge graphs and llms for complex reasoning (2025). arXiv:2505.24478. URL [https://arxiv.org/abs/2505.24478](https://arxiv.org/abs/2505.24478)

\[23] X. Xu, D. Zhang, W. Zhang, Q. Lu, L. Zhu, Design process for retrieval augmented generation systems, in: 2025 IEEE 22nd International Conference on Software Architecture Companion (ICSA-C), 2025, pp. 482-487.

\[24] D. Lindner, I. Rahwan, S. Schubert, R. H. J. M. Kurvers, When combinations of humans and ai are useful: A systematic review and meta-analysis, Nature Human Behaviour 8 (8) (2024) 1631-1643. doi:10.1038/s41562-024-02024-1.

\[25] S. Amershi, D. Weld, M. Vorvoreanu, A. Fourney, B. Nushi, P. Collisson, J. Suh, S. Iqbal, P. N. Bennett, K. Inkpen, J. Teevan, R. Kikin-Gil, E. Horvitz, Guidelines for human-ai interaction, in: Proceedings of the 2019 CHI Conference on Human Factors in Computing Systems, CHI2019, New York, NY, USA, 2019, p. 1-13.

\[26] X. Wang, M. Salmani, P. Omidi, X. Ren, M. Rezagholizadeh, A. Eshaghi, Beyond the limits: A survey of techniques to extend the context length in large language models (2024). arXiv:2402.02244. URL [https://arxiv.org/abs/2402.02244](https://arxiv.org/abs/2402.02244)

\[27] F. Duman Keles, P. M. Wijewardena, C. Hegde, On the computational complexity of self-attention, in: S. Agrawal, F. Orabona (Eds.), Proceedings of The 34th International Conference on Algorithmic Learning Theory, Vol. 201 of Proceedings of Machine Learning Research, PMLR, 2023, pp. 597-619.

\[28] V. de Lamo Castrillo, H. K. Gidey, A. Lenz, A. Knoll, Fundamentals of building autonomous llm agents (2025). arXiv:2510.09244. URL [https://arxiv.org/abs/2510.09244](https://arxiv.org/abs/2510.09244)

\[29] J. White, Q. Fu, S. Hays, M. Sandborn, C. Olea, H. Gilbert, A. Elnashar, J. Spencer-Smith, D. C. Schmidt, A prompt pattern catalog to enhance prompt engineering with chatgpt (2023). arXiv:2302.11382. URL [https://arxiv.org/abs/2302.11382](https://arxiv.org/abs/2302.11382)

\[30] P. Liu, W. Yuan, J. Fu, Z. Jiang, H. Hayashi, G. Neubig, Pre-train, prompt, and predict: A systematic survey of prompting methods in natural language processing, ACM Comput. Surv. 55 (9) (Jan. 2023).
