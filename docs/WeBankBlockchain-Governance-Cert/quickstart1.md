# cert-toolkit使用

## 功能介绍
cert-toolkit用于证书生成。支持轻量级jar包接入。

支持如下方式：
*   根证书生成
*   证书请求生成
*   子证书生成   
*   证书文件的读写

## 前置依赖

在使用本组件前，请确认系统环境已安装相关依赖软件，清单如下：

| 依赖软件 | 说明 |备注|
| --- | --- | --- |
| Java | JDK[1.8] | |
| Git | 下载的安装包使用Git | |

如果您还未安装这些依赖，请参考[附录](../appendix.md)。

```eval_rst
.. important::
    FISCO-BCOS 2.0与3.0对比、JDK版本、WeBankBlockChain-Governance及其他子系统的 `兼容版本说明 <https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/compatibility.html>`_
```



## 部署说明

目前支持从源码进行部署。

### 获取源码

通过git下载源码：

```
git clone https://github.com/WeBankBlockchain/Governance-Cert.git
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试：git clone https://gitee.com/WeBankBlockchain/Governance-Cert.git
```

进入目录：
```
cd Governance-Cert
cd cert-toolkit
```

### 编译源码

方式一：如果服务器已安装Gradle
```
gradle build -x test
```

方式二：如果服务器未安装Gradle，使用gradlew编译
```
chmod +x ./gradlew && ./gradlew build -x test
```

### 导入jar包

cert-toolkit编译之后在cert-toolkit目录下会生成dist文件夹，文件夹中包含cert-toolkit.jar。可以将cert-toolkit.jar导入到自己的项目中，例如libs目录下。然后进行依赖配置。gradle依赖配置如下，然后再对自己的项目进行编译。

```

repositories {
	maven {
		url "http://maven.aliyun.com/nexus/content/groups/public/"
	}
	maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
	maven { url "https://dl.bintray.com/ethereum/maven/" }
	mavenLocal()
	jcenter()
}

dependencies {
	testCompile 'junit:junit:4.12'
	compile 'org.slf4j:slf4j-api:1.7.30'
	compile ('org.projectlombok:lombok:1.18.6')
	annotationProcessor ('org.projectlombok:lombok:1.18.6')
	compile('ch.qos.logback:logback-core:1.2.3')
	compile('ch.qos.logback:logback-classic:1.2.3')
	compile "org.apache.commons:commons-lang3:3.6"
	compile "commons-io:commons-io:2.6"
	compile 'commons-codec:commons-codec:1.4'
	compile 'com.lhalcyon:bip32:1.0.0'
	compile 'org.web3j:core:3.4.0'
	compile 'com.lambdaworks:scrypt:1.4.0'
	compile group: 'org.bouncycastle', name: 'bcprov-jdk15on', version: '1.60'
	compile group: 'org.bouncycastle', name: 'bcpkix-jdk15on', version: '1.60'
	compile fileTree(dir:'libs',include:['*.jar'])
}

```


### 示例说明

下面介绍下证书的生成流程

##### 根证书生成

