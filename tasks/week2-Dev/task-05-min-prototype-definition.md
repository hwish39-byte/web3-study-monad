# 任务：定义 Week 2 最小 Web3 / AI 原型

## 我要做的最小功能是什么？

**链上井字棋（On-chain Tic-Tac-Toe）+ AI Agent 对手**

一个部署在 Monad Testnet 上的双人井字棋游戏，加上一个 AI Agent 作为对手。Agent 读取链上棋盘状态，计算落子，然后将落子交易发送上链。

最小功能范围：
- 1 个 Solidity 合约：管理游戏创建、落子、胜负判定
- AI Agent：读取链上棋盘 → 用 minimax 算法（或 LLM）决定落子 → 发送交易
- 用户通过 CLI 或简单前端与合约交互

## 谁会使用它？

- 我自己（学习 Monad 合约开发 + AI Agent 集成）
- 面试官 / 社区成员（作为 proof of work —— 展示全栈 Web3 × AI 能力）
- 后续可以扩展成多人游戏，供 Monad 生态用户测试

## 用户完成的一个动作是什么？

```
1. 用户发起新游戏 →
   合约创建 gameId，用户执 X
2. AI Agent 监听到新游戏 →
   自动落子 O（第一手）
3. 用户落子 →
   合约验证合法 -> 更新棋盘
4. AI Agent 再次落子 →
   直到一方获胜或平局
5. 查询游戏结果 →
   winner 地址 / draw
```

一条操作流的时间线：

```
用户:    ──创建游戏──→──落子──→──落子──→ ...
Agent:               ──落子──→    ──落子──→ ...
                          ↓
                   合约判定胜负 → emit GameEnded 事件
```

## 我需要读哪 1–3 个文档？

| 文档 | 用途 |
|------|------|
| [Monad Docs — Best Practices](https://docs.monad.xyz/developer-essentials/best-practices) | 合约存储优化、事件设计 |
| [Foundry Book — Forge](https://book.getfoundry.sh/forge/) | 合约开发、测试、部署 |
| [Ethers.js / viem 文档](https://viem.sh/) | Agent 连接链上合约、发送交易 |

附加参考：
- Moss 的 agent-skill.md（理解 Agent 如何安全构造交易）
- Solidity by Example — 投票/拍卖合约（事件驱动模式参考）

## 本周真实实现什么？哪些可以 mock？

### 真实实现（Real）

| 模块 | 实现内容 | 验证方式 |
|------|----------|----------|
| TicTacToe.sol | 完整合约：创建游戏、落子、胜负判定、事件 | `forge test` 通过 |
| 部署脚本 | 部署到 Monad Testnet | 获得合约地址，Explorer 可查 |
| Agent 侦察模块 | 监听链上 GameCreated / MoveMade 事件 | 日志输出事件信息 |
| AI 落子逻辑 | Minimax 算法（纯 off-chain 计算，不消耗 Gas） | 单测验证正确性 |

### 可 Mock

| 模块 | Mock 方案 | 何时做实 |
|------|-----------|----------|
| LLM 驱动的 Agent | 先用 Minimax 算法，规则明确、结果可预测 | 原型跑通后，替换为 LLM 调用 |
| 前端 UI | 先全 CLI 交互（`cast send` / 脚本） | 原型稳定后用 React + Wagmi |
| 多游戏并行 | 先只支持单游戏 1v1 | 基础版本验证后加 |
| Agent 自动启动 | 手动触发 Agent 脚本检查 | 后续用 cron / event listener 自动化 |

## 我如何证明它做出来了？

### 验证清单

- [ ] `forge test` 全部通过（合约逻辑完整性）
- [ ] 合约部署到 Monad Testnet，Explorer 可查
- [ ] Agent 脚本可读取链上棋盘状态并正确计算落子
- [ ] Agent 可发送落子交易到链上
- [ ] 一个完整游戏流程跑通：用户创建 → AI 落子 → 用户落子 → ... → 决出胜负
- [ ] 项目代码推送到 GitHub repo 的 `experiments/week2/` 下

### 交付物

```
experiments/week2/tic-tac-toe/
├── foundry.toml            Foundry 配置（Monad Testnet）
├── src/
│   └── TicTacToe.sol       合约源码
├── test/
│   └── TicTacToe.t.sol     合约测试
├── script/
│   └── Deploy.s.sol        部署脚本
├── agent/
│   ├── index.ts             Agent 入口（监听 + 决策 + 发送）
│   └── minimax.ts           Minimax 落子算法
└── README.md                项目说明 & 运行方式
```

### 一句话证明

> 在 Monad Testnet 上部署了 TicTacToe 合约（地址 `0x...`），Agent 脚本在未登录任何前端的情况下完成了 3 轮对弈并获胜。

## 技术选型

| 层 | 技术 | 理由 |
|----|------|------|
| 合约语言 | Solidity ^0.8.20 | 以太坊/ Monad 标准 |
| 开发框架 | Foundry (forge) | 速度快、Monad 官方推荐 |
| 链 | Monad Testnet | 学习目标一致 |
| Agent 语言 | TypeScript + viem | 自己熟悉的栈，viem 比 ethers 更轻 |
| AI 决策 | Minimax（初版）→ LLM（后续） | Minimax 确定性好，适合先验证流程 |

## 参考链接

- [Foundry Book](https://book.getfoundry.sh/)
- [Monad Docs — Best Practices](https://docs.monad.xyz/developer-essentials/best-practices)
- [viem.sh](https://viem.sh/)
- [Monad Testnet Explorer](https://monad-testnet.socialscan.io)
- [Solidity by Example — TicTacToe](https://solidity-by-example.org/app/tic-tac-toe/)（参考思路）

## 输出

- [x] 定义最小功能：链上井字棋 + AI Agent 对手
- [x] 明确用户画像：自己和面试官
- [x] 描述用户完整动作流
- [x] 列出需要阅读的 3 份文档
- [x] 区分真实实现与 mock
- [x] 定义验证标准和交付物结构
