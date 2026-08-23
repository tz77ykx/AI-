---
description: 把本地项目托管到 GitHub，在 ChatGPT 网页版调用 GitHub 插件与 GPT-5.6 Pro 完成仓库级分析和开发，再安全同步远端改动。
---

# ChatGPT 网页版 + GPT-5.6 Pro：用 GitHub 仓库做远端开发

这是一条适合复杂代码任务的补充工作流：**本地项目负责运行与验收，GitHub 负责版本真源和协作，ChatGPT 网页版负责读取仓库、分析代码并在权限允许时推进远端改动。**

OpenAI 将 GPT-5.6 Sol 定位为复杂推理与代码任务的旗舰模型；Pro 模式会在返回最终答案前投入更多模型计算，适合高价值代码开发、审查和深度分析，但通常需要更长等待时间。Pro 是运行模式，不应理解为一个独立模型名称。

## 什么时候适合用

* 需要跨多个文件理解项目结构、定位问题或设计较大改动。
* 任务有清晰验收标准，愿意用更长等待时间换取更充分的分析与验证。
* 项目已经规范使用 Git，能够通过分支、提交或 Pull Request 审阅改动。

以下任务仍应优先留在本地 Codex 或本地开发环境：必须访问本机数据库、内网服务、设备、模拟器、桌面界面，或必须在本机完成构建和视觉验收的任务。GitHub 中的代码快照不能替代真实运行环境。

## 完整流程

### 1. 先把本地项目安全地放到 GitHub

如果项目还不是 Git 仓库，先阅读[《GitHub 新手指南：从注册到协作》](ba-github-jiao-gei-chatgpt-zhuo-mian-duan-codex.md)中的初始化、敏感信息检查和首次推送步骤。

首次推送前至少确认：

* `.env`、API Key、密码、令牌、私钥和个人数据没有进入提交；
* `.gitignore` 已覆盖依赖目录、缓存、构建产物和本地配置；
* 已查看 `git status` 与待提交 diff；
* 项目已有的测试、类型检查和 git hooks 没有被跳过；
* 不确定是否可以公开时，先创建私有仓库。

GitHub 官方文档提醒，不要把密码或 API Key 等敏感信息提交或推送到远端。已经泄露的密钥应先吊销或轮换，不能只靠删除文件解决。

### 2. 在 ChatGPT 网页版连接 GitHub 插件

1. 打开 ChatGPT 网页版的 **Plugins**。
2. 找到并安装 **GitHub**；按提示登录 GitHub。
3. 授权时只开放本次任务需要的仓库。
4. 安装后新建对话，通过 `@GitHub` 明确调用插件。
5. 先做只读检查：让它报告当前可访问仓库、默认分支与关键目录，不修改任何内容。

插件能否读取、提交、创建分支或发起 Pull Request，取决于当前产品界面、插件工具、GitHub 授权范围和账号权限。没有得到工具成功回执时，不要把“已给出代码”当成“已写入仓库”。

### 3. 选择 GPT-5.6 与 Pro 模式

如果当前账号和会话界面提供 GPT-5.6 与 Pro 模式，选择 GPT-5.6，并开启 Pro。对于日常小改、低延迟任务或大量重复任务，标准模式通常更合适。

不要预设 Pro 一定优于所有 Codex 配置。OpenAI 的官方建议是：在相同代表性任务上比较标准模式与 Pro 模式，并同时检查任务成功率、答案完整度、证据、总 token、延迟和成本。

### 4. 用仓库级提示词下达任务

可以直接复制下面的模板，并替换尖括号中的内容：

```
使用 @GitHub 读取 <用户名/仓库名> 的 <目标分支>。

目标：<要实现的功能或修复的问题>
范围：<允许修改的目录或文件>
不要修改：<禁止触碰的目录、接口或行为>
验收标准：
- <可以检查的结果>
- <必须通过的测试或构建命令>

先完成：
1. 阅读 README、AGENTS.md、贡献指南和相关代码。
2. 说明你对现状的理解、计划修改的文件与主要风险。
3. 检查仓库中是否已有同类实现、Issue 或未合并 PR。

实施要求：
- 保留现有项目风格和兼容性。
- 不跳过测试、类型检查或 git hooks。
- 不提交密钥、生成物或无关改动。
- 若当前 GitHub 工具允许写入，在新分支完成修改并创建 PR；否则只返回可审阅的补丁和精确修改说明，不要声称已经写入仓库。

最终报告：
- 修改文件与原因
- 测试命令和真实结果
- 未验证事项
- 分支、提交或 PR 链接（仅在实际创建成功时提供）
```

把目标、边界和验收标准写清楚，比单纯要求“帮我开发一下”更容易得到可检查的结果。

### 5. 在 GitHub 上审阅远端结果

远端有新提交或 PR 后，先检查：

* 改动是否只覆盖约定范围；
* diff 中是否出现密钥、调试日志、依赖锁文件异常变化或大体积生成文件；
* 测试结论是否包含真实命令和结果，而不是只写“应该通过”；
* PR 的目标分支是否正确；
* 自动化检查是否通过。

重要项目不要直接在默认分支上接受大改动。优先使用独立分支和 Pull Request，保留审阅与回退空间。

### 6. 把远端改动安全同步回本地

先确认本地没有尚未处理的改动，再获取远端信息：

```bash
git status
git fetch origin
git log --oneline --decorate HEAD..origin/main
git diff --stat HEAD..origin/main
```

如果默认分支不是 `main`，把命令中的分支名替换为实际名称。确认本地分支可以快进后再同步：

```bash
git pull --ff-only origin main
```

`git fetch` 只获取远端信息，不会直接合并到当前分支；`git pull` 会获取并整合远端变更。存在本地未提交改动、分叉提交或冲突时，先暂停并处理，不要用 force push 或覆盖命令强行同步。

同步完成后仍要在本机安装依赖并运行项目规定的测试、构建和必要的人工验收。远端模型没有访问本机运行环境时，不能替你证明本地功能已经正常。

## 如何理解“效果更好”

“GPT-5.6 Pro 比 5.6 Sol Ultra 效果更好”可以记录为特定使用者在特定任务中的经验，但不是官方给出的普遍结论。更稳妥的写法是：

**在复杂仓库任务中，GPT-5.6 Pro 可能用更长等待时间换来更完整的分析与更稳定的最终答案；是否优于 5.6 Sol Ultra，应使用同一仓库、同一提示词和同一验收标准做对照。**

建议保存一组自己的代表性任务，比较一次通过率、测试结果、返工次数、总耗时和成本，再决定哪些任务默认使用 Pro。

## 事实核查与资料来源

* 模型定位与 Pro 模式：[OpenAI 模型目录](https://developers.openai.com/api/docs/models)、[GPT-5.6 模型指南](https://developers.openai.com/api/docs/guides/latest-model)
* 插件在 ChatGPT 网页版中的安装、调用与权限：[OpenAI Plugins 文档](https://learn.chatgpt.com/docs/plugins)
* 本地代码首次推送：[GitHub 官方文档：Adding locally hosted code to GitHub](https://docs.github.com/en/migrations/importing-source-code/using-the-command-line-to-import-source-code/adding-locally-hosted-code-to-github)
* 获取与整合远端变更：[GitHub 官方文档：Getting changes from a remote repository](https://docs.github.com/en/get-started/using-git/getting-changes-from-a-remote-repository)
* “Pro 优于 5.6 Sol Ultra”为本次提供的个人使用经验，本文已按经验性观察处理，没有改写成官方结论或普遍保证。
