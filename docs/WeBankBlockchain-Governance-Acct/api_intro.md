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
	    implementation 'com.github.WebankBlockchain:Governance-Account:v3.0.0-SNAPSHOT'
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

包含了创建治理合约类的接口。只有**治理者**才能发起下述接口的交易。

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
