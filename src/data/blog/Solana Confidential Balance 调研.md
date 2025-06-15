---
author: PH
pubDatetime: 2025-05-20T00:00:00Z
title: Solana Confidential Transaction
slug: Solana Confidential Transaction
featured: true
draft: false
tags:
  - docs
  - solana
description:
  Solana Confidential
---

# 核心功能

由之前的 Confidential Transfers 扩展为 Confidential Balances，基于零知识证明，核心功能包括：

* 机密转账：在交易中隐藏转账金额

* 机密手续费：隐藏交易手续费

* 机密铸造与销毁：代币发行者可隐藏铸造、销毁的代币数量

  * 合规性：通过可选的“审计密钥”，监管机构可验证交易合法性而不暴露细节



注：此特性仅适用于Token2022代币，不适用于原生币 和 Legacy SPL Token。



# 当前状态

当前2025-04-09：

* 主网已支持，Rust-SDK已实现，可在Server端管理用户指令、生成零知识证明、处理密钥

* 用于钱包的官方JS库尚在开发中，计划2025下半年面世

&#x20;



# 操作流程

![](/assets/Solana_Confidential_Balance_调研/image-7.png)

1. 创建 confidential transfer extension 下的 mint account

2. 为sender/recipient创建confidential transfer extension下的 token account

3. Mint：向sender account 铸造代币

4. Deposit：从sender的公开余额中，向confidential pending balance中存入金额，需指定金额

5. Apply：把sender的pending balance 转为 confidential available balance，不可指定金额

6. Transfer：sender向recipient进行confidential transfer（机密转账），需指定金额

7. Apply：从recipient的pending balance申请金额到confidential available balance，不可指定金额

8. Withdraw: 从recipient的confidential available balance提现到public balance，需指定金额



# 余额模型

![](/assets/Solana_Confidential_Balance_调研/diagram.png)

## 为什么设置pending余额和available余额

攻击者可以通过抢先交易（front-running）来破环机密账户的使用。ZK根据账户的加密余额验证，考虑如下流程：

* 第一步：Alice生成了当前余额的ZK证明

* 第二步：恶意用户Bob向Alice转账代币且交易先被处理

* 第三步：Alice余额变化，第一步中的ZK证明失效，发起交易会被Token Program拒绝

如果恶意转账持续发起，Alice账户则无法使用。为了防止此类攻击，将账户加密余额分为两个独立部分：

* pending余额：账户的任何收入都添加到pending余额

* available余额：账户的任何支出都从available余额中扣除

# 机密转账Demo分析

https://solana.com/zh/docs/tokens/extensions/confidential-transfer



