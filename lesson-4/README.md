# Lesson 4: Delegatecall Proxy Pattern

## 概述

本课程实现了一个完整的delegatecall代理模式，展示了如何使用delegatecall来执行逻辑合约的函数，同时保留调用者的状态。这是智能合约升级和可扩展性的关键模式。

## 文件结构

- **Logic.sol** - 逻辑合约，包含实际的业务逻辑
- **Proxy.sol** - 代理合约，使用delegatecall调用逻辑合约
- **Proxy.test.js** - 完整的测试套件
- **hardhat.config.js** - Hardhat配置文件
- **package.json** - 项目依赖配置

## 核心概念

### Delegatecall vs Call

| 特性 | Call | Delegatecall |
|------|------|-------------|
| 执行上下文 | 被调用合约 | 调用合约 |
| msg.sender | 调用者 | 调用者 |
| 存储位置 | 被调用合约的存储 | 调用合约的存储 |
| 用途 | 普通函数调用 | 代理模式、库 |

### 为什么使用Delegatecall？

1. **合约升级** - 更新逻辑而不改变存储
2. **代码复用** - 多个合约共享同一逻辑
3. **存储管理** - 保持调用合约的状态

## 逻辑合约 (Logic.sol)

### 状态变量
- `counter` - 全局计数器
- `owner` - 合约所有者
- `userCounters` - 用户特定的计数器

### 主要函数
- `initialize(address _owner)` - 初始化合约
- `increment()` - 增加计数器1
- `incrementBy(uint256 amount)` - 增加指定数量
- `decrement()` - 减少计数器1
- `reset()` - 重置计数器（仅owner）
- `changeOwner(address newOwner)` - 更改所有者（仅owner）

## 代理合约 (Proxy.sol)

### 关键特性

1. **状态变量对齐** - 与逻辑合约相同的存储布局
2. **Delegatecall包装** - 通过delegatecall调用逻辑合约
3. **逻辑合约升级** - 支持更新逻辑合约地址
4. **Fallback函数** - 转发未知函数调用

### 主要函数
- `initialize(address _owner)` - 初始化代理
- `increment()` - 通过delegatecall调用increment
- `incrementBy(uint256 amount)` - 通过delegatecall调用incrementBy
- `decrement()` - 通过delegatecall调用decrement
- `reset()` - 通过delegatecall调用reset
- `updateLogicContract(address newLogicContract)` - 更新逻辑合约

## 安装和运行

### 1. 安装依赖
```bash
npm install
```

### 2. 编译合约
```bash
npx hardhat compile
```

### 3. 运行测试
```bash
npx hardhat test
```

### 4. 启动本地节点
```bash
npx hardhat node
```

## 测试覆盖

### 部署测试 (3个)
- 合约部署验证
- 逻辑合约地址设置
- Owner初始化

### 状态保留测试 (3个)
- 通过delegatecall保留计数器状态
- 保留用户特定状态
- 不同用户的独立状态

### 增加操作测试 (5个)
- 增加1
- 多次增加
- 增加指定数量
- 零值检查
- 用户计数器追踪

### 减少操作测试 (3个)
- 减少1
- 防止负数
- 多次减少

### 重置操作测试 (3个)
- 重置为零
- 权限检查
- Owner权限验证

### Owner管理测试 (4个)
- 更改owner
- 权限检查
- 零地址检查
- 新owner权限验证

### 逻辑合约更新测试 (4个)
- 更新逻辑合约
- 权限检查
- 零地址检查
- 状态保留验证

### 复杂场景测试 (3个)
- 复杂操作序列
- 状态一致性
- 事件发出

### Delegatecall行为测试 (3个)
- msg.sender保留
- 存储位置保留
- 多个delegatecall

### 错误处理测试 (3个)
- 错误消息处理
- 未初始化状态
- 不存在函数调用

**总计：36个测试用例**

## 使用示例

```javascript
// 部署合约
const logic = await Logic.deploy();
const proxy = await Proxy.deploy(logic.address);

// 初始化
await proxy.initialize(owner.address);

// 通过代理调用increment
await proxy.increment();
console.log(await proxy.getCounter()); // 1

// 增加指定数量
await proxy.incrementBy(5);
console.log(await proxy.getCounter()); // 6

// 减少
await proxy.decrement();
console.log(await proxy.getCounter()); // 5

// 重置（仅owner）
await proxy.reset();
console.log(await proxy.getCounter()); // 0

// 更新逻辑合约
const newLogic = await Logic.deploy();
await proxy.updateLogicContract(newLogic.address);
```

## 关键要点

### 1. 存储布局一致性
逻辑合约和代理合约必须有相同的存储变量顺序和类型，否则delegatecall会导致数据混乱。

```solidity
// Logic.sol
contract Logic {
    uint256 public counter;
    address public owner;
    mapping(address => uint256) public userCounters;
}

// Proxy.sol
contract Proxy {
    uint256 public counter;        // 相同顺序
    address public owner;          // 相同顺序
    mapping(address => uint256) public userCounters; // 相同顺序
}
```

### 2. Delegatecall的安全性
- 逻辑合约中的selfdestruct可能销毁代理
- 逻辑合约中的delegatecall可能导致重入
- 需要谨慎处理权限检查

### 3. 升级策略
- 新逻辑合约必须保持存储布局兼容
- 只能添加新状态变量，不能删除或重新排序
- 需要充分测试升级过程

## 事件

- `LogicContractUpdated(address indexed newLogicContract)` - 逻辑合约更新
- `DelegateCallExecuted(address indexed target, bytes data)` - Delegatecall执行
- `CounterIncremented(address indexed caller, uint256 newValue)` - 计数器增加
- `CounterDecremented(address indexed caller, uint256 newValue)` - 计数器减少
- `CounterReset(address indexed caller)` - 计数器重置
- `OwnerChanged(address indexed newOwner)` - Owner更改

## 提交信息

本实现已提交到GitHub：
https://github.com/papermoonio/2025-h2-solidity-native-polkadot-homework/tree/main/lesson-4

## 许可证

MIT

## 参考资源

- [Solidity Delegatecall文档](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#delegatecall-callcode-and-libraries)
- [代理模式最佳实践](https://docs.openzeppelin.com/contracts/4.x/api/proxy)
- [EIP-1967: Proxy Storage Slots](https://eips.ethereum.org/EIPS/eip-1967)
