# Key-core快速开始

Key-core支持两种使用模式：可视化模式和sdk模式。在可视化模式下，会启动一个本地的网站，上面包含了私钥操作的核心功能。在sdk模式下，用户可将sdk引入到自己的项目中，并根据sdk提供的私钥功能来实现自己的业务代码。

```eval_rst
.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBankBlockChain-Governance及其他子系统的 `兼容版本说明 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_
```

## 源码下载

通过git 下载源码.
```
cd ~
git clone https://github.com/WeBankBlockchain/Governance-Key.git
cd Governance-Key
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试：git clone https://gitee.com/WeBankBlockchain/Governance-Key.git
```


## 使用可视化界面
进入目录：
```
cd key-core-web
```

编译代码：
```
gradle bootJar
```

编译后，会生成dist目录，包含key-core-web.jar包。

启动可视化界面：
```
cd dist
java -jar key-core-web.jar
```

启动成功后，会自动弹出浏览器页面。如果未自动弹出，也可以访问localhost:8001端口。网页效果如下：

![](img/keycoreweb.png)

可以使用密钥格式转换、助记词等功能。注意文件格式仅支持.keystore, .json, .pem, .p12这几种格式。

## 使用sdk

### 源码编译

进入目录:

```
cd ~/Governance-Key/key-core
```

编译代码：
```
gradle build -x test
```
完成编译之后，在根目录下会生成dist文件夹，文件夹中包含key-core.jar。

### 引入jar包
将dist目录中的key-core.jar包导入到自己的项目中，例如放到libs目录下。然后进行依赖配置，以gradle为例，依赖配置如下：
```
repositories {
    maven {
        url "http://maven.aliyun.com/nexus/content/groups/public/"
    }
    maven { url "https://oss.sonatype.org/service/local/staging/deploy/maven2"}
    maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
    mavenLocal()
    mavenCentral()
}

dependencies {
    compile 'com.webank:webankblockchain-crypto-core:1.0.0-SNAPSHOT'

    compile "org.apache.commons:commons-lang3:3.6"
    compile group: 'org.bouncycastle', name: 'bcprov-jdk15on', version: '1.60'
    compile group: 'org.bouncycastle', name: 'bcpkix-jdk15on', version: '1.60'
    compile 'org.web3j:core:3.4.0'
    compile ('org.fisco-bcos.java-sdk:java-sdk:2.7.0')
    compile "commons-io:commons-io:2.6"
    compile 'com.lambdaworks:scrypt:1.4.0'
    compile 'commons-codec:commons-codec:1.9'
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile fileTree(dir:'libs',include:['*.jar'])
}

```
### 常用场景示例

#### 随机数方式生成私钥

```java
import com.webank.keygen.model.PkeyInfo;
import com.webank.keygen.service.PkeyByRandomService;
import com.webank.keygen.service.PkeySM2ByRandomService;
import com.webank.keygen.utils.KeyPresenter;

public class KeyGenerationDemo {

    public static void main(String[] args) throws Exception{
        //生成非国密私钥
        PkeyByRandomService eccService = new PkeyByRandomService();
        PkeyInfo eccKey = eccService.generatePrivateKey();
        System.out.println("private key:"+ KeyPresenter.asString(eccKey.getPrivateKey()));
        System.out.println("public key:" + KeyPresenter.asString(eccKey.getPublicKey().getPublicKey()));
        System.out.println("address:" +eccKey.getAddress());

        //生成国密私钥
        PkeySM2ByRandomService gmService = new PkeySM2ByRandomService();
        PkeyInfo gmPkey = gmService.generatePrivateKey();
        System.out.println("private key:"+ KeyPresenter.asString(gmPkey.getPrivateKey()));
        System.out.println("public key:" + KeyPresenter.asString(gmPkey.getPublicKey().getPublicKey()));
        System.out.println("address:" +gmPkey.getAddress());
    }
}
```

#### 私钥转换为公钥及地址

当持有一个私钥明文时，可将其转换为公钥和地址。

```java
import com.webank.keygen.enums.EccTypeEnums;
import com.webank.keygen.model.PkeyInfo;
import com.webank.keygen.utils.KeyPresenter;

