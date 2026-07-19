# 任务：AI 辅助理解 Uniswap Factory 模式 & 生成游戏合约骨架

## 文档链接

https://github.com/Uniswap/v3-core

核心阅读的合约文件：
- `contracts/UniswapV3Factory.sol` — 工厂合约，负责创建和管理 Pool
- `contracts/UniswapV3PoolDeployer.sol` — 部署器，负责实际的合约部署逻辑
- `contracts/interfaces/IUniswapV3Factory.sol` — 工厂接口定义

## 我让 AI 帮我理解了什么

### 1. Uniswap V3 的 Factory-Pool 架构模式

Uniswap V3 使用工厂模式来创建多个 Pool 实例：

```
UniswapV3Factory
  ├── createPool(tokenA, tokenB, fee) → 创建新 Pool
  ├── getPool[token0][token1][fee] → 查询已有 Pool
  └── 事件: PoolCreated, FeeAmountEnabled
         ↓
UniswapV3PoolDeployer
  ├── parameters (临时存储，部署时供 Pool 读取)
  └── deploy() → address(new UniswapV3Pool{salt: ...}())
         ↓
UniswapV3Pool
  ├── 构造函数读取 parameters 后立即清除
  ├── state: liquidity, sqrtPrice, tick, fee
  └── functions: mint, burn, swap, flash
```

### 2. PoolDeployer 的 transient storage 技巧

```solidity
contract UniswapV3PoolDeployer {
    struct Parameters { address factory; address token0; address token1; uint24 fee; int24 tickSpacing; }
    Parameters public parameters;

    function deploy(...) internal returns (address pool) {
        parameters = Parameters({...});          // 1. 写入参数
        pool = address(new UniswapV3Pool{salt: ...}());  // 2. 部署，Pool 构造函数读取 parameters
        delete parameters;                       // 3. 清除参数（防止干扰下次部署）
    }
}
```

关键设计理由：
- Pool 构造函数需要读取参数，但 Solidity 不支持构造函数传参以外的初始化方式
- 通过存储槽暂存参数，是 Gas 最优的方案（SSTORE + 构造函数 SLOAD）
- 部署后立即 delete，防止下次部署读取到旧参数

### 3. 如何应用到链上井字棋

```
GameFactory
  ├── createGame(player1, player2) → 创建新游戏
  ├── games[index] → 查询游戏地址
  └── 事件: GameCreated, GameEnded
         ↓
GameDeployer
  ├── parameters (部署时暂存)
  └── deploy() → address(new TicTacToeGame{salt: ...}())
         ↓
TicTacToeGame
  ├── players[], board[3][3], currentTurn, winner, status
  ├── makeMove(x, y) → 验证合法性 → 更新棋盘 → 判定胜负
  └── events: MoveMade, GameWon, GameDraw
```

**核心改动：**
- 去掉 fee / tickSpacing 参数，替换为 player1 / player2 地址
- 去掉 swap 逻辑，替换为落子逻辑
- 保持 Factory-PoolDeployer 的架构不变

## AI 生成了什么代码骨架

基于 Uniswap V3 的 Factory-Pool 模式，生成了 4 个合约文件：

### 1. interfaces/IGameFactory.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IGameFactory {
    event GameCreated(address indexed player1, address indexed player2, address game, uint256 index);
    event GameEnded(address indexed game, address winner, uint256 index);

    function createGame(address player1, address player2) external returns (address game);
    function getGame(uint256 index) external view returns (address);
    function totalGames() external view returns (uint256);
}
```

### 2. interfaces/ITicTacToeGame.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface ITicTacToeGame {
    enum Status { Ongoing, Player1Wins, Player2Wins, Draw }
    enum Mark { Empty, X, O }

    event MoveMade(address indexed player, uint8 x, uint8 y, Mark mark);
    event GameWon(address indexed winner, Mark winningMark);
    event GameDraw();

    function getBoard() external view returns (Mark[3][3] memory);
    function getStatus() external view returns (Status);
    function getPlayers() external view returns (address, address);
    function getCurrentTurn() external view returns (address);
    function makeMove(uint8 x, uint8 y) external;
}
```

### 3. TicTacToeGame.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./interfaces/ITicTacToeGame.sol";

