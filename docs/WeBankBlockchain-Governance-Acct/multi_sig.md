## 使用多签制治理模式

### 引入治理合约创建器
SDK提供了可通过手动传参，创建GovernContractInitializer对象。
```java
GovernContractInitializer GovernContractInitializer = new GovernContractInitializer(client, cryptoKeypair);
```

或通过自动注入的方式，则用户私钥为默认配置的用户私钥来操作智能合约。
```java
// 自动注入GovernAccountInitializer对象
@Autowired private GovernContractInitializer governContractInitializer;
```

### 创建治理合约
例如，以下平台方选择了治理委员会的治理模式，一共有三个参与者参与治理，治理的规则为任意的交易请求获得其中两方的同意，即可获得通过。那么我们接下来将创建一个治理账户。

<br />**具体调用示例：**<br />

```java
// 1. 配置治理账户信息
GovernAccountGroup governAccountGroup = new GovernAccountGroup();
// 投票阈值2， 初始设置3个治理账户。
governAccountGroup.setThreshold(2);
governAccountGroup.addGovernUser("user1", governanceUser1Keypair.getAddress());
governAccountGroup.addGovernUser("user2", governanceUser2Keypair.getAddress());
governAccountGroup.addGovernUser("user3", governanceUser3Keypair.getAddress());
// 2. 创建治理合约
WEGovernance governance = governAccountInitializer.createGovernAccount(governAccountGroup);
```

执行后返回日志：
```s
...
After add governUser: GovernAccountGroup default group info: 
GovernUser[user1]: weight is [1] external account is [0x1fa875988195e30a9b7e3ddaa7a2870bc84dc468]
GovernUser[user2]: weight is [1] external account is [0xac3fb1d6e748697e40672ec1986e58da7d762696]
GovernUser[user3]: weight is [1] external account is [0x2bdb3029fa9eca8a640101b440ad8de07b015813]
threshold is [2], total weight is [3]
-------------------------------------------------------------------------------------------------
Governance account create succeed [ 0xfd667db2bf205fef03ee8c303f06feaf8e20f3b8 ] 
```

<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

### 创建管理员模式的控制器

治理委员会模式下的管理功能均位于 VoteModeGovernManager 类中。

可通过手动传参创建管理员模式的控制器。
```java
VoteModeGovernManager voteModeGovernManager = new VoteModeGovernManager(governance, client, cryptoKeypair);
```

### 调用控制接口
控制接口包括了重置私钥、冻结账户、解冻账户、注销账户、设置治理账户投票阈值、添加治理账户、删除治理账户等功能。

<br />在本模式下，执行任何账户相关的业务操作需要遵循以下步骤：<br />

1. 发起一个投票请求；
2. 治理账户成员赞同该投票；
3. 投票发起者确认投票已经通过后，发起操作。

#### 治理委员会成员投票
<br />我们首先来介绍下通用的投票接口：<br />

**具体调用示例：**

```java
TransactionReceipt tr = voteModeGovernManager.vote(requestId, true);
```

<br />**函数签名：**<br />

```java
TransactionReceipt vote(BigInteger requestId, boolean agreed)
```

<br />**输入参数：**<br />
- requestId  发起投票的requestId。
- agreed 是否同意，true/false

<br />**返回参数：**<br />
- TransactionReceipt 交易回执


#### 重置用户私钥

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestResetAccount(address1, address2);
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user2);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user1);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.resetAccount(requestId, address2, address1);
```


<br />**参考执行返回日志：**<br />
```s
 Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request to reset account, from [ 0xb6a42b5dcdb7b49fce99ec708a42355e042b2d56 ] to [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ]
Vote request id is [ 10001 ]
 start vote, Request id: [ 10001 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10001 ] 
 request address is [ 0xb6a42b5dcdb7b49fce99ec708a42355e042b2d56 ] 
 vote type: [ change credential ] 
 threshod is [ 2 ] 
 weight is [ 1 ] 
 vote passed? [ false ] 

