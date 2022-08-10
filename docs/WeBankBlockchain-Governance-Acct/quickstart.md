# 快速开始

<br />本章节以尽量短的时间，为使用者提供最简单的WeBankBlockchain-Governance-Account的快速入门。<br />

## 前置依赖

| 依赖软件 | 说明 |备注|
| --- | --- | --- |
| FISCO-BCOS | 3.0.0及以上版本 | rc4及以上版本 |
| Java | JDK[1.8] | |
| Git | 下载的安装包使用Git | |

```eval_rst
.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBankBlockChain-Governance及其他子系统的 `兼容版本说明 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_
```

```eval_rst
.. note::
    - JDK1.8 或者以上版本，推荐使用OracleJDK。CentOS的yum仓库的OpenJDK缺少JCE(Java Cryptography Extension)，会导致JavaSDK无法正常连接区块链节点。
    - 参考  `Java环境配置 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/sdk/java_sdk/quick_start.html#id2>`_  
    - FISCO BCOS区块链环境搭建参考 `FISCO BCOS安装教程 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html>`_  
    - 网络连通性。检查所连接的FISCO BCOS节点`channel_listen_port`是否能telnet通，若telnet不通，需要检查网络连通性和安全策略。
```


## 获取源码

使用git下载源码：
```
git clone https://github.com/WeBankBlockchain/Governance-Account.git
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试：git clone https://gitee.com/WeBankBlockchain/Governance-Account.git
```

- 所有智能合约文件位于src/main/contracts路径下
- 合约demo位于src/main/contracts/samples路径下
- 合约测试代码位于src/test/java路径下
- 其余部分代码为Java SDK代码。


## 使用组件合约

<br />本章节介绍的是只使用合约本身的方式进行账户治理。如果需要使用SDK来治理的，可跳过本章节。<br />
<br />在完成FISCO BCOS链环境初始化以后，通过将WeBankBlockchain-Governance-Account作为独立的插件，可使用控制台来部署账户治理的合约。可参考控制台的[配置与运行](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console_of_java_sdk.html)。<br />

### 使用控制台部署和调用合约

<br />本方式可直接通过控制台或WeBASE-Front来操作。<br />
<br />在此我们以控制台为例，进行演示，关于控制台的使用说明请参考[控制台命令列表](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console_of_java_sdk.html#id19)。<br />

### 准备和登入控制台环境

<br />在进入以下的操作前，首先我们进入控制台，准备至少四组操作私钥。<br />

```
# ./get_account.sh
[INFO] Account Address   : 0x1c7560296c101171eb8015cdc6cfbda26c866189
[INFO] Private Key (pem) : accounts/0x1c7560296c101171eb8015cdc6cfbda26c866189.pem
[INFO] Public  Key (pem) : accounts/0x1c7560296c101171eb8015cdc6cfbda26c866189.public.pem
```

<br />执行以上操作，会在控制台的accounts目录下生成对应的pem私钥文件。<br />
<br />连续执行四次后，查看accounts目录，会发现已得到了四组密钥文件。<br />

```
ll accounts
total 32
-rw-r--r-- 1 root root 237 Jul  5 11:46 0x14b0b2e52a156fcd310acee0692501ca23bb8a3e.pem
-rw-r--r-- 1 root root 174 Jul  5 11:46 0x14b0b2e52a156fcd310acee0692501ca23bb8a3e.public.pem
-rw-r--r-- 1 root root 237 Jul  5 11:47 0x1c7560296c101171eb8015cdc6cfbda26c866189.pem
-rw-r--r-- 1 root root 174 Jul  5 11:47 0x1c7560296c101171eb8015cdc6cfbda26c866189.public.pem
-rw-r--r-- 1 root root 237 Jul  5 11:56 0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8.pem
-rw-r--r-- 1 root root 174 Jul  5 11:56 0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8.public.pem
-rw-r--r-- 1 root root 237 Jul  5 11:46 0x7abb8086fcffd0417d9a4c9b615426aca5e4182d.pem
-rw-r--r-- 1 root root 174 Jul  5 11:46 0x7abb8086fcffd0417d9a4c9b615426aca5e4182d.public.pem
```

<br />由于每次生成的密钥都是随机的，故而密钥的地址都是不同的。<br />
<br />接下来，我们使用最上面的私钥作为操作地址，登入控制台：<br />

```
./start.sh 1 -pem accounts/0x14b0b2e52a156fcd310acee0692501ca23bb8a3e.pem  #1为群组ID
=============================================================================================
Welcome to FISCO BCOS console(1.0.10)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
[group:1]>
```


### 部署治理合约

<br />为了便于部署WeBankBlockchain-Governance-Account治理合约，我们共提供了三个Builder合约，来便于快速部署，分别是：<br />

1. AdminGovernBuilder
2. VoteGovernBuilder
3. WeightVoteGovernBuilder



#### 超级管理员模式


```shell
deploy AdminGovernBuilder
```

<br />在控制台中，执行上述的命令，然后可以调用_governance函数来获得治理账户合约的地址，然后调用WEGovernnance合约的getAccountManager函数来获得账户管理合约的地址和账户管理控制器的地址。<br />

```shell
# 部署超级管理员模式的治理账户
[group:1]> deploy AdminGovernBuilder
contract address: 0x6b10051756bf259efe7b4c22fad3925700ab2e1e
# 获取治理合约的地址
[group:1]> call AdminGovernBuilder 0x6b10051756bf259efe7b4c22fad3925700ab2e1e getGovernance
0x9d8ff088555122e7bcb0827130a9ac64780dff25
# 获取账户管理控制器的地址
[group:1]> call AdminGovernBuilder 0x6b10051756bf259efe7b4c22fad3925700ab2e1e getAccountManager
0x26ceac6bb9e727ed69fa391ae2eba9a3f5c0c8d0
```


#### 治理委员会模式

<br />需要输入部署的治理账户的外部账户地址列表和阈值。<br />

```shell
deploy VoteGovernBuilder(externalAccounts, threshold)
```

<br />我们可以配置每个投票账户拥有相同的投票权重，这种投票方式也被称为多签制。<br />例如，我们部署三个地址，分别为三个不同的地址，三个账户的投票权限相同，阈值为2：<br />

```shell
# 部署多签制模式的治理账户
[group:1]> deploy VoteGovernBuilder ["0x14b0b2e52a156fcd310acee0692501ca23bb8a3e","0x1c7560296c101171eb8015cdc6cfbda26c866189","0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8"] 2
contract address: 0xe96b9fcb5c9d1e0d9a4baa2ad304578363c05bd6
# 获取治理合约的地址
[group:1]> call VoteGovernBuilder 0xe96b9fcb5c9d1e0d9a4baa2ad304578363c05bd6 getGovernance
0x003a512af46eb456b1c57a70b95caf104db1df1b
# 获取账户管理控制器的地址
[group:1]> call VoteGovernBuilder 0xe96b9fcb5c9d1e0d9a4baa2ad304578363c05bd6 getAccountManager
0x50c903e4580682060984b971270f7c7864a6cc81
```

<br />当然我们也可以配置每个投票的账户拥有不同的权重，这种投票方式也被称为权重制。<br />例如，我们部署三个地址，分别为三个不同的地址，对应的权重为1，2，3， 阈值为2：<br />

```shell
# 部署多签制模式的治理账户
[group:1]> deploy WeightVoteGovernBuilder ["0x14b0b2e52a156fcd310acee0692501ca23bb8a3e","0x1c7560296c101171eb8015cdc6cfbda26c866189","0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8"] [1,2,3] 2
contract address: 0xbf8b5357a01232ab2e3ac8922fbe9a425afba026
# 获取治理合约的地址
[group:1]> call WeightVoteGovernBuilder 0xbf8b5357a01232ab2e3ac8922fbe9a425afba026 getGovernance
0x9ba26b43e00fb112da14b509715b868bb8fb1a2d
# 获取账户管理控制器的地址
[group:1]> call WeightVoteGovernBuilder 0xbf8b5357a01232ab2e3ac8922fbe9a425afba026 getAccountManager
0xf0da09d98fe320371fe30d1bd035f0cabc305eaf
```

<br />以上，我们分别展示了如何在链上部署超级管理员模式和治理委员会模式（包括多签制和权重制）的治理合约的部署。<br />
<br />在一个链上，可以基于不同的业务应用或场景来采用不同的管理模式。一般在一种独立的业务或场景中，部署一个治理合约。例如，在链的监管场景中，推荐使用超级管理员模式。在区块链分布式商业中，推荐使用多签制。在类似董事会的治理场景中，推荐采用可设置投票人不同权重的权重投票制。<br />
<br />
<br />

### 调用治理合约

<br />在部署了上述合约的前提下，可根据上一章节获得WEGovernance和AccountManager合约的地址。<br />

#### 账户相关的常见操作

<br />上述所有的Manager都继承了BasicManager基础Manager管理器，支持以下操作：<br />

- newAccount 创建普通账户
- getExternalAccount 查询普通账户地址
- hasAccount 查询普通账户是否已开户
- isExternalAccountNormal 查询外部账户状态是否正常


<br />相关的使用实例如下。<br />
<br />创建一个普通账户<br />

```
[group:1]>  call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf newAccount "0x1c7560296c101171eb8015cdc6cfbda26c866189"
transaction hash: 0xeee7badb2ecdddd080aca04cda58498f27709efd0d1773fa770ed1c91f587119
---------------------------------------------------------------------------------------------
Output
function: newAccount(address)
return type: (bool, address)
return value: (true, 0x39be54ce16ccb720aa0ffedea27a7e3621565598)
---------------------------------------------------------------------------------------------
Event logs
event signature: LogSetOwner(address,address) index: 0
event value: (0x1c7560296c101171eb8015cdc6cfbda26c866189, 0x39be54ce16ccb720aa0ffedea27a7e3621565598)
event signature: LogManageNewAccount(address,address,address) index: 0
event value: (0x1c7560296c101171eb8015cdc6cfbda26c866189, 0x39be54ce16ccb720aa0ffedea27a7e3621565598, 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf)
---------------------------------------------------------------------------------------------
```

<br />查询普通账户地址<br />

```
[group:1]> call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf getUserAccount "0x1c7560296c101171eb8015cdc6cfbda26c866189"
0x39be54ce16ccb720aa0ffedea27a7e3621565598
```

<br />查询普通账户是否已开户<br />

```
[group:1]> call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf hasAccount "0x1c7560296c101171eb8015cdc6cfbda26c866189"
true
```

<br />查询外部账户状态是否正常<br />

```
[group:1]> call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf isExternalAccountNormal "0x1c7560296c101171eb8015cdc6cfbda26c866189"
true
```


#### 治理者的相关操作

<br />治理者可分为超级管理员和治理委员会两种模式，其对应的治理操作也有所不同。<br />
<br />**在超级管理员模式下，包含了以下操作：**<br />获得Governance合约以后，可进行以下操作<br />

- transferOwner 移交超级管理员账户
- setExternalAccount 重置用户私钥
- doOper 账户相关操作， 3-冻结账户 4-解冻账户 5-cancelAccount 注销账户


<br />**移交超级管理员账户**<br />
<br />在控制台中进行操作。其中，被移交的账户已在上一章节中开户。<br />

```
[group:1]> call WEGovernance 0xbba738f0549cde5e5b0bab2dffc125501556018c transferOwner "0x1c7560296c101171eb8015cdc6cfbda26c866189"
transaction hash: 0x849d1fadfd95be911b6c4cd3be52a3799b51aa9f8d71086039e7ce1b48513fb2
Event logs
event signature: LogSetOwner(address,address) index: 0
event value: (0x1c7560296c101171eb8015cdc6cfbda26c866189, 0xbba738f0549cde5e5b0bab2dffc125501556018c)
---------------------------------------------------------------------------------------------
```

<br />**切换登录管理台的外部账户**<br />
<br />可以看到该治理合约的权限已被移交到新的『0x1c7560296c101171eb8015cdc6cfbda26c866189』地址中。故我们需要退出并切换新的私钥来登录控制台。<br />

```
[group:1]> exit
[root@instance-zw7wgjv0 console]# ./start.sh 1 -pem accounts/0x1c7560296c101171eb8015cdc6cfbda26c866189.pem
=============================================================================================
Welcome to FISCO BCOS console(1.0.10)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
[group:1]>
```

<br />**重置用户私钥**<br />

```
[group:1]> call WEGovernance 0xbba738f0549cde5e5b0bab2dffc125501556018c setExternalAccount 2 "0x7abb8086fcffd0417d9a4c9b615426aca5e4182d" "0x14b0b2e52a156fcd310acee0692501ca23bb8a3e"
transaction hash: 0x8201112d9bce00d751e9f6fb13ef3182f4c0afafa750334a34820a5f5101fcc7
---------------------------------------------------------------------------------------------
Output
function: setExternalAccount(uint256,address,address)
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------
Event logs
---------------------------------------------------------------------------------------------
```

<br />**冻结、解冻与销户**<br />

```
## 冻结
[group:1]> call WEGovernance 0xbba738f0549cde5e5b0bab2dffc125501556018c doOper 3 "0x7abb8086fcffd0417d9a4c9b615426aca5e4182d" 3
transaction hash: 0x645ccdb7569d53387ab28ede5b3b066971a87ff8ab45ab9ef663b5f4bfde3204
---------------------------------------------------------------------------------------------
Output
function: doOper(uint256,address,uint8)
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------
Event logs
---------------------------------------------------------------------------------------------

