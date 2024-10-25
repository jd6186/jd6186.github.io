---
layout: post
title: "백엔드 개발에서 고려할 데이터베이스 종류별 특징과 활용 예시 - MySQL, PostgreSQL, MongoDB, Hadoop, Redis, Cassandra, Redshift"
tags: [BackEnd, Database, MySQL, PostgreSQL, MongoDB, Hadoop, Redis, Cassandra, Redshift]
---

# Intro
안녕하세요 Noah입니다.<br/>
오늘은 백엔드 개발에 필수적인 다양한 데이터 저장 매체의 특징과 활용 예시를 소개해 드리려 합니다. 데이터베이스마다 특정한 기능과 장점이 있으며, 그 특성에 따라 최적의 성능을 발휘할 수 있는 환경이 다릅니다. 

이 글에서는 관계형 데이터베이스(RDBMS)와 객체 관계형 데이터베이스(ORDBMS), 그리고 NoSQL의 다양한 유형을 포괄적으로 다룹니다.

특히, **MySQL**은 RDBMS, **PostgreSQL**은 ORDBMS, **MongoDB**는 문서 지향 NoSQL로 대표되는 데이터베이스입니다. 더불어 **Hadoop, Redis, Cassandra, Redshift**와 같은 분산 파일 시스템과 메모리 기반, 열 기반 데이터베이스도 살펴봅니다. 

각 데이터베이스의 주요 기능과 Java 예시 코드까지 포함해 백엔드 개발에서 데이터베이스 선택 시 참고할 수 있도록 구성했습니다.

<br/><br/><br/><br/>

