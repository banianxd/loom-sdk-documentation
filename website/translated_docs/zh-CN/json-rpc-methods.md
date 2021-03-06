---
id: json-rpc-methods
title: JSON RPC方法
sidebar_label: JSON RPC方法
---
## 概述

为了与[Web3.js](https://github.com/ethereum/web3.js)兼容，LoomProvider添加了一些与 [以太坊JSON RPC方法](https://github.com/ethereum/wiki/wiki/JSON-RPC#json-rpc-api)完全兼容的方法，这些方法可以由Loom `QueryService`或`LoomProvider`直接调用，在本教程中我们将讨论`LoomProvider`。

### LoomProvider 调用 JSON RPC 方法

提供程序应该是客户端和Loom DAppChain之间的桥梁，下面的代码是`LoomProvider`实例化的示例，并调用`JSON RPC`来获取`eth_accounts`的帐户。

```javascript
const privateKey = CryptoUtils.generatePrivateKey();
const publicKey = CryptoUtils.publicKeyFromPrivateKey(privateKey);

// 创建客户端
const client = new Client(
  "default",
  "ws://127.0.0.1:46657/websocket",
  "ws://127.0.0.1:9999/queryws"
);

// 函数调用者的地址
const from = LocalAddress.fromPublicKey(publicKey).toString();

//实例化Loom提供程序
const loomProvider = new LoomProvider(client, privateKey);

// eth_accounts JSON RPC 调用
const jsonRPCString = '{"id": 1,"jsonrpc": "2.0", "method": "eth_accounts", "params": []}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
//   "id":1,
//   "jsonrpc": "2.0",
//   "result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]
// }
```

## eth_accounts

* * *

#### 说明

返回 LoomProvider 拥有的地址列表。

#### 参数

无

#### 返回值

`Array of DATA`, 20个字节 - 客户端拥有的地址。

#### 示例

```Javascript
// eth_accounts JSON RPC调用
const jsonRPCString = '{"id": 1,"jsonrpc": "2.0", "method": "eth_accounts", "params": []}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
//   "id":1,
//   "jsonrpc": "2.0",
//   "result": ["0x407d73d8a49eeb85d32cf465507dd71d507100c1"]
// }
```

## eth_blockNumber

* * *

#### 说明

返回最近完成的区块的编号。

#### 参数

无

#### 返回值

`QUANTITY` - 客户端所在的当前区块编号。

#### 示例

```Javascript
// eth_blockNumber JSON RPC调用
const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":83}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
//   "id":83,
//   "jsonrpc": "2.0",
//   "result": "0x4b7" // 1207
// }
```

## eth_call

* * *

#### 说明

立即执行一个新的信息调用，而不在区块俩上创建事务。

#### 参数

1. Object - 事务调用对象

- from: DATA, 20个字节 - 事务发送的地址
- to: DATA, 20个字节 - 交易所针对的地址。
- data: DATA - 散列方法签名和编码参数。 有关详细信息，请参阅以太坊合同ABI

#### 返回值

`DATA` -已执行合同的返回值。

#### 示例

```Javascript
// eth_call JSON RPC调用
const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_call","params":[{see above}],"id":1}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
//   "id":1,
//   "jsonrpc": "2.0",
//   "result": "0x"
// }
```

## eth_getBlockByNumber

* * *

#### 说明

按块编号返回有关块的信息。

#### 参数

1. `QUANTITY|TAG` -块编号的整数，或“最早”，“最新”，“待定”的默认块参数字符串。
2. `Boolean` - 如果为true，则返回完整的事务对象，如果为false，则仅返回事务的哈希值。

#### 返回值

`Object` - 区块对象，或`null`当没有找到任何区块时：

- `number`: `QUANTITY` - 块号。当挂起块时为null。
- `hash`: `DATA`, 32 字节 - 区块的哈希值。当区块待处理时为null。
- `parentHash`: DATA`, 32字节 - 父块的哈希值。
- `logsBloom`: `DATA`, 256字节 - 块日志的Bloom过滤器。 当挂起块时为null。
- `timestamp`: `QUANTITY` - 整理块时的unix时间戳。
- `transactions`: `Array` - 事务对象数组。 或32字节事务哈希，具体取决于给定的最新参数。

#### 示例

```Javascript
// eth_getBlockByNumber JSON RPC调用
const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x1b4", true],"id":1}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
// "id":1,
// "jsonrpc":"2.0",
// "result": {
//     "number": "0x1b4", // 436
//     "hash": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
//     "parentHash": "0x9646252be9520f6e71339a8df9c55e4d7619deeb018d2a3f2d21fc165dde5eb5",
//     "logsBloom": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
//     "timestamp": "0x54e34e8e" // 1424182926
//     "transactions": [{...},{ ... }]
//   }
// } }]
//   }
// }
```

## eth_getBlockByHash

* * *

#### 说明

通过哈希返回有关块的信息。

#### 参数

1. `DATA` - `32字节` - 块的哈希。
2. `Boolean` - 如果为true，则返回完整的事务对象，如果为false，则仅返回事务的哈希值。

#### 返回值

`Object` - 区块对象，或`null`当没有找到任何区块时：

- `number`: `QUANTITY` - 块号。当区块待处理时为null。
- `hash`: `DATA`, 32 字节 - 区块的哈希值。当区块待处理时为null。
- `parentHash`: `DATA`, 32字节 - 父块的哈希值。
- `logsBloom`: `DATA`, 256字节 - 块日志的Bloom过滤器。 当挂起块时为null。
- `timestamp`: `QUANTITY` - 整理块时的unix时间戳。
- `transactions`: `Array` - 事务对象数组。 或32字节事务哈希，具体取决于给定的最新参数。

#### 示例

```Javascript
// eth_getBlockByHash JSON RPC调用
const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_getTransactionByHash","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
// "id":1,
// "jsonrpc":"2.0",
// "result": {
//     "number": "0x1b4", // 436
//     "hash": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
//     "parentHash": "0x9646252be9520f6e71339a8df9c55e4d7619deeb018d2a3f2d21fc165dde5eb5",
//     "logsBloom": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
//     "timestamp": "0x54e34e8e" // 1424182926
//     "transactions": [{...},{ ...
  }]
