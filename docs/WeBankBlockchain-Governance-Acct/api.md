# Java语言版本的SDK使用说明
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
账户治理类控制器包括了治理账户控制器（GovernAccountInitializer）、普通账户控制器（EndUserOperManager）、管理员模式的控制器（AdminModeGovernManager）、投票模式的控制器（VoteModeGovernManager）和社交投票控制器（SocialVoteManager）。

其中，可按以下维度划分：
1. 基础用户账户操作。
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
包含了用户账户创建，查询用户账户映射的地址，查询用户外部地址和查询用户账户状态等基础功能。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 获取用户的内部映射账户的地址 | String getAccountAddress() |  |
| 根据外部账户地址创建用户账户 | String createAccount(String externalAccount) | 传入参数为用户外部账户地址。 |
| 指定账户管理器地址和外部账户地址创建用户账户 | String createAccount(AccountManager accountManager, String externalAccount) | 传入参数为账户管理器地址和外部账户地址。 |
| 根据内部映射账户地址获取外部账户地址 |  String getExternalAccount(String userAccount)  | 传入参数为内部映射账户的地址。|
| 根据外部账户地址获取用户账户对象 | UserAccount getUserAccount(String externalAccount) | 传入参数为用户外部账户地址。 |
| 获取账户状态 |  int getUserAccountStatus(String externalAccount) | 传入参数为用户外部账户地址。|
| 根据外部账户地址获取内部映射账户的地址 |  String getBaseAccountAddress(String externalAccount)  | 传入参数为用户外部账户地址。 |
| 判断外部账户地址是否已创建 |  boolean hasAccount(String externalAccount) | 传入参数为用户外部账户地址。|
| 判断账户状态是否正常 | boolean isExternalAccountNormal(String externalAccount) | 传入参数为用户外部账户地址。|
| 修改绑定的私钥对 |   void changeCredentials(CryptoKeyPair credentials)  | 传入参数为新的私钥对。|


### 治理账户类

#### GovernAccountInitializer

包含了创建治理治理合约类的接口。只有治理者才能发起下述接口的交易。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 创建管理员模式的治理合约 | WEGovernance createGovernAccount(CryptoKeyPair credential) | 传入参数为管理员账户的私钥。 |
| 创建多签制模式的治理合约 | WEGovernance createGovernAccount(List<String> externalAccountList, int threshold) | 传入参数为治理成员外部账户地址列表和通过的阈值。|
| 创建权重投票模式的治理合约 | WEGovernance createGovernAccount(List<String> externalAccountList, List<BigInteger> weights, int threshold) | 传入参数为治理成员外部账户地址列表、各治理账户对应的权重和通过的阈值。 |

#### AdminModeGovernManager
包含了管理员模式下的各类操作接口。只有超级管理员才能发起下述接口的交易。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 移交管理员权限 | TransactionReceipt transferAdminAuth(String newAdminAddr) | 传入参数为新的管理员外部账户地址。 |
| 重置用户账户私钥 | TransactionReceipt resetAccount(String oldAccount, String newAccount) | 传入参数为旧的以及新的用户外部账户地址。|
| 冻结用户账户 | TransactionReceipt freezeAccount(String externalAccount) | 传入参数为要冻结的外部账户地址。 |
| 解冻用户账户 | TransactionReceipt unfreezeAccount(String externalAccount) | 传入参数为要解冻的外部账户地址。 |
| 注销用户账户 | TransactionReceipt cancelAccount(String externalAccount) | 传入参数为要注销的外部账户地址。 |

