# 任务：课程钱包准备与 Monad Testnet 配置

## 背景

Web3 开发的第一步是拥有一个独立、安全的开发用钱包。课程期间所有链上操作（领测试币、部署合约、测试交互）都应使用专用钱包，避免与主力资产混淆。

## 目标

- 创建/准备一个课程专用的 EVM 钱包
- 添加 Monad Testnet 网络并确认连通
- 学会使用区块浏览器查询链上信息
- 理解链上产品与传统互联网产品的核心区别

## 实操记录

### 1. 钱包准备

| 项目 | 内容 |
|------|------|
| 钱包工具 | MetaMask 浏览器扩展 |
| 钱包类型 | EOA（Externally Owned Account） |
| 课程钱包地址 | 使用 Monad Course 专用账户 |
| 助记词备份 | 已离线物理保存，未截图/上传 |
| 安全策略 | 课程钱包与主力钱包分离，仅存放测试代币 |

### 2. Monad Testnet 网络配置

手动添加至 MetaMask：

- **网络名称**：Monad Testnet
- **RPC URL**：（从 Monad Docs 获取）
- **Chain ID**：（从 Monad Docs 获取）
- **货币符号**：MON
- **区块浏览器**：https://monad-testnet.socialscan.io

配置完成后可正常查询余额和交易记录。

### 3. 区块浏览器初体验

第一次打开 Monad Testnet Explorer 时看到的信息：

- 搜索框：可按交易哈希 / 地址 / 区块高度 / 合约地址搜索
- 最新区块列表：区块高度、时间戳、交易数量、验证者
- 最新交易列表：交易哈希、From / To、金额、Gas 费
- 地址页面：ETH 余额、交易历史、Token 持有、合约交互记录

区块浏览器是链上世界的"搜索引擎"——所有公开的链上数据都可以在这里查到。

### 4. 链上产品 vs 互联网产品的理解

**互联网产品（Web2）：**
- 数据存储在公司的服务器上，公司可以修改、删除、审查
- 用户身份由平台控制（账号密码在平台方手里）
- 规则由平台方单方面决定，用户没有话语权

**链上产品（Web3）：**
- 数据存储在区块链上，任何人无法篡改或删除
- 用户通过私钥控制自己的身份和资产，"Not your keys, not your coins"
- 规则由智能合约代码执行，公开透明、不可篡改
- 链上产品本质上是一个**公共状态机**——任何人都可以读状态、提交交易改变状态

**一句话总结：** 互联网产品是你向平台租用服务，链上产品是你直接和代码交互、自己拥有数据和控制权。

## 参考材料

- [Monad Docs - Testnets](https://docs.monad.xyz/developer-essentials/testnets)
- [Monad Docs - Block Explorers](https://docs.monad.xyz/tooling-and-infra/block-explorers)
- [Ethereum Wallets](https://ethereum.org/en/wallets/)
- [Web3 实习手册 - 安全指南](https://web3intern.xyz/zh/security/)

## 输出

- [x] 课程专用钱包已创建 & 助记词已备份
- [x] Monad Testnet 已添加至 MetaMask
- [x] 区块浏览器可正常查询地址
- [x] 已理解链上产品与 Web2 产品的核心差异
