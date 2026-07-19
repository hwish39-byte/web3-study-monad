# 任务：GitHub 开源协作流程记录 — 完成 Moss Issue #97

## 背景

本次协作目标是参与 Moss 开源项目的一个文档类 Issue：

- Issue: https://github.com/nishuzumi/moss/issues/97
- 标题：`[docs] Add a post-simulation intent-alignment checklist`
- 类型：Documentation

这个 Issue 的核心问题是：Moss 已经强调“模拟成功且没有 Warning”并不等于“可以安全签名”，但新开发者仍然可能误以为 `simulate` 成功就代表交易符合用户意图。因此项目需要补充一个可复用的“模拟后意图对齐检查清单”，帮助 Agent / 开发者在签名前系统性检查 Receipt / Outcome 是否与用户原始请求一致。

## 目标理解

完成这个 Issue 不是写业务代码，而是修改文档，让 Moss 的安全边界表达得更清楚。

要强调的核心观点：

```text
Clean simulation proves Receipt coverage.
It does not prove user-intent alignment or approval to sign.
```

也就是说：

- `simulate` 没有 Warning，只说明 Moss 成功解析了观察到的链上变化；
- 它不代表交易一定符合用户的原始意图；
- 它也不代表用户已经同意签名；
- Agent 必须把模拟结果和用户请求逐项对齐。

## 开源协作整体流程

本次流程可以概括为：

```text
阅读 Issue -> Fork 仓库 -> Clone 自己的 Fork -> 新建分支 -> 修改文档
-> 本地测试 -> Commit -> Push -> 创建 Pull Request -> 填写 PR 模板
```

这是参与 GitHub 开源项目最常见的协作方式。

## Step 1：Fork 原仓库

因为普通贡献者通常没有权限直接向原仓库 `nishuzumi/moss` 推送代码，所以需要先 fork 一份到自己的 GitHub 账号下。

操作步骤：

1. 打开 Moss 原仓库：https://github.com/nishuzumi/moss
2. 点击右上角 `Fork`
3. Owner 选择自己的 GitHub 账号，例如 `hwish39-byte`
4. Repository name 保持 `moss`
5. 点击 `Create fork`

完成后，会得到自己的副本：

```text
https://github.com/hwish39-byte/moss
```

Fork 的意义是：在自己的仓库副本上自由修改，再通过 Pull Request 请求原项目合并。

## Step 2：Clone 自己的 Fork

不要把 Moss 源码直接放进当前学习仓库 `web3-study-monad`。学习仓库只用于记录笔记和任务输出，真正修改 Moss 项目要在单独的 Moss 仓库目录中进行。

本地操作：

```bash
cd ~/project
git clone https://github.com/hwish39-byte/moss.git
cd moss
```

进入 Moss 仓库后，可以添加上游仓库：

```bash
git remote add upstream https://github.com/nishuzumi/moss.git
```

检查远程配置：

```bash
git remote -v
```

常见结果：

```text
origin    https://github.com/hwish39-byte/moss.git
upstream  https://github.com/nishuzumi/moss.git
```

其中：

- `origin` 是自己的 fork；
- `upstream` 是原项目仓库。

## Step 3：新建工作分支

不要直接在 `main` 上改。每个 Issue 建一个独立分支，方便提交 PR 和后续维护。

本次分支名：

```bash
git checkout -b docs/post-simulation-intent-checklist
```

确认当前分支：

```bash
git branch --show-current
```

应输出：

```text
docs/post-simulation-intent-checklist
```

## Step 4：分析需要修改的文件

根据 Issue 内容，最合适的修改位置是文档目录中的三个文件：

| 文件 | 修改目的 |
|------|----------|
| `docs/agent-skill.md` | 新增通用的 post-simulation intent-alignment checklist |
| `docs/getting-started.md` | 在 Kuru swap 的意图对齐章节引用通用 checklist |
| `docs/mcp-tools.md` | 在 MCP `simulate` 说明中提醒：clean simulation 不等于意图匹配 |

这个 Issue 的主要修改点应该放在 `docs/agent-skill.md`，因为它是 Agent 使用 Moss 的安全规则文档。

## Step 5：修改 `docs/agent-skill.md`

在 `## Required flow` 和 `## Hard rules` 之间新增一节：

```md
## Post-simulation intent-alignment checklist

After `simulate` returns with no Warnings, do not treat the result as safe to sign automatically. A clean simulation proves that Moss parsed every observed Change; it does not prove the result matches the user's original request.

Before handing transactions to a signer, compare the ordered Receipt texts and structured Outcomes against the recorded intent:

- Operation: the operation matches the user's requested verb, such as `swap`, `transfer`, `approve`, `wrap`, or `unwrap`.
- Protocol: the protocol matches the user's explicit or selected protocol constraint.
- Sender: every transaction sender matches the account authorized by the user.
- Assets: input, output, approved, transferred, wrapped, or unwrapped assets match explicit token identities, not only symbols.
- Amounts: input amounts, output amounts, approval amounts, and native values match the request and loaded parameter units.
- Recipients: every recipient, spender, pool, router, or protocol address is expected.
- Limits: slippage, minimum output, maximum input, deadline, allowance, and other safety limits are preserved.
- Ordering: nested Capability effects appear in the expected order, especially approvals before protocol actions.
- Extra effects: no unexpected transfer, approval, wrap, unwrap, refund, fee, or protocol action appears.
- Warnings: any Warning, halted simulation, missing Receipt, or ambiguous Outcome stops the flow.

If any item cannot be confirmed from the recorded intent and loaded contract, ask the user or stop. Never infer approval from a successful simulation alone.
```

