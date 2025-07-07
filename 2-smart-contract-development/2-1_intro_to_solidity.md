# 2.1: Solidity 语言入门

Solidity 是以太坊智能合约最主流的编程语言，它的语法与 JavaScript 和 C++ 等语言有相似之处。本章节将介绍 Solidity 的核心概念，帮助你快速上手。

## 1. 基本语法和数据类型

### 1.1 Hello, World!

一个最基础的 Solidity 合约看起来是这样的：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract HelloWorld {
    string public greet = "Hello, World!";
}
```

- **SPDX-License-Identifier**: 这是代码的开源许可证声明，是一个好的编程习惯。
- **pragma solidity ^0.8.20;**: 这是编译器版本指令，告诉编译器使用哪个版本的 Solidity 来编译代码。`^` 表示向上兼容，但不包括 `0.9.0` 及以上版本。
- **contract HelloWorld { ... }**: 这是合约的定义，类似于其他语言中的 `class`。
- **string public greet = "Hello, World!";**: 这是一个状态变量 (State Variable)，它会永久地存储在以太坊区块链上。`public` 关键字会自动为它创建一个 getter 函数，允许外部读取它的值。

### 1.2 数据类型 (Data Types)

Solidity 是静态类型语言，变量类型需要在编译时确定。常见的数据类型包括：

- **布尔型 (bool)**: `true` 或 `false`。
- **整型 (int / uint)**: `int` 是有符号整数，`uint` 是无符号整数。可以指定位数，如 `uint8`, `uint256` (默认)。
- **地址 (address)**: 用于存储以太坊地址（例如 `0xAb5801a7D398351b8bE11C439e05C5B3259aeC9B`）。它有两种类型：
  - `address`: 普通地址。
  - `address payable`: 可支付地址，拥有 `transfer` 和 `send` 方法，可以接收以太币。
- **字符串 (string)**: 动态大小的 UTF-8 编码字符串。
- **字节数组 (bytes)**: `bytes1` 到 `bytes32` 是定长字节数组。`bytes` 是动态大小的字节数组。在处理原始数据时，`bytes` 通常比 `string` 更高效。
- **数组 (array)**: `uint[] public numbers;` (动态数组)，`string[5] public names;` (静态数组)。
- **结构体 (struct)**: 用于定义复杂的数据结构。
  ```solidity
  struct Person {
      uint id;
      string name;
  }
  ```
- **映射 (mapping)**: 键值对存储，类似于哈希表或字典。`mapping(address => uint) public balances;` 定义了一个从地址到无符号整数的映射，常用于记录用户余额。

## 2. 合约结构、函数和状态变量

### 2.1 状态变量 (State Variables)

状态变量是永久存储在合约存储空间中的变量。它们的值会构成合约的"状态"，并且会被记录在区块链上。

```solidity
contract SimpleStorage {
    uint public myNumber; // 状态变量
}
```

### 2.2 函数 (Functions)

函数是合约中可执行的代码块。

```solidity
contract SimpleStorage {
    uint public myNumber;

    // 写入数据的函数
    function setNumber(uint _newNumber) public {
        myNumber = _newNumber;
    }

    // 读取数据的函数 (view)
    function getNumber() public view returns (uint) {
        return myNumber;
    }
}
```

**函数可见性 (Visibility Specifiers):**

- **`public`**: 任何账户（外部或内部）都可以调用。
- **`private`**: 只能在当前合约内部调用，子合约也无法调用。
- **`internal`**: 只能在当前合约及继承它的子合约中调用。
- **`external`**: 只能从外部调用，不能在合约内部调用（除非使用 `this.functionName()`）。

**函数状态可变性 (State Mutability):**

- **`view`**: 声明函数不会修改状态。它只能读取状态，不能写入。调用 `view` 函数不需要花费 Gas（如果是外部调用）。
- **`pure`**: 声明函数既不读取也不修改状态。例如，一个只对输入参数进行计算并返回结果的函数。
- **默认 (无修饰符)**: 函数可以读取和修改状态，执行时会改变区块链状态，需要花费 Gas。

## 3. 学习资源推荐

- **[CryptoZombies](https://cryptozombies.io/)**: 一个非常有趣的互动式教程，通过制作一个僵尸游戏来学习 Solidity。非常适合初学者。
- **[Solidity by Example](https://solidity-by-example.org/)**: 提供了大量简洁的代码示例，涵盖了 Solidity 的各种特性。
- **[Solidity 官方文档](https://docs.soliditylang.org/)**: 最权威、最全面的学习资料。当你对某个概念有疑问时，查阅官方文档是最好的选择。 