public class KeyToPubAndAddress {

    public static void main(String[] args) throws Exception{
        String pkeyStr = "0xed1d9dc98c8496b9837cb8c46a2302b9d479aab08f536dc0785115c11990d7f3";
        byte[] privateKey = KeyPresenter.asBytes( pkeyStr);
        PkeyInfo pkeyInfo
                = PkeyInfo.builder()
                .privateKey(privateKey)
                .eccName(EccTypeEnums.SECP256K1.getEccName())
                .build();

        System.out.println("public key :" + KeyPresenter.asString(pkeyInfo.getPublicKey().getPublicKey()));
        System.out.println("address :" + pkeyInfo.getAddress());
    }
}
```

#### 助记词使用

用户可以通过助记词和口令来恢复私钥明文。下述示例中包含了助记词生成、助记词恢复私钥的代码：

```java
import com.webank.keygen.enums.EccTypeEnums;
import com.webank.keygen.model.PkeyInfo;
import com.webank.keygen.service.PkeyByMnemonicService;
import com.webank.keygen.utils.KeyPresenter;

public class MnemonicUsage {

    public static void main(String[] args) throws Exception{
        //助记词生成
        PkeyByMnemonicService mnemonicService = new PkeyByMnemonicService();
        String mnemonic = mnemonicService.createMnemonic();
        System.out.println("Mnemonic:"+mnemonic);
        //生成私钥
        PkeyInfo pkey = mnemonicService.generatePrivateKeyByMnemonic(mnemonic, "passphrase", EccTypeEnums.SECP256K1);
        System.out.println("ECC Private Key:"+ KeyPresenter.asString(pkey.getPrivateKey()));
        System.out.println("ECC Chaincode:"+ KeyPresenter.asString(pkey.getChainCode()));
        System.out.println("ECC Address:"+ pkey.getAddress());
    }
}
```

#### 密钥派生

当用户要在不同场景，甚至不同区块链间使用私钥时，若每次都生成一个私钥，那么随着场景的增多，要保管的私钥势必也会增多，增加了私钥丢失、泄露的可能。所以，用户可以精心保管一个（或多个）根私钥，对于每个特殊场景，通过派生的方式得到子私钥。

就派生方式而言，支持私钥派生私钥，公钥派生公钥。私钥派生的私钥，与私钥对应公钥所派生的公钥是匹配的。此外，BIP-44规范提供了一组按场景组织的规范化派生方式。

```java

import com.webank.keygen.hd.bip32.ExtendedPrivateKey;
import com.webank.keygen.hd.bip32.ExtendedPublicKey;
import com.webank.keygen.hd.bip44.path.Purpose44Path;
import com.webank.keygen.model.PkeyInfo;
import com.webank.keygen.service.PkeyByRandomService;
import com.webank.keygen.service.PkeyHDDeriveService;
import com.webank.keygen.utils.KeyPresenter;

public class KeyDerive {

    public static void main(String[] args) throws Exception{
        //获得一个根私钥
        PkeyByRandomService generateService = new PkeyByRandomService();
        PkeyInfo pkeyInfo = generateService.generatePrivateKey();

        PkeyHDDeriveService deriveService = new PkeyHDDeriveService();
        ExtendedPrivateKey rootKey = deriveService.buildExtendedPrivateKey(pkeyInfo);
        //私钥派生子私钥
        ExtendedPrivateKey subPrivKey = rootKey.deriveChild(2);
        System.out.println("child no.2: "+ KeyPresenter.asString(subPrivKey.getPkeyInfo().getPrivateKey()));
        //子私钥转子公钥
        ExtendedPublicKey subPubKey= subPrivKey.neuter();
        System.out.println("pubkey for child no.2: "+ KeyPresenter.asString(subPubKey.getPubInfo().getPublicKey()));
        //BIP-44派生
        Purpose44Path derivePath = deriveService.getPurpose44PathBuilder().m()
                .purpose44().sceneType(2)
                .account(3).change(4).addressIndex(5).build();
        ExtendedPrivateKey derived = derivePath.deriveKey(rootKey);
        System.out.println("derived for bip 44 "+ KeyPresenter.asString(derived.getPkeyInfo().getPrivateKey()));
    }
}