#### VoteModeGovernManager
包含了投票模式下的各类操作接口，包含了多签制和阈值投票。其中，多签制也可被视为一种特殊的阈值投票，即所有治理账户的投票阈值为1。只有治理者才能发起下述接口的交易。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 请求重设投票阈值 | BigInteger requestResetThreshold(int newThreshold) | 传入参数为新投票阈值。 |
| 请求删除治理账户 |  BigInteger requestRemoveGovernAccount(String externalAccount) | 传入参数为请求删除的用户外部账户地址。|
| 请求重置治理账户 | BigInteger requestResetGovernAccount(String externalAccount, int weight) | 传入参数为待重置的外部账户地址和投票的权重。 |
| 请求添加治理账户 | BigInteger requestAddGovernAccount(String externalAccount, int weight) | 传入参数为待添加的外部账户地址和投票的权重。 |
| 请求添加治理账户 | BigInteger requestAddGovernAccount(String externalAccount) | 传入参数为待添加的外部账户地址，投票权重默认为1。 |
| 请求重置用户账户 |  BigInteger requestResetAccount(String newExternalAccount, String oldExternalAccount) | 传入参数为待重置的新旧两个用户外部账户地址。|
| 请求冻结用户账户 |  BigInteger requestFreezeAccount(String externalAccount) | 传入参数为待冻结的用户外部账户地址。|
| 请求解冻用户账户 |  BigInteger requestUnfreezeAccount(String externalAccount)  | 传入参数为待解冻的外部账户地址。 |
| 请求注销用户账户 | BigInteger requestCancelAccount(String externalAccount) | 传入参数为待注销的外部账户地址。 |
| 对请求内容进行投票 | TransactionReceipt vote(BigInteger requestId, boolean agreed) | 传入参数为投票事务的ID和投票意见。 |
| 执行重置用户账户 | TransactionReceipt resetAccount(BigInteger requestId, String newExternalAccount, String oldExternalAccount) | 传入参数为投票事务ID，重置的新旧两个用户外部账户地址。 |
| 执行冻结用户账户 | TransactionReceipt freezeAccount(BigInteger requestId, String externalAccount) | 传入参数为投票事务ID和操作的外部账户地址。|
| 执行解冻用户账户 | TransactionReceipt unfreezeAccount(BigInteger requestId, String externalAccount) | 传入参数为投票事务ID和操作的外部账户地址。 |
| 执行注销用户账户 |  TransactionReceipt cancelAccount(BigInteger requestId, String externalAccount) | 传入参数为投票事务ID和操作的外部账户地址。 |
| 执行重设投票阈值 | TransactionReceipt resetThreshold(BigInteger requestId, int threshold)  | 传入参数为投票事务ID和重设的阈值。 |
| 执行删除治理账户 | TransactionReceipt removeGovernAccount(BigInteger requestId, String externalAccount) | 传入参数为投票事务ID和操作的外部账户地址。 |
| 执行重置治理账户私钥 | TransactionReceipt resetGovernAccount(BigInteger requestId, String externalAccount, int weight) | 传入参数为投票事务ID、操作的外部账户地址和重置的阈值。|
| 执行添加治理账户 | TransactionReceipt addGovernAccount(BigInteger requestId, String externalAccount, int weight) |传入参数为投票事务ID、操作的外部账户地址和重置的阈值。 |
| 执行添加治理账户 | TransactionReceipt addGovernAccount(BigInteger requestId, String externalAccount) | 传入参数为投票事务ID和操作的外部账户地址。默认的投票阈值为1. |
| 根据交易回执获取投票ID | BigInteger getId(TransactionReceipt tr) | 传入参数为交易回执。 |
| 根据投票ID获取投票请求信息 | VoteRequestInfo getVoteRequestInfo(BigInteger requestId) | 传入参数为投票事务ID。 |
| 获取治理合约的投票权重信息 |  WeightInfo getWeightInfo() |  |

### 普通用户账户类

#### EndUserOperManager
包含了普通用户账户相关的操作。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 重置账户私钥 | TransactionReceipt resetAccount(String newCredential) | 传入参数为新的外部账户地址。 |
| 删除社交好友方式重置私钥 | TransactionReceipt modifyManagerType() |  |
| 修改关联社交好友 | TransactionReceipt modifyManagerType(List<String> voters)  | 传入参数为三个关联的社交好友。 |
| 添加关联的社交好友账户 | TransactionReceipt addRelatedAccount(String externalAccount) | 传入参数为要添加的社交好友外部账户地址。|
| 删除关联的社交好友账户 | TransactionReceipt removeRelatedAccount(String externalAccount) | 传入参数为要删除的社交好友外部账户地址。 |
| 注销用户账户 | TransactionReceipt cancelAccount() | |
| 查询私钥重置方式 | int getUserStatics()  |  |

#### SocialVoteManager
包含了社交好友在行使重置私钥投票功能相关的操作。操作的主体为被其他用户设置并关联的社交好友账户。例如小明设置了三个社交好友的外部账户地址，其中一个好友为小华，当小明丢失了私钥后，可让小华替他发起重置的投票，只要三个社交好友中有两个投票通过，就可以发起重置小明私钥的操作了。这里的SocialVoteManager就是提供了小华相关的操作。

