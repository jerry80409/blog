---
title: 我要寫一篇我討厭 Oracle 的原因
cover: /img/home-bg.jpg
date: 2019-08-22 18:17:28
categories: ['dba']
tags: ['oracle']
---
### I hate the Oracle
Oracle 算是老牌的資料庫, 但實在是滿令人挫折的, 
如果是習慣在 terminal 下操作的使用者, 光 accepted 那個 licence, 
就讓我覺得自己是一個愚蠢的使用者, 官方的文件又難以快速查詢到想要的,
Oracle 版本又很多, 對應不同的 OS 環境又有各種奇奇怪怪的設定方式, 
光是確認 ORACLE_SID 就浪費了不少時間, 
config 文件也是用特規的方式去撰寫, 不是 `.ymal` 也不是 `.conf` 而是 `.ora`,
`to_date()` 也不走 ISO 規範, 硬要跟別人不一樣...
基於各種麻煩的問題, 我真的不喜歡 Oracle

---

### Env
* CentOS 7   (CentOS Linux release 7.6.1810 (Core))
* Oracle 19c (Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production)
---

### Difficult setting
**[安裝教學](https://oracle-base.com/articles/19c/oracle-db-19c-rpm-installation-on-oracle-linux-7)**, 這篇算是我找到比較適合我的教學, 我的安裝記錄如下 
```bash
# 建立資料夾
mkdir -p ~/Download
cd ~/Download

# 下載 rpm
curl -o oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm

# 安裝
sudo yum -y localinstall oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
```

### Uncomfortable licence 
如果想下載官方的 rpm 就要想辦法通過 licence...
1. acceped and click download
![get-accepted-licence-url-link](/img/i-hate-oracle/oracle-accepted-and-clicked-download.png)

2. get accepted licence url link
![get-accepted-licence-url-link](/img/i-hate-oracle/get-accepted-licence-url-link.png)

3. wget
```bash
wget https://download.oracle.com/otn/linux/oracle19c/190000/oracle-databas-ee-19c-1.0-1.x86_64.rpm?AuthParam=some-param
```

---

### Change the oracle user in CentOS
如果跟我一樣是手動安裝, 那可能會需要設定一下 oracle 的 `passwd`
```bash
# 設定 oracle 這個 user 的密碼
sudo passwd oracle

# 切換 oracle 才能調整 oracle 相關 config
su oracle

# 登入 sqlplus
sqlplus / as sysdba
```
---

### Sqlplus command not found
如果發生了 **sqlplus command not found** 是因為缺少環境變數, 確認 env
```bash
# 確認你的環境變數是否有 ORACLE_BASE, ORACLE_HOME, ORACLE_SID
env | grep ORACLE
```

如果沒有 `ORACLE_BASE`, `ORACLE_HOME`, `ORACLE_SID` 就手動設定一下吧, 
但我沒辦法保證每個人的環境都跟我的一樣, 在設定時請手動確認一下路徑是不是跟我一樣
```bash 
# 如果沒有就手動設定吧
echo "export ORACLE_BASE=/opt/oracle" >> ~/.bash_profile
echo "export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1" >> ~/.bash_profile

# 這個設定花很多力氣才找到
echo "export ORACLE_SID=ORCLCDB" >> ~/.bash_profile

source ~/.bash_profile
```
---

### ORA-01034: ORACLE NOT AVAILABLE
這個問題是因為 Oracle 環境變數沒有設定好, 因為我找不到 ORACLE_SID
```bash
# 我也知道可以用 echo, 但就是沒有啊
echo $ORACLE_SID 
```

然後 [教學](http://www.dba-oracle.com/t_find_oracle_sid.htm) 說可以進去資料庫 select, 事實上是 ORACLE_SID 沒設定好, 你也無法用 sqlplus 去做登入, 後來找到可以在 `tnsnames.ora` 找到 ORACLE_SID
```bash
# my path is /opt/oracle/product/19c/dbhome_1/network/admin/tnsnames.ora
cd $ORACLE_HOME/network/admin
cat tnsnames.ora
```

### The .ora config file 
為了找 ORACLE_SID 翻遍所有文件, 才在某個文件的小小角落翻到
> Change the value of ORACLE_SID to your new value in your .profile, .cshrc, .login, oratab, and tnsnames.ora files.

說明 ORACLE_SID 寫在 **.profile**, **.cshrc**, **.login**, **oratab**, **tnsnames.ora**, 只想罵髒話

```bash
# ORCLCDB 就是預設的 SID
ORCLCDB =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = ORCLCDB)
    )
  )

LISTENER_ORCLCDB =
  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
```
---

### Unknow Charset 
亂碼問題, 必須先找到 CHARSET
```sql
SQL> select userenv('language') from dual;

USERENV('LANGUAGE')
----------------------------------------------------
TRADITIONAL CHINESE_TAIWAN.AL32UTF8
```

再把這個愚蠢的 CHARSET 設定到 ENV
```bash
echo "export NLS_LANG='TRADITIONAL CHINESE_TAIWAN.AL32UTF8'" >> ~/.bash_profile
source ~/.bash_profile
```

---

### ORA-12541: TNS:no listener
先檢查 CentOS 的防火牆, 記得把 1521 打開
```bash
# add port 1521 to firewall
sudo firewall-cmd --add-port=1521/tcp

# list firewall available ports
sudo firewall-cmd --list-all

# stdout
target: default
  icmp-block-inversion: no
  interfaces: enp3s0
  sources:
  services: ssh dhcpv6-client
  ports: 22/tcp 1521/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

調整 listener.ora 監聽全部打開, 修改 HOST 為 `0.0.0.0`, 讓遠端的 Client 也可以連線
```bash
# 修改 HOST 為 0.0.0.0 
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

透過 lsnrctl 重新啟動 listener
```bash
# lsnrctl 工具
lsnrctl

LSNRCTL> reload

LSNRCTL> stop

LSNRCTL> start

LSNRCTL> status

```

#### Insert Records : ORA-01843
`ORA-01843` 的錯誤代碼說明是 **不是有效的月份**, 原因是 `to_date()` 造成的
```sql
SQL> Insert into EMPLOYEES (EMPLOYEE_ID,FIRST_NAME,LAST_NAME,EMAIL,PHONE,HIRE_DATE,MANAGER_ID,JOB_TITLE) values (107,'Summer','Payne','summer.payne@example.com','515.123.8181',to_date('07-JUN-16','DD-MON-RR'),106,'Public Accountant');

ORA-01843: 不是有效的月份
```

資料庫就你最特別, 完全不照 ISO 的 Datetime 格式, 硬要自己訂一套, 還要吃 ENV 的 `NLS_DATE_LANGUAGE`
```sql
SQL> Insert into EMPLOYEES (EMPLOYEE_ID,FIRST_NAME,LAST_NAME,EMAIL,PHONE,HIRE_DATE,MANAGER_ID,JOB_TITLE) values (107,'Summer','Payne','summer.payne@example.com','515.123.8181',to_date('07-JUN-16','DD-MON-RR', 'NLS_DATE_LANGUAGE = AMERICAN'),106,'Public Accountant');
```

---

#### Creaet user : ORA-00900
`ORA-00900` 的錯誤代碼說明是 **SQL 敘述句無效**, 但實際上這樣的錯誤代碼說明有說跟沒說一樣...
原因是某些操作的權限不足, 像 `SHOW USER`, 就只能在 Server 上操作
```sql
SQL> SHOW USER;

ORA-00900: SQL 敘述句無效
```