credentials change to [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] from [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ]
 start vote, Request id: [ 10001 ] 
 --------------------------------------  
 voter: [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10001 ] 
 request address is [ 0xb6a42b5dcdb7b49fce99ec708a42355e042b2d56 ] 
 vote type: [ change credential ] 
 threshod is [ 2 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

credentials change to [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] from [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ]
reset account succeed, from [ 0xb6a42b5dcdb7b49fce99ec708a42355e042b2d56 ] to [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ]
```

##### 发起重置用户私钥投票申请

**函数签名：**

```java
BigInteger requestResetAccount(String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- oldCredential  用户的外部账户的原私钥地址。
- newCredential  该账户被重置后的私钥地址


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


##### 重置用户私钥

**函数签名：**

```java
TransactionReceipt resetAccount(BigInteger requestId, String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- newCredential  该账户被重置后的私钥地址
- oldCredential  用户的外部账户的原私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 冻结普通账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestFreezeAccount(address2);
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user1);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user2);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.freezeAccount(requestId, address2);
```

<br />**参考执行返回日志：**<br />
```s
Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request to freeze external account [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ]
Vote request id is [ 10002 ]
credentials change to [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] from [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ]
 start vote, Request id: [ 10002 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10002 ] 
 request address is [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] 
 vote type: [ freeze account ] 
 threshod is [ 2 ] 
 weight is [ 1 ] 
 vote passed? [ false ] 

credentials change to [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] from [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ]
 start vote, Request id: [ 10002 ] 
 --------------------------------------  
 voter: [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10002 ] 
 request address is [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] 
 vote type: [ freeze account ] 
 threshod is [ 2 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

freeze account [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] succeed 
```

##### 发起冻结用户账户投票申请

**函数签名：**

```java
BigInteger requestFreezeAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


##### 冻结用户账户

**函数签名：**

```java
TransactionReceipt freezeAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 解冻普通账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestUnreezeAccount(address2);
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user1);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user2);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.unfreezeAccount(requestId, address2);
```

<br />**参考执行返回日志：**<br />
```s
Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request to unfreeze external account [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ]
Vote request id is [ 10003 ]
credentials change to [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] from [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ]
 start vote, Request id: [ 10003 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10003 ] 
 request address is [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] 
 vote type: [ unfreeze account ] 
 threshod is [ 2 ] 
 weight is [ 1 ] 
 vote passed? [ false ] 

credentials change to [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] from [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ]
 start vote, Request id: [ 10003 ] 
 --------------------------------------  
 voter: [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10003 ] 
 request address is [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] 
 vote type: [ unfreeze account ] 
 threshod is [ 2 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

unfreeze account [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] succeed 
```

##### 发起解冻用户账户投票申请

**函数签名：**

```java
BigInteger requestunfreezeAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


##### 解冻用户账户

**函数签名：**

```java
TransactionReceipt unfreezeAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 账户强制注销

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestCancelAccount(address2);
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user2);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user1);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.cancelAccount(requestId, address2);
```

<br />**参考执行返回日志：**<br />
```s
Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request to cancel external account [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ]
Vote request id is [ 10004 ]
credentials change to [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] from [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ]
 start vote, Request id: [ 10004 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10004 ] 
 request address is [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] 
 vote type: [ cancel account ] 
 threshod is [ 2 ] 
 weight is [ 1 ] 
 vote passed? [ false ] 

credentials change to [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] from [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ]
 start vote, Request id: [ 10004 ] 
 --------------------------------------  
 voter: [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10004 ] 
 request address is [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] 
 vote type: [ cancel account ] 
 threshod is [ 2 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

cancel account [ 0xc952d06d9b5d7179b533901aab9146f5e99afb7e ] succeed 

```


##### 发起注销用户账户投票申请

**函数签名：**

```java
BigInteger requestCancelAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。



##### 注销用户账户

**函数签名：**

```java
TransactionReceipt cancelAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 设置治理账户投票的阈值

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestResetThreshold(newThreshold);
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user2);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user1);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.resetThreshold(requestId, newThreshold);
```

<br />**参考执行返回日志：**<br />
```s
Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request reset threshold to [ 1 ]
Vote request id is [ 10005 ]
credentials change to [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] from [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ]
 start vote, Request id: [ 10005 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10005 ] 
 request address is [ 0x56542c78f625648d9a93f12099264113fad84197 ] 
 vote type: [ reset threshold ] 
 threshod is [ 2 ] 
 weight is [ 1 ] 
 vote passed? [ false ] 