| 功能介绍 | 接口函数签名 | 参数说明|
| --- | --- | --- |
| 申请重置社交好友的私钥 | TransactionReceipt requestResetAccount(String newExternalAccount, String oldExternalAccount)| 传入参数为需要重置的新旧两个外部账户地址。 |
| 所关联的社交好友进行投票 |TransactionReceipt vote(String oldExternalAccount, boolean agreed) | 传入参数为待重置的外部账户地址和投票意见。|
| 执行重置私钥操作 | TransactionReceipt resetAccount(String newExternalAccount, String oldExternalAccount)| 传入参数为需要重置的新旧两个外部账户地址。 |


## 治理账户功能使用详细说明

### 创建治理合约
#### 管理员模式
假如平台方采用管理员的治理模式，那么需要首先生成一个管理员的治理账户。
<br />**具体调用示例：**<br />

```
    // 自动注入AdminModeGovernAccountInitializer对象
    @Autowired
    private GovernAccountInitializer governAccountInitializer;
    // 自动注入CryptoKeyPair对象
    @Autowired 
    private CryptoKeyPair cryptoKeyPair;
    // 使用自动注入的cryptoKeyPair随机创建一组管理员账户的私钥对
    CryptoKeyPair governanceUser1Keypair = cryptoKeyPair.generateKeyPair();
    // 调用 createGovernAccount 方法生成管理员账户
    WEGovernance govern = adminModeManager.createGovernAccount(governUserKeypair);
```

<br />**函数签名：**<br />

```
WEGovernance createGovernAccount(Credentials credential)
```

<br />**输入参数：**<br />

- credential 管理员的私钥


<br />**返回参数：**<br />

- WEGovernance 返回的治理账户对象


<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

#### 多签制治理模式

例如，以下平台方选择了治理委员会的治理模式，一共有三个参与者参与治理，治理的规则为任意的交易请求获得其中两方的同意，即可获得通过。那么我们接下来将创建一个治理账户。
<br />**具体调用示例：**<br />

```
    // 自动注入GovernAccountInitializer对象
    @Autowired
    private GovernAccountInitializer governAccountInitializer;
    // 准备3个治理账户管理成员的公钥地址，生成治理账户
    List<String> list = new ArrayList<>();
    list.add(address1);
    list.add(address2);
    list.add(address3);
    // 创建账户
    WEGovernance govern = adminModeManager.createGovernAccount(list, 2);
```

<br />**函数签名：**<br />

```
WEGovernance createGovernAccount(List<Credentials> credentials, int threshold)
```

<br />**输入参数：**<br />

- externalAccountList  治理委员会成员的外部账户信息。
- threshold 投票阈值，如超过该投票阈值，表示投票通过。


<br />**返回参数：**<br />

- WEGovernance 返回的治理账户对象


<br />调用成功后，函数会返回对应的WEGovernance治理账户对象，通过getContractAddress()方法可以获得对应的治理合约的地址。<br />

#### 权重投票治理模式

本模式类似于上一种多签制，区别在于每个投票者的投票权重可以是不相同的。
<br />例如，以下平台方选择了治理委员会的权重投票的治理模式，一共有三个参与者参与治理，投票的权重分别为1、2、3，阈值为4，也就是说任意的赞同选票权重相加超过阈值即可获得通过。那么我们接下来将创建一个治理账户。<br />
<br />**具体调用示例：**<br />

```
    // 自动注入GovernAccountInitializer对象
    @Autowired
    private GovernAccountInitializer governAccountInitializer;
    // 准备3个治理账户管理成员的公钥地址，生成治理账户
    List<String> list = new ArrayList<>();
    list.add(address1);
    list.add(address2);
    list.add(address3);
    // 设置3个投票账户的权重分别为1、2、3
    List<BigInteger> weights = new ArrayList<>();
    weights.add(BigInteger.valueOf(1));
    weights.add(BigInteger.valueOf(2));
    weights.add(BigInteger.valueOf(3));
    // 创建账户
    WEGovernance govern = adminModeManager.createGovernAccount(list, weights, 4);
```

<br />**函数签名：**<br />

```
WEGovernance createGovernAccount(List<String> credentialsList, List<BigInteger> weights, int threshold)
```

<br />**输入参数：**<br />

- externalAccountList  治理委员会成员的外部账户信息。
- weights  治理委员会成员对应的投票权重。
- threshold 投票阈值，如超过该投票阈值，表示投票通过。


<br />**返回参数：**<br />

- WEGovernance 返回的治理账户对象


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