```plain&#x20;text
Payer: 7sTwHDmF2GTqPgFS37vXLeu6A2LcqASKM2y1pzz3xxPA
Mint keypair generated: 7aumuQ8Sm6AYKh9K59uoFSrZrvzcGeADKiXkJDcEv9Vh
Creating token mint with confidential transfer extension...
Mint Address: 7aumuQ8Sm6AYKh9K59uoFSrZrvzcGeADKiXkJDcEv9Vh
Mint Creation Transaction Signature: Signature: 4GjHApSTqm7qa4co5HA8wvCymb2aP6cfSPt3yrg6UsYb8k76tDz2f5qwsyTHwzHyLPRoFRP1Df4cCQQ15XLftuJd

Creating token account for owner: 7sTwHDmF2GTqPgFS37vXLeu6A2LcqASKM2y1pzz3xxPA
Token Account Address: 2AjUGpzi581BePUmeynbtVD9Ymyxn3zQr58Amo28PnFz
Create Token Account Transaction Signature: 4VtH3JynbaaQWDL6dh1mL59Z2PKDxBivaGpsp59Ljkgc2yeUnbg9R5Bi14ThRmCoio8jxX5Y2Kf9AChFwiodVPnc
Recipient owner: 7jCQxxvVo59tsmakoekQ3U7qjnQTSFWdMnPT8Pefjq49
Funding account 7jCQxxvVo59tsmakoekQ3U7qjnQTSFWdMnPT8Pefjq49 with 10000000 lamports...
Fund Transaction Signature: 5PLR6AN21C35YM4Eut6Et1cZZDoA47BTgvytrf7LYhix2yJYcr5sbMu5kNGjTCNSKFREWqrgMjHRtXDVAMuQEaJB
Creating token account for owner: 7jCQxxvVo59tsmakoekQ3U7qjnQTSFWdMnPT8Pefjq49
Token Account Address: DVtSFKBLpsPMqzmn9W9NPbRYSiX9m6hp7g1G6jTpEV8h
Create Token Account Transaction Signature: 4tKcwv2KRE5GPcWtVjGuAi4Ucbwc1b3NrC9DfewHVtu7m3cLJ3hArVT54i31PjZa9f7Hg3SWShbxLoe8nyfChRx9
Minting 10000 tokens to account: 2AjUGpzi581BePUmeynbtVD9Ymyxn3zQr58Amo28PnFz
Token Minting Transaction Signature: Signature: 29KNFP8rHn4NvtmMakDqGa1jjy691GKEsP3SaMtBMaMzmsHATH1MWYiJASyF2GF3orVz7GJGNP7C7Gm3xUxCAiVm

Depositing 10000 tokens to confidential pending balance...
Confidential Transfer Deposit Signature: Signature: 3ViYZYbKSTuGAYm2WR5hjqwffa98F5YGGi1Tz9qzhtR2qJooVDsGAasw8wTRPPctRx6R1UCTHqAFg9m6BNucX4cL

Applying pending balance for account: 2AjUGpzi581BePUmeynbtVD9Ymyxn3zQr58Amo28PnFz
Apply Pending Balance Signature: Signature: 38cjnw7o6CvuPeqEYMmQ1C7TUtkayPXoF2TrHD8uDkYJdkTvmqa8dad3aP2PvzWVGGaBfbujbPfgnZ3KKG1S5S83

Performing confidential transfer of 5000 tokens...
Creating equality proof context state account...
Create Equality Proof Context State Account: Signature: 5K97UkoR8FJYdxa8UZZzZbXKDzCyuSh9r486RWtmXLA9ZXfJPywxL7CVbFYkDHugXdEP7REYoFMGHBWhxcGHpNPV

Creating ciphertext validity proof context state account...
Create Ciphertext Validity Proof Context State Account: Signature: 2ZqRDCS5QPdPhnYLrJ2kUDF9QC5n6fSm9J7dhQ9Z6jfcNMJPyzQQwf9HWjgYrDd8qTVZH64S9wtwowFNmunkx5zj

Creating range proof context state account...
Create Range Proof Context State Account: Signature: 6m4XEa5mM9cBvb5AnZYUzVLb36VvxZFib37VXkq6bUJTMk4mKycgN7wo4jvc2SxAFFZeh8HbDYPXrkxYx5c7vy9

Executing confidential transfer transaction...
Confidential Transfer Signature: Signature: 3Wc3BsZdfSWGaTh2oNu6WqZ934UhzRZtuyurfjyjAK6grRpMeibVN615Aem5os29oZeEShwWkA7EvUSLNp7vQCRf

Closing proof context state accounts...
Close Equality Proof Context State Account: Signature: 32pTRgfV4VdgrDCT89TCwjYUwq3XGivawztxaCtxAT6v6y9EoiXcBVta9rDunAzn7gNSZiC8fB78jyG5tf6Kxyi9

Close Ciphertext Validity Proof Context State Account: Signature: 2nuTBS3keJCkjYGGxMUVogtZ42q767tZQNsgZEmKmnRCf16qEJUmBa6U9upswnmGa3dLM1KQibKbxCVFjUMj8GJV

Close Range Proof Context State Account: Signature: 46ugd7knHXadNo7LrWePXapUDx4oE7uVq6ytehZ3ZE6o8u3BmqY2AMcbLPrKy4FsaYyLKvQw6KLgHHh5GAjbohr3

Applying pending balance for account: DVtSFKBLpsPMqzmn9W9NPbRYSiX9m6hp7g1G6jTpEV8h
Apply Pending Balance Signature: Signature: sgz6UaDSgQFKu2Vqcmuqc5AzArS4Cukvvtx1qav5YDgkjXU8RXrkNiwYuChAWbAFtpVDssqqFrecx8jd67nNhKx

Withdrawing 2500 tokens from confidential available balance to public balance...
Creating equality proof context state account...
Create Equality Proof Context State Account: Signature: 3ZqnxzKHng2egY7iFrGi3tzGqYSQY7pLBgJpDBdtCzXKu7fEPLMTvCjfgFtPgaWDvt2Ae7mx12GaRuJf6UrWGoj8

Creating range proof context state account...
Create Range Proof Context State Account: Signature: 2cSLbxKPuuyXksKstrjR7RBPzYhJXAZXMusDPxESz1NHersV88McpWqH7znYr2pw31a3YsEJdbPUGAXuzkiU57z

Executing withdrawal transaction...
Withdraw Transaction Signature: Signature: hG9pB7TvHkx6o6PfbLGr92bKTcdemWezjE71jBGUP9woRRdpbvamWZJ31oCLReXsTvtMKXwBWzAfHf9ZyNqhi9G

Closing proof context state accounts...
Close Equality Proof Context State Account: Signature: Pmhvw2b7PRuKv5EEZyMovuSjx7QaDhf9y2UjrpkZyK1VAEj6Nd4se22XUtRARfnpbbtKvgRM5e8Bi91kyg4u3sx

Close Range Proof Context State Account: Signature: 3ofkGY558hhtYRKAdiGgG2MhaoAJXp6FtKS7WZzyXfPP7Z5abaR11hQysBP9cfFbbw2HyMPurpvw1hHBLQmfnprk


All operations completed successfully!
Sender Account: 2AjUGpzi581BePUmeynbtVD9Ymyxn3zQr58Amo28PnFz
Recipient Account: DVtSFKBLpsPMqzmn9W9NPbRYSiX9m6hp7g1G6jTpEV8h
```



