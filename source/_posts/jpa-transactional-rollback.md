---
title: Spring boot JPA transactional rollback failed
cover: /img/home-bg.jpg
date: 2019-02-19 12:21:17
categories: ['spring-boot']
tags: ['spring-boot', 'spring-data', 'jpa']
---
### 原因一, 沒有指定 spring.jpa.database-platform
如果用了 `hibernate.ddl-auto`, 會自動選用 **MyISAM** engine 來建立 table schema, 然而 MyISAM 是不支援 Transactional 的, 自然沒有 rollback 功能, 且 spring boot 也不會給你任何警告, **雷爆了!!!**

因此指定 `database-platform: org.hibernate.dialect.MySQL5InnoDBDialect` 很重要。

application.yml 修正為
```yaml
jpa:
    database: MYSQL
    hibernate.ddl-auto: update
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
```

![my-isam-features](/img/jpa-transactional-rollback/my-isam-features.png)

### 原因二, 沒有指定 rollbackfor
> In its default configuration, the Spring Framework’s transaction infrastructure code only marks a transaction for rollback in the case of runtime, unchecked exceptions; that is, when the thrown exception is an instance or subclass of **RuntimeException**. ( Errors will also - by default - result in a rollback). Checked exceptions that are thrown from a transactional method do not result in rollback in the default configuration.

`@Transactional` 預設只對 RuntimeException 有反應, 還有繼承 RuntimeException 的層級, 若沒有特別指定 `rollbackfor = Exception.class` 屬性, 則不會有動作。

```java
package com.example.springtransactionisolation.service;

import com.example.springtransactionisolation.entity.Employee;
import com.example.springtransactionisolation.repository.EmployeeRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
public class EmployeeService {

    private EmployeeRepository repository;

    public EmployeeService(EmployeeRepository repository) {
        this.repository = repository;
    }

    @Transactional(rollbackFor = Exception.class)
    public void batchCreated(List<Employee> employees) throws Exception {

        repository.saveAll(employees);
        employees.clear();
        
        // rollback failed
        throw new Exception("Oops");
    }
}
```

### 手動 rollback
手動 rollback 可以寫成這樣, 參考官方 [文件](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html#transaction-declarative-rolling-back)

```java
@Transactional
public void batchCreated(List<Employee> employees) {

    try {
        repository.saveAll(employees);

        throw new Exception("oops");

    } catch (Exception updateEmployeeExp) {
        System.out.println("Do rollback transaction status");
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();

        // notify someone
            
    } finally {
        employees.clear();
    }
}
```

### Data too long 測試
另外, 在 insert 的時候, 如果發生 `DataException` 也是繼承於 RuntimeException, 所以沒有特別指定 `rollbackfor` 也是會整批 rollback。

Sample data 測試
```json
[
    {
        "birthDate": "1989-02-22",
        "firstName": "Tom",
        "lastName": "Lin",
        "gender": "M",
        "hireDate": "2019-02-10"
    },
    {
        "birthDate": "1985-05-25",
        "firstName": "Judy",
        "lastName": "Chen",
        "gender": "F",
        "hireDate": "2019-02-10"
    },
    {
        "birthDate": "1987-12-25",
        "firstName": "my name is so loooooooooooooooooooooooooooooooooooooooooooooon",
        "lastName": "Wu",
        "gender": "M",
        "hireDate": "2019-02-10"
    }
]
```

```bash
2019-02-19 13:24:51.796  WARN 61330 --- [nio-8080-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : SQL Error: 1406, SQLState: 22001
2019-02-19 13:24:51.796 ERROR 61330 --- [nio-8080-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper   : Data truncation: Data too long for column 'first_name' at row 1
2019-02-19 13:24:51.807 ERROR 61330 --- [nio-8080-exec-1] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; nested exception is org.hibernate.exception.DataException: could not execute statement] with root cause

com.mysql.cj.jdbc.exceptions.MysqlDataTruncation: Data truncation: Data too long for column 'first_name' at row 1
```

### Ref
* [spring boot docs - transaction-declarative-rolling-back](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html#transaction-declarative-rolling-back)
* [mysql docs - MyISAM Storage Engine Features](https://dev.mysql.com/doc/refman/5.6/en/myisam-storage-engine.html)
* [stackoverflow - Create Mysql InnoDB tables instead of MyISAM](https://stackoverflow.com/questions/1459265/hibernate-create-mysql-innodb-tables-instead-of-myisam)