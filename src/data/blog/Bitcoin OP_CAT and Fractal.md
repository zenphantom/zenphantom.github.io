---
author: PH
pubDatetime: 2025-03-20T15:22:00Z
title: Bitcoin OP_CAT and Fractal
slug: Bitcoin OP_CAT and Fractal
featured: true
draft: false
tags:
  - docs
description:
  Bitcoin OP_CAT and Fractal
---


# Bitcoin OP_CAT and Fractal

[一天3个OP_CAT协议，比特币生态要「变天」了？](https://www.feixiaohao.com/news/12896241.html)

web3 行业总是热点驱动学习，虽然不喜欢追高/追热点，但是工作需要就研究记录一下。

## Backgoround
1、OP_CAT 演进背景
OP_CAT (全称为OP_concatenation)原本是比特币中的一个操作码，用于将堆栈中的两个元素进行串联连接并压入堆栈顶部。
它存在于比特币 GitHub 存储库的第一个提交 4405b78 中，在 2010 年被中本聪禁用。
禁用原因就是可能反复调用造成堆栈溢出。
然而自从 Taproot 软分叉升级后（2021年11月14日在区块709,632中正式激活），tapscript 限制单个堆栈中最多 520 字节的数据来避免此问题，这也对重新启用该操作码成为可能。
2023 年 10 月，Ethan Heilman 提出了一项草案，通过重新定义现有的操作码 OP_SUCCESS126，计划将 OP_CAT 重新引入为 tapscript（用于支持 Taproot 的交易的脚本语言）的操作码。
后续被分配到 BIP347。

### 什么是 OP Code？
op code，中文称为操作码，是比特币原生脚本语言中执行特定操作的命令。
这些操作包括算术过程、数据操作和交易验证标准。

[Script Wiki](https://en.bitcoin.it/wiki/Script)

![其中有很多被禁用的](/assets/Snipaste_2024-10-13_14-19-26.png)
其中有很多被禁用的

OP_CAT 用于拼接数据，它将脚本堆栈顶部的两个元素连接为一个元素。例如，以下脚本：

```
<0xABCD> <0x1234> OP_CAT
stack 
```
将会变成：0xABCD1234

1. 最初 OP_CAT 出现在2009年8月30日 比特币的第一次代码提交：4405b78
```
case OP_CAT:
{
    // (x1 x2 -- out)
    if (stack.size() < 2)
        return false;
    valtype& vch1 = stacktop(-2);
    valtype& vch2 = stacktop(-1);
    vch1.insert(vch1.end(), vch2.begin(), vch2.end());
    stack.pop_back();
    if (stacktop(-1).size() > 520)
        return false;
}

```
2. 后续2010 年 7 月 30 日 ，中本聪在提交 757f076 中引入了堆栈大小项限制为 5000 字节：
```
case OP_CAT:
{
    for (int i = 0; i < vch1.size(); i++)
        vch1[i] |= vch2[i];
    // (x1 x2 -- out)
    if (stack.size() < 2)
        return false;
    valtype& vch1 = stacktop(-2);
    valtype& vch2 = stacktop(-1);
    vch1.insert(vch1.end(), vch2.begin(), vch2.end());
    stack.pop_back();
    if (stacktop(-1).size() > 5000)
        return false;
}
```
3. 仅两天后，中本聪就在提交 6ff5f71 中引入了在一次交易的脚本中的 200 个操作码的数量限制；通过以上两项：每个脚本 200 个操作码和每个堆栈项 5000 字节的限制，可以降低堆栈增长的风险，进一步提高安全性。
```
if (opcode > OP_16 && nOpCount++ > 200)
    return false;

```
4. 两周后，即 2010 年 8 月 16 日 ，中本聪在提交 4bd188c 中停用了各种操作码，包括 OP_CAT，同时引入了 520 字节的堆栈项大小限制（从之前的 5000 字节），没有明确解释这一变化。
```
if (vchPushValue.size() > 520)
    return false;
if (opcode > OP_16 && nOpCount++ > 200)
    return false;

if (opcode == OP_CAT ||
    opcode == OP_SUBSTR ||
    opcode == OP_LEFT ||
    opcode == OP_RIGHT ||
    opcode == OP_INVERT ||
    opcode == OP_AND ||
    opcode == OP_OR ||
    opcode == OP_XOR ||
    opcode == OP_2MUL ||
    opcode == OP_2DIV ||
    opcode == OP_MUL ||
    opcode == OP_DIV ||
    opcode == OP_MOD ||
    opcode == OP_LSHIFT ||
    opcode == OP_RSHIFT)
    return false;
```
### 为什么禁用了OP_CAT？
这个就像一个你在接手了一个离职同事的代码之后试图通过提交记录理解一样，在中本聪消失之前，他共删除了 15 个操作码。
例如，当 OP_CAT 与 OP_DUP（一个复制堆栈顶部项的操作码）结合使用时可能会将 1 字节的值扩展为超过 1 TB，通过将一个 1 字节的值推入堆栈，接着利用 OP_DUP 操作码进行复制，再通过 OP_CAT 操作码进行 40 次连接，便可能导致堆栈值膨胀至超过 1 TB ：
```    
    // 创建一个新的脚本构建器
    builder := txscript.NewScriptBuilder()
    // 向脚本中添加一个1字节的值
    builder.AddData([]byte{0xff})
    // 模拟40次OP_DUP和OP_CAT操作
    for i := 0; i < 40; i++ {
        builder.AddOp(txscript.OP_DUP)
        builder.AddOp(txscript.OP_CAT) // 假设启用OP_CAT操作码
    }
    // 构建脚本
    script, err := builder.Script()
    if err != nil {
        log.Fatalf("Failed to build script: %v", err)
    }

    // 输出生成的脚本
    fmt.Printf("Generated script: %x\n", script)
```

1. Bitcoin signet 测试网支持了很多软分叉提案或重大更新
2. BCH，bitcoincash.org/spec/may-2018-reenabled-opcodes.md
3. Catnet：用于测试比特币脚本中的 Circle STARK 验证器
4. Fractal Bitcoin：OP_CAT 在测试网上启用，主网支持将于 2024 年 9 月启动
```
case OP_CAT:
{
    if (stack.size() < 2)
        return set_error(serror, SCRIPT_ERR_INVALID_STACK_OPERATION);
    valtype& vch1 = stacktop(-2);
    valtype& vch2 = stacktop(-1);
    if (vch1.size() + vch2.size() > MAX_SCRIPT_ELEMENT_SIZE)
        return set_error(serror, SCRIPT_ERR_PUSH_SIZE);
    vch1.insert(vch1.end(), vch2.begin(), vch2.end());
    popstack(stack);
}
```


## 比特币原生脚本极简入门
签名指的是一套数学椭圆曲线计算的过程，有私钥、公钥、地址、签名数据、签名结果
私钥（Private Key）：随机生成的一个大整数
公钥（Public Key）：通过椭圆曲线加密（如 ECDSA 或 Schnorr 签名算法），可以从私钥推导出公钥。公钥是公开的，并且用于验证签名的有效性。私钥可以推导出公钥，但通过公钥无法反推出私钥，这是一种单向门限函数的特性
地址（Address）：为了缩短公钥，便于记录，所以公钥经过哈希处理后得到地址，比如BTC里是2次哈希（SHA-256 ， RIPEMD-160），EVM里是1次哈希Keccak-256 ，再截取哈希值的后20字节。--》隐藏知识点，就是BTC里无法验证EVM的原生签名（影响跨链、乐观证明等）。
总之，他们之间的关系是
- 推导过程：私钥 -> 公钥 -> 地址 
- 签名过程：私钥对签名数据进行签名得到签名结果。
- 验签过程：签名结果结合签名数据，再相同签名算法下，可以得到公钥。

1. BTC地址本质就是各种脚本
   每一种地址，对应的就是不同的脚本解锁条件。

   **P2PKH（Pay-to-PubKey-Hash）地址**
    地址格式: 以 1 开头的字符串（例如：1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa）。
    脚本锁定是 OP_DUP OP_HASH160 \<PubKeyHash> OP_EQUALVERIFY OP_CHECKSIG。

    **P2SH（Pay-to-Script-Hash）地址**
    地址格式: 以 3 开头的字符串（例如：3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy）。
    脚本锁定是 OP_HASH160 \<ScriptHash> OP_EQUAL。

    **Bech32（SegWit）地址（P2WPKH 和 P2WSH）**
    地址格式: 以 bc1 开头的字符串（例如：bc1qar0srrr7xnx0a2w5n3f4k2j2l5e58nljl8v5z）。
    特征: 包括 P2WPKH（用于单个公钥的SegWit地址）和 P2WSH（用于脚本哈希的SegWit地址）。

    **P2TR（Pay-to-Taproot）地址**
    地址格式: 以 bc1p 开头的字符串（例如：bc1p5cyxnuxmeuwuvkwfem96d8hvyyq8tqfdtdr2a2x4u4v8z46gse）。
    特征: P2TR 是 Taproot 地址类型，支持更高效的脚本处理和隐私保护。


2. P2PK地址解锁样例
   P2PK（pay-to-Pubkey）直接给某公钥支付，解锁条件是验签解出公钥

    https://mempool.space/zh/tx/0437cd7f8525ceed2324359c2d0ba26006d92d856a9c20fa0241106ee5a597c9#flow=&vout=0

    ![图片](/assets/Snipaste_2024-10-13_14-41-29.png)

    ![解锁条件](/assets/Snipaste_2024-10-13_14-43-43.png)

    这里的脚本就是
    ```
    OP_PUSHBYTES_65 <0x001122> OP_CHECKSIG
    41 
    04ae1a62fe09c5f51b13905f07f06b99a2f7159b2225f374cd378d71302fa28414e7aab37397f554a7df5f142c21c1b7303b8a0626f1baded5c72a704f7e6cd84c
    ac
    ```

    解锁的操作
    ![解锁的操作](/assets/Snipaste_2024-10-13_14-46-24.png)
    这里的签名对象和解签所用的数据，就是btc的核心数据对象（大概是该交易哈希，output、版本号等）。

3. P2PKH地址解锁样例
   
    ```    
    P2PKH（Pay-to-PubKey-Hash）
    scriptPubKey: OP_DUP OP_HASH160 \<pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG
    scriptSig: \<sig> \<pubKey>
    ```

    <> = 表示传入某些数据

    OP_DUP = 复制顶部堆栈

    OP_HASH160 = 将堆栈顶部值进行哈希计算

    PubKeyHash = 传入一个公钥的哈希

    OP_EQUALVERIFY = 检查栈顶两个元素是否相等

    OP_CHECKSIG = 检测签名是否相等

    ![图片](/assets/Snipaste_2024-10-13_14-48-56.png)


    https://mempool.space/zh/tx/0b6461de422c46a221db99608fcbe0326e4f2325ebf2a47c9faf660ed61ee6a4

    ![图片](/assets/Snipaste_2024-10-13_15-40-43.png)

    ```
    483045022100b95ef71baebf456275693eca9d474ed13acbabe2ca94a4b42510f3a16f20b9ec022075a93a7064b60fe82887f2ba65f6e5280b277ffdbf15e83e1116ef2b51aeb229012102be8f7ea648d3522731589bca6aaade20fd6767910f77f1c7ae2c51d1048c2abc

    OP_PUSHBYTES_72 48 
    <sig> 3045022100b95ef71baebf456275693eca9d474ed13acbabe2ca94a4b42510f3a16f20b9ec022075a93a7064b60fe82887f2ba65f6e5280b277ffdbf15e83e1116ef2b51aeb22901
    OP_PUSHBYTES_33 21
    <pubKey> 02be8f7ea648d3522731589bca6aaade20fd6767910f77f1c7ae2c51d1048c2abc
    ```
    OP_DUP 复制出\<pubKey>

    OP_HASH160 计算复制出的数据的hash160

    OP_PUSHBYTES_20 c6df807683c6a26f25eebf2add7017098a55cc8a 之前utxo记录的公钥哈希

    OP_EQUALVERIFY 与最初输入的\<pubKey>比对一致性

    OP_CHECKSIG 与与最初输入的\<sig>比对签名结果

### 比特币脚本小结
    当前BTC 脚本能做的
    - 线性执行模式，进行类似于if-else的分支操作
    - 可以对32位数字进行有限的算术运算，即加法和减法。
    - 可以将数据哈希化，且可以检查“整体的”ECDSA和Schnorr签名。

    不能做的
    - 没有循环、跳转、递归，按位操作，乘除法，即非图灵完备，编程能力非常有限。
    - 不能连接堆栈上的元素。
    - 没有读取并检查链上交易数据的能力
    - 没有内存，即只能通过stack操作信息。
    - 复杂功能函数的缺失：尤其是局部验签功能，字符串局部操作功能等
    - 缺乏存储：链上只能验证utxo合法性，即只存有公钥和数额，缺乏其他状态

### BTC与EVM合约的根本区别
    更地道的说法，BTC的是契约（Covenant），受制与UTXO模型与无存储架构

    1、契约是一次有效的交易合约，无法并行，等于是按区块出块来顺序调用，如需并行，则需要提前部署出大量的UTXO以供消费。

    2、存储的改变会颠覆Bitcore的架构，所以大概率发展需要沿用链下叠加协议的模式（runes、avm）