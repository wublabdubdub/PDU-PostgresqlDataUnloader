# PostgreSQL灾难恢复工具PDU

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-10--17-336791?logo=postgresql) ![Version](https://img.shields.io/badge/Version-2.5-success?style=flat&color=2ea44f) ![License](https://img.shields.io/badge/License-Apache-green?logo=open-source-initiative)

## 作者介绍
**张晨（ZhangChen）**，具有丰富的Postgresql数据库恢复经验，主导过能源行业TB数据量级别的Postgresql数据文件抽取恢复工作。  
最新的PDU软件下载与功能进展均可通过**唯一官方微信公众号`ZhangChen-PDU`** 获取

<img src="PDU二维码.jpg" alt="alt text" style="display: block; margin: 0 auto; width: 200px;">

**PDU工具的具体使用场景也可在本项目的`wiki`中获取，长期更新中.**


## 项目介绍
假设一下Postgresql数据库中的这些场景
1. 数据库一致性完全损坏无法打开
2. 数据被误删除
3. 数据文件被误删除

解决方案也许有很多，pg_filedump、pg_dirtyread、pg_resetlogs、pg_waldump，但是以上每一样工具都有自己的独特的用法，无疑**增加了使用者的学习成本**，并且无法对上述三个场景的有效性作出保证，同样**增加了数据恢复时的试错成本**。



极端场景下的数据拯救是各种数据库的生态中重要的一环。在此类场景中，
- Oracle可以用odu/dul直接对数据文件或者ASM磁盘进行数
- Postgresql也有生态中的pg_filedump工具可以在***知道表结构的情况下***对单表进行挖掘。但是对于全库崩溃的情况，**如何有效地获取全库的数据字典，并有序便捷地实现数据导出**，是PG生态面临的一个**重要问题**。


本项目**PDU(Postgresql Data Unloader)**，是一款集数据文件挖掘、误删/误更新数据恢复等功能一体的工具，特点是**学习成本低、恢复效率高**。


PDU工具的文件结构简单，仅由两部分组成，***pdu可执行文件+PGDATA.ini配置文件***，整体的设计理念就是降低使用者的学习成本。
## 核心功能
**PostgreSQL Data Unloader (PDU)** 是针对PostgreSQL 10-17版本的灾难恢复工具，主要功能：
1. 从归档WAL中恢复DELETE/UPDATE的原数据
2. 在数据库无法启动时直接从数据文件中提取数据
3. 支持单表/整库/自定义数据文件级别的恢复
4. drop table/truncate table的碎片扫描数据恢复  
## PDU支持解析的数据类型

分类 | 数据类型 | 是否支持 |
---- | ---- | ---- |
数值类型 | `smallint`, `integer`, `bigint`, `numeric`, `real`, `double precision` |  :white_check_mark:
序列类型 | `smallserial`, `serial`, `bigserial` |  :white_check_mark:
货币类型 | `money` |  :white_check_mark:
字符类型 | `character(n)`, `varchar(n)`, `text` |  :white_check_mark:
二进制类型 | `bytea` |  :white_check_mark:
日期/时间 | `date`, `time`, `timestamp`, `interval` |  :white_check_mark:
布尔类型 | `boolean` |  :white_check_mark:
UUID | `uuid` |  :white_check_mark:
XML | `xml` |  :white_check_mark:
JSON | `json`, `jsonb` |  :white_check_mark:
数组 | 所有基础类型的数组（如 `integer[]`） |  :white_check_mark:
网络地址 | `cidr`, `inet`, `macaddr` |  :white_check_mark:
位串类型 | `bit(n)`, `bit varying(n)` | :white_check_mark:
枚举类型 | 用户自定义枚举类型 | :x:
PostGis几何类型 | `point`, `line`, `lseg`, `box`, `path`, `polygon`, `circle` | :white_check_mark:
文本搜索 | `tsvector`, `tsquery` | :x:
复合类型 | 用户自定义类型 | :x:
范围类型 | `int4range`, `int8range`, `numrange`, `tsrange`, `tstzrange`, `daterange` | :x:

## 使用帮助
```bash
PDU数据拯救工具 | 命令帮助
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
  **基础操作**
  b;                                      │ 初始化数据库元信息
  <exit;>|<\q;>                           │ 退出工具
  ----------------------------------------.--------------------------------

  **数据库切换**
  use <db>;                               │ 指定当前数据库（例: use logs;）
  set <schema>;                           │ 指定当前模式（例: set recovery;）
  ----------------------------------------.--------------------------------

  **元数据展示**
  \l;                                     │ 列出所有数据库
  \dn;                                    │ 列出当前数据库所有模式
  \dt;                                    │ 列出当前模式下的所有表
  \d+ <table>;                            │ 查看表结构详情（例: \d+ users;）
  \d <table>;                             │ 查看表列类型（例: \d users;）
  ----------------------------------------.--------------------------------

  **数据导出**
  u|unload tab <table>;                   │ 导出表数据到CSV（例: unload tab orders;）
  u|unload sch <schema>;                  │ 导出整个模式数据（例: unload sch public;）
  u|unload ddl;                           │ 生成当前模式DDL语句文件
  u|unload copy;                          │ 生成CSV的COPY语句脚本
  ----------------------------------------.--------------------------------

  **误操作数据恢复**
  scan [t1|manual];                       │ 扫描误删表/从manual目录初始化元数据
  restore del/upd [<TxID>|all];           │ 按事务号/时间区间恢复数据
  add <filenode> <表名> <字段类型列表>;   │ 手动添加表信息（例: add 12345 t1 varchar,...）[!] 需将数据文件放入restore/datafile
  restore db <库名> <路径>;               │ 初始化自定义数据库目录（例: restore db xmandb /home/...）
  ----------------------------------------.--------------------------------

  **Drop Table恢复**
  dropscan/ds;                            │ 针对文件restore/tab.config中配置的表进行碎片扫描恢复
  dropscan/ds repair;                     │ 针对此前扫描失败的TOAST表进行恢复
  dropscan/ds clean;                      │ 删除restore/dropscan下的所有目录
  dropscan/ds copy;                       │ 生成restore/dropscan下的所有表文件的COPY命令
  ----------------------------------------.--------------------------------

  **参数设置**
  p|param startwal/endwal <WAL文件>;      │ 设置WAL扫描范围（默认归档目录首尾）
  p|param starttime/endtime <时间>;       │ 设置时间扫描范围（例: 2025-01-01 00:00:00）
  p|param resmode tx|time;                │ 设置恢复模式（事务号/时间区间）
  p|param restype delete|update;          │ 设置恢复类型（删除/更新）
  p|param exmode csv|sql;                 │ 设置导出格式（默认CSV）
  p|param encoding utf8|gbk;              │ 设置字符编码（默认utf8）
  reset <参数名>|all;                     │ 重置指定参数|所有参数
  show;                                   │ 查看所有参数状态
  t;                                      │ 查看当前支持的数据类型
└──────────────────────────────────────────────────────────────────────────────────────────────────┘

语法规则
◈ 所有指令必须以`;`结尾
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

# 联系我
PDU软件在使用上有任何疑问或程序出现了bug/core dump，都欢迎联系我本人进行处理
```bash
  • WeChat: x1987LJ2020929
  • Email:  1109315180@qq.com
  • Tel:    15251853831
```