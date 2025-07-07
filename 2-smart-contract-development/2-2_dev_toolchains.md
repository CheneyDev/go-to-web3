# 2.2: 智能合约开发工具链

为了高效、安全地开发智能合约，我们需要依赖专业的工具链。这些工具可以帮助我们完成代码编译、单元测试、本地部署、主网部署以及与合约交互等一系列工作。目前，最主流的工具链是基于 JavaScript/TypeScript 的 Hardhat。

## 1. Hardhat: 主流的以太坊开发环境

[Hardhat](https://hardhat.org/) 是一个灵活、可扩展且运行快速的以太坊开发环境。它几乎是当前 Web3 项目开发的事实标准。

### 1.1 主要特性

- **本地以太坊网络 (Hardhat Network)**: Hardhat 内置了一个专门为开发设计的本地以太坊节点。它能让你在本地快速部署和测试合约，无需连接到公共测试网，并且提供了强大的调试功能，如 `console.log` 和错误堆栈跟踪。
- **自动化测试框架**: Hardhat 与 Mocha、Chai 和 Ethers.js 等库深度集成，可以方便地编写和运行自动化单元测试和集成测试。
- **插件系统**: Hardhat 拥有一个丰富的插件生态系统。例如，`hardhat-ethers` 用于与 Ethers.js 集成，`hardhat-waffle` 提供了更便捷的测试断言，`hardhat-gas-reporter` 可以估算 Gas 消耗。
- **TypeScript 支持**: Hardhat 对 TypeScript 提供了一流的支持，可以帮助你编写类型更安全的代码。

### 1.2 核心工作流

使用 Hardhat 的一般开发流程如下：

1.  **项目初始化**:
    ```bash
    mkdir my-project && cd my-project
    npm init -y
    npm install --save-dev hardhat
    npx hardhat
    ```
    执行 `npx hardhat` 后，它会引导你创建一个示例项目，包含基本的目录结构、配置文件 (`hardhat.config.js`) 和示例合约。

2.  **编写合约**: 在 `contracts/` 目录下编写你的 Solidity 合约。

3.  **编译合约**:
    ```bash
    npx hardhat compile
    ```
    该命令会编译 `contracts/` 目录下的所有合约，并将编译结果（ABI 和字节码）存放在 `artifacts/` 目录下。

4.  **编写和运行测试**: 在 `test/` 目录下编写测试脚本（通常使用 JavaScript 或 TypeScript）。
    ```bash
    npx hardhat test
    ```
    Hardhat 会启动一个临时的本地网络，在上面运行你的测试用例。

5.  **编写部署脚本**: 在 `scripts/` 目录下编写部署脚本。
    ```javascript
    // scripts/deploy.js
    async function main() {
      const MyContract = await hre.ethers.getContractFactory("MyContract");
      const myContract = await MyContract.deploy();
      await myContract.deployed();
      console.log("MyContract deployed to:", myContract.address);
    }
    main().catch(console.error);
    ```

6.  **部署到网络**:
    ```bash
    # 部署到本地网络
    npx hardhat run scripts/deploy.js --network localhost

    # 部署到 Sepolia 测试网 (需在 hardhat.config.js 中配置)
    npx hardhat run scripts/deploy.js --network sepolia
    ```

## 2. Foundry: 基于 Rust 的高性能工具集 (进阶)

[Foundry](https://book.getfoundry.sh/) 是一个基于 Rust 构建的、速度极快的以太坊开发工具集。与 Hardhat 不同，Foundry 允许你直接用 Solidity 编写测试用例，这对于许多 Solidity 开发者来说更加直观。

### 2.1 核心组件

- **Forge**: Foundry 的测试框架。你可以用 Solidity 编写测试合约。
- **Cast**: Foundry 的命令行工具，用于执行智能合约调用、发送交易和读取链上数据。功能非常强大，可以看作是 Web3 的 "Swiss Army Knife"。
- **Anvil**: Foundry 的本地测试节点，类似于 Hardhat Network。
- **Chisel**: 一个高级的 Solidity REPL (Read-Eval-Print Loop)。

### 2.2 主要优势

- **速度快**: 由于其底层是 Rust，编译和测试速度通常比 Hardhat 快很多。
- **Solidity-Native 测试**: 直接用 Solidity 写测试，无需在 JavaScript/TypeScript 和 Solidity 之间切换上下文。
- **强大的 Fuzzing 功能**: Foundry 内置了强大的模糊测试 (Fuzz Testing) 功能，可以帮助发现边缘情况下的 bug。

对于追求极致性能和希望在 Solidity 环境中完成所有工作的开发者来说，Foundry 是一个绝佳的选择。

---

**总结**: 对于初学者，我们强烈建议从 **Hardhat** 开始。它拥有庞大的社区、丰富的文档和成熟的生态系统。当你对智能合约开发流程非常熟悉后，可以尝试使用 **Foundry** 来提升开发效率。 