大致流程：发送者deposit 10000 tokens，机密转账给接受者5000，接受者提现2500



## 创建代币Mint

```rust
// Create extension initialization parameters for the mint
    // This configures the confidential transfer extension on the mint
    let extension_initialization_params =
        vec![ExtensionInitializationParams::ConfidentialTransferMint {
            authority: Some(payer.pubkey()), // Authority that can modify confidential transfer settings
            auto_approve_new_accounts: true, // Automatically approve new accounts for confidential transfers
            auditor_elgamal_pubkey: None,    // No auditor for this example
        }];

    // Create and initialize the mint with the ConfidentialTransferMint extension
    let transaction_signature = token
        .create_mint(
            &payer.pubkey(),                 // Mint authority
            Some(&payer.pubkey()),           // Freeze authority (optional)
            extension_initialization_params, // Extension parameters
            &[mint],                         // Signers needed for the transaction
        )
        .await?;
```

可见相关配置有三个参数：

* authority：有权限修改此mint的机密转账配置的账户；若auto\_approve\_new\_accounts=false, 则由此账户审批新创建的token account

* auto\_approve\_new\_accounts：

  * true：可任意创建机密mint的token account

  * false：新token account需authority账户审批

* auditor\_elgamal\_pubkey：审计者ElGamal公钥

实验验证，auto\_approve\_new\_accounts=false 时，虽然可以创建token account，但执行deposit时会失败，无法执行机密转账，符合预期。报错信息：

```rust
Depositing 10000 tokens to confidential pending balance...
Error: client error: RPC response error -32002: Transaction simulation failed: Error processing Instruction 0: custom program error: 0x18; 5 log messages:
  Program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb invoke [1]
  Program log: ConfidentialTransferInstruction::Deposit
  Program log: Error: Account not approved for confidential transfers
  Program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb consumed 1882 of 200000 compute units
  Program TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb failed: custom program error: 0x18
```



创建代币Mint ：https://explorer.solana.com/tx/mnX2YZFEd385PQM6Cd1EiZbm7x9jFydDjAobBvGoZQ5ULcygwDc83UmEkneQb5345wEPuRFhvzwaNAjPXPE4yjR?cluster=devnet

![](/assets/Solana_Confidential_Balance_调研/image.png)

创建token account依然可成功：https://explorer.solana.com/tx/3jDtwZmKx72inhDJJaYybMANkbbFYM7xwR5nVLEu14erEgHy99MTp48zEuaAb6q4AqigPhizxLZWiq6XR2kT9jcs?cluster=devnet





## 创建Sender TokenAccount

https://solscan.io/tx/4VtH3JynbaaQWDL6dh1mL59Z2PKDxBivaGpsp59Ljkgc2yeUnbg9R5Bi14ThRmCoio8jxX5Y2Kf9AChFwiodVPnc?cluster=devnet

初始余额零值是加密过的：

![](/assets/Solana_Confidential_Balance_调研/image-1.png)



### 创建TokenAccount细节

