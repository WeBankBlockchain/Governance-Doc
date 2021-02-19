# SDK使用说明
为了便于使用，我们提供了Java版本的SDK。相关的使用说明可参考下文。

同时，提供了一个基于本SDK的[Governance-Account-Demo](https://github.com/WeBankBlockchain/Governance-Account-Demo)。Demo中包含了相关的合约代码和SDK的代码，供参考。

## 引入工程
将Jar包引入到用户自己的Springboot业务项目中，此处的项目名以Governance-Account-Demo为例。

在自己的Java项目中的build.gradle文件中，添加maven仓库
```
    allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```

引入依赖：
```
    dependencies {
	    implementation 'com.github.WeBankBlockchain:Governance-Account:master-SNAPSHOT'
	}
```

## 配置环境

### 拷贝证书

```
cd Governance-Account-Demo
```

将SDK证书拷贝到项目的conf目录下(这里假设SDK证书位于~/fisco/nodes/127.0.0.1/sdk目录)：
```
mkdir -p conf && cp -r ~/fisco/nodes/127.0.0.1/sdk/* conf
```

### 修改配置文件
配置文件位于 src/main/resources/application.properties 目录下

```
## 节点地址和channel端口
system.nodeStr=127.0.0.1:20200
## 群组ID
system.groupId=1

## 密码类型 0-非国密，1-国密
system.encryptType=0

## 配置的客户端私钥，如不配置，则随机生成一个
# system.hexPrivateKey=33b07356be6d05a930a104d20f482e36e55040e2f8d1af6169419e5e231629ac

## 是否默认开启创建治理合约
system.defaultGovernanceEnabled=true
```

### 获取自动注入的对象

#### 自动获取链操作底层的对象
如果正确配置，可获得自动注入的FISCO BCOS SDK常用的对象。详见[FISCO BCOS Java SDK手册](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/index.html)。

```
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

#### 自动获取账户治理接口控制器的对象

相关的业务操作流程可参考[组件介绍](./intro.md)

账户治理类控制器包括了治理账户控制器（GovernAccountInitializer）、普通账户控制器（EndUserOperManager）、管理员模式的控制器（AdminModeGovernManager）、投票模式的控制器（VoteModeGovernManager）和社交投票控制器（SocialVoteManager）。

其中，可按以下维度划分：
1. 通用的**基础用户账户**操作。
   - 通用操作：BasicManager。 
  
2. 与**治理用户账户**操作相关的。继承了BasicManager。
   - 与治理账户通用操作相关的： 如创建治理合约的 GovernAccountInitializer 。
   - 与特定的治理模式下的治理操作相关的：如管理员模式下的 AdminModeGovernManager 和投票模式下的（包含多签制和权重投票制）VoteModeGovernManager 。

3. 与**普通用户账户**操作相关的。继承了BasicManager。
   - 普通用户操作相关的： 如普通用户操作相关的 EndUserOperManager ， 普通用户投票相关的 SocialVoteManager 。

```
@Autowired 
private EndUserOperManager endUserOperManager;
@Autowired 
private SocialVoteManager socialVoteManager;
@Autowired 
private GovernAccountInitializer governAccountInitializer;
@Autowired 
private AdminModeGovernManager adminModeGovernManager;
@Autowired 
private VoteModeGovernManager voteModeGovernManager
```

## 功能列表

### 通用操作类

#### BasicManager
包含了用户账户创建，查询用户账户映射的地址，查询用户外部地址和查询用户账户状态等基础功能。**所有用户**可以发起以下操作。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 获取用户的内部映射账户的地址 | String getAccountAddress() |  |
| 根据外部账户地址创建用户账户 | String createAccount(String externalAccount) | 用户外部账户地址。 |
| 指定账户管理器地址和外部账户地址创建用户账户 | String createAccount(AccountManager accountManager, String externalAccount) | 账户管理器地址和外部账户地址。 |
| 根据内部映射账户地址获取外部账户地址 |  String getExternalAccount(String userAccount)  | 内部映射账户的地址。|
| 根据外部账户地址获取用户账户对象 | UserAccount getUserAccount(String externalAccount) | 用户外部账户地址。 |
| 获取账户状态 |  int getUserAccountStatus(String externalAccount) | 用户外部账户地址。|
| 根据外部账户地址获取内部映射账户的地址 |  String getBaseAccountAddress(String externalAccount)  | 用户外部账户地址。 |
| 判断外部账户地址是否已创建 |  boolean hasAccount(String externalAccount) | 用户外部账户地址。|
| 判断账户状态是否正常 | boolean isExternalAccountNormal(String externalAccount) | 用户外部账户地址。|
| 修改绑定的私钥对 |   void changeCredentials(CryptoKeyPair credentials)  | 新的私钥对。|


### 治理账户类

#### GovernAccountInitializer

包含了创建治理治理合约类的接口。只有**治理者**才能发起下述接口的交易。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 创建管理员模式的治理合约 | WEGovernance createGovernAccount(CryptoKeyPair credential) | 管理员账户的私钥。 |
| 创建投票模式（含多签制和权重投票模式）的治理合约 | GovernAccountGroup governAccountGroup | 包含了治理账户的外部账户地址列表、对应的权重和阈值等配置信息。|
| 创建多签制模式的治理合约 | WEGovernance createGovernAccount(List<String> externalAccountList, int threshold) | 治理成员外部账户地址列表和通过的阈值。|
| 创建权重投票模式的治理合约 | WEGovernance createGovernAccount(List<String> externalAccountList, List<BigInteger> weights, int threshold) | 治理成员外部账户地址列表、各治理账户对应的权重和通过的阈值。 |

#### AdminModeGovernManager
包含了管理员模式下的各类操作接口。只有在超级管理员模式下的**超级管理员**才能发起下述接口的交易。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 移交管理员权限 | TransactionReceipt transferAdminAuth(String newAdminAddr) | 新的管理员外部账户地址。 |
| 重置用户账户私钥 | TransactionReceipt resetAccount(String oldAccount, String newAccount) | 旧的以及新的用户外部账户地址。|
| 冻结用户账户 | TransactionReceipt freezeAccount(String externalAccount) | 要冻结的外部账户地址。 |
| 解冻用户账户 | TransactionReceipt unfreezeAccount(String externalAccount) | 要解冻的外部账户地址。 |
| 注销用户账户 | TransactionReceipt cancelAccount(String externalAccount) | 要注销的外部账户地址。 |

#### VoteModeGovernManager
包含了投票模式下的各类操作接口，包含了多签制和阈值投票。其中，多签制也可被视为一种特殊的阈值投票，即所有治理账户的投票阈值为1。只有在投票模式下的**治理者**才能发起下述接口的交易。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 请求重设投票阈值 | BigInteger requestResetThreshold(int newThreshold) | 新投票阈值。 |
| 请求删除治理账户 |  BigInteger requestRemoveGovernAccount(String externalAccount) | 请求删除的用户外部账户地址。|
| 请求重置治理账户 | BigInteger requestResetGovernAccount(String externalAccount, int weight) | 待重置的外部账户地址和投票的权重。 |
| 请求添加治理账户 | BigInteger requestAddGovernAccount(String externalAccount, int weight) | 待添加的外部账户地址和投票的权重。 |
| 请求添加治理账户 | BigInteger requestAddGovernAccount(String externalAccount) | 待添加的外部账户地址，投票权重默认为1。 |
| 请求重置用户账户 |  BigInteger requestResetAccount(String newExternalAccount, String oldExternalAccount) | 待重置的新旧两个用户外部账户地址。|
| 请求冻结用户账户 |  BigInteger requestFreezeAccount(String externalAccount) | 待冻结的用户外部账户地址。|
| 请求解冻用户账户 |  BigInteger requestUnfreezeAccount(String externalAccount)  | 待解冻的外部账户地址。 |
| 请求注销用户账户 | BigInteger requestCancelAccount(String externalAccount) | 待注销的外部账户地址。 |
| 对请求内容进行投票 | TransactionReceipt vote(BigInteger requestId, boolean agreed) | 投票事务的ID和投票意见。 |
| 执行重置用户账户 | TransactionReceipt resetAccount(BigInteger requestId, String newExternalAccount, String oldExternalAccount) | 投票事务ID，重置的新旧两个用户外部账户地址。 |
| 执行冻结用户账户 | TransactionReceipt freezeAccount(BigInteger requestId, String externalAccount) | 投票事务ID和操作的外部账户地址。|
| 执行解冻用户账户 | TransactionReceipt unfreezeAccount(BigInteger requestId, String externalAccount) | 投票事务ID和操作的外部账户地址。 |
| 执行注销用户账户 |  TransactionReceipt cancelAccount(BigInteger requestId, String externalAccount) | 投票事务ID和操作的外部账户地址。 |
| 执行重设投票阈值 | TransactionReceipt resetThreshold(BigInteger requestId, int threshold)  | 投票事务ID和重设的阈值。 |
| 执行删除治理账户 | TransactionReceipt removeGovernAccount(BigInteger requestId, String externalAccount) | 投票事务ID和操作的外部账户地址。 |
| 执行重置治理账户私钥 | TransactionReceipt resetGovernAccount(BigInteger requestId, String externalAccount, int weight) | 投票事务ID、操作的外部账户地址和重置的阈值。|
| 执行添加治理账户 | TransactionReceipt addGovernAccount(BigInteger requestId, String externalAccount, int weight) |投票事务ID、操作的外部账户地址和重置的阈值。 |
| 执行添加治理账户 | TransactionReceipt addGovernAccount(BigInteger requestId, String externalAccount) | 投票事务ID和操作的外部账户地址。默认的投票阈值为1. |
| 根据交易回执获取投票ID | BigInteger getId(TransactionReceipt tr) | 交易回执。 |
| 根据投票ID获取投票请求信息 | VoteRequestInfo getVoteRequestInfo(BigInteger requestId) | 投票事务ID。 |
| 获取治理合约的投票权重信息 |  WeightInfo getWeightInfo() |  |

### 普通用户账户类

#### EndUserOperManager
包含了普通用户账户相关的操作。**所有用户**可以发起以下操作。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 重置账户私钥 | TransactionReceipt resetAccount(String newCredential) | 新的外部账户地址。 |
| 删除社交好友方式重置私钥 | TransactionReceipt modifyManagerType() |  |
| 修改关联社交好友 | TransactionReceipt modifyManagerType(List<String> voters)  | 三个关联的社交好友。 |
| 添加关联的社交好友账户 | TransactionReceipt addRelatedAccount(String externalAccount) | 要添加的社交好友外部账户地址。|
| 删除关联的社交好友账户 | TransactionReceipt removeRelatedAccount(String externalAccount) | 要删除的社交好友外部账户地址。 |
| 注销用户账户 | TransactionReceipt cancelAccount() | |
| 查询私钥重置方式 | int getUserStatics()  |  |

#### SocialVoteManager
包含了社交好友在行使重置私钥投票功能相关的操作。操作的主体为被其他用户设置并关联的社交好友账户。例如小明设置了三个社交好友的外部账户地址，其中一个好友为小华，当小明丢失了私钥后，可让小华替他发起重置的投票，只要三个社交好友中有两个投票通过，就可以发起重置小明私钥的操作了。这里的SocialVoteManager就是提供了小华相关的操作。

只有被其他用户所委托进行社交好友关联来重置的用户可以发起以下操作。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 申请重置社交好友的私钥 | TransactionReceipt requestResetAccount(String newExternalAccount, String oldExternalAccount)| 需要重置的新旧两个外部账户地址。 |
| 所关联的社交好友进行投票 |TransactionReceipt vote(String oldExternalAccount, boolean agreed) | 待重置的外部账户地址和投票意见。|
| 执行重置私钥操作 | TransactionReceipt resetAccount(String newExternalAccount, String oldExternalAccount)| 需要重置的新旧两个外部账户地址。 |


## 治理账户功能使用详细说明

### 创建治理合约
SDK提供了GovernAccountInitializer类来创建治理合约，该接口可被自动注入到工程中。

CryptoKeyPair也被自动注入，可用于生成私钥对。
```
    // 自动注入GovernAccountInitializer对象
    @Autowired private GovernAccountInitializer governAccountInitializer;
    // 自动注入CryptoKeyPair对象
    @Autowired private CryptoKeyPair cryptoKeyPair;
```

#### 管理员模式
假如治理者确定采用管理员的治理模式，那么需要首先生成一个管理员的治理账户。随后，使用管理员的账户来创建治理合约。
<br />**具体调用示例：**<br />

```
    // 使用自动注入的cryptoKeyPair随机创建一组管理员账户的私钥对
    CryptoKeyPair governanceUser1Keypair = cryptoKeyPair.generateKeyPair();
    // 调用 createGovernAccount 方法创建治理合约
    WEGovernance govern = adminModeManager.createGovernAccount(governUserKeypair);
```

执行返回日志：
```
Governance acct create succeed 0x44209c57874cc7e6654f76e4940d8d05c7c3881a 
```

<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

#### 多签制治理模式

例如，以下平台方选择了治理委员会的治理模式，一共有三个参与者参与治理，治理的规则为任意的交易请求获得其中两方的同意，即可获得通过。那么我们接下来将创建一个治理账户。

<br />**具体调用示例：**<br />

```
    // 创建一个治理委员会的配置
    GovernAccountGroup governAccountGroup = new GovernAccountGroup();
    // 设置投票通过的阈值为2。设置的阈值需小于等于所有治理账户权重之和。
    governAccountGroup.setThreshold(2);
    // 随机生成3个治理账户管理成员的私钥对（本实例仅供演示，实际生产中请妥善保管私钥）。传入用户名。
    GovernUser governUser1 = new GovernUser("user1", cryptoKeyPair.generateKeyPair().getAddress());
    GovernUser governUser2 = new GovernUser("user2", cryptoKeyPair.generateKeyPair().getAddress());
    GovernUser governUser3 = new GovernUser("user3", cryptoKeyPair.generateKeyPair().getAddress());
    // 在治理委员会配置中添加以上创建的治理账户
    governAccountGroup.addGovernUser(governUser1);
    governAccountGroup.addGovernUser(governUser2);
    governAccountGroup.addGovernUser(governUser3);
    // 创建治理合约。
    WEGovernance govern = governAccountInitializer.createGovernAccount(governAccountGroup);
```

执行后返回日志：
```
...
After add governUser: GovernAccountGroup default group info: 
GovernUser[user1]: weight is [1] external account is [0x1fa875988195e30a9b7e3ddaa7a2870bc84dc468]
GovernUser[user2]: weight is [1] external account is [0xac3fb1d6e748697e40672ec1986e58da7d762696]
GovernUser[user3]: weight is [1] external account is [0x2bdb3029fa9eca8a640101b440ad8de07b015813]
threshold is [2], total weight is [3]
-------------------------------------------------------------------------------------------------
Governance acct create succeed 0xfd667db2bf205fef03ee8c303f06feaf8e20f3b8 
```

<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

#### 权重投票治理模式

本模式类似于上一种多签制，区别在于每个投票者的投票权重可以是不相同的。
<br />例如，以下平台方选择了治理委员会的权重投票的治理模式，一共有三个参与者参与治理，投票的权重分别为1、2、3，阈值为4，也就是说任意的赞同选票权重相加超过阈值即可获得通过。那么我们接下来将创建一个治理账户。<br />
<br />**具体调用示例：**<br />

```
    // 创建一个治理委员会的配置
    GovernAccountGroup governAccountGroup = new GovernAccountGroup();
    // 设置投票通过的阈值为4。设置的阈值需小于等于所有治理账户权重之和。
    governAccountGroup.setThreshold(4);
    // 随机生成3个治理账户管理成员的私钥对（本实例仅供演示，实际生产中请妥善保管私钥）。传入用户名。与多签制不同的是，多传入了一个投票的权重值。
    GovernUser governUser1 = new GovernUser("user1", cryptoKeyPair.generateKeyPair().getAddress(), 1);
    GovernUser governUser2 = new GovernUser("user2", cryptoKeyPair.generateKeyPair().getAddress(), 2);
    GovernUser governUser3 = new GovernUser("user3", cryptoKeyPair.generateKeyPair().getAddress(), 3);
    // 在治理委员会配置中添加以上创建的治理账户
    governAccountGroup.addGovernUser(governUser1);
    governAccountGroup.addGovernUser(governUser2);
    governAccountGroup.addGovernUser(governUser3);
    // 创建治理合约。
    WEGovernance govern = governAccountInitializer.createGovernAccount(governAccountGroup);
```

执行返回日志：
```
...
After add governUser: GovernAccountGroup default group info: 
GovernUser[user1]: weight is [1] external account is [0xd91f9fa464ef53b763652fdc170554351f65a207]
GovernUser[user2]: weight is [2] external account is [0xb64cd5cd4a57cbc16f74c46b342f3b35bfb64c33]
GovernUser[user3]: weight is [3] external account is [0xc8abcd2857ba484e0867faf184332e0538b932e3]
threshold is [4], total weight is [6]
-------------------------------------------------------------------------------------------------
Governance acct create succeed 0x7f0eef7303d76c3846827cff71925d24522c60c5 
```

<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />


### 调用控制接口

#### 管理员模式

管理员模式下的管理功能均位于GovernAccountInitializer类中。
<br />首先，注入该类：<br />

```
@Autowired
private AdminModeGovernManager adminModeManager;
```


##### 重置用户私钥

**具体调用示例：**

```
    TransactionReceipt tr = adminModeManager.resetAccount(u1Address, u2Address);
```

<br />**函数签名：**<br />

```
    TransactionReceipt resetAccount(String oldAccount, String newAccount)
```

<br />**输入参数：**<br />

- oldAccount  用户的外部账户的原私钥地址。
- newAccount  该账户被重置后的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执


##### 冻结普通账户

**具体调用示例：**

```
    TransactionReceipt tr = adminModeManager.freezeAccount(u1Address);
```

<br />**函数签名：**<br />

```
    TransactionReceipt freezeAccount(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执


##### 解冻普通账户

**具体调用示例：**

```
    TransactionReceipt tr = adminModeManager.unfreezeAccount(u1Address);
```

<br />**函数签名：**<br />

```
    TransactionReceipt unfreezeAccount(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执


##### 账户强制注销

**具体调用示例：**

```
    TransactionReceipt tr = governAccountInitializer.cancelAccount(u1Address);
```

<br />**函数签名：**<br />

```
    TransactionReceipt cancelAccount(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执



##### 移交管理员的权限

<br />移交管理员账户时，需要确保被移交的账户已注册，且账户状态正常。<br />
<br />**具体调用示例：**<br />

```
    TransactionReceipt tr = adminModeManager.transferAdminAuth(u1Address);
```

<br />**函数签名：**<br />

```
    TransactionReceipt transferAdminAuth(String account)
```

<br />**输入参数：**<br />

- account  用户的外部账户的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执



#### 多签制治理模式

治理委员会模式下的管理功能均位于 VoteModeGovernManager 类中。
<br />首先，注入该类：<br />

```
@Autowired
private VoteModeGovernManager voteModeGovernManager;
```

<br />在本模式下，执行任何账户相关的业务操作需要遵循以下步骤：<br />

1. 发起一个投票请求；
2. 治理账户成员赞同该投票；
3. 投票发起者确认投票已经通过后，发起操作。


<br />我们首先来介绍下通用的投票接口：<br />

##### 治理委员会成员投票

**具体调用示例：**

```
    TransactionReceipt tr = voteModeGovernManager.vote(requestId, true);
```

<br />**函数签名：**<br />

```
    TransactionReceipt vote(BigInteger requestId, boolean agreed)
```

<br />**输入参数：**<br />

- requestId  发起投票的requestId。
- agreed 是否同意，true/false


<br />**返回参数：**<br />

- TransactionReceipt 交易回执



##### 重置用户私钥

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestResetAccount(p2.getAddress(), p1.getAddress());
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.resetAccount(requestId, p2.getAddress(), p1.getAddress());
```


###### 发起重置用户私钥投票申请

**函数签名：**

```
    BigInteger requestResetAccount(String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- oldCredential  用户的外部账户的原私钥地址。
- newCredential  该账户被重置后的私钥地址


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 重置用户私钥

**函数签名：**

```
    TransactionReceipt resetAccount(BigInteger requestId, String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- newCredential  该账户被重置后的私钥地址
- oldCredential  用户的外部账户的原私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 冻结普通账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestFreezeAccount(p2.getAddress());
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.freezeAccount(requestId, p2.getAddress());
```

###### 发起冻结用户账户投票申请

**函数签名：**

```
   BigInteger requestFreezeAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 冻结用户账户

**函数签名：**

```
   TransactionReceipt freezeAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 解冻普通账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestUnreezeAccount(p2.getAddress());
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.unfreezeAccount(requestId, p2.getAddress());
```

###### 发起解冻用户账户投票申请

**函数签名：**

```
   BigInteger requestunfreezeAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 解冻用户账户

**函数签名：**

```
   TransactionReceipt unfreezeAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 账户强制注销

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestCancelAccount(p2.getAddress());
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.cancelAccount(requestId, p2.getAddress());
```


###### 发起注销用户账户投票申请

**函数签名：**

```
   BigInteger requestCancelAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。



###### 注销用户账户

**函数签名：**

```
   TransactionReceipt cancelAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 设置治理账户投票的阈值

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestResetThreshold(newThreshold);
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.resetThreshold(requestId, newThreshold);
```

###### 发起设置治理账户投票申请

**函数签名：**

```
    BigInteger requestRemoveGovernAccount(int newThreshold)
```

<br />**输入参数：**<br />

- newThreshold  新阈值。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 设置新阈值

**函数签名：**

```
   TransactionReceipt resetThreshold(BigInteger requestId, int threshold)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- newThreshold  新阈值。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 治理账户删除一个投票账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestRemoveGovernAccount(p2.getAddress());
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.removeGovernAccount(requestId, p2.getAddress());
```

###### 发起删除一个治理账户投票申请

**函数签名：**

```
   BigInteger requestRemoveGovernAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 删除一个投票账户

**函数签名：**

```
   TransactionReceipt removeGovernAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 治理账户添加一个投票新账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
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

###### 发起添加一个治理账户投票申请

**函数签名：**

```
   BigInteger requestAddGovernAccount(String credential)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 添加一个投票账户

**函数签名：**

```
   TransactionReceipt addGovernAccount(BigInteger requestId, String credential)
```

<br />**输入参数：**<br />

- requestId  之前的投票请求返回的ID
- externalAccount  用户的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


#### 不同权重的投票制治理模式

不同权重的投票制模式总体和多签制非常类似，此处不再做过多的赘述，请参考上节。

##### 治理账户添加一个投票新账户

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    BigInteger requestId = voteModeGovernManager.requestAddGovernAccount(p2.getAddress() , weight);
    // 执行投票
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u1);
    voteModeGovernManager.vote(requestId, true);
    // 切换投票者
    voteModeGovernManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = voteModeGovernManager.addGovernAccount(requestId, p2.getAddress(), weight);
```

###### 发起添加一个治理账户投票申请

**函数签名：**

```
   BigInteger requestAddGovernAccount(String credential, int weight)
```

<br />**输入参数：**<br />

- externalAccount  用户的外部账户地址。
- weight 新加入账户的权重


<br />**返回参数：**<br />

- BigInteger 投票ID，用户需保存该ID便于后续的交互和其他操作。


###### 添加一个投票账户

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


##### 其他交易

其余部分颇为相似，可参考基于相同权重的投票模式

## 普通用户接口使用详细说明


### 普通账户主要功能

普通账户的功能均位于EndUserOperManager类中。
<br />首先，注入该类：<br />

```
@Autowired
private EndUserOperManager endUserOperManager;
```

#### 创建新账户

**具体调用示例：**

```
    String account = endUserAdminManager.createAccount(p1Address);
```

<br />**函数签名：**<br />

```
    String createAccount(String who)
```

<br />**输入参数：**<br />

- externalAccount  待创建的账户的外部账户的私钥地址。


<br />**返回参数：**<br />

- String 新建的账户地址


#### 重置用户私钥

**具体调用示例：**

```
    TransactionReceipt tr = endUserAdminManager.resetAccount(p2Address);
```

<br />**函数签名：**<br />

```
    TransactionReceipt resetAccount(String newCredential)
```

<br />**输入参数：**<br />

- newCredential  待更换的私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执

#### 账户强制注销

**具体调用示例：**

```
    TransactionReceipt tr = endUserAdminManager.cancelAccount();
```

<br />**函数签名：**<br />

```
    TransactionReceipt cancelAccount()
```

<br />**返回参数：**<br />

- TransactionReceipt 交易回执


#### 修改普通账户的管理类型

**具体调用示例：**

```
    List<String> voters = Lists.newArrayList();
    voters.add(u.getAddress());
    voters.add(u1.getAddress());
    voters.add(u2.getAddress());
    TransactionReceipt tr = endUserAdminManager.modifyManagerType(voters);
```

<br />**函数签名：**<br />

```
    TransactionReceipt modifyManagerType(List<String> voters)
    //重载函数，设置为仅自己管理
    TransactionReceipt modifyManagerType()
```

<br />**输入参数：**<br />

- voters （可选） 当变更为支持社交好友投票时需要传入，且voters的大小必须为3。投票本身为2-3的规则。如果voters不传入，则默认不开启投票模式。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执


#### 添加一个社交好友

**具体调用示例：**

```
    TransactionReceipt tr = endUserAdminManager.addRelatedAccount(userAddress);
```

<br />**函数签名：**<br />

```
    TransactionReceipt addRelatedAccount(String account)
```

<br />**输入参数：**<br />

- account  被添加好友的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执


#### 删除一个社交好友

**具体调用示例：**

```
    TransactionReceipt tr = endUserAdminManager.removeRelatedAccount(userAddress);
```

<br />**函数签名：**<br />

```
    TransactionReceipt removeRelatedAccount(String account)
```

<br />**输入参数：**<br />

- account  被删除好友的外部账户地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执


### 使用社交好友投票重置私钥

#### 重置用户私钥

参考上文提及的三个步骤：发起投票请求、投票、执行操作。此处，使用了单SDK来处理多用户的操作，使用了changeCredentials函数来切换不同的用户。
<br />社交好友投票相关的操作在 SocialVoteManager 类中。<br />
<br />**具体调用示例：**<br />

```
    // 发起投票请求
    TransactionReceipt t = socialVoteManager.requestResetAccount(u1.getAddress(), p1.getAddress());
    // 执行投票
    socialVoteManager.vote(requestId, true);
    // 切换投票者
    socialVoteManager.changeCredentials(u1);
    socialVoteManager.vote(requestId, true);
    // 切换投票者
    socialVoteManager.changeCredentials(u);
    // 发起重置私钥操作
    TransactionReceipt tr = socialVoteManager.resetAccount(u1.getAddress(), p1.getAddress());
```

#### 发起重置用户私钥投票申请

**函数签名：**

```
    TransactionReceipt requestResetAccount(String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- oldCredential  用户的外部账户的原私钥地址。
- newCredential  该账户被重置后的私钥地址


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 投票

**函数签名：**

```
    TransactionReceipt vote(String oldCredential, boolean agreed)
```

<br />**输入参数：**<br />

- oldCredential  申请变更账户的外部账户地址。
- agreed  是否同意


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。


##### 重置用户私钥

**函数签名：**

```
    TransactionReceipt resetAccount(String newCredential, String oldCredential)
```

<br />**输入参数：**<br />

- newCredential  该账户被重置后的私钥地址
- oldCredential  用户的外部账户的原私钥地址。


<br />**返回参数：**<br />

- TransactionReceipt 交易回执。