## 解冻
[group:1]>  call WEGovernance 0xbba738f0549cde5e5b0bab2dffc125501556018c doOper 4 "0x7abb8086fcffd0417d9a4c9b615426aca5e4182d" 4
transaction hash: 0xe5ae51e446836abda81294175d508e0f938f12c53517673aabfa496d4f352b3c
---------------------------------------------------------------------------------------------
Output
function: doOper(uint256,address,uint8)
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------
Event logs
---------------------------------------------------------------------------------------------

## 注销
[group:1]> call WEGovernance 0xbba738f0549cde5e5b0bab2dffc125501556018c doOper 5 "0x7abb8086fcffd0417d9a4c9b615426aca5e4182d" 5
transaction hash: 0xd13bfc5cd76fb38bea1d05de7b4bec8463ce86a69f4c6b2a3c66ed8bcce582a4
---------------------------------------------------------------------------------------------
Output
function: doOper(uint256,address,uint8)
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------
Event logs
---------------------------------------------------------------------------------------------
```

<br />**在治理委员会模式下，包含了以下操作**：<br />

- register 申请发起治理操作，包括了： 10-申请重设治理委员会投票阈值， 11-申请重设治理委员会投票权重，2-申请重置账户，3-申请冻结账户，4-申请解冻账户， 5-申请注销账户
- vote 对申请项投票
- setExternalAccount 重置用户私钥
- doOper 投票通过后，发起治理操作，包括了冻结账户、解冻账户、注销账户等。
- setThreshold 设置投票的阈值。
- setWeight 设置投票者的权重。
- getRequestInfo 查看投票详情信息。


<br />详细的操作方式可参考治理合约的相关代码接口。<br />
<br />由于使用控制台操作治理委员会相关操作较为繁琐，我们推荐采用Java SDK的方式来进行。后期，我们会提供通过Web管理台的方式，敬请期待。<br />

#### 普通用户的相关操作

<br />普通用户可通过accountManager来重置和注销账户。<br />

- setExternalAccountByUser 重置用户私钥
- cancelByUser 注销账户


<br />如下所示<br />
<br />使用外部账户地址为『0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8』来注册账户。
```shell
# 注册账户
[group:1]> call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf newAccount "0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8"
transaction hash: 0x0d3ff289ca601906f1fa286561755efb405a832c6295061d3d7cdce447884abb
---------------------------------------------------------------------------------------------
Output
function: newAccount(address)
return type: (bool, address)
return value: (true, 0x5196261524954a59c5f23f05883eef59d6e5fd38)
---------------------------------------------------------------------------------------------
Event logs
event signature: LogSetOwner(address,address) index: 0
event value: (0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8, 0x5196261524954a59c5f23f05883eef59d6e5fd38)
event signature: LogManageNewAccount(address,address,address) index: 0
event value: (0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8, 0x5196261524954a59c5f23f05883eef59d6e5fd38, 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf)
---------------------------------------------------------------------------------------------
```

<br />退出并使用该外部账户地址对应的私钥来载入控制台。
```
[group:1]> exit
[root@instance-zw7wgjv0 console]# ./start.sh 1 -pem accounts/0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8.pem
=============================================================================================
Welcome to FISCO BCOS console(1.0.10)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
```

<br />销户
```
[group:1]> call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf cancelByUser
transaction hash: 0xbced5c2aaf2cbcb0c98ec9877246253f3f594816d456f29a581062d97901e891
---------------------------------------------------------------------------------------------
Output
function: cancelByUser()
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------
Event logs
event signature: LogManageCancel(address,address) index: 0
event value: (0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8, 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf)
---------------------------------------------------------------------------------------------
```

<br />重新开户并设置外部账户地址为『0x1』
```
[group:1]> call AccountManager 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf setExternalAccountByUser "0x1"
transaction hash: 0x0ff2e49f11767826a311562fec36f81decfc9d67242e8d399473c5fbde53db9d
---------------------------------------------------------------------------------------------
Output
function: setExternalAccountByUser(address)
return type: (bool)
return value: (true)
---------------------------------------------------------------------------------------------
Event logs
event signature: LogManageExternalAccount(address,address,address) index: 0
event value: (0x0000000000000000000000000000000000000001, 0x5b15b41277f4cacfdad39ba06a5dcc1295af0fd8, 0xf0da09d98fe320371fe30d1bd035f0cabc305eaf)
---------------------------------------------------------------------------------------------
```

<br />此外，支持用户自主设置账户治理模式，支持以下操作。<br />

- setStatics 修改重置类型，0-不支持社交好友重置私钥，1-支持社交好友重置私钥
- setWeight 设置关联社交好友普通账户的权重
- register 发起投票
- vote 投票
- setExternalAccountBySocial 重置私钥


<br />更多操作，请查看智能合约代码。由于直接操作合约的方式较为繁琐，此处不再进行详细地演示。**我们推荐使用Java SDK集成的方式来进行相关的操作。** 后续，我们将提供web管理台来便于用户使用，敬请期待。<br />

### 将合约引入到用户自身的业务合约中


#### 组件合约引入

<br />用户在具体的业务合约中，可采用引入的方式来使用WeBankBlockchain-Governance-Account智能合约。<br />

##### 在通用的场景中引入治理合约


```
// 在业务合约中声明import账户管理合约。
import "./AccountManager.sol"