credentials change to [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] from [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ]
 start vote, Request id: [ 10005 ] 
 --------------------------------------  
 voter: [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10005 ] 
 request address is [ 0x56542c78f625648d9a93f12099264113fad84197 ] 
 vote type: [ reset threshold ] 
 threshod is [ 2 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

reset threshold [ 1 ] succeed 
```

##### 发起重置阈值投票申请

**函数签名：**

```
BigInteger requestResetThreshold(int newThreshold)
```

<br />**输入参数：**<br />

- newThreshold  新阈值。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


##### 设置新阈值

**函数签名：**

```
TransactionReceipt resetThreshold(BigInteger requestId, int threshold)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- newThreshold  新阈值。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 治理账户删除一个投票账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestRemoveGovernAccount(user3Address);
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user2);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(user1);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.removeGovernAccount(requestId, user3Address);
```

<br />**参考执行返回日志：**<br />
```s
Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request to remove governance account [ 0x92217c219165d5af708afea698a910fe9ba4b0c9 ]
Vote request id is [ 10006 ]
credentials change to [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] from [ 0xc48cc67b2cb6cfa1ba599f98e30adf2dd12e2f47 ]
 start vote, Request id: [ 10006 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10006 ] 
 request address is [ 0x92217c219165d5af708afea698a910fe9ba4b0c9 ] 
 vote type: [ reset weight ] 
 threshod is [ 1 ] 
 weight is [ 1 ] 
 vote passed? [ true ] 

Contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] remove governance account [ 0x92217c219165d5af708afea698a910fe9ba4b0c9 ] succeed 
```

##### 发起删除一个治理账户投票申请

**函数签名：**

```
BigInteger requestRemoveGovernAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


##### 删除一个投票账户

**函数签名：**

```java
TransactionReceipt removeGovernAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 治理账户添加一个投票新账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```java
// 发起投票请求
BigInteger requestId = voteModeGovernManager.requestAddGovernAccount(p2.getAddress());
// 执行投票
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(u1);
voteModeGovernManager.vote(requestId, true);
// 切换投票者
voteModeGovernManager.changeCredentials(u);
// 发起重置私钥操作
TransactionReceipt tr = voteModeGovernManager.addGovernAccount(requestId, p2.getAddress());
```

<br />**参考执行返回日志：**<br />
```s
Governance contract [ 0x56542c78f625648d9a93f12099264113fad84197 ] request to add governance account [ 0x92217c219165d5af708afea698a910fe9ba4b0c9 ], weight [ 1 ]
Vote request id is [ 10007 ]
 start vote, Request id: [ 10007 ] 
 --------------------------------------  
 voter: [ 0xa9d5c2f248f27d4976e99c7430789e12b484fd81 ] 
 voter weight is [ 1 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10007 ] 
 request address is [ 0x92217c219165d5af708afea698a910fe9ba4b0c9 ] 
 vote type: [ reset weight ] 
 threshod is [ 1 ] 
 weight is [ 1 ] 
 vote passed? [ true ] 

add account [ 0x92217c219165d5af708afea698a910fe9ba4b0c9 ] weight [ 1 ] succeed 

```

##### 发起添加一个治理账户投票申请

**函数签名：**

```java
BigInteger requestAddGovernAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


##### 添加一个投票账户

**函数签名：**

```java
TransactionReceipt addGovernAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


<br />**其他治理模式：**<br />

[使用管理员治理模式](./admin_mode.md)

[使用权重投票治理模式](./weight_vote.md)
