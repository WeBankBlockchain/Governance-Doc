

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