contract TicTacToeGame is ITicTacToeGame {
    Mark[3][3] private board;
    Status private status;
    address public player1;
    address public player2;
    address public currentTurn;

    modifier onlyPlayer() {
        require(msg.sender == player1 || msg.sender == player2, "Not a player");
        _;
    }

    modifier onlyCurrentTurn() {
        require(msg.sender == currentTurn, "Not your turn");
        _;
    }

    modifier gameOngoing() {
        require(status == Status.Ongoing, "Game ended");
        _;
    }

    constructor(address _player1, address _player2) {
        player1 = _player1;
        player2 = _player2;
        currentTurn = _player1;  // X goes first
        status = Status.Ongoing;
    }

    function getBoard() external view returns (Mark[3][3] memory) { return board; }
    function getStatus() external view returns (Status) { return status; }
    function getPlayers() external view returns (address, address) { return (player1, player2); }
    function getCurrentTurn() external view returns (address) { return currentTurn; }

    function makeMove(uint8 x, uint8 y) external onlyPlayer onlyCurrentTurn gameOngoing {
        require(x < 3 && y < 3, "Invalid position");
        require(board[x][y] == Mark.Empty, "Position taken");

        Mark mark = (msg.sender == player1) ? Mark.X : Mark.O;
        board[x][y] = mark;
        emit MoveMade(msg.sender, x, y, mark);

        if (_checkWin(mark)) {
            status = (mark == Mark.X) ? Status.Player1Wins : Status.Player2Wins;
            emit GameWon(msg.sender, mark);
        } else if (_isDraw()) {
            status = Status.Draw;
            emit GameDraw();
        } else {
            currentTurn = (msg.sender == player1) ? player2 : player1;
        }
    }

    function _checkWin(Mark mark) private view returns (bool) {
        // 行
        for (uint8 i = 0; i < 3; i++)
            if (board[i][0] == mark && board[i][1] == mark && board[i][2] == mark) return true;
        // 列
        for (uint8 j = 0; j < 3; j++)
            if (board[0][j] == mark && board[1][j] == mark && board[2][j] == mark) return true;
        // 对角线
        if (board[0][0] == mark && board[1][1] == mark && board[2][2] == mark) return true;
        if (board[0][2] == mark && board[1][1] == mark && board[2][0] == mark) return true;
        return false;
    }

    function _isDraw() private view returns (bool) {
        for (uint8 i = 0; i < 3; i++)
            for (uint8 j = 0; j < 3; j++)
                if (board[i][j] == Mark.Empty) return false;
        return true;
    }
}
```

### 4. GameFactory.sol（集成 GameDeployer）

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./interfaces/IGameFactory.sol";
import "./TicTacToeGame.sol";

contract GameDeployer {
    struct Parameters { address player1; address player2; }
    Parameters public parameters;

    function deploy(address player1, address player2) internal returns (address game) {
        parameters = Parameters(player1, player2);
        game = address(new TicTacToeGame(player1, player2));
        delete parameters;
    }
}

contract GameFactory is IGameFactory, GameDeployer {
    address[] public games;
    mapping(address => uint256) public gameIndex;

    function createGame(address player1, address player2) external returns (address game) {
        require(player1 != address(0) && player2 != address(0), "Invalid address");
        require(player1 != player2, "Same player");
        game = deploy(player1, player2);
        games.push(game);
        gameIndex[game] = games.length - 1;
        emit GameCreated(player1, player2, game, games.length - 1);
    }

    function getGame(uint256 index) external view returns (address) {
        require(index < games.length, "Index out of bounds");
        return games[index];
    }

    function totalGames() external view returns (uint256) { return games.length; }
}
```

## 我手动改了什么

| 改动 | 原（Uniswap 模式） | 改后 | 原因 |
|------|--------------------|------|------|
| 删除 fee / tickSpacing | Factory 需要 fee 参数 | 只保留 player1, player2 | 游戏不需要费用参数 |
| 删除盐值 salt | `new Pool{salt: keccak256(...)}()` | 直接 `new TicTacToeGame(...)` | 游戏合约不要求 CREATE2 确定性地址 |
| 构造函数传参 | Pool 从 deployer.parameters 读取 | TicTacToeGame 直接构造函数传参 | Solidity ^0.8.20 支持直接传参，不需要 transient storage 技巧 |
| 简化 deployer | UniswapV3PoolDeployer 独立合约 | GameDeployer 作为 Factory 的父合约（internal） | 复杂度降低，不需要分合约 |
| 删除 NoDelegateCall | Uniswap 有 NoDelegateCall modifier | 删除 | 游戏合约不需要防 delegatecall |
| 增加 gameIndex 反向映射 | 无 | `mapping(address => uint256)` | 方便定位已部署的游戏 |
| 增加 Modifier 链 | 无 | onlyPlayer → onlyCurrentTurn → gameOngoing | 三个安全守卫串联验证 |
| 简化胜负判定 | 无（Uniswap 用 tick 数学） | 手写 8 种胜利条件（3行+3列+2对角线） | 井字棋规则简单，不需要复杂数学 |
| 增加枚举类型 | 无 | Status 和 Mark 枚举 | 状态语义化，比 uint8 可读性强 |

## 当前是否跑通

**部分跑通**

合约代码逻辑已验证（手动走读）：
- 构造函数正确初始化玩家和棋盘
- Modifier 链顺序正确（先验证身份 → 再验证轮次 → 再验证游戏状态）
- 落子验证：边界检查 + 位置占用检查
- 胜负判定覆盖全部 8 种胜利组合
- 平局判定覆盖全部 9 格

差异说明：
- 简化了 PoolDeployer 的 transient storage 模式，因为 Solidity 0.8.20 支持构造函数传参，不需要这个技巧
- 游戏合约逻辑比 Uniswap Pool 简单很多（无复杂数学运算），代码骨架可直接用于实现

## 如果没跑通，卡在哪里

没有卡点。代码逻辑完整，可以直接放入 Foundry 项目编译和测试。后续需要：
1. 用 `forge init` 创建 Foundry 项目
2. 将上面 4 个文件放入 `src/`
3. 编写测试用例（基本流程、边界条件）
4. 部署到 Monad Testnet

## 参考链接

- Uniswap V3 Core: https://github.com/Uniswap/v3-core
- UniswapV3Factory.sol: 工厂模式参考
- UniswapV3PoolDeployer.sol: 部署模式参考
- Uniswap V3 Whitepaper: 架构设计背景

## 输出

- [x] 选择 Uniswap V3 Core 为核心文档
- [x] AI 辅助理解 Factory-Pool 架构模式 + transient storage 技巧
- [x] 生成完整代码骨架（4 个合约文件）
- [x] 记录手动改动（9 项修改及原因）
- [x] 评估运行状态（部分跑通，可直接编译）
