---
title: Spring Boot JPA (Hibernate) 入門 
cover: /img/home-bg.jpg
date: 2019-01-24 11:58:20
categories: ['spring-boot']
tags: ['spring-boot', 'spring-data', 'jpa']
---
### Spring Data
Spring Data 是 Spring 的資料（data , model）處理套件的總稱, 從 Spring data 著手, 比較容易知道 Spring 在資料面的生態系, 底層可搭配這些, 守備範圍很廣, 細節就看 [文件](https://spring.io/projects/spring-data).
* Spring Data JDBC
* Spring Data JPA (底層可搭配 JBoss, Hibernate, OpenJPA, EclipseLink)
* Spring Data LDAP
* Spring Data for Apache Cassandra
* Spring Data DynamoDB
* Spring Data Elasticsearch 

### Transaction
> Transactions are atomic units of work that can be committed or rolled back

簡單解釋, `transaction` 應該是一個不可分割的工作單元, 可以像這樣, 同時包含讀, 寫的一個 **工作單元**.
```sql
START TRANSACTION;
SELECT @A:=SUM(salary) FROM table1 WHERE type=1;
UPDATE table2 SET summary=@A WHERE type=1;
COMMIT;
```

### ACID
ACID 是在設計 Transaction 的重要準則, 但有時候很難兼顧... 
* Atomicity: 原子特性, translaction 應該盡量維持不能分割的狀態, 以上面例子來說 `SELECT` 跟 `UPDATE` 是同一個業務, 不應該拆成 2 個 transaction 去處理
* Consistency: 資料一致性, translaction 在 commit 或 rolled back 都應該保持資料的正確性, 以上面來說假設 `SELECT` 拿到 2 筆資料, 那 `UPDATE` 也應該是 **2** 筆資料, 何時會資料不一致？ 當你沒做任何 locking(樂觀鎖, 悲觀鎖) 又在一段時間內有多個 translaction 在同一張 table 操作, 資料就會髒掉(髒讀, 髒寫)
* Isolation: 隔離性, 延伸 Consistency 的敘述, 為了維持ㄧ致性, 隔離是必要手段, 有幾個策略 `READ_UNCOMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`, 詳細可看 [這篇](https://www.byteslounge.com/tutorials/spring-transaction-isolation-tutorial)
* Durability: 持久性, 這邊通常是靠資料庫的特性去處理, 比如 InnoDB 有 **doublewrite buffer** 來確保資料在斷電, 或一些意外情況可以確實寫入 Disk.

### DDL, DML, DCL, TCL
幾個常見的名詞...
* DDL: Data Definition Language, 建立資料庫 Schema 常用的指令, `CREATE`, `DROP`, `ALTER`, `TRUNCATE`, `COMMENT`, `RENAME`
* DML: Data Manipulation Language, 做資料操作的指令 CRUD 那些, `SELECT`, `INSERT`, `DELETE`, `UPDATE`
* DCL: Data Control Language, 資料庫的權限管理, `GRUNT`, `REVOKE`
* TCL: Transaction Control Language, 做 transaction 操作的, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `SET TRANSACTION`
  
### Hibernate DDL
我自己的開發習慣, 小專案通常都會用 `Spring Data JPA` 搭配 `Hibernate`, Hibernate 支援 DDL, 所以就算沒有 [flyway](https://flywaydb.org/) 或 [liquibase](https://www.liquibase.org/) 支援, 資料庫基本的 DDL 也能幫你處理得很好.

hibernate.ddl-auto 幾個參數
* create: 刪除已存在的 tables, 建立新的 tables
* create-drop: 建立新的 tables, SessionFactory 結束後刪除 tables
* update: 建立新的 tables, 若 entity 欄位有異動, 更新對應的 DDL
* validate: 只驗證 tables 與 columns 是否存在, entity 有該欄位, 但資料庫沒有對應的欄位, 噴 `PersistenceException`
```
javax.persistence.PersistenceException: [PersistenceUnit: default] Unable to build Hibernate SessionFactory;
nested exception is org.hibernate.tool.schema.spi.SchemaManagementException: Schema-validation: missing column 
[locking] in table [employee]
```

#### application.yml
```yml
database:
  host: localhost
  name: spring_transaction_demo
  username: root
  password:

spring:
  datasource:
    username: ${database.username}
    password: ${database.password}
    url: jdbc:mysql://${database.host}:3306/${database.name}?characterEncoding=utf-8&useUnicode=true&useSSL=false&rewriteBatchedStatements=TRUE
  jpa:
    database: MYSQL
    hibernate.ddl-auto: update
    show-sql: true
    properties.hibernate.show_sql: ${spring.jpa.show-sql}
    properties.hibernate.format_sql: ${spring.jpa.show-sql}

logging.level.org.hibernate.SQL: debug
```

#### pom.xml
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 懶人用的 lombok 套件 -->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.4</version>
  <scope>provided</scope>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
```


#### Entity
```java
import lombok.Data;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Version;
import java.time.Instant;
import java.time.LocalDate;

@Data
@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "emp_no", unique = true, nullable = false)
    private Long empNo;

    @Column(name = "birth_date", nullable = false)
    private LocalDate birthDate;

    @Column(name = "first_name", nullable = false, length = 20)
    private String firstName;

    @Column(name = "last_name", nullable = false, length = 20)
    private String lastName;

    @Enumerated(EnumType.STRING)
    @Column(name = "gender", nullable = false)
    private Gender gender;

    @Column(name = "hire_date", nullable = false)
    private LocalDate hireDate;

    @Version
    @Column(name = "version")
    private Instant version;
}
```

#### Repository
```java
import com.example.springtransactionisolation.entity.Employee;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
}
```

#### Test
```java
@Test
public void demo() {
    Employee e1 = new Employee();
    e1.setFirstName("Leo");
    e1.setLastName("Foster");
    e1.setGender(Gender.M);
    e1.setBirthDate(LocalDate.of(1990, 1, 1));
    e1.setHireDate(LocalDate.now());

    repository.save(e1);
}
```

### Ref
* InnoDB and the ACID Model, [https://dev.mysql.com/doc/refman/5.6/en/mysql-acid.html](https://dev.mysql.com/doc/refman/5.6/en/mysql-acid.html)
* MySQL Glossary - [https://dev.mysql.com/doc/refman/5.6/en/glossary.html#glos_acid](https://dev.mysql.com/doc/refman/5.6/en/glossary.html#glos_acid)
* GeeksForGeeks - [https://www.geeksforgeeks.org/sql-ddl-dml-dcl-tcl-commands/](https://www.geeksforgeeks.org/sql-ddl-dml-dcl-tcl-commands/)
* Spring Data docs - [https://spring.io/projects/spring-data](https://spring.io/projects/spring-data)
* Hibernate_User_Guide - [https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#preface](https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#preface)
* Blog byteslounge - [https://www.byteslounge.com/tutorials/spring-transaction-isolation-tutorial](https://www.byteslounge.com/tutorials/spring-transaction-isolation-tutorial)
  