```rust
#[repr(C)]
#[derive(Clone, Copy, Debug, Default, PartialEq, Pod, Zeroable)]
pub struct ConfidentialTransferAccount {
    /// `true` if this account has been approved for use. All confidential
    /// transfer operations for the account will fail until approval is
    /// granted.
    pub approved: PodBool,

    /// The public key associated with ElGamal encryption
    pub elgamal_pubkey: PodElGamalPubkey,

    /// The low 16 bits of the pending balance (encrypted by `elgamal_pubkey`)
    pub pending_balance_lo: EncryptedBalance,

    /// The high 48 bits of the pending balance (encrypted by `elgamal_pubkey`)
    pub pending_balance_hi: EncryptedBalance,

    /// The available balance (encrypted by `encryption_pubkey`)
    pub available_balance: EncryptedBalance,

    /// The decryptable available balance
    pub decryptable_available_balance: DecryptableBalance,

    /// If `false`, the extended account rejects any incoming confidential
    /// transfers
    pub allow_confidential_credits: PodBool,

    /// If `false`, the base account rejects any incoming transfers
    pub allow_non_confidential_credits: PodBool,

    /// The total number of `Deposit` and `Transfer` instructions that have
    /// credited `pending_balance`
    pub pending_balance_credit_counter: PodU64,

    /// The maximum number of `Deposit` and `Transfer` instructions that can
    /// credit `pending_balance` before the `ApplyPendingBalance`
    /// instruction is executed
    pub maximum_pending_balance_credit_counter: PodU64,

    /// The `expected_pending_balance_credit_counter` value that was included in
    /// the last `ApplyPendingBalance` instruction
    pub expected_pending_balance_credit_counter: PodU64,

    /// The actual `pending_balance_credit_counter` when the last
    /// `ApplyPendingBalance` instruction was executed
    pub actual_pending_balance_credit_counter: PodU64,
}
```

注意到：

* allow\_confidential\_credits：若false，则此账户会拒绝所有发来的机密转账

* allow\_non\_confidential\_credits：若false，则此账户会拒绝所有发来的转账



### pending balance credit counters

当调用`ApplyPendingBalance`指令时，

1. 客户端将pending余额和available余额的值相加，AES加密后设为`decryptable_available_balance`

2. 使用以下两个字段追踪pending变更计数器：

   * `expected_pending_balance_credit_counter`: 客户端创建`ApplyPendingBalance`指令时的`pending_balance_credit_counter`值

   * `actual_pending_balance_credit_counter`: `ApplyPendingBalance`指令被处理时的`pending_balance_credit_counter`值



expected/actual 两个值相等，意味着 `decryptable_available_balance`和`available_balance`相等。

当获取token account的`decryptable_available_balance`时，若expected/actual 两个值不相等，客户端需要查找近期能匹配到counter差异的 deposit/transfer 指令，计算出正确的余额。



### 余额对账处理

若expected/actual 两个值不相等，按以下步骤重新计算`decryptable_available_balance`：

1. 从token account中的`decryptable_available_balance`值开始，

2. 从最近的交易中，找到（actual - expected）个带有deposit/transfer指令（因为此处只需关注入账交易）的交易：

   1. 把deposit指令中的公开amount加上去

   2. 把transfer指令中的amount解密后加上去





## 向 Sender ATA 铸造token

mint 10000 tokens，数额公开

https://explorer.solana.com/tx/29KNFP8rHn4NvtmMakDqGa1jjy691GKEsP3SaMtBMaMzmsHATH1MWYiJASyF2GF3orVz7GJGNP7C7Gm3xUxCAiVm?cluster=devnet





## deposit：Sender ATA的token转到pending balance

https://solscan.io/tx/3ViYZYbKSTuGAYm2WR5hjqwffa98F5YGGi1Tz9qzhtR2qJooVDsGAasw8wTRPPctRx6R1UCTHqAFg9m6BNucX4cL?cluster=devnet

这一步是可以看到token变化值的：

![](/assets/Solana_Confidential_Balance_调研/image-2.png)



## apply：把pending balance转到available balance

https://explorer.solana.com/tx/38cjnw7o6CvuPeqEYMmQ1C7TUtkayPXoF2TrHD8uDkYJdkTvmqa8dad3aP2PvzWVGGaBfbujbPfgnZ3KKG1S5S83?cluster=devnet

到这一步，虽然不显示余额变化值，但仍能推断出available balance的数值：

![](/assets/Solana_Confidential_Balance_调研/image-3.png)



## 创建ZKP

相等性证明：

https://explorer.solana.com/tx/5K97UkoR8FJYdxa8UZZzZbXKDzCyuSh9r486RWtmXLA9ZXfJPywxL7CVbFYkDHugXdEP7REYoFMGHBWhxcGHpNPV?cluster=devnet

密文有效性证明：

https://explorer.solana.com/tx/2ZqRDCS5QPdPhnYLrJ2kUDF9QC5n6fSm9J7dhQ9Z6jfcNMJPyzQQwf9HWjgYrDd8qTVZH64S9wtwowFNmunkx5zj?cluster=devnet

范围证明：