# 목차
1. [MySQL (RDBMS - 관계형 데이터베이스)](#mysql-rdbms---관계형-데이터베이스)
2. [PostgreSQL (ORDBMS - 객체 관계형 데이터베이스)](#postgresql-ordbms---객체-관계형-데이터베이스)
3. [MongoDB (NoSQL - 문서 지향 데이터베이스)](#mongodb-nosql---문서-지향-데이터베이스)
4. [Hadoop (HDFS - 분산 파일 시스템)](#hadoop-hdfs---분산-파일-시스템)
5. [Column-oriented DB (열 기반 데이터베이스)](#column-oriented-db-열-기반-데이터베이스)
6. [Cassandra (Wide Column Store - 넓은 열 저장소)](#cassandra-wide-column-store---넓은-열-저장소)
7. [Redis (In-memory Key-Value Store - 메모리 기반 키-값 저장소)](#redis에-연결하고-데이터-추가하기)
8. [총 정리](#총-정리)
9. [Outro](#outro)

<br/><br/><br/><br/>


## MySQL (RDBMS - 관계형 데이터베이스)
### 본질
MySQL은 전통적인 행 기반 저장 방식의 관계형 데이터베이스(RDBMS)로, **구조화된 데이터 관리**와 **높은 데이터 무결성**을 지향합니다. <br/>
그 중에서도 InnoDB 스토리지 엔진을 사용하는 MySQL은 **ACID 트랜잭션**을 준수하여 데이터의 신뢰성과 일관성을 유지합니다. <br/>
기본적으로 고정된 스키마를 요구하며, 데이터를 테이블에 행(row)으로 저장하여 구조화된 데이터에 대해 신속한 CRUD 연산을 제공합니다.

### 사용 예시
- **전자상거래 시스템**: 주문 관리 및 트랜잭션 처리에 최적화되어 있어 높은 무결성과 트랜잭션 지원이 필요합니다.
- **금융 애플리케이션**: 보안과 무결성이 중요한 금융 데이터 처리에 활용됩니다.


### Java 예시 코드 (MySQL 연결 및 간단한 CRUD)
#### MySQL 데이터베이스에 연결
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class MySQLConnection {
    public static Connection getConnection() throws SQLException {
        String url = "jdbc:mysql://localhost:3306/mydatabase";
        String user = "username";
        String password = "password";

        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println("Connected to MySQL database!");
        return connection;
    }
}
```

#### CRUD 예제: 테이블에 데이터 삽입
```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class MySQLCRUD {
    public static void insertData(String name, int age) throws SQLException {
        String query = "INSERT INTO users (name, age) VALUES (?, ?)";

        try (Connection conn = MySQLConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(query)) {
            pstmt.setString(1, name);
            pstmt.setInt(2, age);
            pstmt.executeUpdate();
            System.out.println("Data inserted successfully!");
        }
    }

    public static void main(String[] args) {
        try {
            insertData("John Doe", 30);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
<br/><br/><br/><br/>

## PostgreSQL (ORDBMS - 객체 관계형 데이터베이스)
### 본질
PostgreSQL은 RDBMS의 기능을 넘어, 객체 지향적 특성과 고급 쿼리 기능을 결합하여 복잡한 데이터 처리가 필요한 환경에서 효과적입니다. <br/>
**JSONB와 같은 비정형 데이터 타입**을 지원하고, **GIS(지리 정보 시스템) 데이터 처리**에도 유리해 다양한 데이터 형식과 복잡한 비즈니스 로직을 지원합니다. <br/>
다중 버전 동시성 제어(MVCC)를 통해 동시 작업을 효과적으로 처리하고 ACID 트랜잭션을 보장합니다.

### 사용 예시
- **데이터 분석 플랫폼**: 복잡한 쿼리와 데이터 분석 기능이 필요한 시스템에서 사용됩니다.
- **공간 정보 시스템(GIS)**: 공간 좌표 데이터 및 복잡한 위치 기반 쿼리 처리가 필요한 시스템에서 활용됩니다.

### Java 예시 코드 (PostgreSQL 연결 및 JSON 데이터 처리)
#### PostgreSQL 데이터베이스에 연결
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class PostgreSQLConnection {
    public static Connection getConnection() throws SQLException {
        String url = "jdbc:postgresql://localhost:5432/mydatabase";
        String user = "username";
        String password = "password";

        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println("Connected to PostgreSQL database!");
        return connection;
    }
}
```

#### JSON 데이터 처리 예제
PostgreSQL의 JSONB 타입을 활용하여 JSON 데이터를 테이블에 저장하고 조회할 수 있습니다.

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class PostgreSQLCRUD {
    public static void insertJsonData(String jsonData) throws SQLException {
        String query = "INSERT INTO json_table (data) VALUES (?::jsonb)";

        try (Connection conn = PostgreSQLConnection.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(query)) {
            pstmt.setString(1, jsonData);
            pstmt.executeUpdate();
            System.out.println("JSON data inserted successfully!");
        }
    }

    public static void main(String[] args) {
        String jsonData = "{\"name\": \"John\", \"age\": 30, \"city\": \"New York\"}";
        try {
            insertJsonData(jsonData);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
<br/><br/><br/><br/>

## MongoDB (NoSQL - 문서 지향 데이터베이스)
### 본질
MongoDB는 유연한 스키마를 지원하는 문서 지향 NoSQL 데이터베이스로, JSON과 유사한 BSON 형식의 문서로 데이터를 저장합니다.<br/>
MongoDB는 **고정된 스키마가 필요하지 않으며, 구조가 자주 바뀌거나 비정형 데이터가 많은 경우에 유리**합니다. <br/>
샤딩을 통한 수평 확장성과 클라이언트 측 필드 수준 암호화 기능을 제공해 데이터 보안과 확장성을 모두 갖춘 구조입니다.<br/>
엔진 자체가 쓰기 성능과 단순 조회에서 빠른 속도를 보장하며, **대용량 데이터 처리**와 **실시간 분석**에 적합합니다.<br/>
다만, **트랜잭션 처리**가 필요한 경우나 조회조건이 복잡한 경우에는 RDBMS나 ORDBMS가 더 적합할 수 있습니다.

### 사용 예시
- **소셜 미디어 플랫폼**: 데이터 구조가 다양하고 변화가 잦은 환경에 적합합니다.
- **IoT 애플리케이션**: 대규모 실시간 데이터가 수집되는 상황에 유연한 스키마와 확장성으로 유리합니다.

### Java 예시 코드 (MongoDB 연결 및 데이터 삽입)
#### MongoDB 데이터베이스에 연결
```java
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoDatabase;

public class MongoDBConnection {
    public static MongoDatabase getDatabase() {
        MongoClient mongoClient = MongoClients.create("mongodb://localhost:27017");
        MongoDatabase database = mongoClient.getDatabase("mydatabase");
        System.out.println("Connected to MongoDB database!");
        return database;
    }
}
```

#### MongoDB에 JSON 문서 삽입 예제
```java
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

public class MongoDBCRUD {
    public static void insertDocument() {
        MongoDatabase database = MongoDBConnection.getDatabase();
        MongoCollection<Document> collection = database.getCollection("mycollection");

        Document document = new Document("name", "John Doe")
                .append("age", 30)
                .append("city", "New York");
        collection.insertOne(document);
        System.out.println("Document inserted successfully!");
    }

    public static void main(String[] args) {
        insertDocument();
    }
}
```
<br/><br/><br/><br/>


## Hadoop (HDFS - 분산 파일 시스템)
### 본질
Hadoop의 HDFS는 대규모 데이터를 분산하여 저장하고 관리할 수 있는 분산 파일 시스템입니다.<br/>
데이터는 파일 단위로 블록에 나뉘어 여러 노드에 분산 저장되며, 수평 확장을 통해 대용량 데이터를 처리합니다.<br/> 
데이터 무결성보다는 대규모 배치 처리에 중점을 두며, 비정형 데이터의 수집 및 분석에 적합합니다.

### 사용 예시
- **로그 데이터 분석**: 대량의 로그 데이터를 장기적으로 저장하고 배치 처리하는 데 적합합니다.
- **데이터 웨어하우스**: 주기적인 배치 분석을 통해 저장된 대량의 데이터를 활용하는 BI 분석에 사용됩니다.

### Java 예시 코드 (Hadoop 파일 시스템 연결 및 데이터 업로드)
Java에서는 Hadoop을 사용하는 데 도움이 되는 다양한 API가 제공되며, Java 환경에서 파일을 업로드하고 처리하는 예제를 아래와 같이 작성할 수 있습니다.

#### Hadoop 설정 및 데이터 업로드 예제
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.io.IOException;
import java.net.URI;

public class HadoopExample {
    public static void uploadFileToHDFS(String localFilePath, String hdfsFilePath) throws IOException {
        Configuration configuration = new Configuration();
        configuration.set("fs.defaultFS", "hdfs://localhost:9000");
        FileSystem fs = FileSystem.get(URI.create("hdfs://localhost:9000"), configuration);

        Path srcPath = new Path(localFilePath);
        Path dstPath = new Path(hdfsFilePath);

        fs.copyFromLocalFile(srcPath, dstPath);
        System.out.println("File uploaded to HDFS at: " + hdfsFilePath);

        fs.close();
    }

    public static void main(String[] args) {
        try {
            uploadFileToHDFS("/path/to/local/file.txt", "/path/in/hdfs/file.txt");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
<br/><br/><br/><br/>


## Column-oriented DB (열 기반 데이터베이스)
### 본질
Column-oriented DB는 열(column) 기반 데이터베이스로, 각 열의 데이터를 묶어서 저장하여 특정 컬럼의 집계 작업에서 빠른 성능을 제공합니다.<br/>
BI 및 실시간 데이터 분석에 강점이 있으며, 행 기반 DB보다 쓰기 성능은 떨어질 수 있지만, 읽기 성능이 중요한 환경에서는 매우 유용합니다.

### 사용 예시
- **실시간 데이터 대시보드**: 특정 데이터 열에 대한 집계가 빈번하게 발생하는 상황에 적합합니다.
- **통계 분석 시스템**: 고속 조회가 필요한 통계 및 비즈니스 인텔리전스(BI) 애플리케이션에서 효과적입니다.

### Java 예시 코드 (Redshift 연결 및 간단한 조회)
예시로 든 Redshift는 Java에서 JDBC API를 통해 연결이 가능하며, 주로 분석 작업을 위한 SQL 쿼리를 활용하여 데이터를 추출합니다.

#### 데이터 조회 예제
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class RedshiftExample {
   public static void main(String[] args) {
      String url = "jdbc:redshift://your-redshift-cluster-url:5439/yourdatabase";
      String user = "your-username";
      String password = "your-password";

      try (Connection conn = DriverManager.getConnection(url, user, password);
           Statement stmt = conn.createStatement()) {

         System.out.println("Connected to Amazon Redshift!");

         // 쿼리 예제: 특정 지역의 판매 데이터 조회
         String query = "SELECT COUNT(*), AVG(sales) FROM sales_data WHERE region = 'North America'";
         ResultSet rs = stmt.executeQuery(query);

         // 결과 처리
         while (rs.next()) {
            System.out.println("Count: " + rs.getInt(1));
            System.out.println("Average Sales: " + rs.getDouble(2));
         }

      } catch (Exception e) {
         e.printStackTrace();
      }
   }
}
```
<br/><br/><br/><br/>


## Cassandra (Wide Column Store - 넓은 열 저장소)
### 본질
Cassandra는 **분산 아키텍처**를 기반으로 하는 넓은 열 저장소(Wide Column Store)로 설계된 NoSQL 데이터베이스입니다. <br/>
Cassandra는 **고가용성**과 **확장성**을 목표로 하며, **분산 환경에서의 대량 데이터 처리**에 적합합니다. <br/>
행(Row)은 테이블(Column Family)에 저장되지만, 행 내의 데이터를 열(Column) 단위로 분리하여 저장하는 구조를 가지고 있어, 특정 열이나 열 그룹에 대해 빠르게 접근할 수 있습니다.

### 사용 예시
- **IoT 데이터 수집 및 분석**: 높은 쓰기 성능과 확장성으로 인해 대량의 IoT 데이터를 저장하고 분석하는 데 적합합니다.
- **실시간 분석 플랫폼**: 빠른 쓰기 및 읽기 성능이 요구되는 실시간 분석과 이벤트 스트리밍에 유리합니다.

### Java 예시 코드 (Cassandra 연결 및 데이터 삽입)
#### Cassandra에 연결하고 데이터 추가하기
Cassandra는 DataStax Java 드라이버를 사용해 Java 애플리케이션과 통합할 수 있습니다. <br/>
아래 예제는 `user_data` 테이블에 사용자 데이터를 삽입하고 조회하는 방법을 보여줍니다.

```java
import com.datastax.oss.driver.api.core.CqlSession;
import com.datastax.oss.driver.api.core.cql.PreparedStatement;
import com.datastax.oss.driver.api.core.cql.ResultSet;
import com.datastax.oss.driver.api.core.cql.Row;
import java.net.InetSocketAddress;

public class CassandraExample {
    public static void main(String[] args) {
        try (CqlSession session = CqlSession.builder()
                .addContactPoint(new InetSocketAddress("127.0.0.1", 9042))
                .withKeyspace("mykeyspace")
                .build()) {

            // 데이터 삽입 예제
            String insertQuery = "INSERT INTO user_data (id, name, age) VALUES (?, ?, ?)";
            PreparedStatement preparedStatement = session.prepare(insertQuery);
            session.execute(preparedStatement.bind(1, "Alice", 25));
            System.out.println("Data inserted successfully!");

            // 데이터 조회 예제
            String selectQuery = "SELECT * FROM user_data WHERE id = ?";
            ResultSet resultSet = session.execute(preparedStatement.bind(1));
            for (Row row : resultSet) {
                System.out.println("ID: " + row.getInt("id"));
                System.out.println("Name: " + row.getString("name"));
                System.out.println("Age: " + row.getInt("age"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
<br/><br/><br/><br/>


## Redis (In-memory Key-Value Store - 메모리 기반 키-값 저장소)
### 본질
Redis는 **메모리 기반 키-값 데이터 저장소**로, **고속의 데이터 접근**을 제공하는 NoSQL 데이터베이스입니다. <br/>
모든 데이터를 메모리에 저장하고 필요 시 디스크에 백업하여 영속성을 보장할 수 있어, **빠른 읽기 및 쓰기 속도**가 필요한 환경에서 최적화된 성능을 발휘합니다. <br/>
Redis는 단순한 키-값 저장 방식 외에도 리스트, 해시, 셋, 정렬된 셋 등의 다양한 데이터 구조를 지원하여 유연한 데이터 관리를 돕습니다.

### 사용 예시
- **캐싱 시스템**: 자주 조회되는 데이터를 Redis에 저장하여 DB의 부하를 줄이고, 빠른 응답을 제공하는 캐시 역할을 합니다.
- **실시간 분석**: 게임 리더보드나 실시간 데이터 분석, 스트리밍 처리가 필요한 환경에서 활용됩니다.
- **세션 관리**: 빠른 읽기/쓰기가 필요한 웹 애플리케이션에서 사용자 세션을 관리하는 데 적합합니다.

### Java 예시 코드 (Redis 연결 및 데이터 삽입과 조회)
Java 애플리케이션에서 Redis와 연결하려면 `Jedis` 라이브러리를 사용할 수 있습니다. <br/>
아래 예제는 Redis에 간단한 문자열 데이터를 추가하고 조회하는 방법을 보여줍니다.

### Redis에 연결하고 데이터 추가하기
```java
import redis.clients.jedis.Jedis;

public class RedisExample {
    public static void main(String[] args) {
        // Redis 서버에 연결합니다.
        Jedis jedis = new Jedis("localhost", 6379);
        System.out.println("Connected to Redis!");

        // 데이터 추가 예제
        jedis.set("user:1000", "John Doe");
        System.out.println("Data set successfully!");

        // 데이터 조회 예제
        String user = jedis.get("user:1000");
        System.out.println("Retrieved data: " + user);

        // 연결 종료
        jedis.close();
    }
}
```
<br/><br/><br/><br/>


## 총 정리

| **특징** | **MySQL**                            | **PostgreSQL**                               | **MongoDB**                          | **Hadoop (HDFS)**                           | **Redshift**                    | **Cassandra**                            | **Redis**                           |

| -------- | -------------------------------------- | ----------------------------------------------- | -------------------------------------- | --------------------------------------------- | ---------------------------------- | ------------------------------------------ | -------------------------------------- |

| **주요 용도**      | 트랜잭션 처리, CRUD 중심의 애플리케이션 | 복잡한 쿼리 및 데이터 분석, GIS 지원          | 비정형 데이터 저장 및 대량 데이터 관리 | 대규모 데이터 저장 및 배치 처리            | 실시간 분석 및 대규모 데이터 조회 | 대규모 데이터 쓰기, IoT 데이터 수집      | 실시간 캐싱, 빠른 읽기/쓰기          |

| **데이터 저장 방식** | 행(row) 기반                         | 행(row) 기반 (JSONB 지원)                    | 문서 기반 (BSON)                      | 파일 단위로 블록 분산 저장                 | 열(column) 기반                   | 넓은 열 저장소(Column Family)           | 키-값(Key-Value) 기반                |

| **확장성**         | 주로 수직 확장, 일부 수평 확장 가능     | 수직 및 수평 확장 (리플리케이션 지원)         | 수평 확장 (샤딩)                      | 수평 확장 (노드 추가)                      | 수평 확장 (클러스터링)            | 수평 확장성 우수                         | 수직 및 수평 확장 가능               |

| **트랜잭션 관리**  | ACID 지원                             | ACID 지원 (MVCC 동시성 제어)                 | 제한적 (다중 문서 트랜잭션)           | ACID 미지원, 주로 배치 처리에 적합        | 제한적, 일관성보다 성능 우선       | 제한적 (단일 행 단위)                    | 비영구적 트랜잭션                   |

| **데이터 일관성 및 무결성** | 데이터 일관성 및 무결성 보장       | 데이터 일관성 강력히 보장                    | 일관성보다 가용성 우선                | 일관성보다 가용성 중시                     | 주로 분석 성능에 집중             | 일관성보다 가용성 및 확장성 중시         | 일관성보다 성능 우선                |

| **읽기 성능**      | 인덱스 활용한 빠른 읽기               | 고급 인덱스, 복잡한 쿼리 처리                | 빠른 읽기 성능, 대량 데이터 조회 가능 | 전체 데이터 스캔, 비효율적                | 열 단위 저장, 고속 읽기            | 빠른 읽기, 대규모 데이터 처리           | 매우 빠른 읽기                      |

| **쓰기 성능**      | 간단한 쓰기 작업 최적화               | 복잡한 쓰기 작업 안정적 지원                 | 높은 쓰기 성능, 유연한 구조            | 대용량 데이터 배치 시 빠름                | 데이터 쓰기 성능 낮음               | 빠른 쓰기, 대규모 데이터 처리           | 매우 빠른 쓰기                      |

| **데이터 분석 성능** | 기본적 집계는 가능, 대규모 분석 비효율적 | 복잡한 집계와 분석에 유리                  | 기본적 분석 및 집계 지원               | 대용량 데이터 분석(배치) 최적화          | 고속 쿼리 및 집계 분석 최적화      | 기본적 분석에 적합                      | 기본적 분석에 적합                  |

| **데이터 형식 지원** | 구조화된 데이터(SQL)                | 구조화된 데이터, JSONB                      | 비정형 데이터(BSON, JSON)             | 다양한 데이터 형식 (텍스트, 이미지 등)   | 구조화된 데이터(SQL)               | 구조화 및 반정형 데이터                | 문자열, 숫자, 해시, 리스트 등       |

| **주요 사용 사례** | 전자상거래, 웹 애플리케이션            | 데이터 분석 플랫폼, GIS 시스템               | 소셜 미디어, IoT 데이터 관리           | 로그 분석, 데이터 웨어하우스              | BI, 실시간 대시보드               | IoT, 실시간 데이터 스트리밍              | 세션 캐싱, 실시간 분석              |

| **주요 장점**      | 사용 편리, 빠른 CRUD 처리             | ACID 트랜잭션, 고급 쿼리 성능                | 스키마 유연성, 수평 확장성             | 대규모 데이터 분산 저장                  | 빠른 열 조회 성능, 집계 최적화      | 고가용성, 확장성 우수                    | 메모리 기반 고속 처리              |

| **주요 단점**      | 복잡한 쿼리와 대규모 분석 비효율      | 수평 확장성 제한적                           | 트랜잭션 관리 제한                    | 실시간 데이터 처리 어려움                | 실시간 트랜잭션 한계                | 복잡한 쿼리 처리 비효율                 | 데이터 영구 보관 어려움             |

| **대표적인 예시**   | MySQL, MariaDB                        | PostgreSQL, Amazon Aurora                   | MongoDB, Couchbase                   | Hadoop (HDFS), Google Bigtable           | Amazon Redshift, Google BigQuery | Apache Cassandra                         | Redis, Memcached                   |

| **성능 최적화**    | 인덱스 및 캐시 사용                   | 고급 인덱스, 동시성 제어                     | 샤딩, 메모리 캐시                     | MapReduce, Spark 사용                    | 열 단위 데이터 압축                | 복제 및 샤딩                             | 메모리 기반 처리, TTL 지원         |

| **데이터 저장 유연성** | 고정된 스키마                      | 고정 스키마, JSON 지원                      | 유연한 스키마, 비정형 데이터 지원       | 비정형 데이터 저장 가능                  | 스키마 기반 열 단위 저장            | 고정/유연 스키마 가능                    | 고정 스키마 없음, 다양한 데이터 구조 |

| **백업 및 복구**   | 기본 백업 및 복구 제공                 | 백업 도구, Point-in-Time 복구                | 자동 복제, 수평 복구 지원              | 데이터 복제, 노드 장애 복구              | 분산 백업 및 복구                   | 클러스터 단위 복구                       | 주기적 스냅샷 및 백업              |

| **확장성 방법**    | 리플리케이션, 샤딩                    | 논리적 복제, 샤딩, 클러스터링               | 샤딩 통한 수평 확장                   | 노드 추가 통한 수평 확장                | 클러스터링 지원                      | 샤딩 통한 수평 확장                       | 클러스터 확장 지원                  |

| **대규모 데이터 분석 지원** | 제한적, OLTP 중심                | 고급 쿼리와 데이터 분석                     | 데이터 처리 엔진 활용                 | MapReduce 등 사용                       | 집계 분석 최적화                    | 기본적 데이터 분석                        | 제한적                              |



## Outro
이 글에서는 각 데이터베이스의 본질적 기능, 용도, 그리고 Java 환경에서의 예시 코드를 포함하여 상세히 설명하였습니다.<br/>
각 데이터베이스의 선택은 데이터 특성과 요구 사항에 따라 달라질 수 있으며, 프로젝트 요구사항에 맞는 최적의 데이터베이스를 선택하는 것이 중요합니다.<br/> 
이를 통해 성능과 확장성, 데이터 무결성을 모두 고려한 안정적인 시스템을 구축할 수 있습니다. <br/>

다음 글에서는 특정 데이터베이스의 성능 최적화 방법에 대해 다뤄보겠습니다.

긴 글 읽어주셔서 감사합니다. 더 궁금한 점이나 추가 정보가 필요하시면 언제든지 댓글로 남겨주세요. 감사합니다! 😊
