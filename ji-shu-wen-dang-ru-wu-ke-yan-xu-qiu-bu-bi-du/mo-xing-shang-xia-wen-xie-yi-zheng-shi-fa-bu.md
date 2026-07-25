# 模型上下文协议正式发布

## 模型上下文协议正式发布

发布于 2024 年 11 月 25 日

![一幅关于关键上下文连接到中心枢纽的抽象插图](https://cdn.sanity.io/images/4zrzovbb/website/3aabd8804251c0364cbde9d2e4be6dc8e8c2faec-2880x1620.png)

今天，我们正式开源 [Model Context Protocol](https://modelcontextprotocol.io)（MCP，模型上下文协议），一项用于将 AI 助手连接到数据所在系统的新标准，包括内容仓库、业务工具和开发环境。其目标是帮助前沿模型（frontier models）生成更好、更相关的响应。

随着 AI 助手走向主流，业界在模型能力上投入了大量资源，在推理和质量上取得了长足进步。然而，即使是最先进的模型，也受到其与数据隔离的限制——被困在信息孤岛（data silos）和遗留系统（legacy systems）之后。每一个新的数据源都需要自己的定制实现，这使得真正互联的系统难以扩展。

MCP 正是为了解决这一挑战。它提供了一种通用的开放标准，用于将 AI 系统与数据源连接起来，用单一协议取代碎片化的集成。由此，为 AI 系统提供了一种更简单、更可靠的数据访问方式。

### 模型上下文协议

模型上下文协议（Model Context Protocol, MCP）是一项开放标准，使开发者能够在其数据源与 AI 驱动的工具之间建立安全、双向的连接。其架构非常直观：开发者既可以通过 MCP 服务器（MCP servers）暴露自己的数据，也可以构建连接到这些服务器的 AI 应用（MCP 客户端，MCP clients）。

今天，我们为开发者推出模型上下文协议的三大核心组件：

* 模型上下文协议的[规范与 SDK](https://github.com/modelcontextprotocol)
* [Claude Desktop 应用](https://claude.ai/download)中对本地 MCP 服务器的支持
* 一个 MCP 服务器的[开源仓库](https://github.com/modelcontextprotocol/servers)

Claude 3.5 Sonnet 擅长快速构建 MCP 服务器实现，使组织和个人能够轻松将其最重要的数据集与各种 AI 驱动的工具快速连接起来。为了帮助开发者开始探索，我们共享了面向 Google Drive、Slack、GitHub、Git、Postgres 和 Puppeteer 等流行企业系统的预构建 MCP 服务器。

Block 和 Apollo 等早期采用者已将 MCP 集成到其系统中，而 Zed、Replit、Codeium 和 Sourcegraph 等开发工具公司也正在与 MCP 合作，以增强其平台的能力——使 AI 智能体能够更好地检索相关信息，从而进一步理解编码任务的上下文，并在更少的尝试中生成更细致、更实用的代码。

“在 Block，开源不仅仅是一种开发模式，更是我们工作的基础，也是我们致力于创造能够带来有意义变革、并作为公共利益服务所有人的技术的承诺，”Block 首席技术官 Dhanji R. Prasanna 表示。“像模型上下文协议这样的开放技术，是连接 AI 与现实应用的桥梁，确保创新是可获得、透明且根植于协作的。我们很兴奋能够参与这一协议，并用它来构建智能体系统（agentic systems），让人们从机械性负担中解放出来，专注于创造性工作。”

开发者无需再为每个数据源维护独立的连接器，而是可以基于一个标准协议进行构建。随着生态系统的成熟，AI 系统将在不同工具和数据集之间保持上下文，用更可持续的架构取代当今碎片化的集成。

### 开始使用

开发者今天就可以开始构建和测试 MCP 连接器。所有 [Claude.ai](https://claude.ai) 套餐都支持将 MCP 服务器连接到 Claude Desktop 应用。

Claude for Work 客户可以开始在本地测试 MCP 服务器，将 Claude 连接到内部系统和数据集。我们很快将提供开发者工具包，用于部署可为整个 Claude for Work 组织服务的远程生产级 MCP 服务器。

开始构建：

* 通过 [Claude Desktop 应用](https://claude.ai/download)安装预构建 MCP 服务器
* 跟随我们的[快速入门指南](https://modelcontextprotocol.io/quickstart)构建你的第一个 MCP 服务器
* 为我们的连接器和实现[开源仓库](https://github.com/modelcontextprotocol)做出贡献

### 开放的社区

MCP 由 Anthropic 的 David Soria Parra 和 Justin Spahr-Summers 创建。我们致力于将 MCP 建设为一个协作式的开源项目和生态系统，并期待听到你的反馈。无论你是 AI 工具开发者、希望利用现有数据的企业，还是探索前沿的早期采用者，我们都邀请你共同打造上下文感知 AI（context-aware AI）的未来。

***

_来源：_[_Introducing the Model Context Protocol_](https://www.anthropic.com/news/model-context-protocol)