//   }
// }
```

## eth_getCode

* * *

#### 说明

返回给定地址处的代码。

#### 参数

1. `DATA`,`20字节` - 地址

#### 返回值

`DATA` - 来自给定地址的代码。

#### 示例

```Javascript
// eth_getCode JSON RPC调用
const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_getCode","params":["0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b", "0x2"],"id":1}'

// Parse JSON是发送之前的必要步骤
await loomProvider.sendAsync(JSON.parse(jsonRPCString))

// 返回将是这样的
// {
//   "id":1,
//   "jsonrpc": "2.0",
//   "result": "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b600060078202905091905056"
// }
 
```

## eth_getFilterChanges

* * *

#### 说明

过滤器的轮询方法，返回自上次轮询以来发生的日志数组。

#### 参数

1. `QUANTITY` - 过滤器ID。

#### 返回值

`Array` -日志对象数组, 如果自上次轮询以来没有任何更改, 则为空数组。

- 对于使用 `eth_newBlockFilter` 创建的过滤器, 返回是块的哈希值 (`数据`, 32 字节), 例如 `["0x3454645634534......"]`。
- 对于使用 `eth_newBlockFilter` 创建的过滤器, 返回是事务的哈希值 (`数据`, 32 字节), 例如 `["0x6345343454645......"]`。
- 对于使用` eth_newFilter </ code>创建的过滤器，日志将是具有以下参数的对象：</p>

