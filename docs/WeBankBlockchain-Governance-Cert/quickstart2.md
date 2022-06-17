
# cert-mgr使用

## 功能介绍

cert-mgr用于证书托管。

证书管理中，证书相关的私钥由单独的私钥表保存，还包含了证书表和请求表，生成的证书会保存在证书表中，子证书的请求会保存在请求表中。

证书签名算法目前支持：

*   SHA256WITHRSA
*   SHA256WITHECDSA
*   SM3WITHSM2

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

## 部署教程

目前支持从源码进行部署。

### 获取源码

通过git下载源码：

```
https://github.com/WeBankBlockchain/Governance-Cert.git
```
```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试：git clone https://gitee.com/WeBankBlockchain/Governance-Cert.git
```
进入目录：
```
cd Governance-Cert/cert-mgr
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

cert-mgr编译之后在根目录下会生成dist文件夹，文件夹中包含cert-mgr.jar。可以将cert-mgr.jar导入到自己的项目中，例如拷贝到libs目录下，然后进行依赖配置。gradle推荐依赖配置如下，然后再对自己的项目进行编译。

```
repositories {
    mavenCentral()
    mavenLocal()
    maven {
        url "http://maven.aliyun.com/nexus/content/groups/public/"
    }
}
dependencies {
        compile 'org.springframework.boot:spring-boot-starter'
    	compile 'org.springframework.boot:spring-boot-starter-data-jpa'
    	compile 'org.springframework.boot:spring-boot-starter-jta-atomikos'
    
    	testCompile('org.springframework.boot:spring-boot-starter-test') {
    		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    		//exclude group: 'junit', module: 'junit'
    	}
    	compile ('org.projectlombok:lombok:1.18.6')
    	testCompile ('org.projectlombok:lombok:1.18.6')
    	annotationProcessor 'org.projectlombok:lombok:1.18.6'
    	testAnnotationProcessor 'org.projectlombok:lombok:1.18.6'
    	testCompile 'junit:junit:4.12'
    	compile "org.apache.commons:commons-lang3:3.6"
    	compile "commons-io:commons-io:2.6"
    	compile 'com.lhalcyon:bip32:1.0.0'
    	compile 'org.web3j:core:3.4.0'
    	compile 'com.lambdaworks:scrypt:1.4.0'
    	compile 'commons-codec:commons-codec:1.9'
    	compile 'mysql:mysql-connector-java'
    	compile group: 'org.bouncycastle', name: 'bcprov-jdk15on', version: '1.60'
    	compile group: 'org.bouncycastle', name: 'bcpkix-jdk15on', version: '1.60'
    	compile fileTree(dir:'libs',include:['*.jar'])
}

```


## 使用详解

依赖注入使用前需添加组件扫描，如下所示：

```java
@SpringBootApplication
@ComponentScan(basePackages = { "com.webank.cert" })
public class CertTestApplication {
	public static void main(String[] args) {
		SpringApplication.run(CertTestApplication.class, args);
	}
}
```

### 配置

请参考下面的模板，配置application.properties。
```
## 证书存储db
spring.datasource.url=jdbc:mysql://[ip]:[port]/pkey_mgr?autoReconnect=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2b8
spring.datasource.username=
spring.datasource.password=