根据生成的私钥自签名生成根证书，可以指定私钥也可选择自动生成，组件封装了多种入参方法，可按需使用，参照[根证书生成](https://governance-doc.readthedocs.io/zh_CN/latest/toolkitdoc/com/webank/cert/toolkit/service/CertService.html#generateKPAndRootCert-com.webank.cert.toolkit.model.X500NameInfo-java.lang.String-java.lang.String-)

以generateKPAndRootCert方法为例，示例代码如下：

```
    CertService certService = new CertService();
    X500NameInfo info = X500NameInfo.builder()
            .commonName("chain")
            .organizationName("fisco-bcos")
            .organizationalUnitName("chain")
            .build();
    //自动生成私钥（默认为RSA），并写入指定路径"out/ca"，
    certService.generateKPAndRootCert(info,"out/ca");
```

执行上述方法，会在控制台打印出证书、私钥文件保存结果和路径，证书会保存在out/ca/ca.crt文件中

##### 子证书csr生成

csr全称为Certificate Signing Request，即证书请求文件，根（父）证书通过其私钥对请求文件签名来颁发子证书

csr的生成提供了多种入参方法，参照[证书申请生成](https://governance-doc.readthedocs.io/zh_CN/latest/toolkitdoc/com/webank/cert/toolkit/service/CertService.html#createCertRequest-com.webank.cert.toolkit.model.X500NameInfo-java.security.PublicKey-java.security.PrivateKey-java.lang.String-)

以generateCertRequestByDefaultConf为例，可快速生成csr，示例代码如下：

```
    CertService certService = new CertService();
    X500NameInfo info = X500NameInfo.builder()
            .commonName("chain")
            .organizationName("fisco-bcos")
            .organizationalUnitName("chain")
            .build();
    //自动生成RSA私钥，KeyUtils为证书组件密钥工具类
    KeyPair keyPair = KeyUtils.generateKeyPair();
    //CertUtils工具提供了证书读写解析的相关能力
    String priStr = CertUtils.readPEMAsString(keyPair.getPrivate());
    String csrStr = certService.generateCertRequestByDefaultConf(info, priStr, "out/agency", "agency");
    System.out.println(csrStr);
```

执行上述方法会在控制台打印出csr文件内容，并写入out/agency/agency.csr文件中

其中涉及的CertUtils工具类，该类提供了证书读写解析的相关能力，参照[CertUtils详情](https://governance-doc.readthedocs.io/zh_CN/latest/toolkitdoc/com/webank/cert/toolkit/utils/CertUtils.html)

##### 子证书颁发

通过根证书和其私钥对子证书申请进行签发, 提供多种入参方法，参照[子证书签发](https://governance-doc.readthedocs.io/zh_CN/latest/toolkitdoc/com/webank/cert/toolkit/service/CertService.html#generateChildCertByDefaultConf-boolean-org.bouncycastle.asn1.x509.KeyUsage-java.lang.String-java.lang.String-java.lang.String-)

以文件路径为入参示例，示例代码如下：

```
    //参数为生成相关文件路径
    CertService certService = new CertService();
    String childStr2 = certService.generateChildCertByDefaultConf("out/ca/ca.crt","out/agency/agency.csr","out/ca/ca_pri.key", "out/agency" ,
           "agency");
    System.out.println(childStr2);
```

执行上述方法会在控制台打印出子证书内容,并写入out/agency/agency.crt文件中，可从第二步开始，继续下一级证书的签发。


##### 证书链验证

上述步骤中我们生成了多级证书，对生成的证书链进行验证，查看证书是否有效

示例代码如下：

```
    CertService certService = new CertService();
    try {
        X509Certificate root = null;
        root = CertUtils.readCrt("out/ca/ca.crt");
        X509Certificate child = CertUtils.readCrt("out/agency/agency.crt");
        List<X509Certificate> certChain = new ArrayList<>();
        //可添加多级证书...这里以上述步骤中生成的两个证书为例
        certChain.add(root);
        certChain.add(child);
        System.out.println("证书链验证结果 = " + certService.verify(root, certChain));
    } catch (CertificateException | FileNotFoundException e) {
        e.printStackTrace();
    }
```

执行上述方法，可以在控制台看到证书链的验证结果为true，表明证书链的有效性

##### 证书撤销

我们可以对有效期内的证书进行吊销操作

```
    CertService certService = new CertService();
    try {
        //从文件中读取证书（上述步骤中生成的证书路径）
        X509Certificate root = CertUtils.readCrt("out/ca/ca.crt");
        X509Certificate child = CertUtils.readCrt("out/agency/agency.crt");
        //从文件中读取私钥（上述步骤中生成的私钥路径）
        PrivateKey caPrivateKey = (PrivateKey) CertUtils.readRSAKey("out/ca/ca_pri.key");
        List<X509Certificate> revokeCertificates = new ArrayList<>();
        revokeCertificates.add(child);
        //撤销上述步骤中签发的子证书
        X509CRL X509Crl = certService.createCRL(root,caPrivateKey,revokeCertificates,"SHA256WITHRSA");
        System.out.println("吊销证书路径：out/agency/agency.crt");
        X509Crl.getRevokedCertificates().forEach(x509CRLEntry -> {
            System.out.println("吊销证书序列号：" +  x509CRLEntry.getSerialNumber());
        });
    
        //验证吊销证书后的证书链
        List<X509Certificate> certChain = new ArrayList<>();
        //可添加多级证书...这里以上述步骤中生成的两个证书为例
        certChain.add(root);
        certChain.add(child);
        System.out.println("证书链验证结果 = " + certService.verify(root,certChain,X509Crl));
    } catch (Exception e) {
        e.printStackTrace();
    }
```

执行上述方法，可以在控制台看到吊销后的证书链验证结果为false，表明该子证书已经失效

##### PFX证书读写

crt证书只包含公钥信息

pfx由PKCS#12（Public Key Cryptography Standards #12）标准定义，包含了公钥和私钥信息。

CertUtils工具类提供了pfx的生成和读取方法，示例代码如下:

```
    try {
        List<X509Certificate> list = new ArrayList<>();
        list.add(CertUtils.readCrt("out/ca/ca.crt"));
        //生成pfx文件，参数分别为：证书别名，私钥，keyStore密码，证书信息，保存路径，证书名
        CertUtils.savePfx("fisco",(PrivateKey) CertUtils.readRSAKey("out/ca/ca_pri.key"),"123",list,"out/ca","ca");
        //从pfx中导出私钥信息
        PrivateKey key = CertUtils.readPriKeyFromPfx("out/ca/ca.pfx","123");
        //在控制台输出导出私钥的BASE64编码信息
        System.out.println(Base64.toBase64String(key.getEncoded()));
    } catch (Exception e) {
        e.printStackTrace();
    }
```
执行上述方法，可以在out/ca路径下看到ca.pfx证书文件的生成，以及从该pfx文件中导出的私钥Base64编码信息


### 接口说明

CertService提供了多种功能接口：
- createRootCertificate：生成根证书，即自签名证书
- createCertRequest：生成证书请求
- createChildCertificate：生成子证书
- verify：验证证书
- createCRL：吊销证书

为方便调用，针对上述接口封装了默认配置（签名算法：SHA256WITHRSA,有效期10年）的生成接口：
- generateRootCertByDefaultConf：生成根证书
- generateCertRequestByDefaultConf：生成证书请求
- generateChildCertByDefaultConf：生成子证书
- generateKPAndRootCert：生成密钥对和根证书

KeyUtils和CertUtils两个工具类，提供了对证书和私钥的相关读写操作。

更多参照[Java doc](https://governance-doc.readthedocs.io/zh_CN/latest/toolkitdoc/navigation.html)
