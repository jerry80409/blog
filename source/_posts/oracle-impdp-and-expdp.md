---
title: Oracle 11g 資料匯出與匯入
cover: /img/home-bg.jpg
date: 2019-08-16 11:53:23
categories: ['dba']
tags: ['oracle']
---
### Oracle 11g sample database
因為我是用 Docker 建置 Oracle 11g 的環境, 而 Oracle 11g 已經算是舊版, 官方的文件實在很難找到一個只下載 sample database 的地方...

** 參考這個資源 [Oracle Sample Database](https://www.oracletutorial.com/getting-started/oracle-sample-database/) **, 

裡面的 Sample Database 其實是 Oracle 12 之後才有支援的語法, 所以我調整了一下 Schema, 後續的資料匯入再按照該資源的 sql 操作就好了。

```sql
-- regions
CREATE TABLE regions
  (
    region_id NUMBER,
    region_name VARCHAR2( 50 ) NOT NULL
  );
ALTER TABLE regions ADD (CONSTRAINT pk_region PRIMARY KEY (region_id));

-- countries table
CREATE TABLE countries
  (
    country_id   CHAR( 2 ) PRIMARY KEY  ,
    country_name VARCHAR2( 40 ) NOT NULL,
    region_id    NUMBER                 , -- fk
    CONSTRAINT fk_countries_regions FOREIGN KEY( region_id )
      REFERENCES regions( region_id ) 
      ON DELETE CASCADE
  );

-- location
CREATE TABLE locations
  (
    location_id NUMBER,
    address     VARCHAR2( 255 ) NOT NULL,
    postal_code VARCHAR2( 20 )          ,
    city        VARCHAR2( 50 )          ,
    state       VARCHAR2( 50 )          ,
    country_id  CHAR( 2 )               , -- fk
    CONSTRAINT fk_locations_countries 
      FOREIGN KEY( country_id )
      REFERENCES countries( country_id ) 
      ON DELETE CASCADE
  );							
ALTER TABLE locations ADD (CONSTRAINT pk_locations PRIMARY KEY (location_id));

-- warehouses
CREATE TABLE warehouses
  (
    warehouse_id NUMBER, 
    warehouse_name VARCHAR( 255 ) ,
    location_id    NUMBER( 12, 0 ), -- fk
    CONSTRAINT fk_warehouses_locations 
      FOREIGN KEY( location_id )
      REFERENCES locations( location_id ) 
      ON DELETE CASCADE
  );
ALTER TABLE warehouses ADD (CONSTRAINT pk_warehouses PRIMARY KEY (warehouse_id));

-- employees
CREATE TABLE employees
  (
    employee_id NUMBER,
    first_name VARCHAR( 255 ) NOT NULL,
    last_name  VARCHAR( 255 ) NOT NULL,
    email      VARCHAR( 255 ) NOT NULL,
    phone      VARCHAR( 50 ) NOT NULL ,
    hire_date  DATE NOT NULL          ,
    manager_id NUMBER( 12, 0 )        , -- fk
    job_title  VARCHAR( 255 ) NOT NULL
  );	
ALTER TABLE employees ADD (CONSTRAINT pk_employees PRIMARY KEY (employee_id));
ALTER TABLE employees ADD (CONSTRAINT fk_employees_manager FOREIGN KEY( manager_id ) REFERENCES employees( employee_id ) ON DELETE CASCADE);
-- CREATE SEQUENCE employees_seq START WITH 1;

-- product category
CREATE TABLE product_categories
  (
    category_id NUMBER,
    category_name VARCHAR2( 255 ) NOT NULL
  );
ALTER TABLE product_categories ADD (CONSTRAINT pk_product_categories PRIMARY KEY (category_id));
-- CREATE SEQUENCE product_categories_seq START WITH 1;

-- products table
CREATE TABLE products
  (
    product_id NUMBER,
    product_name  VARCHAR2( 255 ) NOT NULL,
    description   VARCHAR2( 2000 )        ,
    standard_cost NUMBER( 9, 2 )          ,
    list_price    NUMBER( 9, 2 )          ,
    category_id   NUMBER NOT NULL         ,
    CONSTRAINT fk_products_categories 
      FOREIGN KEY( category_id )
      REFERENCES product_categories( category_id ) 
      ON DELETE CASCADE
  );	
ALTER TABLE products ADD (CONSTRAINT pk_products PRIMARY KEY (product_id));
-- CREATE SEQUENCE products_seq START WITH 1;

-- customers
CREATE TABLE customers
  (
    customer_id NUMBER ,
    name         VARCHAR2( 255 ) NOT NULL,
    address      VARCHAR2( 255 )         ,
    website      VARCHAR2( 255 )         ,
    credit_limit NUMBER( 8, 2 )
  );
ALTER TABLE customers ADD (CONSTRAINT pk_customers PRIMARY KEY (customer_id));
-- CREATE SEQUENCE customers_seq START WITH 1;

-- contacts
CREATE TABLE contacts
  (
    contact_id NUMBER, 
    first_name  VARCHAR2( 255 ) NOT NULL,
    last_name   VARCHAR2( 255 ) NOT NULL,
    email       VARCHAR2( 255 ) NOT NULL,
    phone       VARCHAR2( 20 )          ,
    customer_id NUMBER                  ,
    CONSTRAINT fk_contacts_customers 
      FOREIGN KEY( customer_id )
      REFERENCES customers( customer_id ) 
      ON DELETE CASCADE
  );
ALTER TABLE contacts ADD (CONSTRAINT pk_contacts PRIMARY KEY (contact_id));
-- CREATE SEQUENCE contacts_seq START WITH 1;

-- orders table
CREATE TABLE orders
  (
    order_id NUMBER, 
    customer_id NUMBER( 6, 0 ) NOT NULL, -- fk
    status      VARCHAR( 20 ) NOT NULL ,
    salesman_id NUMBER( 6, 0 )         , -- fk
    order_date  DATE NOT NULL          ,
    CONSTRAINT fk_orders_customers 
      FOREIGN KEY( customer_id )
      REFERENCES customers( customer_id )
      ON DELETE CASCADE,
    CONSTRAINT fk_orders_employees 
      FOREIGN KEY( salesman_id )
      REFERENCES employees( employee_id ) 
      ON DELETE SET NULL
  );
ALTER TABLE orders ADD (CONSTRAINT pk_orders PRIMARY KEY (order_id));
-- CREATE SEQUENCE orders_seq START WITH 1;

-- order items
CREATE TABLE order_items
  (
    order_id   NUMBER( 12, 0 )                                , -- fk
    item_id    NUMBER( 12, 0 )                                ,
    product_id NUMBER( 12, 0 ) NOT NULL                       , -- fk
    quantity   NUMBER( 8, 2 ) NOT NULL                        ,
    unit_price NUMBER( 8, 2 ) NOT NULL                        ,
    CONSTRAINT pk_order_items 
      PRIMARY KEY( order_id, item_id ),
    CONSTRAINT fk_order_items_products 
      FOREIGN KEY( product_id )
      REFERENCES products( product_id ) 
      ON DELETE CASCADE,
    CONSTRAINT fk_order_items_orders 
      FOREIGN KEY( order_id )
      REFERENCES orders( order_id ) 
      ON DELETE CASCADE
  );

-- inventories
CREATE TABLE inventories
  (
    product_id   NUMBER( 12, 0 )        , -- fk
    warehouse_id NUMBER( 12, 0 )        , -- fk
    quantity     NUMBER( 8, 0 ) NOT NULL,
    CONSTRAINT pk_inventories 
      PRIMARY KEY( product_id, warehouse_id ),
    CONSTRAINT fk_inventories_products 
      FOREIGN KEY( product_id )
      REFERENCES products( product_id ) 
      ON DELETE CASCADE,
    CONSTRAINT fk_inventories_warehouses 
      FOREIGN KEY( warehouse_id )
      REFERENCES warehouses( warehouse_id ) 
      ON DELETE CASCADE
  );
```
<br>
### Expdp
我不太確定每個 Oracle Server 的環境是否都是這個路徑, 建議在參考這份紀錄的時候, 先確定一下自己的 Oracle 環境
```bash
# check $ORACLE_HOME
echo "$ORACLE_HOME"
/u01/app/oracle/product/11.2.0/EE
```
#### DATA_PUMP_DIR
`expdp` 指令是用來匯出資料的, 預設的匯出位置是 `DATA_PUMP_DIR`, 這個位置可以透過 select 查詢, 這份紀錄的 DATA_PUMP_DIR 是 **/u01/app/oracle/admin/EE/dpdump/**, 表示資料的匯入匯出預設 dmp 檔案會輸出到該路徑。

```sql
SQL> select * from ALL_DIRECTORIES;

OWNER			       DIRECTORY_NAME
------------------------------ ------------------------------
DIRECTORY_PATH
--------------------------------------------------------------------------------
SYS			       XMLDIR
/ade/b/2125410156/oracle/rdbms/xml

SYS			       IMPDP
/docker-entrypoint-initdb.d

SYS			       DATA_PUMP_DIR
/u01/app/oracle/admin/EE/dpdump/


OWNER			       DIRECTORY_NAME
------------------------------ ------------------------------
DIRECTORY_PATH
--------------------------------------------------------------------------------
SYS			       ORACLE_OCM_CONFIG_DIR
/u01/app/oracle/product/11.2.0/EE/ccr/state
```

#### expdp 用法
```bash
# 顯示操作文件
expdp -help

# 基本的用法 expdp [username/pwd] DIRECTORY=dmpdir DUMPFILE=scott.dmp
Example: expdp scott/tiger DIRECTORY=dmpdir DUMPFILE=scott.dmp

# 使用範例, 是透過 key=value 的方式去操作參數
...
Format:  expdp KEYWORD=value or KEYWORD=(value1,value2,...,valueN)
Example: expdp scott/tiger DUMPFILE=scott.dmp DIRECTORY=dmpdir SCHEMAS=scott
               or TABLES=(T1:P1,T1:P2), if T1 is partitioned table
```
---

在操作 expdp 是需要 `EXP_FULL_DATABASE` 的角色(Role), 可以用 DBA 查詢 USER 相關權限...
```sql
-- 查看 user 或 role 系統權限
SQL> SELECT * FROM DBA_SYS_PRIVS
    WHERE GRANTEE = 'OT';

-- 查看 user 系統 objects(table, index, seq, etc.) 使用權限
SQL> SELECT * FROM DBA_TAB_PRIVS
    WHERE GRANTEE = 'EXP_FULL_DATABASE';

-- 查看 user 擁有的 role
SQL> SELECT * FROM DBA_ROLE_PRIVS
    WHERE GRANTEE = 'OT';

-- 授權 EXP_FULL_DATABASE role
SQL> GRANT EXP_FULL_DATABASE TO OT;

-- 如果嫌麻煩就直接給 DBA 的角色吧（測試環境）
SQL> GRANT DBA TO OT;
```

```bash
# 匯出 OT schema 的所有資料
expdp OT/PASSWORD \
    full=y \
    schemas=OT \
    dumpfile=expdp.dmp

# 會在 /u01/app/oracle/admin/EE/dpdump/ 看到匯出的 dmp file
ls /u01/app/oracle/admin/EE/dpdump/
```

### impdp 用法
一樣需要 `DATAPUMP_IMP_FULL_DATABASE` 的角色, 
```sql
-- 授權 DATAPUMP_IMP_FULL_DATABASE role
SQL> GRANT DATAPUMP_IMP_FULL_DATABASE TO OT;
```
```bash
# 匯入
impdp OT/PASSWORD \
    full=y \
    schemas=WMS \
    dumpfile=expdp.dmp
```
--- 
若有需要 TABLESPACE 
```sql
SQL> CREATE BIGFILE TABLESPACE "OT_DATA" DATAFILE '/u01/app/oracle/oradata/EE/ot_data01.dbf' SIZE 1G AUTOEXTEND ON NEXT 1G MAXSIZE UNLIMITED LOGGING EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

### Reference
* [Oracle Sample Database](https://www.oracletutorial.com/getting-started/oracle-sample-database/)
* [Creating an Oracle Database](http://sergeivovk.com/walkthrough-creating-an-oracle-database-docker-image.html)
* [Oracle Docs](https://docs.oracle.com/cd/E11882_01/server.112/e22490/dp_import.htm)
