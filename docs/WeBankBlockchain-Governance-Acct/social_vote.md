## 使用社交好友投票重置私钥

### 引入控制器

可通过手动传参创建控制器。
```java
SocialVoteManager socialVoteManager = new SocialVoteManager(governance, client, cryptoKeypair);
```

在配置了自动创建治理合约的场景下，也可通过Spring自动注入对象。

### 重置用户私钥

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />社交好友投票相关的操作在 SocialVoteManager 类中。<br />
<br />**具体调用示例：**<br />

```java
// 发起投票请求
TransactionReceipt t = socialVoteManager.requestResetAccount(newAddress, oldAddress);
// 执行投票
socialVoteManager.vote(requestId, true);
// 切换操作者
socialVoteManager.changeCredentials(user2);
socialVoteManager.vote(requestId, true);
// 切换操作者
socialVoteManager.changeCredentials(user1);
// 发起重置私钥操作
TransactionReceipt tr = socialVoteManager.resetAccount(newAddress, oldAddress);
```

<br />**参考执行返回日志：**<br />
```s
 Request reset account [ 0x508ec78ffb74ed7bd4b52090915c985ec9a2e15b ] to new account [ 0x781372c133eb2088df8de277f8fc3516a6be2c1f ]
credentials change to [ 0xa4382265425421a435cdb2428a67caf3e190f99d ] from [ 0x508ec78ffb74ed7bd4b52090915c985ec9a2e15b ]
 start vote of account config: [ 0xe529c202d32af26b0efba39661cbdcf9fcc67ec9 ] 
 --------------------------------------  
 voter: [ 0xa4382265425421a435cdb2428a67caf3e190f99d ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 2 ] 
 request address is [ 0x508ec78ffb74ed7bd4b52090915c985ec9a2e15b ] 
 vote type: [ change credential ] 
 threshod is [ 2 ] 
 weight is [ 1 ] 
 vote passed? [ false ] 

credentials change to [ 0x13ef955bb5c0a97b38671ecfb684d52109f4b5d0 ] from [ 0xa4382265425421a435cdb2428a67caf3e190f99d ]
 start vote of account config: [ 0xe529c202d32af26b0efba39661cbdcf9fcc67ec9 ] 
 --------------------------------------  
 voter: [ 0x13ef955bb5c0a97b38671ecfb684d52109f4b5d0 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 2 ] 
 request address is [ 0x508ec78ffb74ed7bd4b52090915c985ec9a2e15b ] 
 vote type: [ change credential ] 
 threshod is [ 2 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

credentials change to [ 0x508ec78ffb74ed7bd4b52090915c985ec9a2e15b ] from [ 0x13ef955bb5c0a97b38671ecfb684d52109f4b5d0 ]
External account reset to [ 0x781372c133eb2088df8de277f8fc3516a6be2c1f ] from [ 0x508ec78ffb74ed7bd4b52090915c985ec9a2e15b ] 
```

#### 发起重置用户私钥投票申请

**函数签名：**

```java
    TransactionReceipt requestResetAccount(String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- oldCredential  用户的外部账户的原私钥地址。
- newCredential  该账户被重置后的私钥地址


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 投票

**函数签名：**

```java
TransactionReceipt vote(String oldCredential, boolean agreed)
```

<br />**输入参数：**<br />

- oldCredential  申请变更账户的外部账户地址。
- agreed  是否同意


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 重置用户私钥

**函数签名：**

```java
TransactionReceipt resetAccount(String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- newCredential  该账户被重置后的私钥地址
- oldCredential  用户的外部账户的原私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。