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


<br />**其他治理模式：**<br />

[使用多签制治理模式](./multi_sig.md)

[使用权重投票治理模式](./weight_vote.md)
