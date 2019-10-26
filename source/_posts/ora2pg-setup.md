---
title: ora2pg 安裝
cover: /img/home-bg.jpg
date: 2019-08-19 18:20:32
categories: ['dba']
tags: ['oracle', 'postgres']
---
### Summary
在處理 Oracle 遷移到 Postgres ...
Postgres [官方](https://wiki.postgresql.org/wiki/Converting_from_other_Databases_to_PostgreSQL#Utilities.2C_tools.2C_scripts_etc.) 其實提供了許多 Migration tools, 但大部分都需要付費購買, 但這種一次的作業實在很難爭取到公司資源, 在 open source 上相關資源較多的也只有 `Ora2pg` 與 `Orafce` 以及 `AWS Schema Conversion Tool`, 然而這些工具支持的面向皆不一樣, 以下簡單的介紹一下。

* **[Ora2pg](http://ora2pg.darold.net/)** - open source, perl 語言, 目前看起來功能最完整的工具支援 Oracle Table, Functaion, Trigger, etc. 轉換, 但設定有點複雜。
  
* **[AWS Schema Conversion Tool](http://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP_SchemaConversionTool.Installing.html)** - AWS 的工具, 個人覺得報表功能很好用, 缺點是沒用 AWS 的服務只能幫你轉 Oracle 的 schema。
  
* **[Orafce](https://github.com/orafce/orafce)** - open source, c 語言, 針對 oracle 的 function 與 package 做調整的工具。
  
* [fullconvert](https://www.spectralcore.com/fullconvert) - 看起來滿完整, 但試用版有滿多限制。

* [Database migration for Oracle and PostgreSQL](https://dbconvert.com/oracle/postgresql/) - 沒有支援 Mac 使用者。
  
* [Oracle-to-PostgreSQL](http://www.convert-in.com/ora2pgs.htm) - 沒有支援 Mac 使用者。

### Install Perl
* Ora2pg 由 perl 編寫, 所以需要 perl 環境
* 由於體積不大, 所以我直接把 Ora2pg 安裝在 Oracle Server (CentOS)

```bash
# check your perl version, 
# if it does't work, you should install perl
perl -v

# setup perl by yum, it's will setup all perl packagies
yum install perl*

# and setup cpan
yum install cpan

# then check perl
perl -v

# this is my perl info
This is perl 5, version 16, subversion 3 (v5.16.3) built for x86_64-linux-thread-multi
(with 39 registered patches, see perl -V for more detail)

Copyright 1987-2012, Larry Wall

Perl may be copied only under the terms of either the Artistic License or the
GNU General Public License, which may be found in the Perl 5 source kit.

Complete documentation for Perl, including FAQ lists, should be found on
this system using "man perl" or "perldoc perl".  If you have access to the
Internet, point your browser at http://www.perl.org/, the Perl Home Page.
```
---
### Ora2pg setup 
```bash
# install ora2pg v.20
cd ~/Download
wget https://github.com/darold/ora2pg/archive/v20.0.tar.gz
tar zxvf v20.0.tar.gz

# make and install
cd ora2pg-20.0
perl Makefile.PL
make && make install

# set env
echo "PERL5LIB=$PERL5LIB:/root/perl5/lib/perl5" >> ~/.bash_profile
echo "ORACLE_HOME=$ORACLE_HOME:/u01/app/oracle/product/11.2.0/EE" >> ~/.bash_profile
echo "LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib" >> ~/.bash_profile

source ~/.bash_profile

# install DBD:Oracle
perl -MCPAN -e 'install DBD::Oracle'

# make and install DBD:Oracle
cd ~/.cpan/build/DBD-Oracle*
perl Makefile.PL
make 
make install
```

```bash
# 網路上流傳的 check perl XD
vim check.pl
```

```perl
#!/usr/bin/perl
use ExtUtils::Installed;
my $inst = ExtUtils::Installed->new();
print join "\n",$inst->modules();
```

```bash
# 執行看看
chmod +x check.pl
perl check.pl

# 順利的話, stdout 會輸出
DBD::Oracle
Ora2Pg
Perl
Test::NoWarnings
```

```bash
# 確認 Ora2pg version
ora2pg -v
Ora2Pg v20.0
```

### Ora2pg init project
官方文件有說明專案式的 Ora2pg
```bash
ora2pg --project_base ~/ora2pg_projects/ --init_project project_name
Creating project test_project.
/app/migration/test_project/
        schema/
                dblinks/
                directories/
                functions/
                grants/
                mviews/
                packages/
                partitions/
                procedures/
                sequences/
                synonyms/
                tables/
                tablespaces/
                triggers/
                types/
                views/
        sources/
                functions/
                mviews/
                packages/
                partitions/
                procedures/
                triggers/
                types/
                views/
        data/
        config/
        reports/
 
Generating generic configuration file
Creating script export_schema.sh to automate all exports.
Creating script import_all.sh to automate all imports.
```

### Setting ORACLE connection
```
ORACLE_DSN	dbi:Oracle:host=localhost;sid=EE;port=1521
ORACLE_USER	system
ORACLE_PWD	foobar
```

### Export / Import schema
`--init_project` 提供了兩個方便的 scripts 方便作業 
* `export_schema.sh` 用來匯出 data objects schema.
* `import_all.sh` 用來匯入 data objects schema.

```bash
# execute
./export_schema.sh

# stdout
[========================>] 52/52 tables (100.0%) end of scanning.
[========================>] 11/11 objects types (100.0%) end of objects auditing.
Running: ora2pg -p -t TABLE -o table.sql -b ./schema/tables -c ./config/ora2pg.conf
[========================>] 52/52 tables (100.0%) end of scanning.
[========================>] 52/52 tables (100.0%) end of table export.
Running: ora2pg -p -t PACKAGE -o package.sql -b ./schema/packages -c ./config/ora2pg.conf
[========================>] 1/1 packages (100.0%) end of output.
Running: ora2pg -p -t VIEW -o view.sql -b ./schema/views -c ./config/ora2pg.conf
[========================>] 0/0 views (100.0%) end of output.
Running: ora2pg -p -t GRANT -o grant.sql -b ./schema/grants -c ./config/ora2pg.conf
WARNING: Exporting privilege as non DBA user is not allowed, see USER_GRANT
Running: ora2pg -p -t SEQUENCE -o sequence.sql -b ./schema/sequences -c ./config/ora2pg.conf
[========================>] 1/1 sequences (100.0%) end of output.
Running: ora2pg -p -t TRIGGER -o trigger.sql -b ./schema/triggers -c ./config/ora2pg.conf
[========================>] 82/82 triggers (100.0%) end of output.
Running: ora2pg -p -t FUNCTION -o function.sql -b ./schema/functions -c ./config/ora2pg.conf
[========================>] 4/4 functions (100.0%) end of functions export.
Running: ora2pg -p -t PROCEDURE -o procedure.sql -b ./schema/procedures -c ./config/ora2pg.conf
[========================>] 0/0 procedures (100.0%) end of procedures export.
Running: ora2pg -p -t TABLESPACE -o tablespace.sql -b ./schema/tablespaces -c ./config/ora2pg.conf
WARNING: Exporting tablespace as non DBA user is not allowed, see USER_GRANT
Running: ora2pg -p -t PARTITION -o partition.sql -b ./schema/partitions -c ./config/ora2pg.conf
[========================>] 0/0 partitions (100.0%) end of output.
Running: ora2pg -p -t TYPE -o type.sql -b ./schema/types -c ./config/ora2pg.conf
[========================>] 0/0 types (100.0%) end of output.
Running: ora2pg -p -t MVIEW -o mview.sql -b ./schema/mviews -c ./config/ora2pg.conf
[========================>] 0/0 materialized views (100.0%) end of output.
Running: ora2pg -p -t DBLINK -o dblink.sql -b ./schema/dblinks -c ./config/ora2pg.conf
[========================>] 0/0 dblink (100.0%) end of output.
Running: ora2pg -p -t SYNONYM -o synonym.sql -b ./schema/synonyms -c ./config/ora2pg.conf
[========================>] 0/0 synonyms (100.0%) end of output.
Running: ora2pg -p -t DIRECTORY -o directorie.sql -b ./schema/directories -c ./config/ora2pg.conf
[========================>] 0/0 directory (100.0%) end of output.
Running: ora2pg -t PACKAGE -o package.sql -b ./sources/packages -c ./config/ora2pg.conf
[========================>] 1/1 packages (100.0%) end of output.
Running: ora2pg -t VIEW -o view.sql -b ./sources/views -c ./config/ora2pg.conf
[========================>] 0/0 views (100.0%) end of output.
Running: ora2pg -t TRIGGER -o trigger.sql -b ./sources/triggers -c ./config/ora2pg.conf
[========================>] 82/82 triggers (100.0%) end of output.
Running: ora2pg -t FUNCTION -o function.sql -b ./sources/functions -c ./config/ora2pg.conf
[========================>] 4/4 functions (100.0%) end of functions export.
Running: ora2pg -t PROCEDURE -o procedure.sql -b ./sources/procedures -c ./config/ora2pg.conf
[========================>] 0/0 procedures (100.0%) end of procedures export.
Running: ora2pg -t PARTITION -o partition.sql -b ./sources/partitions -c ./config/ora2pg.conf
[========================>] 0/0 partitions (100.0%) end of output.
Running: ora2pg -t TYPE -o type.sql -b ./sources/types -c ./config/ora2pg.conf
[========================>] 0/0 types (100.0%) end of output.
Running: ora2pg -t MVIEW -o mview.sql -b ./sources/mviews -c ./config/ora2pg.conf
[========================>] 0/0 materialized views (100.0%) end of output.

To extract data use the following command:

# you can follow this command export data
ora2pg -t COPY -o data.sql -b ./data -c ./config/ora2pg.conf
```