XXContract {
  // 创建AccountManager合约
  AccountManager _accountManager;
  
  construct(address accountManager) {
	    // 传入已部署的AccountManager地址。此地址由上面的部署治理合约步骤获得。
        _accountManager = AccountManager(accountManager);
  } 
  
    // ……
  function doSomeBiz() public {
        // 获得普通用户的地址
        address userAccountAddress = _accountManager.getUserAccount(msg.sender);
		// **特别注意**： 在具体的业务中，需使用此普通用户的地址来代替以往的外部地址。
        doBiz(userAccountAddress);
  }

}
```


##### 在涉及转账的场景中引入治理合约

<br />在涉及到转账的场景中，引入合约的方式更为典型，本例来源于我们提供的使用demo，限于篇幅，此处略去了和示例非密切相关的部分，获取完整的demo代码可查找src/main/contracts/samples/TransferDemo/路径。<br />

```
import "./AccountManager.sol";
contract TransferDemo {
	// ……
    mapping(address => uint256) private _balances;
    // import AccountManager
    AccountManager _accountManager;

    constructor(address accountManager, uint256 initBalance) public {
        // 初始化accountManager，此地址由上面的部署治理合约步骤获得。
        _accountManager = AccountManager(accountManager);
		// 合约的owner设置为用户普通账户的地址
        address owner = _accountManager.getUserAccount(msg.sender);
        _balances[owner] = initBalance;
    }

    modifier validateAccount(address addr) {
        require(
            // 调用accountManager来判断该外部账户的地址是否已被注册及账户状态是否正常等。
            _accountManager.isExternalAccountNormal(addr),
            "Account is abnormal!"
        );
        _;
    }
   // ……
    function balance(address owner) public view returns (uint256) {
        // 1.先根据外部账户的地址查询普通账户的地址，然后仔查询余额
        return _balances[_accountManager.getUserAccount(owner)];
    }
    function transfer(address toAddress, uint256 value)
        public
        // 检查转入方和转出方的外部账户地址是否合法。
        validateAccount(msg.sender)
        validateAccount(toAddress)
        checkTargetAccount(toAddress)
        returns (bool)
    {
        // 1. 查询转出方的普通账户地址
        address fromAccount = _accountManager.getUserAccount(msg.sender);
        // 2. 查询转出方普通账户余额
        uint256 balanceOfFrom = _balances[fromAccount].sub(value);
        // 3. 转出方金额扣除
        _balances[fromAccount] = balanceOfFrom;
        // 4. 查询转入方的普通账户地址
        address toAccount = _accountManager.getUserAccount(toAddress);
        // 5. 转入方余额增加
        uint256 balanceOfTo = _balances[toAccount].add(value);
        // 设置转入方普通账户的余额
        _balances[toAccount] = balanceOfTo;
        return true;
    }
}
```

<br />接下来，用户可到FISCO BCOS控制台中，将XXContract的业务合约编译为具体的Java代码。控制台提供一个专门的编译合约工具，方便开发者将Solidity合约文件编译为Java合约文件，具体使用方式参考[合约编译工具](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/console/console_of_java_sdk.html#id5)<br />
<br />也可以通过控制台或WeBASE-Front等工具部署到链上，在此不再赘述。<br />
<br />更加完整地引入治理合约的例子和使用方式可以参考工程中附带的samples demo。我们提供了基于存证和积分转账场景的两个demo。<br />