https://explorer.solana.com/tx/6m4XEa5mM9cBvb5AnZYUzVLb36VvxZFib37VXkq6bUJTMk4mKycgN7wo4jvc2SxAFFZeh8HbDYPXrkxYx5c7vy9?cluster=devnet



## 机密转账

https://explorer.solana.com/tx/3Wc3BsZdfSWGaTh2oNu6WqZ934UhzRZtuyurfjyjAK6grRpMeibVN615Aem5os29oZeEShwWkA7EvUSLNp7vQCRf?cluster=devnet

从发送者available余额转5000tokens到接收者pending余额，但这一步是看不到转账金额的：

![](/assets/Solana_Confidential_Balance_调研/image-4.png)



相关代码：

```rust
let transfer_signature = token
        .confidential_transfer_transfer(
            sender_token_account,    // Source token account
            recipient_token_account, // Destination token account
            &sender_owner.pubkey(),  // Owner of the source account
            Some(&spl_token_client::token::ProofAccount::ContextAccount(
                equality_proof_context_state_pubkey, // Equality proof context state account
            )),
            Some(&ciphertext_validity_proof_account_with_ciphertext), // Ciphertext validity proof
            Some(&spl_token_client::token::ProofAccount::ContextAccount(
                range_proof_context_state_pubkey, // Range proof account
            )),
            amount,                   // Amount to transfer
            None,                     // Optional auditor info (none in this case)
            sender_elgamal_keypair,   // Sender's ElGamal keypair
            sender_aes_key,           // Sender's AES key
            recipient_elgamal_pubkey, // Recipient's ElGamal public key
            None,                     // Optional auditor ElGamal public key
            &[&sender_owner],       // Signers
        )
        .await?;
```



## 关闭Proof账户

相等性证明：

https://explorer.solana.com/tx/32pTRgfV4VdgrDCT89TCwjYUwq3XGivawztxaCtxAT6v6y9EoiXcBVta9rDunAzn7gNSZiC8fB78jyG5tf6Kxyi9?cluster=devnet

密文有效性证明：

https://explorer.solana.com/tx/2nuTBS3keJCkjYGGxMUVogtZ42q767tZQNsgZEmKmnRCf16qEJUmBa6U9upswnmGa3dLM1KQibKbxCVFjUMj8GJV?cluster=devnet

范围证明：

https://explorer.solana.com/tx/46ugd7knHXadNo7LrWePXapUDx4oE7uVq6ytehZ3ZE6o8u3BmqY2AMcbLPrKy4FsaYyLKvQw6KLgHHh5GAjbohr3?cluster=devnet



## apply：接收者将pending balance 转为 available balance

https://explorer.solana.com/tx/sgz6UaDSgQFKu2Vqcmuqc5AzArS4Cukvvtx1qav5YDgkjXU8RXrkNiwYuChAWbAFtpVDssqqFrecx8jd67nNhKx?cluster=devnet

这一步看不到金额数值：

![](/assets/Solana_Confidential_Balance_调研/image-5.png)



## withdraw：从available余额提现到public余额

创建相关ZKP后，进行withdraw：

https://explorer.solana.com/tx/hG9pB7TvHkx6o6PfbLGr92bKTcdemWezjE71jBGUP9woRRdpbvamWZJ31oCLReXsTvtMKXwBWzAfHf9ZyNqhi9G?cluster=devnet

这一步是能看到金额的：

![](/assets/Solana_Confidential_Balance_调研/image-6.png)



# 加密原理

使用公钥加密+对称加密方案：

* 公钥加密：Twisted ELGamal，加密转账金额、pending余额

* 对称加密：使用 AES-GCM-SIV，加密available余额



## Twisted ELGamal 加密

Twisted ELGamal 是标准ELGamal的一个变体，密文分为两部分：

* 加密消息的Pedersen承诺，此部分与公钥无关

* 一个“解密句柄”，用于将加密随机因子和特定的EIGamal公钥绑定，此部分与被加密消息无关



可以将其理解为一个支持ZKP的公钥加密算法。



## ELGamal 解密

使用ELGamal的弊端是解密效率低下，原因在于即使拥有正确的密钥，为了解密也必须求解一个离散对数问题，解密时间会随着被加密数字的位数指数增长。

32位消息解密几秒内可完成，但Solana中的`u64`余额被加密后就无法解密，因此会把`u64`拆分成两个32位的部分分开处理。



转账金额：限制为48位，拆分为两部分：

* amount\_lo: 低16位

* amount\_hi: 高32位



## 线性同态