```

#### 私钥加密导出到目录
```java

import com.webank.keygen.enums.EccTypeEnums;
import com.webank.keygen.model.PkeyInfo;
import com.webank.keygen.service.PkeyEncryptService;
import org.web3j.utils.Numeric;

public class KeyExport {
    public static void main(String [] args) throws Exception{
        //生成一个私钥
        PkeyInfo pkeyInfo
                = PkeyInfo.builder()
                .privateKey(Numeric.hexStringToByteArray("252ffefe4e3856eb84a4fba5f07fc2066d3043a763cb74ed16ff093ac79b52d6"))
                .eccName(EccTypeEnums.SECP256K1.getEccName())
                .build();
        byte[] privateKeyBytes = pkeyInfo.getPrivateKey();
        EccTypeEnums eccTypeEnums = EccTypeEnums.getEccByName(pkeyInfo.getEccName());
        //pem导出到
        PkeyEncryptService encryptService = new PkeyEncryptService();
        encryptService.encryptPEMFormat(privateKeyBytes, eccTypeEnums, System.getProperty("user.dir"));
        //keystores导出
        String password = "123456";
        encryptService.encryptKeyStoreFormat(password, privateKeyBytes,eccTypeEnums, System.getProperty("user.dir"));
        //p12导出
        encryptService.encryptP12Format(password, privateKeyBytes,eccTypeEnums, System.getProperty("user.dir"));
    }
}

```
#### 数据分片与还原

支持将数据分片，分片的数据可还原为原始数据。分片模式可指定为(n, t)，其中n表示数据要分为多少片，t表示只需要多少片即可还原出原始数据，例如分片模式若为（3，2），那么原始数据可分解为3片，当且仅当持有其中2片的时候，即可还原出原始数据。

```java
import com.webank.keygen.model.PkeyInfo;
import com.webank.keygen.service.PkeyByRandomService;
import com.webank.keygen.service.PkeyShardingService;
import com.webank.keygen.utils.KeyPresenter;
import java.util.ArrayList;
import java.util.List;

public class Shard {
    public static void main(String[] args) throws Exception{
        //生成一个私钥
        PkeyByRandomService generateService = new PkeyByRandomService();
        PkeyInfo pkeyInfo = generateService.generatePrivateKey();
        System.out.println("Before sharding "+KeyPresenter.asString(pkeyInfo.getPrivateKey()));
        //开始分片，分解为5片，凑齐任意3片才能还原
        PkeyShardingService shardingService
                = new PkeyShardingService();
        List<String> shards = shardingService.shardingPKey(pkeyInfo.getPrivateKey(), 5, 3);
        //还原
        List<String> recoveredShards = new ArrayList<>();
        recoveredShards.add(shards.get(0));
        recoveredShards.add(shards.get(2));
        recoveredShards.add(shards.get(3));
        byte[] recovered = shardingService.recoverPKey(recoveredShards);
        System.out.println("After recovered "+ KeyPresenter.asString(recovered));
    }
}

```
#### 密码学操作
下述例子包含了签名、验签、数据加密、数据解密。
```java

import com.webank.keysign.service.ECCEncryptService;
import com.webank.keysign.service.ECCSignService;
import com.webank.keysign.service.SM2EncryptService;
import com.webank.keysign.service.SM2SignService;