这段 checklist 的作用是把“模拟后必须检查什么”变成一组可复用规则。

## Step 6：修改 `docs/getting-started.md`

找到这一节：

```md
## 9. Align structured Outcomes with intent
```

在标题下面新增一句：

```md
Use the reusable checklist in [Agent safety rules](./agent-skill.md#post-simulation-intent-alignment-checklist), then apply the Kuru-specific checks below.
```

这样 Getting Started 中的 Kuru swap 示例就和通用 checklist 建立了连接。

这里不需要重写整个教程，因为原文已经包含 Kuru swap 的具体检查项。当前 Issue 要的是“compact, reusable checklist”，不是替换已有示例。

## Step 7：修改 `docs/mcp-tools.md`

找到 `## simulate` 章节，在说明 clean simulation / Receipt coverage 的位置附近新增：

```md
A clean `simulate` response proves Receipt coverage, not user-intent alignment. Agents must run the post-simulation intent-alignment checklist before handing transactions to a signer.
```

这句话提醒 MCP 工具使用者：`simulate` 的职责是验证 Receipt 覆盖，不是替用户判断意图是否匹配。

## Step 8：检查本地修改

修改完成后运行：

```bash
git diff
```

查看 diff 时，终端可能进入分页模式。退出方法是按：

```text
q
```

确认只修改了这三个文件：

```text
docs/agent-skill.md
docs/getting-started.md
docs/mcp-tools.md
```

如果发现改动范围不对，应先修正再提交。

## Step 9：运行测试

本次是文档修改，理论上不影响代码逻辑，但仍然建议跑一次测试，给 PR 提供 Evidence。

最初运行：

```bash
pnpm test:offline
```

终端报错：

```text
[ERR_PNPM_RECURSIVE_RUN_FIRST_FAIL] @themoss/core@0.1.0 test: `vitest run`
Exit status 1
```

继续查看完整错误，发现真正原因是：

```text
Error: EACCES: permission denied, mkdir '/var/folders/zz/...'
```

这不是文档改动导致的失败，而是 Vitest 在 macOS 默认临时目录下没有创建文件夹权限。解决方法是手动指定项目内的临时目录：

```bash
mkdir -p .tmp
TMPDIR="$PWD/.tmp" pnpm test:offline
```

使用 `TMPDIR="$PWD/.tmp"` 后测试通过。

这个经验很重要：遇到测试失败时，不要只看 pnpm 最后一行错误，要往上找真正的 `Error:` / `FAIL` 信息。

## Step 10：暂存和提交

确认 diff 和测试后，暂存文件：

```bash
git add docs/agent-skill.md docs/getting-started.md docs/mcp-tools.md
```

提交：

```bash
git commit -m "docs: add post-simulation intent alignment checklist"
```

提交信息使用 Conventional Commit 风格：

- `docs:` 表示文档修改；
- 后面说明具体变化。

## Step 11：推送到自己的 Fork

首次推送这个分支：

```bash
git push -u origin docs/post-simulation-intent-checklist
```

这里推送到的是自己的 fork：

```text
https://github.com/hwish39-byte/moss
```

不是直接推送到原项目 `nishuzumi/moss`。

## Step 12：创建 Pull Request

推送成功后，打开自己的 fork 页面：

```text
https://github.com/hwish39-byte/moss
```

GitHub 通常会显示 `Compare & pull request` 按钮。点击后确认 PR 方向：

```text
base repository: nishuzumi/moss
base branch: main

head repository: hwish39-byte/moss
compare branch: docs/post-simulation-intent-checklist
```

PR 标题：

```text
docs: add post-simulation intent-alignment checklist
```

## Step 13：填写 PR 描述

Moss 仓库自带 PR 模板，内容包括：

```md
## What and why

## Type of change

## Framework and package impact

## Verification

### Protocol changes

## Evidence
```

本次是文档 PR，可以这样填写。

### What and why

```md
Closes #97

This PR adds a reusable post-simulation intent-alignment checklist for Agents.

A clean simulation proves that Moss parsed every observed Change, but it does not prove that the result matches the user's original request or is safe to sign. The checklist makes that safety boundary easier to apply across Capabilities, not only in the Getting Started Kuru swap example.

Changes:
- Adds a reusable checklist to `docs/agent-skill.md`.
- Links the Getting Started intent-alignment section to the checklist.
- Adds a short MCP `simulate` note clarifying that Receipt coverage is not user-intent alignment.
```