明文x0，x1的和与差和密文ct0, ct1的和与差的解密值相等：

```rust
let (sk, pk) = PKE::keygen();

let ct_0 = PKE::encrypt(pk, x_0);
let ct_1 = PKE::encrypt(pk, x_1);

assert_eq!(x_0 + x_1, PKE::decrypt(sk, ct_0 + ct_1));
```

由于线性同态仅在密文使用同一公钥时才成立，因此转账金额必须分别使用发送方和接收方加密：

```rust
Transfer {
  amount_sender: PKE::encrypt(pubkey_sender, 10),
  amount_receiver: PKE::encrypt(pubkey_receiver, 10),
}
```

收到这种形式的转账指令后，Token Program可以在发送方和接收方账户中减去和增加密文金额：

```rust
Account {
    mint: Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB,
    owner: 5vBrLAPeMjJr9UfssGbjUaBmWtrXTg2vZuMN6L4c8HE6, // pubkey_sender
    amount: PKE::encrypt(pubkey_sender, 50) - PKE::encrypt(pubkey_sender, 10),
    ...
}

Account {
    mint: Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB,
    owner: 0x89205A3A3b2A69De6Dbf7f01ED13B2108B2c43e7, // pubkey_receiver
    amount: PKE::encrypt(pubkey_receiver, 50) + PKE::encrypt(pubkey_receiver, 10),
    ...
}
```

由于加密，Token Program无法检查金额有效性，例如余额50的账户不能向外账转70代币，因此转账指令需包含零知识证明。



## available balance

```thrift
struct ConfidentialTransferAccount {
  /// `true` if this account has been approved for use. All confidential
  /// transfer operations for
  /// the account will fail until approval is granted.
  approved: PodBool,

  /// The public key associated with ElGamal encryption
  encryption_pubkey: ElGamalPubkey,

  /// The pending balance (encrypted by `encryption_pubkey`)
  pending_balance: ElGamalCiphertext,

  /// The available balance (encrypted by `encryption_pubkey`)
  available_balance: ElGamalCiphertext,

  /// The decryptable available balance
  decryptable_available_balance: AeCiphertext,
}
```

`available_balance`是用Elgamal加密的密文，在客户端难以解密，仅用于创建交易指令时生成ZKP。



因此用`decryptable_available_balance`保存着使用AES加密的密文，客户端应该使用它解密得到available balance，相应的密钥作为独立密钥存在客户端，或者从所有者签名密钥中动态派生。



账户的available balance仅会在两个指令执行后变化，且这两个指令都需要传入`new_decryptable_available_balance`作为参数：

* `ApplyPendingBalance`：从pending余额apply到available余额

* `Transfer`：向外转账

因此available余额**仅由其所属账户控制**。



## pending balance

```thrift
struct ConfidentialTransferAccount {
  /// `true` if this account has been approved for use. All confidential
  /// transfer operations for
  /// the account will fail until approval is granted.
  approved: PodBool,

  /// The public key associated with ElGamal encryption
  encryption_pubkey: ElGamalPubkey,

  /// The low-bits of the pending balance (encrypted by `encryption_pubkey`)
  pending_balance_lo: ElGamalCiphertext,

  /// The high-bits of the pending balance (encrypted by `encryption_pubkey`)
  pending_balance_hi: ElGamalCiphertext,

  /// The available balance (encrypted by `encryption_pubkey`)
  available_balance: ElGamalCiphertext,

  /// The decryptable available balance
  decryptable_available_balance: AeCiphertext,
}
```

与available余额类似，可以为pending余额保存一个可解密的`decryptable_pending_balance`，但与available余额情况不同，pending余额会随着转账的到来而不断变化。由于`decryptable_pending_balance`相应的密钥只有账户所有者知道，`Transfer`指令的发送者无法更改接收者账户的可解密余额。

因此TokenProgram保存着两个ElGamal密文，将64位余额分成高32位和低32位两部分：

```thrift
struct ConfidentialTransferAccount {
  /// `true` if this account has been approved for use. All confidential
  /// transfer operations for
  /// the account will fail until approval is granted.
  approved: PodBool,

  /// The public key associated with ElGamal encryption
  encryption_pubkey: ElGamalPubkey,

  /// The low-bits of the pending balance (encrypted by `encryption_pubkey`)
  pending_balance_lo: ElGamalCiphertext,

  /// The high-bits of the pending balance (encrypted by `encryption_pubkey`)
  pending_balance_hi: ElGamalCiphertext,

  /// The available balance (encrypted by `encryption_pubkey`)
  available_balance: ElGamalCiphertext,

  /// The decryptable available balance
  decryptable_available_balance: AeCiphertext,
}
```

