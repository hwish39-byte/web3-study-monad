# Week 2 Dev Summary — 链上井字棋 + AI Agent 原型

## 1. 我想做什么

构建一个部署在 Monad Testnet 上的**链上井字棋（Tic-Tac-Toe）+ AI Agent 对手**，作为 Web3 × AI 全栈能力的 proof of work。

### 完整愿景

```
用户发起游戏 → 合约创建游戏实例 → AI Agent 自动落子 → 交替下棋 → 胜负判定上链

合约层:  TicTacToeGame.sol 管理每个游戏的状态、落子验证、胜负判定
工厂层:  GameFactory.sol   创建和追踪所有游戏实例
Agent层: 监听链上事件 → Minimax 决策 → 发送落子交易
交互层:  CLI 或前端和合约交互
```

### 用户故事

> 作为 Monad 生态的一个开发者，我想在链上下一盘井字棋，由 AI 作为对手。每一步落子都是一笔链上交易，验证 Monad 的低 Gas 和高 TPS 能否支撑链上游戏的实时交互。

---

## 2. 实际做到了哪一步

### 已完成 ✅

| 模块 | 状态 | 说明 |
|------|------|------|
| 原型定义 | ✅ 完成 | task-05-min-prototype-definition.md — 明确功能、用户、技术选型 |
| Factory-Pool 模式研读 | ✅ 完成 | 分析 Uniswap V3 Core 架构，理解 Factory-PoolDeployer-Pool 三层模式 |
| AI 辅助生成代码骨架 | ✅ 完成 | task-06-ai-code-skeleton.md — 4 个合约文件 + 9 项手动修改记录 |
| 架构分析笔记 | ✅ 完成 | Uniswap transient storage 技巧、Modifier 链设计、枚举类型选择 |
| 知识覆盖 | ✅ 完成 | Solidity 合约模式、Foundry 流程、Uniswap 架构、Moss Agent 工作流 |

### 未完成 ❌

| 模块 | 状态 | 卡点 |
|------|------|------|
| Foundry 项目初始化 | ❌ 未做 | 需要 `forge init` + 配置 Monad Testnet 网络 |
| 合约编译 & 测试 | ❌ 未做 | 代码骨架已验证逻辑，但未跑 `forge test` |
| 部署到 Monad Testnet | ❌ 未做 | 需要 RPC 配置和测试 MON |
| AI Agent 代码 | ❌ 未做 | Minimax 算法未实现 |
| CLI 交互脚本 | ❌ 未做 | viem + TypeScript 脚本未编写 |
| README 文档 | ❌ 未做 | 项目级别的 README 未创建 |

### 产出物清单

```
tasks/week2-Dev/
├── task-05-min-prototype-definition.md   ← 原型定义
└── task-06-ai-code-skeleton.md           ← 代码骨架 + AI 协作记录
```

### 代码骨架（已验证逻辑，未编译运行）

```solidity
// 核心合约架构
IGameFactory.sol       ← 工厂接口（createGame, getGame, totalGames）
ITicTacToeGame.sol     ← 游戏接口（makeMove, getBoard, getStatus, Events）
TicTacToeGame.sol      ← 游戏实现（棋盘、Modifier 链、胜负判定）
GameFactory.sol        ← 工厂实现（创建+追踪游戏，集成 GameDeployer）
```

---

## 3. AI Collaboration Log

| 阶段 | AI 做了什么 | 人工做了什么 |
|------|------------|-------------|
| 原型定义 | 基于用户背景推荐了 Tic-Tac-Toe 方案，分析技术选型 | 确认游戏方向，补充 Monad 生态定位 |
| 文档选择 | 推荐 Uniswap V3 Core 作为 Factory 模式参考 | 选择 UniswapV3Factory.sol 作为核心文档 |
| 架构理解 | 解释 Factory-PoolDeployer-Pool 三层模式 + transient storage 技巧 | 评估该模式在游戏场景的适用性 |
| 代码生成 | 生成 4 个合约骨架（接口 + 实现 + 工厂） | 手动修改 9 项：去 fee/salt/NoDelegateCall，增 Modifier 链/枚举/反向映射 |
| 代码审查 | 检查 Modifier 顺序、胜负判定完整性 | 确认 8 种胜利组合覆盖、边角 case 处理 |

**Key Insight**: AI 在 "理解成熟协议模式并迁移到新场景" 上非常高效。Uniswap 的 Factory 模式 → Tic-Tac-Toe 的 GameFactory 模式，核心架构几乎不变，改动集中在业务逻辑层。人工修改的重点是 "去掉金融复杂度，保留架构精髓"。

---

## 4. Known Issues

| Issue | 描述 | 优先级 |
|-------|------|--------|
| Foundry 项目未初始化 | `forge init` 未执行，合约文件未纳入编译管线 | HIGH |
| 合约未部署 | 无 Monad Testnet 上的合约地址 | HIGH |
| AI Agent 未实现 | Minimax 算法和 viem 交互脚本未写 | HIGH |
| Gas 未测试 | 井字棋 9 步完成，合约存储成本未知 | MEDIUM |
| CREATE2 确定地址 | 当前用 `new` 部署，后续可用 CREATE2 做确定性地址 | LOW |
| 前端 UI | 计划中可以完全 mock，不作 Week 2 目标 | LOW |

---

## 5. Week 3 我能承担的开发角色

### 首选：Smart Contract Developer（合约开发）

Week 2 已完成 Uniswap Factory 模式研读和 Tic-Tac-Toe 合约设计，Week 3 可以直接转入实现阶段：

- 初始化 Foundry 项目，将代码骨架迁移到正式编译管线
- 编写完整的 Forge 测试（单元测试 + 集成测试 + 边界条件）
- 部署到 Monad Testnet 并验证

### 次选：AI Agent Developer（Agent 开发）

- 用 TypeScript + viem 实现链上事件监听
- 实现 Minimax 落子算法
- 集成 Agent 与合约的完整交互流程

### 也可承担：Full-stack Builder

- 合约 + Agent 的全链路集成
- CLI 交互脚本
- 技术文档和 README

### 一句话定位

> 能读 Uniswap 源码，能写 Solidity 合约，能集成 AI Agent —— Week 3 最适合的角色是 **Smart Contract Developer**，把 Week 2 的设计变成跑在 Monad Testnet 上的真实合约。

---

## 6. 输出

- [x] Dev Plan（task-05 — 原型定义）
- [x] Code Skeleton（task-06 — 4 个合约文件 + 9 项人工修改记录）
- [x] AI Collaboration Log（本节 3）
- [x] Known Issues（本节 4）
- [x] Week 3 开发角色建议（本节 5）
