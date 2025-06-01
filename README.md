# PostgreSQL灾难恢复工具PDU使用指南

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-10--17-336791?logo=postgresql) ![Version](https://img.shields.io/badge/Version-2.5-success?style=flat&color=2ea44f) ![License](https://img.shields.io/badge/License-Apache-green?logo=open-source-initiative)

## 项目介绍
假设一下Postgresql数据库中的这些场景
1. 数据库一致性完全损坏无法打开
2. 数据被误删除
3. 数据被误更新

解决方案也许有很多，pg_filedump、pg_dirtyread、pg_resetlogs、pg_waldump，以上每一样工具都有自己的独特的用法，无疑**增加了使用者的学习成本**，并且无法对上述三个场景的有效性作出保证，同样**增加了数据恢复时的试错成本**。



极端场景下的数据拯救是各种数据库的生态中重要的一环。在此类场景中，
- Oracle可以用odu/dul直接对数据文件或者ASM磁盘进行数据提取；
- Postgresql也有生态中的pg_filedump工具可以在知道表结构的情况下对单表进行挖掘。但是对于全库崩溃的情况，**如何有效地获取全库的数据字典，并有序便捷地实现数据导出**，是PG生态面临的一个**重要问题**。


本项目**PDU(Postgresql Data Unloader)**，是一款集数据文件挖掘、误删/误更新数据恢复等功能一体的工具，特点是学习成本低、恢复效率高。


PDU工具的文件结构简单，仅由两部分组成，***pdu可执行文件+PGDATA.ini配置文件***，整体的设计理念就是降低使用者的学习成本。
## 核心功能
**PostgreSQL Data Unloader (PDU)** 是针对PostgreSQL 10-17版本的灾难恢复工具，主要功能：
- 从归档WAL中恢复DELETE/UPDATE的原数据
- 在数据库无法启动时直接从数据文件中提取数据
- 支持单表/整库/自定义文件恢复
- 提供事务级/时间区间级数据恢复

## 使用帮助
```bash
PDU.public=# h;

PDU Data Rescue Tool | Command Reference
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
  **Basic Operations**
  b;                                      │ Initialize database metadata
  exit; | \q;                             │ Exit the tool
  ----------------------------------------.--------------------------------

  **Database Context**
  use <db>;                               │ Set current database (e.g. use logs;)
  set <schema>;                           │ Set current schema (e.g. set recovery;)
  ----------------------------------------.--------------------------------

  **Metadata Display**
  \l;                                     │ List all databases
  \dn;                                    │ List all schema in current database
  \dt;                                    │ List tables in current schema
  \d+ <table>;                            │ View table structure details (e.g. \d+ users;)
  \d <table>;                             │ View column types (e.g. \d users;)
  ----------------------------------------.--------------------------------

  **Data Export**
  u|unload tab <table>;                   │ Export table to CSV (e.g. unload tab orders;)
  u|unload sch <schema>;                  │ Export entire schema (e.g. unload sch public;)
  u|unload ddl;                           │ Generate DDL statements of current schema
  u|unload copy;                          │ Generate COPY statements for CSVs
  ----------------------------------------.--------------------------------

  **Data Recovery**
  scan [t1|manual];                       │ Scan deleted/update records of tables/Init metadata from manual
  restore del/upd [<TxID>|all];           │ Restore data by transaction ID/time range
  add <filenode> <table> <columns>;       │ Manually add table info (e.g. add 12345 t1 varchar,...) [!] Datafile should be put into path 'restore/datafile'
  restore db <db> <path>;                 │ Init customized database directory (e.g. restore db xmandb /home/...)
  ----------------------------------------.--------------------------------

  **Parameters**
  p|param startwal/endwal <WAL>;          │ Set WAL scan range (default archive boundaries)
  p|param starttime/endtime <TIME>;       │ Set time scan range (e.g. 2025-01-01 00:00:00)
  p|param resmode tx|time;                │ Set recovery mode (Transaction/Time)
  p|param restype delete|update;          │ Set recovery type (Delete/Update)
  p|param exmode csv|sql;                 │ Set export format (default CSV)
  p|param encoding utf8|gbk;              │ Set character encoding (default utf8)
  reset <param>|all;                      │ Reset specified parameter|all parameter
  show;                                   │ Display all parameters
└──────────────────────────────────────────────────────────────────────────────────────────────────┘

Syntax Rules
◈ All commands must end with `;`

```

## 快速部署
![EL-7/8/9](https://img.shields.io/badge/EL-7/8/9-red?style=flat&logo=redhat&logoColor=red) ![LINUX ARM64](https://img.shields.io/badge/LINUX-ARM-%23FCC624?style=flat&logo=linux&logoColor=black&labelColor=FCC624) ![LINUX X86](https://img.shields.io/badge/LINUX-X86-%23FCC624?style=flat&logo=linux&logoColor=black&labelColor=FCC624)
### 1、环境准备
- 将安装包上传到需要恢复的数据库服务器上
- 根据服务器架构选择对应的安装包
  
```bash
PDU{版本号}_for_Postgresql10-17社区版_{发布日期}_X86
PDU{版本号}_for_Postgresql10-17社区版_{发布日期}_ARM
```

#### 解压安装包

```bash
mkdir pdu
cd pdu && unzip ../PDU2.0_for_Postgresql10-17社区版_20250521_x86.zip 
```

#### 填写配置文件，填入数据目录和归档目录
```bash
vim pdu.ini

PGDATA=/home/postgres/data
ARCHIVE_DEST=/home/postgres/wal_arch
```

### 2、启动pdu
```bash
[root@node1 xman]# ./pdu

╔══════════════════════════════════════════════════════╗
║  Copyright 2024-2025 ZhangChen. All rights reserved  ║
║  PDU: PostgreSQL Data Unloader                       ║
║  Version 2.5.0 (2025-05-22)                          ║
╚══════════════════════════════════════════════════════╝

  Current DB Supported Version:
  ──────────────────────────
  • PostgreSQL 10

╔═══════════════════════════════════════════╗
║           PROFESSIONAL EDITION            ║
╠═══════════════════════════════════════════╣
║ • Licensed to zc                          ║
║ • Full functionality                      ║
║ • No limitations                          ║
╚═══════════════════════════════════════════╝

  Contact Me:
  ───────────────────
  • WeChat: x1987LJ2020929
  • Email:  1109315180@qq.com
  • Tel:    15251853831


PDU.public=#
```

### 3、数据字典初始化
使用命令<b;>就可以快速实现数据字典的获取，后续可以通过\l、\dt、\dn、\d+、\d等PG常用命令查看当前的数据库、表、模式、表结构等信息。
```bash
PDU.public=# b;

Initializing...
 -pg_database:</home/10pg/data/global/1262>

Database:postgres
      -pg_schema:</home/10pg/data/base/13214/2615>
      -pg_class:</home/10pg/data/base/13214/1259> 69 Records
      -pg_attribute:</home/10pg/data/base/13214/1249> 2579 Records
      Schema:
        ▌ public 0 tables

Database:xman
      -pg_schema:</home/10pg/tbsxman/PG_10_201707211/213244/2615>
      -pg_class:</home/10pg/tbsxman/PG_10_201707211/213244/1259> 493 Records
      -pg_attribute:</home/10pg/tbsxman/PG_10_201707211/213244/1249> 8697 Records
      Schema:
        ▌ public 0 tables
        ▌ xman 212 tables

```
## PDU支持解析的数据类型
| 一级分类 | 二级分类 | 具体类型     |
|:--------:|:--------:|:------------|
| 数值类型 | 整数类型 | smallserial |
|          |          | smallint    |
|          |          | int         |
|          |          | tinyint     |
|          |          | oid         |
|          |          | xid         |
|          |          | serial      |
|          |          | bigint      |
|          |          | bigserial   |
|          | 浮点类型 | float4      |
|          |          | float8      |
|          |          | numeric     |
|          | 日期类型 | time        |
|          |          | timetz      |
|          |          | date        |
|          |          | timestamp   |
|          |          | timestamptz |
|          | 字符类型 | name        |
|          |          | charn       |
|          |          | char        |
|          |          | varchar     |
|          |          | bpchar      |
|          |          | text        |
|          |          | json        |
|          |          | jsonb       |
|          |          | xml         |
|          |          | clob        |
|          |          | blob        |
|          |          | bytea       |