相应地，`Transfer`指令数据中的amount也分成两部分加密：

```thrift
// Actual cryptographic components are organized in `VerifyTransfer`
// instruction data
struct ConfidentialTransferInstructionData {
  /// The transfer amount encrypted under the sender ElGamal public key
  encrypted_amount_sender: ElGamalCiphertext,
  /// The low-bits of the transfer amount encrypted under the receiver
  /// ElGamal public key
  encrypted_amount_lo_receiver: ElGamalCiphertext,
  /// The high-bits of the transfer amount encrypted under the receiver
  /// ElGamal public key
  encrypted_amount_hi_receiver: ElGamalCiphertext,
}
```

一个很自然的分割64位数字的方案是将其分为高32位和低32位，这个长度解密的效率也足够高，但这个方案有个问题是：32位的`pending_balance_lo`容易溢出。例如：两笔交易amount都是`2^32-1`，`pending_balance_lo`值为`2^32`,这是一个33位的数字，密文的解密更难。



为了解决溢出问题，设计为：

* `pending_balance_lo`：低16位

* `pending_balance_hi`：高48位

并在account state中添加了2个字段：

```rust
struct ConfidentialTransferAccount {
  ... // `approved`, `encryption_pubkey`, available balance fields omitted

  /// The low bits of the pending balance (encrypted by `encryption_pubkey`)
  pending_balance_lo: ElGamalCiphertext,

  /// The high bits of the pending balance (encrypted by `encryption_pubkey`)
  pending_balance_hi: ElGamalCiphertext,

  /// The maximum number of `Deposit` and `Transfer` instructions that can credit
  /// `pending_balance` before the `ApplyPendingBalance` instruction is executed
  pub maximum_pending_balance_credit_counter: u64,

  /// The number of incoming transfers since the `ApplyPendingBalance` instruction
  /// was executed
  pub pending_balance_credit_counter: u64,
}
```

* `pending_balance_credit_counter`：记录上次`ApplyPendingBalance`指令以来的入账交易笔数。

* `maximum_pending_balance_credit_counter`：用于限制下次执行`ApplyPendingBalance`指令之前，此账户还能接受入账交易的笔数，这个值可以通过`ConfirureAccount`配置，通常应设置为`2^16`



对`Transfe`指令数据做如下更改：

* 交易amount限制为`48位`

* amount被拆分为两部分：

  * `encrypted_amount_lo_receiver`：低16位

  * `encrypted_amount_hi_receiver`：高32位

```thrift
// Actual cryptographic components are organized in `VerifyTransfer`
// instruction data
struct ConfidentialTransferInstructionData {
  /// The transfer amount encrypted under the sender ElGamal public key
  encrypted_amount_sender: ElGamalCiphertext,
  /// The low *16-bits* of the transfer amount encrypted under the receiver
  /// ElGamal public key
  encrypted_amount_lo_receiver: ElGamalCiphertext,
  /// The high *32-bits* of the transfer amount encrypted under the receiver
  /// ElGamal public key
  encrypted_amount_hi_receiver: ElGamalCiphertext,
}
```

这个设计是在ElGamal解密效率和confidential transfer可用性之间取得平衡的一个折衷选择。



考虑`maximum_pending_balance_credit_counter`设置为`2^16`:

* `encrypted_amount_lo_receiver`保存着一个最大16位的数字，这样即使在`2^16`个转入交易后，`pending_balance_lo`最大是`32位`可以容纳。

* 同理，`encrypted_amount_hi_receiver`最大32位，在在`2^16`个转入交易后，`pending_balance_hi`会是一个最大`48位`的数字。

  48位数字的解密很慢，但对绝大多数app来说，具有较高amount值的交易比较少，只有当账户接受到巨量的较高amount的交易才会出现这个情况。另外，客户端可以经常提交`ApplyPendingBalance`指令，把账户的pending余额刷新到available余额，这样可以防止`pending_balance_hi`值过于巨大。



## 零知识证明

Confidential Balance中使用的ZKP专门为此场景设计，用例简单，因此不需要任何的可信配置和复杂的电路设计。

主要包含两部分：

* Sigma协议

* Bulletproofs



### ELGamal

一条转账指令**必须包含**三个ElGamal公钥：发送方、接收方、审计方。

```rust
struct TransferPubkeys {
  source_pubkey: ElGamalPubkey,
  destination_pubkey: ElGamalPubkey,
  auditor_pubkey: ElGamalPubkey,
}

struct TransferData {
  transfer_pubkeys: TransferPubkeys,
}
```