## spring jpa config
spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
```

### 建表

如果在上述配置中指定了**spring.jpa.properties.hibernate.hbm2ddl.auto=update**，则jpa会帮助用户自动建立数据表。

如果不希望自动建立数据表，请先关闭jpa建表开关：
```
spring.jpa.properties.hibernate.hbm2ddl.auto=validate
```
然后按下面方式手动建表，默认开启自动建表

1） 在数据源运行下述建表语句：

```
 -- Create syntax for TABLE 'cert_keys_info'
 drop table if exists cert_keys_info;
 CREATE TABLE `cert_keys_info` (
   `pk_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
   `user_id` varchar(255) NOT NULL,
   `key_alg` varchar(8) NOT NULL,
   `key_pem` longtext NOT NULL,
   `creat_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
   `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   PRIMARY KEY (`pk_id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
 -- Create syntax for TABLE 'cert_info'
 drop table if exists cert_info;
 CREATE TABLE `cert_info` (
   `pk_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
   `user_id` varchar(255) NOT NULL,
   `subject_pub_key` longtext NOT NULL,
   `cert_content` longtext NOT NULL,
   `issuer_key_id` bigint(20) NOT NULL,
   `subject_key_id` bigint(20) NOT NULL,
   `parent_cert_id` bigint(20),
   `serial_number` varchar(255) NOT NULL,
   `issuer_org` varchar(255) NOT NULL,
   `issuer_cn` varchar(255) NOT NULL,
   `subject_org` varchar(255) NOT NULL,
   `subject_cn` varchar(255) NOT NULL,
   `is_ca_cert` int(4) NOT NULL,
   `creat_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
   `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   PRIMARY KEY (`pk_id`),
   UNIQUE KEY (`parent_cert_id`,`serial_number`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 
 -- Create syntax for TABLE 'cert_request_info'
 drop table if exists cert_request_info;
 CREATE TABLE `cert_request_info` (
   `pk_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
   `parent_cert_id` bigint(20),
   `subject_key_id` bigint(20) NOT NULL,
   `user_id` varchar(255) NOT NULL,
   `cert_request_content` longtext NOT NULL,
   `subject_org` varchar(255) NOT NULL,
   `subject_cn` varchar(255) NOT NULL,
   `creat_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
   `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
   PRIMARY KEY (`pk_id`),
   UNIQUE KEY (`parent_cert_id`,`subject_key_id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 接口说明

CertManagerService类是证书管理的统一入口，覆盖证书管理的全生命周期，包含如下功能：

*   createRootCert : 生成根证书
*   createRootCertByHexPriKey ：私钥Hex格式作为入参生成证书
*   createCertRequest：生成请求
*   createCertRequestByHexPriKey ：私钥Hex格式作为入参生成请求
*   createChildCert：生成子证书
*   resetCertificate：证书重置
*   queryCertList：证书列表查询
*   queryCertRequestList：请求列表查询
*   queryCertKeyList：证书私钥列表查询
*   queryCertInfoByCertId：根据id查询证书
*   queryCertRequestByCsrId：根据id证书请求
*   exportCertToFile：证书导出

参考[Java doc](https://governance-doc.readthedocs.io/zh_CN/latest/certmgrdoc/navigation.html)

### 示例说明

下面介绍使用的例子

#### 实例注入

在Java项目test中创建一个测试类，在测试类中注入依赖

建议通过Spring自动注入KeysManagerService服务，示例如下：

```
    @SpringBootTest
    @RunWith(SpringRunner.class)
    public class Example {
        @Autowired
        private CertManagerService certManagerService;
    }  
```


#### 生成根证书

生成根证书，提供了多种封装接口，可按需使用

示例代码如下

```
    @Test
    public void testCreateRootCert() throws Exception {
        X500NameInfo issuer = X500NameInfo.builder()
                .commonName("chain")
                .organizationName("fisco-bcos")
                .organizationalUnitName("chain")
                .build();
        //用户id，如Bob
        String userId = "Bob";
        //有效期
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        //默认配置生成：采用RSA密钥对，也可调用其他封装接口，自定义密钥类型，支持pem和Hex格式
        CertVO cert = certManagerService.createRootCert(userId,issuer,beginDate,endDate);
        System.out.println("证书id: " + cert.getPkId());
        System.out.println("证书签发者: " + cert.getUserId());
        System.out.println("父证书id: " + cert.getPCertId());
        System.out.println("签发私钥id: " + cert.getIssuerKeyId());
        System.out.println("证书内容: " + cert.getCertContent());
    }

```
执行上述代码，可在控制台看到生成的证书内容，证书和相关联的签名私钥会保存在db中

#### 证书列表查询

可以通过接口按一定条件查询已签发的证书列表

示例代码如下

```
    @Test
    public void testQueryCertList() throws Exception {
        List<CertVO> list = certManagerService.queryCertInfoList();
        list.forEach(certVO -> {
            System.out.println("证书id：" + certVO.getPkId() + "\t");
            System.out.println("证书id：" +certVO.getUserId() + "\t");
            System.out.println("父证书id: " + certVO.getPCertId() + "\t");
            System.out.println("签发私钥id: " + certVO.getIssuerKeyId() + "\t");
            System.out.println("---------");
        });
    }

```
执行上述代码，可在控制台看到所有签发的证书列表及相关信息


#### 子证书csr生成

从上述步骤查询结果中，选择证书作为根证书，请求子证书。

示例代码如下

```
    @Test
    public void testCreateCertRequest() throws Exception {
        X500NameInfo subject = X500NameInfo.builder()
                .commonName("agancy")
                .organizationName("fisco-bcos")
                .organizationalUnitName("agancy")
                .build();
        String userId = "Alice";
        //自动生成密钥对，入参为：当前用户userId，父证书issuerCertId（可根据上述步骤查询得到），申请机构信息subject
        CertRequestVO csr = certManagerService.createCertRequest(userId, 1, subject);
        System.out.println("证书申请id：" + csr.getPkId());
        System.out.println("父证书签发者：" + csr.getPCertUserId());
        System.out.println("证书申请者：" + csr.getUserId());
        System.out.println("父证书id：" + csr.getPCertId());
        System.out.println("证书申请内容" + csr.getCertRequestContent());
    }


```
执行上述代码，证书申请和相关联的签名私钥会保存在db中，同时在db中将证书申请和其对应的父证书关联，便于后续子证书签发。


#### 证书申请列表查询

可以通过接口按一定条件查询证书申请列表

示例代码如下

```
    @Test
    public void testQueryCertRequestList() throws Exception {
        List<CertRequestVO> list = certManagerService.queryCertRequestList();
        list.forEach(csr -> {
            System.out.println("证书申请id: " + csr.getPkId());
            System.out.println("父证书签发者: " + csr.getPCertUserId());
            System.out.println("证书申请者: " + csr.getUserId());
            System.out.println("父证书id: " + csr.getPCertId());
            System.out.println("---------");
        });
    }
```
执行上述代码，可在控制台看到所有证书申请的列表及相关信息

#### 子证书签发

从上述步骤查询结果中，可选择证书申请签发子证书。

示例代码如下

```
    @Test
    public void testCreateChildCert() throws Exception {
        String userId = "Bob";
        //参数：签发者userId，证书申请id
        CertVO child = certManagerService.createChildCert(userId, 1);
        System.out.println("证书id: " + child.getPkId());
        System.out.println("证书签发者: " + child.getUserId());
        System.out.println("父证书id: " + child.getPCertId());
        System.out.println("签发私钥id: " + child.getIssuerKeyId());
        System.out.println("证书内容: " + child.getCertContent());
    } 
```

执行上述代码，可在控制台看到签发的子证书详情，可从第二步开始，继续签发下一级证书

#### 证书重置

证书重置支持对证书有效期和用途配置进行重置，方法为exportCertToFile。
可通过第二步的查询证书列表，选择证书重置，如选择证书id为1的证书进行重置，示例代码如下：

```
    @Test
    public void testResetCertificate() throws Exception {
        String userId = "John";
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        CertVO cert = certManagerService.resetCertificate(userId,1,new KeyUsage(KeyUsage.dataEncipherment),beginDate,endDate);
        System.out.println("证书id: " + cert.getPkId());
        System.out.println("证书签发者: " + cert.getUserId());
        System.out.println("父证书id: " + cert.getPCertId());
        System.out.println("签发私钥id: " + cert.getIssuerKeyId());
        System.out.println("证书内容: " + cert.getCertContent());
    }
```

执行上述代码可以在控制台看到重置后的证书详情

#### 证书导出

可通过第二步的查询证书列表，选择证书导出，如选择证书id为1的证书进行导出，示例代码如下：

```
    @Test
    public void testExportCertToFile() throws Exception {
        String filePath = "src/ca.crt";
        certManagerService.exportCertToFile(1L,filePath);
        System.out.println("导出完成，保存路径为：" + filePath);
    }
```

#### 更多示例
参考[证书管理接口示例](./mgr_api.md)
<br/>
参考[Java doc](https://governance-doc.readthedocs.io/zh_CN/latest/certmgrdoc/navigation.html)
