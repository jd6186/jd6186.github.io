---
layout: post
title: "[JAVA] Properties 내 환경변수 중 민감정보 암호화 처리"
tags: [BackEnd JAVA]
---

## Intro

안녕하세요 **Noah**입니다

오늘은 Java Properties 내 민감정보를 암호화하는 방법에 대해서 알아보도록 하겠습니다.

보통 DB를 사용하게 되면 아래와 같이 DB를 구분하여 사용하고 있으실 겁니다.


```java
    // Master DB
    @Bean(name = "masterDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.master.hikari")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create()
                .build();
    }
    
    // Slave DB
    @Bean(name = "slaveDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.slave.hikari")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create()
                .build();
    }
```

이 때 prefix로 지정한 부분부터 properties의 내용을 비교하여 필요한 환경변수를 활용하여 DB Connection을 진행하게 됩니다. 

하지만 이렇게 되면 properties 내 암호화되지 않은 민감정보를 저장해야 하는 이슈가 존재합니다.

이런 민감 정보를 노출하지 않으면서도 위 로직은 그대로 유지할 수 있도록 도와주는 API가 바로 <strong style="color: #bb4177;">'Jasypt'</strong>입니다.

추가로 Jasypt 기본으로 적용 가능한 암호화 방식이 MD5 또는 SHA-1 암호화 방식밖에 없어 추가로 bcprov API를 활용해 SHA-256암호화 방식을 사용하고자 한다

<br/><br/><br/><br/>


## Jasypt를 활용해 Properties 내 데이터 암호화 처리 방법(with. SHA-256)

1. pom.xml 설정

```xml

<!-- jasypt > 보안 -->
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>
<!-- 암호화 알고리즘 -->
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.69</version>
</dependency>
```

<br/><br/>

1. Config 작성

```java
import org.bouncycastle.jce.provider.BouncyCastleProvider;
import org.jasypt.encryption.StringEncryptor;
import org.jasypt.encryption.pbe.PooledPBEStringEncryptor;
import org.jasypt.encryption.pbe.config.SimpleStringPBEConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JasyptConfig {
    private final static StringKEY= "공개키";
    private final static StringALGORITHM= "PBEWithSHA256And128BitAES-CBC-BC"; // SHA-256 알고리즘 선택한 모습
    private final static StringCNT= "1000"; // 최대 문자열
    private final static StringPOOl_SIZE= "2"; // 2가 권장
    private final static StringBASE64= "base64";

    @Bean(name="jasyptStringEncryptor")
    public StringEncryptor stringEncryptor(){

        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword(KEY); // 암호화할 때 사용하는 키
        config.setKeyObtentionIterations(CNT); // 반복할 해싱 회수
        config.setPoolSize(POOl_SIZE); // 암호화 인스턴스 pool
        config.setStringOutputType(BASE64); //인코딩 방식

        PooledPBEStringEncryptor encryptor = new PooledPBEStringEncryptor();
        encryptor.setProvider(new BouncyCastleProvider()); // bcprov API 적용
        encryptor.setConfig(config); // 위 세팅 내용 설정
        encryptor.setAlgorithm(ALGORITHM); // 알고리즘 설정
        return encryptor;
    }
}
```

<br/><br/>

1. 테스트를 작성하여 원하는 데이터들을 1차적으로 암호화 작업 진행

```java

@SpringBootTest
class ServerApplicationTests {
   @Autowired
   StringEncryptor jasyptStringEncryptor;

   @Test
   void contextLoads(){
      String encryptVal = jasyptStringEncryptor.encrypt("암호화하고싶은문자열을 넣어주세요오");
      System.out.println("encrypt된 문자열 >>>>>>>>>> " + encryptVal);
      System.out.println("decrypt된 문자열 >>>>>>>>>> " + jasyptStringEncryptor.decrypt(encryptVal));
   }
}
```

<br/><br/>

1. Properties 내 암호화한 데이터 저장
    - ENC(”Encrypt된 문자열 넣기”) 를 활용하여 복호화해 사용할 데이터에만 앞에 ENC를 붙여주기

```
######### DB #########
spring.datasource.master.hikari.jdbc-url=ENC(jYKFM2Zv2dKN6AsfVQZyquAl6Nq/pnZPXSpCeW9WIQ8rvsHt5ZCVR163VxkXJFWhR+EwABKY1w9UIfhRo37/YFo4e)
spring.datasource.master.hikari.username=ENC(IMNfmkOf4/7D6613/tbcaCuDCnXiEE/1tSRt515r07s=)
spring.datasource.master.hikari.password=ENC(sSnjU0LViKPMjfUNG0Oo+4El5UgiGMQYCHJzrp/h2fw=)
spring.datasource.master.hikari.driverClassName=net.sf.log4jdbc.sql.jdbcapi.DriverSpy
spring.datasource.master.hikari.validationQuery=select 1
spring.datasource.master.hikari.autoReconnect=true
spring.datasource.master.hikari.read-only=true

spring.datasource.slave.hikari.jdbc-url=ENC(DJxuerxi9OQGMUBjnKgWOnJFo7DwAlOp/)
spring.datasource.slave.hikari.username=ENC(IMNfmkOf4/7D6613/tbcaCuDCnXiEE/1tSRt515r07s=)
spring.datasource.slave.hikari.password=ENC(sSnjU0LViKPMjfUNG0Oo+4El5UgiGMQYCHJzrp/h2fw=)
spring.datasource.slave.hikari.driverClassName=net.sf.log4jdbc.sql.jdbcapi.DriverSpy
spring.datasource.slave.hikari.validationQuery=select 1
spring.datasource.slave.hikari.autoReconnect=true
spring.datasource.slave.hikari.read-only=true
```

<br/><br/><br/><br/>

## 글을 마치며 
독자분들께 마지막으로 전하고 싶은 말은 SHA-1, MD5의 취약점이 발견되면서 더 이상 사용되고 있지 않다는 점을 인지하시고 

꼭 bcprov API가 아니더라도 다른 API를 활용해서라도 SHA-256 이상을 사용하실 것을 권장드립니다.

이상으로 글을 마치겠습니다. 감사합니다.