public class Crypto {
    public static void main(String[] args){
        //Case 1: Ecc(secp256k1) sign and verify
        String eccMsg = "HelloEccSign";
        String eccPrivateKey = "28018238ac7eec853401dfc3f31133330e78ac27a2f53481270083abb1a126f9";
        String eccPublicKey = "0460fc2bce5795ee2ac34d1f584f603b4e2920a95d8d3db5f5c664244a99fd76405831ffaf932f64eae3ec67bc8ff7bfed9039f29bf39ce6583d55ca449b64319e";

        ECCSignService eccSignService = new ECCSignService();
        String eccSignature = eccSignService.sign(eccMsg, eccPrivateKey);
        System.out.println("ecc signature:"+eccSignature);

        boolean eccVerifyResult = eccSignService.verify(eccMsg, eccSignature, eccPublicKey);
        System.out.println("ecc verify result:"+eccVerifyResult);

        //Case 2: Ecc(secp256k1) encryption and decryption
        ECCEncryptService eccEncryptService = new ECCEncryptService();
        String eccCipherText = eccEncryptService.encrypt(eccMsg, eccPublicKey);
        System.out.println("ecc encryption cipher:"+eccCipherText);

        String eccPlainText = eccEncryptService.decrypt(eccCipherText, eccPrivateKey);
        System.out.println("ecc decryption result:"+eccPlainText);

        //Case3: Gm(sm2p256v1) sign and verify
        String gmMsg = "HelloGM";
        String gmPrivateKey = "73c8a8054b5e42b0d089e24f16c665bc82a132082d258c5efb54c49a3b7273f9";
        String gmPublicKey = "0451c895673d372267a565c4a7711102108138132b21f22ed556df08fb4c8cfdcaf17dcb605f8a6394f8684aa1916df60929532faf808c36c133ce52356d0f45f3";

        SM2SignService sm2SignService = new SM2SignService();
        String gmSignature = sm2SignService.sign(gmMsg, gmPrivateKey);
        System.out.println("gm signature:"+gmSignature);

        boolean gmVerifyResult = sm2SignService.verify(gmMsg, gmSignature, gmPublicKey);
        System.out.println("gm verify result:"+gmVerifyResult);

        //Case4: Gm(sm2p256v1) encryption and decryption
        SM2EncryptService gmEncryptService = new SM2EncryptService();
        String gmCipherText = gmEncryptService.encrypt(gmMsg, gmPublicKey);
        System.out.println("gm encryption cipher:"+gmCipherText);

        String gmPlainText = gmEncryptService.decrypt(gmCipherText, gmPrivateKey);
        System.out.println("gm decryption result:"+gmPlainText);
    }
}

```

### SDK核心接口

下面包含核心接口：
* [PkeyByRandomService](https://governance-doc.readthedocs.io/zh_CN/latest/keycoredoc/com/webank/keygen/service/PkeyByRandomService.html):随机数方式私钥生成（非国密）
* [PkeySM2ByRandomService](https://governance-doc.readthedocs.io/zh_CN/latest/keycoredoc/com/webank/keygen/service/PkeySM2ByRandomService.html)：随机数方式私钥生成（国密）
* [PkeyByMnemonicService](https://governance-doc.readthedocs.io/zh_CN/latest/keycoredoc/com/webank/keygen/service/PkeyByMnemonicService.html)：助记词生成；助记词恢复私钥
* [PkeyHDDeriveService](https://governance-doc.readthedocs.io/zh_CN/latest/keycoredoc/com/webank/keygen/service/PkeyHDDeriveService.html)：密钥派生
* [PkeyEncryptService](https://governance-doc.readthedocs.io/zh_CN/latest/keycoredoc/com/webank/keygen/service/PkeyEncryptService.html)：私钥按不同格式导出和导入
* [PkeyShardingService](https://governance-doc.readthedocs.io/zh_CN/latest/keycoredoc/com/webank/keygen/service/PkeyShardingService.html)：分片与还原
