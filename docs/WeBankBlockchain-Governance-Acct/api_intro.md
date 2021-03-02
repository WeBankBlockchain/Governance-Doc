# SDK使用说明
为了便于使用，我们提供了Java版本的SDK。相关的使用说明可参考下文。

同时，提供了一个基于本SDK的[Governance-Account-Demo](https://github.com/WeBankBlockchain/Governance-Account-Demo)。Demo中包含了相关的合约代码和SDK的代码，供参考。

## 引入工程
将Jar包引入到用户自己的Springboot业务项目中，此处的项目名以Governance-Account-Demo为例。

在自己的Java项目中的build.gradle文件中，添加maven仓库
```groovy
    allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```

引入依赖：
```groovy
    dependencies {
	    implementation 'com.github.WeBankBlockchain:Governance-Account:master-SNAPSHOT'
	}
```

## 配置环境

### 拷贝证书

```shell
cd Governance-Account-Demo
```

将SDK证书拷贝到项目的conf目录下(这里假设SDK证书位于~/fisco/nodes/127.0.0.1/sdk目录)：
```shell
mkdir -p conf && cp -r ~/fisco/nodes/127.0.0.1/sdk/* conf
```

### 修改配置文件
配置文件位于 src/main/resources/application.properties 目录下

```s
## 节点地址和channel端口
system.nodeStr=127.0.0.1:20200
## 群组ID
system.groupId=1

## 密码类型 0-非国密，1-国密
system.encryptType=0

## 配置的客户端私钥，如不配置，则随机生成一个
# system.hexPrivateKey=33b07356be6d05a930a104d20f482e36e55040e2f8d1af6169419e5e231629ac

## 是否默认开启创建治理合约，打开后才会自动创建一个管理员模式的治理合约。
system.defaultGovernanceEnabled=true
```

### 获取自动注入的对象

#### 自动获取链操作底层的对象
如果正确配置，可获得自动注入的FISCO BCOS SDK常用的对象。详见[FISCO BCOS Java SDK手册](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/index.html)。

```java
// 自动注入bcosSDK对象,主要包含了SDK相关的操作。
@Autowired 
private BcosSDK bcosSDK;
// 自动注入Client对象,主要包含客户端相关的操作，
@Autowired 
private Client client;
// 自动注入CryptoKeyPair对象,主要用于私钥相关的操作，包含了密码学相关操作的接口。
@Autowired 
private CryptoKeyPair cryptoKeyPair;

```

#### 创建账户治理接口控制器的对象

相关的业务操作流程可参考[组件介绍](./intro.md)

账户治理类控制器包括了治理账户创建器（GovernContractInitializer）、普通账户控制器（EndUserOperManager）、管理员模式的控制器（AdminModeGovernManager）、投票模式的控制器（VoteModeGovernManager）和社交投票控制器（SocialVoteManager）。

其中，可按以下维度划分：
1. 通用的**基础用户账户**操作。
   - 通用操作：BasicManager。 
  
2. 与**治理用户账户**操作相关的。继承了BasicManager。
   - 与治理账户通用操作相关的： 如创建治理合约的 GovernContractInitializer 。
   - 与特定的治理模式下的治理操作相关的：如管理员模式下的 AdminModeGovernManager 和投票模式下的（包含多签制和权重投票制）VoteModeGovernManager 。

3. 与**普通用户账户**操作相关的。继承了BasicManager。
   - 普通用户操作相关的： 如普通用户操作相关的 EndUserOperManager ， 普通用户投票相关的 SocialVoteManager 。

##### 手动创建
以下是创建方法, 可选择自动注入GovernContractInitializer对象，然后创建其他的控制器。
```java
@Autowired 
private GovernContractInitializer GovernContractInitializer;
```

也可以手动传参，创建GovernContractInitializer对象。
```java
GovernContractInitializer GovernContractInitializer = new GovernContractInitializer(client, cryptoKeypair);
```

选择治理模式，使用GovernContractInitializer对象创新新的治理合约后，进一步创建各自模式和角色的控制器对象。
```java
// 使用治理账户创建器（GovernContractInitializer）所创建的治理合约对象governance 
EndUserOperManager endUserOperManager = new EndUserOperManager(governance, client, cryptoKeypair);
SocialVoteManager socialVoteManager = new SocialVoteManager(governance, client, cryptoKeypair);
AdminModeGovernManager adminModeManager = new AdminModeGovernManager(governance, client, cryptoKeypair);
VoteModeGovernManager voteModeGovernManager = new VoteModeGovernManager(governance, client, cryptoKeypair);

```

##### 自动注入
如果打开了默认开启创建治理合约，则会使用配置的私钥来默认创建一个管理员模式的治理账户（一般用于快速演示）。相关的合约管理器也会被自动注入进来。

```java
@Autowired 
private GovernContractInitializer GovernContractInitializer;
@Autowired 
private EndUserOperManager endUserOperManager;
@Autowired 
private SocialVoteManager socialVoteManager;
@Autowired 
private AdminModeGovernManager adminModeGovernManager;
```

## 功能列表

### 通用操作类

#### BasicManager
包含了用户账户创建，查询用户账户映射的地址，查询用户外部地址和查询用户账户状态等基础功能。**所有用户**可以发起以下操作。

| 功能介绍                                     | 接口函数签名                                                                | 参数说明                       |
| -------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------ |
| 获取用户的内部映射账户的地址                 | String getAccountAddress()                                                  |                                |
| 根据外部账户地址创建用户账户                 | String createAccount(String externalAccount)                                | 用户外部账户地址。             |
| 指定账户管理器地址和外部账户地址创建用户账户 | String createAccount(AccountManager accountManager, String externalAccount) | 账户管理器地址和外部账户地址。 |
| 根据内部映射账户地址获取外部账户地址         | String getExternalAccount(String userAccount)                               | 内部映射账户的地址。           |
| 根据外部账户地址获取用户账户对象             | UserAccount getUserAccount(String externalAccount)                          | 用户外部账户地址。             |
| 获取账户状态                                 | int getUserAccountStatus(String externalAccount)                            | 用户外部账户地址。             |
| 根据外部账户地址获取内部映射账户的地址       | String getBaseAccountAddress(String externalAccount)                        | 用户外部账户地址。             |
| 判断外部账户地址是否已创建                   | boolean hasAccount(String externalAccount)                                  | 用户外部账户地址。             |
| 判断账户状态是否正常                         | boolean isExternalAccountNormal(String externalAccount)                     | 用户外部账户地址。             |
| 修改绑定的私钥对                             | void changeCredentials(CryptoKeyPair credentials)                           | 新的私钥对。                   |


### 治理账户类

#### GovernContractInitializer

包含了创建治理治理合约类的接口。只有**治理者**才能发起下述接口的交易。

| 功能介绍                                         | 接口函数签名                                                                                                | 参数说明                                                       |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 创建管理员模式的治理合约                         | WEGovernance createGovernAccount(CryptoKeyPair credential)                                                  | 管理员账户的私钥。                                             |
| 创建投票模式（含多签制和权重投票模式）的治理合约 | GovernAccountGroup governAccountGroup                                                                       | 包含了治理账户的外部账户地址列表、对应的权重和阈值等配置信息。 |
| 创建多签制模式的治理合约                         | WEGovernance createGovernAccount(List<String> externalAccountList, int threshold)                           | 治理成员外部账户地址列表和通过的阈值。                         |
| 创建权重投票模式的治理合约                       | WEGovernance createGovernAccount(List<String> externalAccountList, List<BigInteger> weights, int threshold) | 治理成员外部账户地址列表、各治理账户对应的权重和通过的阈值。   |

#### AdminModeGovernManager
包含了管理员模式下的各类操作接口。只有在超级管理员模式下的**超级管理员**才能发起下述接口的交易。

| 功能介绍         | 接口函数签名                                                          | 参数说明                       |
| ---------------- | --------------------------------------------------------------------- | ------------------------------ |
| 移交管理员权限   | TransactionReceipt transferAdminAuth(String newAdminAddr)             | 新的管理员外部账户地址。       |
| 重置用户账户私钥 | TransactionReceipt resetAccount(String oldAccount, String newAccount) | 旧的以及新的用户外部账户地址。 |
| 冻结用户账户     | TransactionReceipt freezeAccount(String externalAccount)              | 要冻结的外部账户地址。         |
| 解冻用户账户     | TransactionReceipt unfreezeAccount(String externalAccount)            | 要解冻的外部账户地址。         |
| 注销用户账户     | TransactionReceipt cancelAccount(String externalAccount)              | 要注销的外部账户地址。         |

#### VoteModeGovernManager
包含了投票模式下的各类操作接口，包含了多签制和阈值投票。其中，多签制也可被视为一种特殊的阈值投票，即所有治理账户的投票阈值为1。只有在投票模式下的**治理者**才能发起下述接口的交易。

| 功能介绍                   | 接口函数签名                                                                                                | 参数说明                                           |
| -------------------------- | ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| 请求重设投票阈值           | BigInteger requestResetThreshold(int newThreshold)                                                          | 新投票阈值。                                       |
| 请求删除治理账户           | BigInteger requestRemoveGovernAccount(String externalAccount)                                               | 请求删除的用户外部账户地址。                       |
| 请求重置治理账户           | BigInteger requestResetGovernAccount(String externalAccount, int weight)                                    | 待重置的外部账户地址和投票的权重。                 |
| 请求添加治理账户           | BigInteger requestAddGovernAccount(String externalAccount, int weight)                                      | 待添加的外部账户地址和投票的权重。                 |
| 请求添加治理账户           | BigInteger requestAddGovernAccount(String externalAccount)                                                  | 待添加的外部账户地址，投票权重默认为1。            |
| 请求重置用户账户           | BigInteger requestResetAccount(String newExternalAccount, String oldExternalAccount)                        | 待重置的新旧两个用户外部账户地址。                 |
| 请求冻结用户账户           | BigInteger requestFreezeAccount(String externalAccount)                                                     | 待冻结的用户外部账户地址。                         |
| 请求解冻用户账户           | BigInteger requestUnfreezeAccount(String externalAccount)                                                   | 待解冻的外部账户地址。                             |
| 请求注销用户账户           | BigInteger requestCancelAccount(String externalAccount)                                                     | 待注销的外部账户地址。                             |
| 对请求内容进行投票         | TransactionReceipt vote(BigInteger requestId, boolean agreed)                                               | 投票事务的ID和投票意见。                           |
| 执行重置用户账户           | TransactionReceipt resetAccount(BigInteger requestId, String newExternalAccount, String oldExternalAccount) | 投票事务ID，重置的新旧两个用户外部账户地址。       |
| 执行冻结用户账户           | TransactionReceipt freezeAccount(BigInteger requestId, String externalAccount)                              | 投票事务ID和操作的外部账户地址。                   |
| 执行解冻用户账户           | TransactionReceipt unfreezeAccount(BigInteger requestId, String externalAccount)                            | 投票事务ID和操作的外部账户地址。                   |
| 执行注销用户账户           | TransactionReceipt cancelAccount(BigInteger requestId, String externalAccount)                              | 投票事务ID和操作的外部账户地址。                   |
| 执行重设投票阈值           | TransactionReceipt resetThreshold(BigInteger requestId, int threshold)                                      | 投票事务ID和重设的阈值。                           |
| 执行删除治理账户           | TransactionReceipt removeGovernAccount(BigInteger requestId, String externalAccount)                        | 投票事务ID和操作的外部账户地址。                   |
| 执行重置治理账户私钥       | TransactionReceipt resetGovernAccount(BigInteger requestId, String externalAccount, int weight)             | 投票事务ID、操作的外部账户地址和重置的阈值。       |
| 执行添加治理账户           | TransactionReceipt addGovernAccount(BigInteger requestId, String externalAccount, int weight)               | 投票事务ID、操作的外部账户地址和重置的阈值。       |
| 执行添加治理账户           | TransactionReceipt addGovernAccount(BigInteger requestId, String externalAccount)                           | 投票事务ID和操作的外部账户地址。默认的投票阈值为1. |
| 根据交易回执获取投票ID     | BigInteger getId(TransactionReceipt tr)                                                                     | 交易回执。                                         |
| 根据投票ID获取投票请求信息 | VoteRequestInfo getVoteRequestInfo(BigInteger requestId)                                                    | 投票事务ID。                                       |
| 获取治理合约的投票权重信息 | WeightInfo getWeightInfo()                                                                                  |                                                    |

### 普通用户账户类

#### EndUserOperManager
包含了普通用户账户相关的操作。**所有用户**可以发起以下操作。

| 功能介绍                 | 接口函数签名                                                    | 参数说明                       |
| ------------------------ | --------------------------------------------------------------- | ------------------------------ |
| 重置账户私钥             | TransactionReceipt resetAccount(String newCredential)           | 新的外部账户地址。             |
| 删除社交好友方式重置私钥 | TransactionReceipt modifyManagerType()                          |                                |
| 修改关联社交好友         | TransactionReceipt modifyManagerType(List<String> voters)       | 三个关联的社交好友。           |
| 添加关联的社交好友账户   | TransactionReceipt addRelatedAccount(String externalAccount)    | 要添加的社交好友外部账户地址。 |
| 删除关联的社交好友账户   | TransactionReceipt removeRelatedAccount(String externalAccount) | 要删除的社交好友外部账户地址。 |
| 注销用户账户             | TransactionReceipt cancelAccount()                              |                                |
| 查询私钥重置方式         | int getUserStatics()                                            |                                |

#### SocialVoteManager
包含了社交好友在行使重置私钥投票功能相关的操作。操作的主体为被其他用户设置并关联的社交好友账户。例如小明设置了三个社交好友的外部账户地址，其中一个好友为小华，当小明丢失了私钥后，可让小华替他发起重置的投票，只要三个社交好友中有两个投票通过，就可以发起重置小明私钥的操作了。这里的SocialVoteManager就是提供了小华相关的操作。

只有被其他用户所委托进行社交好友关联来重置的用户可以发起以下操作。

| 功能介绍                 | 接口函数签名                                                                                 | 参数说明                         |
| ------------------------ | -------------------------------------------------------------------------------------------- | -------------------------------- |
| 申请重置社交好友的私钥   | TransactionReceipt requestResetAccount(String newExternalAccount, String oldExternalAccount) | 需要重置的新旧两个外部账户地址。 |
| 所关联的社交好友进行投票 | TransactionReceipt vote(String oldExternalAccount, boolean agreed)                           | 待重置的外部账户地址和投票意见。 |
| 执行重置私钥操作         | TransactionReceipt resetAccount(String newExternalAccount, String oldExternalAccount)        | 需要重置的新旧两个外部账户地址。 |


## 使用管理员治理模式

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
假如治理者确定采用管理员的治理模式，则传入管理者的密钥，创建新的治理合约。
<br />**具体调用示例：**<br />

```
// 调用 createGovernAccount 方法创建治理合约
WEGovernance governance = adminModeManager.createGovernAccount(cryptoKeyPair);
```

执行返回日志：
```s
Governance account create succeed [ 0xa84b6989931ec1352a799a0edae3108d8e19bed0 ] 
```

<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

### 创建管理员模式的控制器
可通过手动传参创建管理员模式的控制器。
```java
AdminModeGovernManager adminModeManager = new AdminModeGovernManager(governance, client, cryptoKeyPair);
```
也可通过自动注入的方式引入。

### 调用控制接口

控制接口包括了重置私钥、冻结账户、解冻账户、注销账户、移交管理员权限等功能。

#### 重置用户私钥

**具体调用示例：**

```java
TransactionReceipt tr = adminModeManager.resetAccount(u1Address, u2Address);
```

<br />**函数签名：**<br />

```java
TransactionReceipt resetAccount(String oldAccount, String newAccount)
```

<br />**输入参数：**<br />

- oldAccount  用户的外部账户的原私钥地址。
- newAccount  该账户被重置后的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

<br />**参考执行返回日志：**<br />
```s
reset account to [ 0x8a73e5c031c176ecf4bb156c59271847e985fc25 ] from [ 0xfc51a229193e1277844fc6c547fe6877c9f0bbac ]
```


#### 冻结普通账户

**具体调用示例：**

```java
TransactionReceipt tr = adminModeManager.freezeAccount(u1Address);
```

<br />**函数签名：**<br />

```java
TransactionReceipt freezeAccount(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

<br />**参考执行返回日志：**<br />
```s
freeze account [ 0xfc51a229193e1277844fc6c547fe6877c9f0bbac ]
```


#### 解冻普通账户

**具体调用示例：**

```java
    TransactionReceipt tr = adminModeManager.unfreezeAccount(u1Address);
```

<br />**函数签名：**<br />

```java
    TransactionReceipt unfreezeAccount(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

<br />**参考执行返回日志：**<br />
```s
unfreeze account [ 0xfc51a229193e1277844fc6c547fe6877c9f0bbac ]
```

#### 账户强制注销

**具体调用示例：**

```java
TransactionReceipt tr = GovernContractInitializer.cancelAccount(u1Address);
```

<br />**函数签名：**<br />

```java
TransactionReceipt cancelAccount(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

<br />**参考执行返回日志：**<br />
```s
cancel account [ 0xfc51a229193e1277844fc6c547fe6877c9f0bbac ]
```

#### 移交管理员的权限

<br />移交管理员账户时，需要确保被移交的账户已注册，且账户状态正常。<br />
<br />**具体调用示例：**<br />

```java
TransactionReceipt tr = adminModeManager.transferAdminAuth(u1Address);
```

<br />**函数签名：**<br />

```java
TransactionReceipt transferAdminAuth(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

<br />**参考执行返回日志：**<br />
```s
Contract [ 0x7a3868055f88b98a1a5aac81efa20b6ebe10c2ff ] transfer owner to address [ 0x8a73e5c031c176ecf4bb156c59271847e985fc25 ]
```


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



## 使用权重投票治理模式

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

本模式类似于上一种多签制，区别在于每个投票者的投票权重可以是不相同的。
<br />例如，以下平台方选择了治理委员会的权重投票的治理模式，一共有三个参与者参与治理，投票的权重分别为1、2、3，阈值为4，也就是说任意的赞同选票权重相加超过阈值即可获得通过。那么我们接下来将创建一个治理账户。<br />
<br />**具体调用示例：**<br />

```java
// 1. 创建账户和权重列表
GovernAccountGroup governAccountGroup = new GovernAccountGroup();
// 创建3个治理账户，合约投票阈值为4，三个账户的投票权重分别为1、2、3
governAccountGroup.setThreshold(4);
governAccountGroup.addGovernUser("user1", governanceUser1Keypair.getAddress(), 1);
governAccountGroup.addGovernUser("user2", governanceUser2Keypair.getAddress(), 2);
governAccountGroup.addGovernUser("user3", governanceUser3Keypair.getAddress(), 3);
```

执行返回日志：
```s
...
After add governUser: GovernAccountGroup default group info: 
GovernUser[user1]: weight is [1] external account is [0xd91f9fa464ef53b763652fdc170554351f65a207]
GovernUser[user2]: weight is [2] external account is [0xb64cd5cd4a57cbc16f74c46b342f3b35bfb64c33]
GovernUser[user3]: weight is [3] external account is [0xc8abcd2857ba484e0867faf184332e0538b932e3]
threshold is [4], total weight is [6]
-------------------------------------------------------------------------------------------------
Governance acct create succeed [ 0x7f0eef7303d76c3846827cff71925d24522c60c5 ]
```

<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

### 创建管理员模式的控制器
治理委员会模式下的管理功能均位于 VoteModeGovernManager 类中。

可通过手动传参创建管理员模式的控制器。
```java
VoteModeGovernManager voteModeGovernManager = new VoteModeGovernManager(governance, client, cryptoKeypair);
```

### 调用控制接口
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
    BigInteger requestId = voteModeGovernManager.requestResetAccount(address2, address1);
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
Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request to reset account, from [ 0x3527e5ee82a84c4038287a34efdfb2efa5dfb13c ] to [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ]
 Vote request id is [ 10001 ]
credentials change to [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] from [ 0x0cfd01d563e33e884bc0fb8cf28ea968919a4d27 ]
 start vote, Request id: [ 10001 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10001 ] 
 request address is [ 0x3527e5ee82a84c4038287a34efdfb2efa5dfb13c ] 
 vote type: [ change credential ] 
 threshod is [ 4 ] 
 weight is [ 2 ] 
 vote passed? [ false ] 

credentials change to [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] from [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ]
 start vote, Request id: [ 10001 ] 
 --------------------------------------  
 voter: [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 voter weight is [ 3 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10001 ] 
 request address is [ 0x3527e5ee82a84c4038287a34efdfb2efa5dfb13c ] 
 vote type: [ change credential ] 
 threshod is [ 4 ] 
 weight is [ 5 ] 
 vote passed? [ true ] 

credentials change to [ 0x0cfd01d563e33e884bc0fb8cf28ea968919a4d27 ] from [ 0x816973dd7755176e9197bbe097287409b6c795a9 ]
reset account succeed, from [ 0x3527e5ee82a84c4038287a34efdfb2efa5dfb13c ] to [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ]

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
Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request to freeze external account [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ]
Vote request id is [ 10002 ]
credentials change to [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] from [ 0x0cfd01d563e33e884bc0fb8cf28ea968919a4d27 ]
 start vote, Request id: [ 10002 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10002 ] 
 request address is [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] 
 vote type: [ freeze account ] 
 threshod is [ 4 ] 
 weight is [ 2 ] 
 vote passed? [ false ] 

credentials change to [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] from [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ]
 start vote, Request id: [ 10002 ] 
 --------------------------------------  
 voter: [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 voter weight is [ 3 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10002 ] 
 request address is [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] 
 vote type: [ freeze account ] 
 threshod is [ 4 ] 
 weight is [ 5 ] 
 vote passed? [ true ] 

freeze account [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] succeed 
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
Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request to unfreeze external account [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ]
Vote request id is [ 10003 ]
credentials change to [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] from [ 0x816973dd7755176e9197bbe097287409b6c795a9 ]
 start vote, Request id: [ 10003 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10003 ] 
 request address is [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] 
 vote type: [ unfreeze account ] 
 threshod is [ 4 ] 
 weight is [ 2 ] 
 vote passed? [ false ] 

credentials change to [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] from [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ]
 start vote, Request id: [ 10003 ] 
 --------------------------------------  
 voter: [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 voter weight is [ 3 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10003 ] 
 request address is [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] 
 vote type: [ unfreeze account ] 
 threshod is [ 4 ] 
 weight is [ 5 ] 
 vote passed? [ true ] 

unfreeze account [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] succeed
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
 Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request to cancel external account [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ]
Vote request id is [ 10004 ]
credentials change to [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] from [ 0x816973dd7755176e9197bbe097287409b6c795a9 ]
 start vote, Request id: [ 10004 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10004 ] 
 request address is [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] 
 vote type: [ cancel account ] 
 threshod is [ 4 ] 
 weight is [ 2 ] 
 vote passed? [ false ] 

credentials change to [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] from [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ]
 start vote, Request id: [ 10004 ] 
 --------------------------------------  
 voter: [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 voter weight is [ 3 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10004 ] 
 request address is [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] 
 vote type: [ cancel account ] 
 threshod is [ 4 ] 
 weight is [ 5 ] 
 vote passed? [ true ] 

cancel account [ 0x995e75d6bb9269f5349089f582bd9664e4a3cbca ] succeed 

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
Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request reset threshold to [ 1 ]
Vote request id is [ 10005 ]
credentials change to [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] from [ 0x816973dd7755176e9197bbe097287409b6c795a9 ]
 start vote, Request id: [ 10005 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10005 ] 
 request address is [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] 
 vote type: [ reset threshold ] 
 threshod is [ 4 ] 
 weight is [ 2 ] 
 vote passed? [ false ] 

credentials change to [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] from [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ]
 start vote, Request id: [ 10005 ] 
 --------------------------------------  
 voter: [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 voter weight is [ 3 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10005 ] 
 request address is [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] 
 vote type: [ reset threshold ] 
 threshod is [ 4 ] 
 weight is [ 5 ] 
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
credentials change to [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] from [ 0x816973dd7755176e9197bbe097287409b6c795a9 ]
Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request to remove governance account [ 0x816973dd7755176e9197bbe097287409b6c795a9 ]
Vote request id is [ 10006 ]
 start vote, Request id: [ 10006 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10006 ] 
 request address is [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 vote type: [ reset weight ] 
 threshod is [ 1 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

Contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] remove governance account [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] succeed 
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
 Governance contract [ 0x2af2476ddd68d113a01b9319f49c743decb42aa6 ] request to add governance account [ 0x816973dd7755176e9197bbe097287409b6c795a9 ], weight [ 5 ]
Vote request id is [ 10007 ]
 start vote, Request id: [ 10007 ] 
 --------------------------------------  
 voter: [ 0xa18f4517b651999aa8552c5a8cdebdfbcdb537a1 ] 
 voter weight is [ 2 ] 
 agreed: [ true ] 

the current vote info: 
 -------------------------------------- 
 request id [ 10007 ] 
 request address is [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] 
 vote type: [ reset weight ] 
 threshod is [ 1 ] 
 weight is [ 2 ] 
 vote passed? [ true ] 

add account [ 0x816973dd7755176e9197bbe097287409b6c795a9 ] weight [ 5 ] succeed 

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



##### 添加一个治理账户

**函数签名：**

```
TransactionReceipt addGovernAccount(BigInteger requestId, String credential, int weight)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。
- weight 新加入账户的权重


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。



## 普通用户操作接口


### 创建控制器

可通过手动传参创建控制器。
```java
EndUserOperManager endUserOperManager = new EndUserOperManager(governance, client, endUser1Keypair);
```

在配置了自动创建治理合约的场景下，也可通过Spring自动注入对象。

### 创建新账户

**具体调用示例：**

```java
String account = endUserAdminManager.createAccount(p1Address);
```

<br />**参考执行返回日志：**<br />
```s
User account created is [ 0x0559525b0dbf461bcd450f687c227d3c5ba9607d ]
new account created: [ 0xdb2ab65d2aba4f57db8fb3866ed8a2d5727b30cc ], created by [ 0x22caf10fd5b32a7413e0e1bf4bd07fc28566e513 ]
```

<br />**函数签名：**<br />

```java
    String createAccount(String who)
```

<br />**输入参数：**<br />

- externalAccount  待创建的账户的外部账户的私钥地址。


<br />**返回参数：**<br />

- String 新建的账户地址


### 重置用户私钥

**具体调用示例：**

```java
TransactionReceipt tr = endUserAdminManager.resetAccount(p2Address);
```

<br />**参考执行返回日志：**<br />
```s
External account [ 0x22caf10fd5b32a7413e0e1bf4bd07fc28566e513 ] reset by self to new external account [ 0x092fa98bfa86dbf4684d90364cf656e88e075e05 ] 
```

<br />**函数签名：**<br />

```
TransactionReceipt resetAccount(String newCredential)
```

<br />**输入参数：**<br />

- newCredential  待更换的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

### 账户强制注销

**具体调用示例：**

```java
TransactionReceipt tr = endUserAdminManager.cancelAccount();
```

<br />**参考执行返回日志：**<br />
```s
External account canceled by self: [ 0x092fa98bfa86dbf4684d90364cf656e88e075e05 ] 
```

<br />**函数签名：**<br />

```java
TransactionReceipt cancelAccount()
```

<br />**返回参数：**<br />

- TransactionReceipt 交易回执


### 修改普通账户的管理类型

**具体调用示例：**

```java
List<String> voters = Lists.newArrayList();
voters.add(u.getAddress());
voters.add(u1.getAddress());
voters.add(u2.getAddress());
TransactionReceipt tr = endUserAdminManager.modifyManagerType(voters);
```
<br />**参考执行返回日志：**<br />
```s
Set Account [ 0x22caf10fd5b32a7413e0e1bf4bd07fc28566e513 ] to social reset mode.
 --------------------------------------  
 threshold is 2 
 Voters: ["0x1a26865e04fdd739949ca2ee519071cf7457ce9c","0x0c3f18a50003c70546aa17f72d47c9e6b7bacd28","0xc86308dc9d20b7edcb17988e4dafb3dc98540bd8"] 
```

<br />**函数签名：**<br />

```java
TransactionReceipt modifyManagerType(List<String> voters)
//重载函数，设置为仅自己管理
TransactionReceipt modifyManagerType()
```

<br />**输入参数：**<br />

- voters （可选） 当变更为支持社交好友投票时需要传入，且voters的大小必须为3。投票本身为2-3的规则。如果voters不传入，则默认不开启投票模式。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执




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