### Type of change

只勾选：

```md
- [x] Documentation / example
```

其他不勾。

### Framework and package impact

```md
None. Documentation-only change. No public types, package boundaries, Capability tree, Change/Receipt behavior, or runtime behavior changed.
```

### Verification

因为实际跑的是 `pnpm test:offline`，它内部等价于跳过 E2E 的测试流程，可以勾：

```md
- [x] `pnpm test`
- [x] Docs and examples match the implemented API
```

没有跑的命令不要勾：

```md
- [ ] `pnpm build`
- [ ] `pnpm typecheck`
- [ ] `pnpm lint`
```

Changeset 不需要勾，因为没有用户包变更：

```md
- [ ] User-facing package changes include a changeset
```

### Protocol changes

本次不是 Protocol PR，写：

```md
N/A for this documentation-only PR.
```

下面的 Protocol checklist 不需要勾。

### Evidence

推荐填写：

```md
Ran `TMPDIR="$PWD/.tmp" pnpm test:offline`.

Result: passed.

`TMPDIR` was set because the default macOS temp directory returned an `EACCES: permission denied, mkdir '/var/folders/zz/...'` error when Vitest tried to create worker/cache files.
```

这段 Evidence 说明了验证命令，也解释了为什么需要设置 `TMPDIR`。

## 本次遇到的问题

### 1. 不知道是否需要新建文件夹

一开始容易混淆学习仓库和贡献仓库。正确做法是：

- `web3-study-monad`：记录学习笔记和任务输出；
- `moss`：fork 后 clone 下来的真实开源项目，用于提交 PR。

不要把 Moss 源码直接放进学习仓库。

### 2. 不理解 Fork

Fork 是把别人的仓库复制一份到自己的 GitHub 账号下。普通贡献者没有权限直接推送到原仓库，所以需要先修改自己的 fork，再通过 PR 请求原仓库合并。

### 3. `git diff` 不知道如何退出

`git diff` 默认可能进入分页器，按：

```text
q
```

即可退出。

### 4. 测试失败但不是代码问题

一开始 `pnpm test:offline` 失败，错误显示在 `@themoss/core` 的 `vitest run`。真正原因是：

```text
EACCES: permission denied, mkdir '/var/folders/zz/...'
```

解决方法：

```bash
mkdir -p .tmp
TMPDIR="$PWD/.tmp" pnpm test:offline
```

这说明测试环境问题和代码问题要分开判断。

## 本次学到的 GitHub 协作知识

### 1. 原仓库、Fork、Upstream、Origin 的区别

| 名称 | 含义 |
|------|------|
| 原仓库 | 项目维护者的仓库，例如 `nishuzumi/moss` |
| Fork | 自己账号下的项目副本，例如 `hwish39-byte/moss` |
| origin | 本地仓库默认远程，通常指向自己的 fork |
| upstream | 手动添加的上游仓库，指向原项目 |

### 2. 一个 Issue 对应一个分支

为每个 Issue 创建独立分支，可以让修改范围清晰，也方便维护者 review。

本次分支：

```text
docs/post-simulation-intent-checklist
```

### 3. 文档 PR 也要认真验证

即使只改 Markdown，也应该尽量跑项目提供的测试命令。如果测试命令因为环境问题失败，要在 PR 的 Evidence 中如实说明。

### 4. PR 描述要回答维护者关心的问题

好的 PR 描述应该说明：

- 改了什么；
- 为什么改；
- 影响范围是什么；
- 如何验证；
- 是否关闭某个 Issue。

本次使用：

```md
Closes #97
```

这样 PR 合并后 GitHub 会自动关闭对应 Issue。

## 总结

这次 Moss Issue #97 是一次完整的轻量级开源贡献练习。虽然修改内容只是文档，但流程覆盖了真实开源协作中的关键步骤：

- 阅读 Issue；
- 理解项目安全模型；
- fork 原仓库；
- clone 自己的 fork；
- 新建分支；
- 修改文档；
- 运行测试；
- 处理本地测试环境问题；
- commit；
- push；
- 创建 PR；
- 按模板填写说明和 Evidence。

这类文档 Issue 很适合作为第一次开源贡献入口。它不要求立刻理解全部源码，但要求准确理解项目的设计边界，并用维护者能接受的方式补充文档。

本次最大的收获是：开源贡献不只是“改文件”，更重要的是把问题、修改、验证和影响范围讲清楚。维护者看到 PR 后，应该能快速判断这个修改为什么存在、是否安全、如何验证。

## 参考链接

- Moss 原仓库：https://github.com/nishuzumi/moss
- Issue #97：https://github.com/nishuzumi/moss/issues/97
- Fork 示例：https://github.com/hwish39-byte/moss
- GitHub Pull Request 文档：https://docs.github.com/en/pull-requests