<ul>
<li><code>removed`: `TAG` - 由于链重新配置而删除日志时` true </ 0>。 如果启用了日志记录，则<code> false </ 0>。</li>
<li><code> logIndex </ code>：<code> QUANTITY </ code> - 区块中日志索引位置的整数。 当日志为待处理时为null。</li>
<li><code> transactionIndex`：` QUANTITY ` - 事务索引位置的整数，日志是由此创建的。当日志为待处理时为null。</li> 
    
    - ` transactionHash`：` DATA`，32字节 - 事务的哈希，这个日志根据此创建。当日志为待处理时为null。
    - `blockHash`: `DATA`, 32 字节 - 这个日志所在区块的哈希。当它待处理时为null。当日志待处理时为null。
    - `blockNumber`: `QUANTITY` - 这个日志所在的区块号。当它待处理时为null。当日志待处理时为null。
    - `address`: `DATA`, 20 字节 - 这个日志起源的地址。
    - `data`: `DATA` - 包含日志的一个或多个32字节的非索引参数。
    - `topics`: `Array of DATA` - 数组0到 4 32 字节 索引日志参数的`DATA`。 （在Solidity：第一个topic是事件（例如Deposit(address,bytes32,uint256)）签名的哈希，除非你用匿名说明符声明了该事件。）</ul></li> </ul> 
    
    #### 示例
    
    ```Javascript
    // eth_getFilterChanges JSON RPC调用
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_getFilterChanges","params":["0x16"],"id":73}'
    
    // Parse JSON是发送之前的必要步骤
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // 返回将是这样的
    // {
    //   "id":1,
    //   "jsonrpc":"2.0",
    //   "result": [{
    //     "logIndex": "0x1", // 1
    //     "blockNumber":"0x1b4", // 436
    //     "blockHash": "0x8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcfdf829c5a142f1fccd7d",
    //     "transactionHash":  "0xdf829c5a142f1fccd7d8216c5785ac562ff41e2dcfdf5785ac562ff41e2dcf",
    //     "transactionIndex": "0x0", // 0
    //     "address": "0x16c5785ac562ff41e2dcfdf829c5a142f1fccd7d",
    //     "data":"0x0000000000000000000000000000000000000000000000000000000000000000",
    //     "topics": ["0x59ebeb90bc63057b6515673c3ecf9438e5058bca0f92585014eced636878c9a5"]
    //     },{
    //       ...
    //     }]
    // }
    ```
    
    ## eth_getLogs
    
    * * *
    
    #### 说明
    
    返回与给定过滤器对象匹配的所有日志的数组。
    
    #### 参数
    
    1. `Object` - 过滤器选项：
    
    - `fromBlock`: `QUANTITY|TAG` - (可选，默认：`"latest"`) 整数区块号，或最后挖出来的区块链或”pending 待处理“为“latest“，还没有挖的事务为”earliest“。
    - `toBlock`: `QUANTITY|TAG` - (可选，默认：`"latest"`) 整数区块号，或最后挖出来的区块链或”pending 待处理“为“latest“，还没有挖的事务为”earliest“。
    - `address`: `DATA`|数组, 20 字节 - (可选) 合约地址或者日志应该由此起源的地址列表。
    - `topics`: `DATA`的数组 - (可选) 32 字节的`DATA` 主题的数组。 主题是与顺序相关的。 每个主题也可以是 `DATA` 的数组，其中有 "or" 选项。
    - `blockhash`: `DATA`, 32 字节 - (可选, 未来) 添加 EIP-234 后，blockhash 将成为新的过滤器选项，它使用32字节的哈希 blockHash 来限制返回到单个区块的日志。 使用 blockHash 等同于 fromBlock = toBlock = 带有哈希 blockHash 的区块号。 如果过滤器条件中出现了 blockHash，则 fromBlock 和 toBlock 都不会被允许。
    
    #### 返回值
    
    请参见 [eth_getFilterChanges](#eth-getfilterchanges)
    
    #### 示例
    
    ```Javascript
    // eth_getFilterChanges JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_getLogs","params":[{"topics":["0x000000000000000000000000a94f5374fce5edbc8e2a8697c15331677e6ebf0b"]}],"id":74}'
    
    // 解析JSON是发送等待loomProvider.sendAsync(JSON.parse(jsonRPCString)之前的必要步骤)
    ```
    
    Result see [eth_getFilterChanges](#eth-getfilterchanges)
    
    ## eth_getTransactionReceipt
    
    * * *
    
    #### Description
    
    Returns the receipt of a transaction by transaction hash.
    
    **Note** That the receipt is not available for pending transactions.
    
    #### Parameters
    
    1. `DATA`, 32 Bytes - hash of a transaction
    
    #### Returns
    
    `Object` - A transaction receipt object, or `null` when no receipt was found:
    
    - `transactionHash`: `DATA`, 32 Bytes - hash of the transaction.
    - `transactionIndex`: `QUANTITY` - integer of the transactions index position in the block.
    - `blockHash`: `DATA`, 32 Bytes - hash of the block where this transaction was in.
    - `blockNumber`: `QUANTITY` - block number where this transaction was in.
    - `from`: `DATA`, 20 Bytes - address of the sender.
    - `to`: `DATA`, 20 Bytes - address of the receiver. null when its a contract creation transaction.
    - `contractAddress`: `DATA`, 20 Bytes - The contract address created, if the transaction was a contract creation, otherwise null.
    - `logs`: `Array` - Array of log objects, which this transaction generated.
    - `status`: `QUANTITY` either 1 (success) or 0 (failure)
    
    #### Example
    
    ```Javascript
    // eth_getTransactionReceipt JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_getTransactionReceipt","params":["0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"],"id":1}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    // "id":1,
    // "jsonrpc":"2.0",
    // "result": {
    //      transactionHash: '0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238',
    //      transactionIndex:  '0x1', // 1
    //      blockNumber: '0xb', // 11
    //      blockHash: '0xc6ef2fc5426d6ad6fd9e2a26abeab0aa2411b7ab17f30a99d3cb96aed1d1055b',
    //      contractAddress: '0xb60e8dd61c5d32be8058bb8eb970870f07233155', // or null, if none was created
    //      logs: [{
    //          // logs as returned by getFilterLogs, etc.
    //      }, ...],
    //      status: '0x1'
    //   }
    // }
    ```
    
    ## eth_newBlockFilter
    
    * * *
    
    #### Description
    
    Creates a filter in the node, to notify when new pending transactions arrive. To check if the state has changed, call [eth_getFilterChanges](#eth-getfilterchanges).
    
    #### Parameters
    
    None
    
    #### Returns
    
    `QUANTITY` - A filter id.
    
    #### Example
    
    ```Javascript
    // eth_newBlockFilter JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_newBlockFilter","params":[],"id":73}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    //   "id":1,
    //   "jsonrpc":  "2.0",
    //   "result": "0x1" // 1
    // }
    ```
    
    ## eth_newFilter
    
    * * *
    
    #### Description
    
    Creates a filter object, based on filter options, to notify when the state changes (logs). To check if the state has changed, call [eth_getFilterChanges](#eth-getfilterchanges).
    
    ##### A note on specifying topic filters:
    
    Topics are order-dependent. A transaction with a log with topics [A, B] will be matched by the following topic filters:
    
    - `[]` "anything"
    - `[A]` "A in first position (and anything after)"
    - `[null, B]` "anything in first position AND B in second position (and anything after)"
    - `[A, B]` "A in first position AND B in second position (and anything after)"
    - `[[A, B], [A, B]]` "(A OR B) in first position AND (A OR B) in second position (and anything after)"
    
    #### Parameters
    
    1. `Object` - The filter options:
    
    - `fromBlock`: `QUANTITY|TAG` - (optional, default: "latest") Integer block number, or "latest" for the last mined block or "pending", "earliest" for not yet mined transactions.
    - `toBlock`: `QUANTITY|TAG` - (optional, default: "latest") Integer block number, or "latest" for the last mined block or "pending", "earliest" for not yet mined transactions.
    - `address`: `DATA|Array`, 20 Bytes - (optional) Contract address or a list of addresses from which logs should originate.
    - `topics`: `Array of DATA`, - (optional) Array of 32 Bytes DATA topics. Topics are order-dependent. Each topic can also be an array of DATA with "or" options.
    
    #### Returns
    
    `QUANTITY` - A filter id
    
    #### Example
    
    ```Javascript
    // eth_newFilter JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_newFilter","params":[{"topics":["0x12341234"]}],"id":73}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    //   "id":1,
    //   "jsonrpc": "2.0",
    //   "result": "0x1" // 1
    // }
    ```
    
    ## eth_sendTransaction
    
    * * *
    
    #### Description
    
    Creates new message call transaction or a contract creation, if the data field contains code.
    
    #### Parameters
    
    1. `Object` - The transaction object
    
    - `from`: `DATA`, 20 Bytes - The address the transaction is send from.
    - `to`: `DATA`, 20 Bytes - (optional when creating new contract) The address the transaction is directed to.
    - `data`: `DATA` - The compiled code of a contract OR the hash of the invoked method signature and encoded parameters. For details see Ethereum Contract ABI
    
    #### Returns
    
    `DATA`, 32 Bytes - the transaction hash, or the zero hash if the transaction is not yet available.
    
    Use [eth_getTransactionReceipt](#eth-gettransactionreceipt) to get the contract address, after the transaction was mined, when you created a contract.
    
    #### Example
    
    ```Javascript
    // eth_sendTransaction JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_sendTransaction","params":[{see above}],"id":1}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    //   "id":1,
    //   "jsonrpc": "2.0",
    //   "result": "0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331"
    // }
    ```
    
    ## eth_subscribe
    
    * * *
    
    #### Description
    
    It works by subscribing to particular events. The node will return a subscription id. For each event that matches the subscription a notification with relevant data is send together with the subscription id.
    
    #### Parameters
    
    1. `object` with the following (optional) fields
    
    - `address`, either an address or an array of addresses. Only logs that are created from these addresses are returned (optional)
    - `topics`, only logs which match the specified topics (optional)
    
    #### Returns
    
    Subscription id
    
    #### Example
    
    ```Javascript
    // eth_subscribe JSON RPC call
    const jsonRPCString = '{"id": 1, "method": "eth_subscribe", "params": ["logs", {"address": "0x8320fe7702b96808f7bbc0d4a888ed1468216cfd", "topics": ["0xd78a0cb8bb633d06981248b816e7bd33c2a35a6089241d099fa519e361cab902"]}]}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    //   "jsonrpc":"2.0",
    //   "id":2,
    //   "result":"0x4a8a4c0517381924f9838102c5a4dcb7"
    // }
    ```
    
    ## eth_uninstallFilter
    
    * * *
    
    #### Description
    
    Uninstalls a filter with given id. Should always be called when watch is no longer needed. Additonally Filters timeout when they aren't requested with [eth_getFilterChanges](#eth-getfilterchanges) for a period of time.
    
    #### Parameters
    
    1. `QUANTITY` - The filter id
    
    #### Returns
    
    `Boolean` - `true` if the filter was successfully uninstalled, otherwise `false`.
    
    #### Example
    
    ```Javascript
    // eth_uninstallFilter JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"eth_uninstallFilter","params":["0xb"],"id":73}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    //   "id":1,
    //   "jsonrpc": "2.0",
    //   "result": true
    // }
    ```
    
    ## net_version
    
    * * *
    
    #### Description
    
    Returns the current network id.
    
    #### Parameters
    
    None
    
    #### Returns
    
    `String` - The current network id.
    
    - "474747": Currently there's now network id defined, the number returned is simply `474747`
    
    #### Example
    
    ```Javascript
    // net_version JSON RPC call
    const jsonRPCString = '{"jsonrpc":"2.0","method":"net_version","params":[],"id":67}'
    
    // Parse JSON is a necessary step before send
    await loomProvider.sendAsync(JSON.parse(jsonRPCString))
    
    // Return should be something like
    // {
    //   "id":67,
    //   "jsonrpc": "2.0",
    //   "result": "474747"
    // }
    ```