如果没有审计者，那么`auditor_pubkey`就是 32 个零字节。



### 转账金额

金额被限制为48位，分为16位`amount_lo`和32位`amount_hi`，两者都需要用**三个ELGamal公钥**加密，且需要附带密文的**有效性证明**，以证明密文是正确生成的。



由于有效性证明仅能保证Twisted ELGamal密文的生成正确，无法证明其他属性，比如金额必须是正数。因此，还需要包含一个**范围证明**，以证明`amount_lo`和`amount_hi`分别是正16位值和正32位值。

```rust
struct TransferAmountEncryption {
  commitment: PedersenCommitment,
  source_handle: DecryptHandle,
  destination_handle: DecryptionHandle,
  auditor_handle: DecryptHandle,
}

struct TransferProof {
  validity_proof: ValidityProof,
  range_proof: RangeProof,
}

struct TransferData {
  ciphertext_lo: TransferAmountEncryption,
  ciphertext_hi: TransferAmountEncryption,
  transfer_pubkeys: TransferPubkeys,
  proof: TransferProof,
}
```



### 余额

用户还需证明账户有足够的余额进行转账，Bulletproofs支持证明聚合，因此这个证明聚合到了转账金额的范围证明中。





# 业务影响

TODO 不支持有哪些影响

## Web3钱包

* 向外转账：可选“使用机密转账”，开启后则使用机密模式转账

  * TODO 如何判断代币是否支持机密转账

  * TODO 确认能否将Deposit、Apply操作都放在一笔交易中

    * 待讨论 如果自动处理这个过程，Deposit/Apply金额多少合适？如果转多少就Deposit/Apply多少，容易被反推出交易金额

* 查看机密转账信息，包括金额、手续费

  * TODO 账户拥有者如何将加密数据进行解密的处理细节

* Deposit操作

  * 待讨论 是否单独支持此功能

* Apply操作

  * 待讨论 是否单独支持此功能

* Withdraw操作

  * 必须支持，否则用户无法把通过机密转账收到的代币提现到public余额



若不支持：

* 已有代币不受影响

* 用户无法向外机密转账

* 机密交易无法查看金额等信息

* 通过机密转账收到的代币无法提现使用或交易



## CEX

* 资产充提隐私化：用户可将资产汇聚到机密地址再充到CEX，降低链上行为被关联

* 对冲基金、做市商可以用机密余额隐藏大额持仓，避免市场狙击

* 交易所需开发升级相关系统，以支持机密余额等新特性



若不支持：

* 交易问题：通过机密转账充值到CEX账户的pending余额，后续在CEX无法正常交易

* 退币问题：会导致较多退币客诉，且退币功能也需要额外开发才能支持



## DEX

头部dex：orca、meteora、raydium

* MEV防护：DEX支持机密余额，可避免MEV机器人狙击大额订单

* 流动性保护：做市商、机构可隐藏持仓规模，降低被套利风险

* 流动性碎片化：可能出现“公开池”和“隐私池”分离，降低整体资金效率

* AMM：现有AMM协议依赖公开余额计算价格，合约需重构以支持加密余额的处理



TODO 产品形态细化



## Dapp

* 数据读取方式变化：用户需授权Dapp访问其加密余额

* 交互模式调整：

  * 借贷协议支持加密余额，通过ZKP证明抵押品价值，无需公开具体数额

  * DAO投票可隐藏成员持票数量，仅公布投票结果



## 签名SDK

* deposit

* apply

* ZKP 生成

* 机密转账交易处理

* withdraw



整体来说，机密余额会对Solana生态产生广泛的影响：

* 相关产品功能设计复杂度上升

* Solana可能吸引Monero、Zcash等隐私链的用户，改变公链竞争态势



# TODO

* [ ] 发行方mint时的机密等级配置

* [ ] ZKP创建、验证的性能

* [ ] ElGamal加解密的性能

# 相关资料

官方推文：https://x.com/solana\_devs/status/1909629525035741306

Confidential Transfer 官方文档：https://solana.com/zh/docs/tokens/extensions/confidential-transfer

EIGamal Encryption加密原理：https://spl.solana.com/confidential-token/deep-dive/encryption

ZK证明：https://spl.solana.com/confidential-token/deep-dive/zkps

HELIUS博文：https://www.helius.dev/blog/confidential-balances

官方 Confidential-Balance-Sample: https://github.com/solana-developers/Confidential-Balances-Sample

官方Demo：https://solana.com/zh/docs/tokens/extensions/confidential-transfer

