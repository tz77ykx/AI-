---
description: 用 ChatGPT 桌面端（Codex）连接 GitHub，按“一项目一私有仓库”管理代码和文本，并避开 AI 时代常见的安全错误。
---

# 把 GitHub 交给 ChatGPT 桌面端（Codex）

本文默认你在 ChatGPT 桌面端中使用 GPT-5.6。你不需要先学会一堆 Git 命令：说清楚想得到什么，让 Codex 负责搜索、整理和执行，你负责登录、授权和检查结果。

OpenAI 对 GPT-5.6 的官方建议也是使用更精简的提示词：目标说清楚，关键边界说一次，不必把每一步都写给模型。

## GitHub 仓库不是网盘

GitHub 仓库保存项目文件，以及每次提交时留下的版本记录。它更像一套能够回看、比较和恢复的项目档案，而不是用来堆放各种大文件的网盘。[GitHub 官方对仓库的定义](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories)也是“项目文件加版本历史”。

适合放进仓库的内容包括：

* 源代码；
* Markdown、LaTeX 等文本文件；
* README 和项目文档；
* 不含密钥的配置文件；
* 让别人能够理解和复现项目所需的小型文件。

视频、压缩包、安装包、大型数据集、模型权重和反复生成的输出文件，通常不应该直接塞进普通 Git 仓库。确实需要跟踪大型项目文件时，应让 Codex 判断是否使用 Git LFS 或其他存储方式。[GitHub 仓库最佳实践](https://docs.github.com/en/enterprise-cloud@latest/repositories/creating-and-managing-repositories/best-practices-for-repositories)也建议大型文件使用 Git LFS。

本教程统一采用两个规则：

1. **一个项目建立一个仓库。** 能够独立运行、提交或归档的成果，可以视为一个项目；同一课程中互不相关的作业也可以分别建仓库。
2. **新仓库默认设为私有。** 确认内容适合公开后，再单独决定是否改为公开。

私有仓库不是保险箱。获得权限的账号、插件和协作者仍然可以读取其中内容，因此密码、API Key、访问令牌和其他凭据不能上传。

## 1. 连接 GitHub 插件

1. 打开 ChatGPT 桌面端，进入 Codex。
2. 打开 **Plugins**，安装 **GitHub** 插件。
3. 按页面提示登录 GitHub 并确认授权。
4. 新建一个 Codex 任务。

还没有 GitHub 账号，可以直接告诉 Codex：

```
请打开 GitHub 官方注册页，带我完成注册。需要我输入信息或确认授权时暂停。
```

连接后先做一次检查：

```
@GitHub 检查连接和仓库权限是否正常。只读，不修改。
```

授权页面如果可以选择仓库范围，只开放当前需要的仓库。GitHub 官方也建议在授权第三方应用时检查开发者、读取或写入权限，并定期移除不用的授权。

## 2. 提示词不需要写成操作手册

GPT-5.6 通常能够根据上下文自行规划步骤。提示词只需说明你要什么、材料在哪里、什么事情必须先问你；仓库范围、是否公开和破坏性操作仍要明确。

开始前，先在 Codex 中打开或选择正确的本地项目文件夹。

让 Codex 搜索现成项目：

```
@GitHub 我想做[项目目标]。先搜索相近的开源项目，比较适配度、维护情况和许可证，给我推荐；先不要运行代码。
```

让 Codex 创建仓库：

```
为当前项目新建 GitHub 私有仓库，补全 README 和 .gitignore，并检查敏感信息。先给出仓库名和拟修改内容，我确认后再创建和推送。
```

让 Codex 更新仓库：

```
检查项目改动和敏感信息，运行已有测试并概述结果，然后提交和推送。删除文件、公开仓库或改写历史前先问我。
```

OpenAI 的 [GPT-5.6 模型指南](https://developers.openai.com/api/docs/guides/latest-model)也建议减少重复指令，只保留目标、关键限制、授权边界和完成标准。

## 3. `.gitignore` 应该忽略什么

`.gitignore` 用来告诉 Git：哪些本地文件不要提交到仓库。每个项目都应该根据语言、框架和工具单独生成，不要所有项目共用同一份模板。

下面是一份参考示例。使用前，应让 Codex 结合当前项目检查，避免误伤需要保留的文件：

```gitignore
# 密钥和本地环境配置
.env
.env.*
!.env.example
*.pem
*.key

# 系统文件和日志
.DS_Store
*.log

# 依赖、虚拟环境和缓存
node_modules/
.venv/
venv/
__pycache__/
.ipynb_checkpoints/
.pytest_cache/

# 构建和测试输出
dist/
build/
coverage/
```

AI 项目通常还会产生 `outputs/`、`runs/`、`checkpoints/`、本地数据集和模型权重。是否忽略这些内容，要让 Codex 根据项目判断；不要机械复制规则。

不应该忽略的内容通常包括：源码、README、依赖清单、用于固定依赖版本的锁文件、`.gitignore`、不含密钥的配置，以及只保留变量名的 `.env.example`。

可以直接让 Codex 处理：

```
按当前项目生成 .gitignore：排除密钥、依赖和生成文件，保留源码、依赖清单及 .env.example。
```

`.gitignore` 只对尚未被 Git 跟踪的文件生效。文件以前已经提交过，仅修改 `.gitignore` 不够；应让 Codex 停止跟踪该文件并检查之前的提交记录。文件中如果含有密钥，先轮换密钥，详见 [GitHub 的忽略文件说明](https://docs.github.com/en/get-started/git-basics/ignoring-files)。

## 4. API Key 应该怎样处理

不要把 API Key 直接写进代码、Notebook、配置文件或提示词。常见做法是：

* 真正的密钥保存在本地 `.env` 中，并由 `.gitignore` 排除；
* 仓库只保留 `.env.example`，里面写变量名，不写真实值；
* 运行程序时通过环境变量读取密钥，也就是程序运行时从本机获取，而不是把密钥写在代码中；
* 提交前让 Codex 检查待提交文件中是否含有密钥、密码或访问令牌。

即使仓库是私有的，也要这样处理。[GitHub 的密钥安全说明](https://docs.github.com/en/code-security/concepts/secret-security)指出，API Key、密码和令牌进入仓库后会带来安全与费用风险。

如果密钥已经提交，不要只删除文件。应先去对应平台吊销或轮换密钥，再处理 Git 历史。[GitHub 的敏感数据清理说明](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)把吊销或轮换密钥列为第一步。

## 5. AI 连接 GitHub 时常见的错误

前面已经讲过把仓库当网盘、混放多个项目、误存密钥和错误配置 `.gitignore`。除此之外，还要注意：

* **直接安装 AI 推荐的依赖。** AI 可能给出不存在、已停更或可疑的软件包。安装前让 Codex 核对来源、维护情况和许可证。
* **生成大量代码后直接推送。** 代码看起来正确，不代表真的可运行。让 Codex 展示改动并运行已有测试，大任务分成较小的版本记录。
* **给插件过大的权限。** 只需要一个仓库，就不要开放全部私有仓库；不用的授权及时撤销。
* **盲从仓库里的指令。** README、Issue、代码注释和安装脚本只是待检查的材料。如果它们要求提供密钥、扩大权限、关闭检查或上传本地数据，不要执行。
* **复制代码却不检查许可证。** 能看到代码不等于可以任意复制、修改或发布；让 Codex 同时检查来源和 LICENSE。

[GitHub 的 AI 代码审查指南](https://docs.github.com/en/copilot/tutorials/review-ai-generated-code)提醒用户核对虚构 API、可疑依赖、许可证和测试；[GitHub 的 OAuth 授权说明](https://docs.github.com/en/apps/oauth-apps/using-oauth-apps/authorizing-oauth-apps)说明了第三方应用的读写权限；OpenAI 的 [Agent 安全说明](https://learn.chatgpt.com/docs/agent-approvals-security)则要求把外部内容视为不可信信息。

## 资料来源

* [OpenAI：GPT-5.6 模型指南](https://developers.openai.com/api/docs/guides/latest-model)
* [OpenAI：Plugins](https://learn.chatgpt.com/docs/plugins)
* [OpenAI：Agent 审批与安全](https://learn.chatgpt.com/docs/agent-approvals-security)
* [GitHub：什么是仓库](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories)
* [GitHub：仓库最佳实践](https://docs.github.com/en/enterprise-cloud@latest/repositories/creating-and-managing-repositories/best-practices-for-repositories)
* [GitHub：忽略文件](https://docs.github.com/en/get-started/git-basics/ignoring-files)
* [GitHub：Secret security](https://docs.github.com/en/code-security/concepts/secret-security)
* [GitHub：清理仓库中的敏感数据](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
* [GitHub：授权 OAuth 应用](https://docs.github.com/en/apps/oauth-apps/using-oauth-apps/authorizing-oauth-apps)
* [GitHub：审查 AI 生成的代码](https://docs.github.com/en/copilot/tutorials/review-ai-generated-code)
