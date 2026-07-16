# 任务：Moss 开源贡献计划

## 背景

在完成 Moss 项目研读后，结合自己的 Builder 身份（Web3 × AI Agent 全栈开发），制定一份实际可执行的贡献计划。

## 自我能力评估

| 维度 | 匹配度 | 说明 |
|------|--------|------|
| TypeScript / Node.js | ⭐⭐⭐ | 有 React + MCP Server 开发经验 |
| Solidity | ⭐⭐⭐ | 写过 TrustPay 合约，了解 ERC20/ERC721 |
| Web3 协议理解 | ⭐⭐⭐ | 做过 DeFi 场景分析，熟悉 DEX/Lending 基本流程 |
| AI Agent | ⭐⭐⭐⭐ | CGHub 的 Agent 模块 + Hermes Agent 日常使用 |
| 中文文档能力 | ⭐⭐⭐⭐⭐ | 母语中文，适合做中文内容贡献 |
| Monad 生态 | ⭐⭐ | Week 1 刚入门，正在学习中 |

## 选择贡献方向

### 方向评估

| 方向 | 适合的工作 | 匹配理由 |
|------|-----------|----------|
| Dev Builder | 协议 Adapter、Bug Fix、Demo | ✅ 有合约和 TS 基础，但 Moss 代码库尚不熟悉 |
| Research Builder | 架构分析、技术笔记、文档补充 | ✅ 能从 AI Agent 视角输出独特见解 |
| Ops Builder | README、Tutorial、FAQ | ✅ 中文是优势，但已有中文文档 |

### 选定方向：Research Builder → Dev Builder（本周路线）

**第一优先级：Research Builder** — 本周先深入理解 Moss 架构，输出有价值的技术笔记
**第二优先级：Dev Builder** — 下周尝试提交一个简单的 Protocol Adapter

**为什么这样选：**
1. Moss 是 Alpha 阶段项目，代码和文档都可能变化，直接写代码前先吃透设计
2. Agent 工作流的理解是我的优势（CGHub Agent 模块经验），能输出独到的分析
3. Issue #12（FastLane shMONAD staking adapter）标了 "good first issue" + "starter"，是理想的切入点

## 本周贡献计划

### Day 1：架构研读（2h）

- [ ] 阅读 Moss 源码结构：core（Registry, Plan, ActionFlow）/ simulator / erc / system
- [ ] 重点理解 `Registry.action()` 和 `simulate()` 的调用链路
- [ ] 通读现有的 ADR（Architecture Decision Records）
- [ ] 输出：Moss 架构笔记（中文）

### Day 2：Agent Workflow 分析（2h）

- [ ] 阅读 agent-skill.md + mcp-tools.md，理解 Agent 的 4 步工作流
- [ ] 运行 example-simple-flow 和 agent-swap，理解实际执行流程
- [ ] 对比 CGHub Agent 模块的设计差异，输出对比分析
- [ ] 输出：Moss Agent Workflow 技术笔记（中文）

### Day 3：协议 Adapter 预研（2h）

- [ ] 阅读 protocol-onboarding.md + system/src/wmon.ts（参考 adapter）
- [ ] 分析 FastLane shMONAD 合约文档，确定需要实现的函数
- [ ] 确认需要的测试用例
- [ ] 输出：FastLane shMONAD adapter 开发方案

### Day 4：贡献 Issue 互动（1h）

- [ ] 在 Issue #12 下回复，说明自己有兴趣贡献该 adapter
- [ ] 向维护者确认技术方案、测试环境和 PR 规范
- [ ] 如果方案确认，创建 WIP PR 或 fork 开始开发

## 优先考虑 Issue

### Issue #12：Adapter: FastLane shMONAD staking

| 维度 | 内容 |
|------|------|
| 标签 | `good first issue` `adapter` `staking` `difficulty:starter` |
| 难度 | Starter — 形状与现有 adapter 类似 |
| 价值 | 填补 Monad 生态的质押协议适配 |
| 先决条件 | 读 protocol-onboarding.md + 参考 wmon.ts |

### Issue #16：Documentation — Beginner-friendly Quick Start Guide

| 维度 | 内容 |
|------|------|
| 标签 | 无 |
| 难度 | 低，纯文档 |
| 价值 | 降低新用户的入门门槛 |
| 注意 | 已有 getting-started.md，需要看缺口在哪里 |
| 中文优势 | 可以同时输出中英文版 |

### Issue #28：Tooling — Fetch verified ABIs through Monadscan API

| 维度 | 内容 |
|------|------|
| 标签 | `enhancement` `ready-for-agent` |
| 难度 | 中等，需要理解 Monadscan API + Moss 的 token fallback 机制 |
| 价值 | 自动化 ABI 获取，减少手动配置 |
| 注意 | 标了 `ready-for-agent`，也可能被其他贡献者接走 |

## 输出计划

| 输出 | 格式 | 存放位置 |
|------|------|----------|
| Moss 架构笔记 | Markdown | `experiments/week2/moss-architecture-notes.md` |
| Agent Workflow 分析 | Markdown | `experiments/week2/moss-agent-workflow-analysis.md` |
| FastLane shMONAD 方案 | Markdown | `tasks/week2/task-03-moss-contribution-plan.md`（本文件已含） |

## 预期成果

- 本周：输出 2 份技术笔记 + 1 份开发方案
- 下周目标：提交 FastLane shMONAD adapter 的 PR（或至少完成 WIP）
- 所有产出同步到本 repo 作为 proof of work

## 参考链接

- Moss GitHub: https://github.com/nishuzumi/moss
- Moss 文档目录: docs/（含 ADR、agent-skill.md、protocol-onboarding.md）
- Issue #12 (FastLane shMONAD): https://github.com/nishuzumi/moss/issues/12
- Issue #16 (Quick Start Guide): https://github.com/nishuzumi/moss/issues/16
- Issue #28 (Monadscan ABI): https://github.com/nishuzumi/moss/issues/28

## 输出

- [x] 完成自我能力评估
- [x] 选定 Research Builder → Dev Builder 的渐进路径
- [x] 制定本周 4 天贡献计划（研读 → 分析 → 预研 → 互动）
- [x] 分析 3 个候选 Issue 的可行性和优先级
- [x] 明确输出物和存放位置
