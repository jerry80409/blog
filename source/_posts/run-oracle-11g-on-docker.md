---
title: run-oracle-11g-on-docker
cover: /img/home-bg.jpg
date: 2020-01-13 16:15:20
categories: ['dba']
tags: ['oracle']
---
## Oracle Enterprise 11g Docker Image
這個紀錄主要是協助團隊, 於本地 Local 運行 Oracle Database, 方便大家開發, 避免開發上的資料衝突,

我採用的這個 image 非官方 Oracle 的映像, 但幫我省去了許多 oracle licence 同意處理的問題...
```
https://hub.docker.com/r/loliconneko/oracle-ee-11g
```

### Docker
如果是習慣使用 docker 的人, 可以參照文件
```bash
# docker pull
docker pull loliconneko/oracle-ee-11g

# run docker
docker run -d -p 8080:8080 -p 1521:1521 loliconneko/oracle-ee-11g
```

運行後, 基本的 connection 資訊
```markdown
hostname: localhost
port: 1521
sid: EE
service name: EE.oracle.docker
username: system
password: oracle
```

### Docker Compose
我自己平常的開發習慣是用 docker-compose(3.7);
```yml
version: '3.7'
services:
  # https://hub.docker.com/r/loliconneko/oracle-ee-11g
  oracle:
    image: loliconneko/oracle-ee-11g:latest
    container_name: oracle
    working_dir: /u01/app/oracle/oradata
    restart: always
    ports:
      - "1521:1521"
    environment:
      - WEB_CONSOLE=false
      - IMPORT_FROM_VOLUME=true
    volumes:
      # 這個 volumn 用來放 Oracle 本體
      - "./oradata:/u01/app/oracle"
      # 這個 volumn 用來匯入 Oracle 的資料
      - "./import_oracle:/u01/app/oracle/admin/EE/dpdump"
    networks:
      - backend

networks:
  backend:
    name: docker_backend
    driver: bridge
```

---
## Import data to Oracle
在 Docker volumn 的部分, 我在 local 規劃了 `oradata` 與 `import_oracle` 兩個資料夾,
* ordata 用來放 Oracle 本體, 避免 docker compose down 後, 資料遺失
* import_oracle 用來方便資料庫匯入
* 資料匯入匯出的操作, 盡量用 sysdba 的 role 操作


### Export dev Oracle
DEV 的 Oracle 是 Oracle Server, 簡單的 SSH 登入後, 把資料匯出後下載
```bash
# expdp explain
expdp -help

# expdp 匯出 schema, 匯出的檔案會位於 /u01/app/oracle/admin/EE/dpdump/expdat.dmp
expdp \"sys\/your-password as sysdba\" schemas=your-schema

# download expdat.dmp form dev to import_oracle folder
scp user@dev-server:/u01/app/oracle/admin/EE/dpdump/expdat.dmp ~/project/docker/import_oracle
```

### Import expdat.dmp to Oracle
因為用 `expdp` 操作資料匯出, 所以需要用 `impdp` 才能對應匯入操作, `loliconneko/oracle-ee-11g` 預設的帳號密碼
* user: sys / system
* password: oracle

```bash
# execute oracle bash, this instruction will run oracle container bash
docker-compose exec oracle bash

# in oracle container, go to the dpdump will see the expdat.dmp
cd /u01/app/oracle/admin/EE/dpdump
ls -al

# the instruction will scan /u01/app/oracle/admin/EE/dpdump and import expdat.dmp 
impdp -help
impdp \"sys\/oracle as sysdba\" schemas=your-schema;
```

---
## Memo
在整個轉移過程中其實 trouble shooting 非常多, 但大多皆可於 google 搜尋到對應的解決方式, 但細部的原因我也沒有很理解, Oracle 是個複雜龐大的系統, 我這邊整理一些我遇到的問題, 希望有幫到大家 XD

### 1. failed: ORA-28000: the account is locked
```sql
ALTER USER your-account ACCOUNT UNLOCK;
```

### 2. failed: ORA-01000:maximum open cursors exceeded.
```sql
ALTER SYSTEM SET open_cursors=1000;
```
