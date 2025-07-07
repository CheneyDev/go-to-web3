# 2.3: 第一个智能合约项目：SimpleStorage

理论学习之后，最好的巩固方式就是动手实践。本章将指导你从零开始，使用 Hardhat 创建、测试和部署一个名为 `SimpleStorage` 的简单智能合约。

这个合约的功能非常简单：
- 存储一个数字 (`uint`)。
- 允许任何人读取这个数字。
- 允许任何人更新这个数字。

## 1. 准备工作：初始化 Hardhat 项目

首先，确保你已经安装了 [Node.js](https://nodejs.org/) (推荐 LTS 版本)。然后按照 [2.2 章节](2-2_dev_toolchains.md) 的指引初始化一个 Hardhat 项目。

```bash
mkdir simplestorage-project
cd simplestorage-project
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox
npx hardhat
```

在 `npx hardhat` 的交互式引导中，选择 `Create a TypeScript project`，并同意所有默认选项。项目创建成功后，你的目录结构应该如下：

```
simplestorage-project/
├── contracts/
├── scripts/
├── test/
├── hardhat.config.ts
└── package.json
```

## 2. 编写 `SimpleStorage` 合约

删除 `contracts/` 目录下的 `Lock.sol` 示例文件，并创建一个新文件 `SimpleStorage.sol`。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleStorage {
    uint private favoriteNumber;

    // 定义一个事件，当数字被改变时触发
    event NumberChanged(uint indexed oldNumber, uint indexed newNumber, address changer);

    function store(uint _newNumber) public {
        uint oldNumber = favoriteNumber;
        favoriteNumber = _newNumber;
        emit NumberChanged(oldNumber, _newNumber, msg.sender);
    }

    function retrieve() public view returns (uint) {
        return favoriteNumber;
    }
}
```

- **`favoriteNumber`**: 这是一个 `private` 的状态变量，只能在合约内部访问。
- **`store(uint _newNumber)`**: 这是一个公共函数，用于更新 `favoriteNumber` 的值。它还触发了一个 `NumberChanged` 事件。
- **`retrieve()`**: 这是一个 `view` 函数，用于读取 `favoriteNumber` 的当前值。
- **`event NumberChanged(...)`**: 事件 (Event) 是合约与外部世界（例如你的 DApp 前端）通信的重要机制。当状态发生变化时，可以发出事件，外部应用可以监听这些事件并作出反应。`msg.sender` 是一个全局变量，表示调用当前函数的账户地址。

## 3. 编写和运行单元测试

测试是确保智能合约安全可靠的关键步骤。删除 `test/` 目录下的 `Lock.ts` 示例文件，并创建一个新文件 `SimpleStorage.ts`。

```typescript
import { ethers } from "hardhat";
import { expect } from "chai";
import { SimpleStorage } from "../typechain-types";

describe("SimpleStorage", function () {
  let simpleStorage: SimpleStorage;

  beforeEach(async function () {
    const SimpleStorageFactory = await ethers.getContractFactory("SimpleStorage");
    simpleStorage = await SimpleStorageFactory.deploy();
    await simpleStorage.deployed();
  });

  it("Should start with a favorite number of 0", async function () {
    const currentValue = await simpleStorage.retrieve();
    expect(currentValue).to.equal(0);
  });

  it("Should update the favorite number when store is called", async function () {
    const newValue = 42;
    const tx = await simpleStorage.store(newValue);
    await tx.wait(); // 等待交易被打包

    const currentValue = await simpleStorage.retrieve();
    expect(currentValue).to.equal(newValue);
  });
  
  it("Should emit a NumberChanged event when store is called", async function () {
    const newValue = 77;
    await expect(simpleStorage.store(newValue))
      .to.emit(simpleStorage, "NumberChanged")
      .withArgs(0, newValue, (await ethers.getSigners())[0].address);
  });
});
```

- **`describe` 和 `it`**: 来自 Mocha 测试框架，用于组织测试用例。
- **`beforeEach`**: 在每个 `it` 测试用例运行前都会执行一次，这里我们用它来部署一个新的 `SimpleStorage` 合约，确保测试环境是干净的。
- **`expect`**: 来自 Chai断言库，用于验证结果是否符合预期。
- **`ethers.getContractFactory`**: Hardhat-Ethers 插件提供的功能，用于获取合约的"工厂"实例。
- **`to.emit(...).withArgs(...)`**: 一个强大的测试功能，用于验证合约是否正确地触发了事件以及事件参数是否正确。

现在，运行测试：

```bash
npx hardhat test
```

如果一切顺利，你应该会看到所有测试都通过了。

## 4. 部署到测试网

### 4.1 获取测试网 ETH

部署合约需要支付 Gas 费用，即使在测试网上也是如此。你需要从 "水龙头" (Faucet) 获取免费的测试网 ETH。

1.  **安装钱包**: 安装 [MetaMask](https://metamask.io/) 浏览器插件钱包。
2.  **切换网络**: 在 MetaMask 中，将网络切换到 "Sepolia" 测试网络。
3.  **获取 ETH**: 访问一个 Sepolia 水龙头网站，例如 [sepoliafaucet.com](https://sepoliafaucet.com/) 或 [infura.io/faucet/sepolia](https://www.infura.io/faucet/sepolia)。根据网站提示输入你的钱包地址，获取测试用的 ETH。这可能需要几分钟的时间。

### 4.2 配置 Hardhat

为了让 Hardhat 能将合约部署到 Sepolia，你需要提供三样东西：
- Sepolia 测试网的 RPC URL。
- 你钱包的私钥 (Private Key)。
- Etherscan 的 API 密钥（用于验证合约）。

**强烈建议**: 使用 `.env` 文件来管理这些敏感信息，避免将它们硬编码到代码中。

首先，安装 `dotenv` 包：
```bash
npm install --save-dev dotenv
```

然后，在项目根目录创建 `.env` 文件：
```
# .env
SEPOLIA_RPC_URL="YOUR_SEPOLIA_RPC_URL"
PRIVATE_KEY="YOUR_WALLET_PRIVATE_KEY"
ETHERSCAN_API_KEY="YOUR_ETHERSCAN_API_KEY"
```

- **`SEPOLIA_RPC_URL`**: 你可以从 [Infura](https://www.infura.io/) 或 [Alchemy](https://www.alchemy.com/) 等节点服务商处免费获取。
- **`PRIVATE_KEY`**: 在 MetaMask 中，点击账户详情 -> 导出私钥。**警告：私钥绝不能泄露给任何人！**
- **`ETHERSCAN_API_KEY`**: 在 [Etherscan](https://etherscan.io/) 网站注册一个账户，然后免费创建一个 API 密钥。

最后，修改 `hardhat.config.ts` 文件，使其能够读取 `.env` 变量并配置网络：

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import "dotenv/config"; // 导入 dotenv

const SEPOLIA_RPC_URL = process.env.SEPOLIA_RPC_URL || "https://sepolia.infura.io/v3/your-infura-id";
const PRIVATE_KEY = process.env.PRIVATE_KEY || "your-private-key";
const ETHERSCAN_API_KEY = process.env.ETHERSCAN_API_KEY || "your-etherscan-api-key";

const config: HardhatUserConfig = {
  solidity: "0.8.20",
  networks: {
    sepolia: {
      url: SEPOLIA_RPC_URL,
      accounts: [PRIVATE_KEY],
      chainId: 11155111,
    },
  },
  etherscan: {
    apiKey: ETHERSCAN_API_KEY,
  },
};

export default config;
```

### 4.3 编写部署脚本

删除 `scripts/` 目录下的 `deploy.ts` 示例文件，并创建一个新文件 `deploySimpleStorage.ts`。

```typescript
import { ethers, run } from "hardhat";

async function main() {
  const SimpleStorageFactory = await ethers.getContractFactory("SimpleStorage");
  console.log("Deploying contract...");
  const simpleStorage = await SimpleStorageFactory.deploy();
  await simpleStorage.deployed();
  console.log(`Deployed contract to: ${simpleStorage.address}`);

  // 如果在测试网，则等待几个区块确认后再验证
  if (process.env.ETHERSCAN_API_KEY) {
    console.log("Waiting for block confirmations...");
    await simpleStorage.deployTransaction.wait(6);
    await verify(simpleStorage.address, []);
  }
}

async function verify(contractAddress: string, args: any[]) {
  console.log("Verifying contract...");
  try {
    await run("verify:verify", {
      address: contractAddress,
      constructorArguments: args,
    });
  } catch (e: any) {
    if (e.message.toLowerCase().includes("already verified")) {
      console.log("Already verified!");
    } else {
      console.log(e);
    }
  }
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```
这个脚本不仅部署了合约，还会自动在 Etherscan 上验证合约源码。

### 4.4 执行部署

现在，运行以下命令将合约部署到 Sepolia 测试网：

```bash
npx hardhat run scripts/deploySimpleStorage.ts --network sepolia
```

部署成功后，你会在终端看到合约的地址。

## 5. 与合约交互

部署成功后，你可以在 Etherscan 上查看你的合约。访问 `https://sepolia.etherscan.io/address/你的合约地址`。

在 Etherscan 的 "Contract" 标签页，你会看到你的合约源码已被验证。你还可以使用 "Read Contract" 和 "Write Contract" 功能与你的合约进行交互。

- **`retrieve`**: 点击 "Read Contract" 下的 `retrieve` 按钮，可以看到当前存储的数字（初始为 0）。
- **`store`**: 点击 "Write Contract"，连接你的 MetaMask 钱包，然后在 `store` 函数下输入一个新数字，点击 "Write" 并确认交易。交易成功后，再次调用 `retrieve`，你就会发现数字已经被更新了。

恭喜你！你已经成功地在以太坊测试网上部署并交互了你的第一个智能合约！ 