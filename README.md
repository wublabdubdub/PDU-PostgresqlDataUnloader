# PostgreSQL Disaster Recovery Tool PDU

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-10--17-336791?logo=postgresql) ![Version](https://img.shields.io/badge/Version-2.5-success?style=flat&color=2ea44f) ![License](https://img.shields.io/badge/License-Apache-green?logo=open-source-initiative)

## Author Introduction
**Zhang Chen (ZhangChen)**, with extensive experience in PostgreSQL database recovery, has led PostgreSQL data file extraction and recovery work at the terabyte level in the energy industry.  
The latest PDU software downloads and feature updates can be obtained through the **official WeChat public account `ZhangChen-PDU`**.

<img src="PDU二维码.jpg" alt="alt text" style="display: block; margin: 0 auto; width: 200px;">

**Specific usage scenarios for the PDU tool can also be found in the project's `wiki`, which is continuously updated.**

## Project Introduction
Consider these scenarios in a PostgreSQL database:
1. The database integrity is completely corrupted and cannot be opened.
2. Data is accidentally deleted.
3. Data files are accidentally deleted.

There might be many solutions: pg_filedump, pg_dirtyread, pg_resetlogs, pg_waldump. However, each of these tools has its own unique usage methods, which undoubtedly **increases the learning curve for users** and also **adds trial-and-error costs during data recovery** without guaranteeing effectiveness for the above three scenarios.

Data rescue in extreme scenarios is a crucial part of any database's ecosystem. In such scenarios,
- Oracle can use odu/dul to directly mine data from data files or ASM disks.
- PostgreSQL also has the pg_filedump tool in its ecosystem, which can mine single tables **provided the table structure is known**. However, for cases of complete database corruption, **how to effectively obtain the entire database's data dictionary and achieve orderly and convenient data export** is a **significant challenge** facing the PG ecosystem.

This project, **PDU (PostgreSQL Data Unloader)**, is a tool that integrates data file mining and recovery of accidentally deleted/updated data. Its characteristics are **low learning cost and high recovery efficiency**.

The PDU tool's file structure is simple, consisting of only two parts: the ***pdu executable file and the PGDATA.ini configuration file***. The overall design philosophy is to lower the learning cost for users.

## Core Features
**PostgreSQL Data Unloader (PDU)** is a disaster recovery tool for PostgreSQL versions 10-17, with main features:
1. Recover original data from DELETE/UPDATE operations in archived WALs.
2. Extract data directly from data files when the database cannot be started.
3. Support recovery at the level of single tables/entire databases/custom data files.
4. Data recovery via fragment scanning for dropped table/truncated table operations.

## Data Types Supported by PDU

Category | Data Type | Supported? |
---- | ---- | ---- |
Numeric Types | `smallint`, `integer`, `bigint`, `numeric`, `real`, `double precision` |  :white_check_mark:
Serial Types | `smallserial`, `serial`, `bigserial` |  :white_check_mark:
Monetary Type | `money` |  :white_check_mark:
Character Types | `character(n)`, `varchar(n)`, `text` |  :white_check_mark:
Binary Type | `bytea` |  :white_check_mark:
Date/Time | `date`, `time`, `timestamp`, `interval` |  :white_check_mark:
Boolean Type | `boolean` |  :white_check_mark:
UUID | `uuid` |  :white_check_mark:
XML | `xml` |  :white_check_mark:
JSON | `json`, `jsonb` |  :white_check_mark:
Arrays | Arrays of all basic types (e.g., `integer[]`) |  :white_check_mark:
Network Address | `cidr`, `inet`, `macaddr` |  :white_check_mark:
Bit String Types | `bit(n)`, `bit varying(n)` | :white_check_mark:
Enumeration Types | User-defined enumeration types | :x:
PostGIS Geometric Types | `point`, `line`, `lseg`, `box`, `path`, `polygon`, `circle` | :white_check_mark:
Text Search | `tsvector`, `tsquery` | :x:
Composite Types | User-defined types | :x:
Range Types | `int4range`, `int8range`, `numrange`, `tsrange`, `tstzrange`, `daterange` | :x:


## Quick Start
![EL-7/8/9](https://img.shields.io/badge/EL-7/8/9-red?style=flat&logo=redhat&logoColor=red) ![LINUX ARM64](https://img.shields.io/badge/LINUX-ARM-%23FCC624?style=flat&logo=linux&logoColor=black&labelColor=FCC624) ![LINUX X86](https://img.shields.io/badge/LINUX-X86-%23FCC624?style=flat&logo=linux&logoColor=black&labelColor=FCC624)
### 1. Environment Preparation
- Upload the installation package to the database server that needs recovery.
- Choose the corresponding installation package based on the server architecture.

```bash
PDU3.0.25.12_for_Postgresql10-18_Community_Edition_20251203_x86.zip
```

#### 1.1 Extract the Installation Package

```bash
mkdir pdu
cd pdu && unzip ../PDU3.0.25.12_for_Postgresql10-18_Community_Edition_20251203_x86.zip
Archive:  PDU3.0.25.12_for_Postgresql10-18_Community_Edition_20251203_x86.zip
 extracting: pdu10
 extracting: pdu11
 extracting: pdu12
 extracting: pdu13
 extracting: pdu14
 extracting: pdu15
 extracting: pdu16
 extracting: pdu17
 extracting: pdu18
 extracting: pdu.ini

```

#### 1.2 Edit the configuration file, filling in the data directory and archive directory.
```bash
vim pdu.ini

PGDATA=/home/postgres/data
ARCHIVE_DEST=/home/postgres/wal_arch
```

### 2. Choose the right version of pdu and Get in
```bash
[root@node1 PDU]# ./pdu18

╔══════════════════════════════════════════════════════╗
║  Copyright 2024-2025 ZhangChen. All rights reserved  ║
║  PDU: PostgreSQL Data Unloader                       ║
║  Version 3.0.25.12  (2025-12-03)                     ║
║  Valid Until 2124-12-03 10:00:00                     ║
╚══════════════════════════════════════════════════════╝

  Current DB Supported Version:
  ──────────────────────────
  • PostgreSQL 18

╔═══════════════════════════════════════════╗
║              COMMUNITY VERSION            ║
╠═══════════════════════════════════════════╣
║ • Max 100000 Records Per Table (unload)   ║
║ • Max 100000 Records Per Table (restore)  ║
║ • Max 1 GB Per Table                      ║
║ • Max 50 Columns Per Table                ║
║ • Max 500 Object Per Schema               ║
╚═══════════════════════════════════════════╝

  Contact Me:
  ───────────────────
  • WeChat: x1987LJ2020929
  • Email:  1109315180@qq.com
  • Tel:    15251853831


PDU.public=#
```

### 3. Bootstrap First
Use the command `<b;>` to quickly bootstrap. Afterwards, you can use common PostgreSQL commands like `\l`, `\dt`, `\dn`, `\d+`, `\d` to view the current databases, tables, schemas, table structures, etc.
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
### 4. Usage Help
```bash
PDU Data Rescue Tool | Command Reference
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
  **Basic Operations**
  b;                                      │ Initialize database metadata
  <exit>;<\q>;                            │ Exit the tool
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

  **Accidental Operation Data Recovery**
  scan [t1|manual];                       │ Scan deleted/update records of tables/Init metadata from manual
  restore del/upd [<TxID>|all];           │ Restore data by transaction ID/time range
  add <filenode> <table> <columns>;       │ Manually add table info (e.g. add 12345 t1 varchar,...) [!] Datafile should be put into path 'restore/datafile'
  restore db <db> <path>;                 │ Init customized database directory (e.g. restore db xmandb /home/...)
  ----------------------------------------.--------------------------------

  **Drop Table Recovery**
  dropscan/ds;                            │ Perform fragment scan recovery for tables configured in restore/tab.config
  dropscan/ds repair;                     │ Recover previously failed TOAST table scans
  dropscan/ds clean;                      │ Delete all directories under restore/dropscan
  dropscan/ds copy;                       │ Generate COPY commands for all table files under restore/dropscan
  ----------------------------------------.--------------------------------

  **Parameters**
  p|param startwal/endwal <WAL>;          │ Set WAL scan range (default archive boundaries)
  p|param starttime/endtime <TIME>;       │ Set time scan range (e.g. 2025-01-01 00:00:00)
  p|param resmode tx|time;                │ Set recovery mode (Transaction/Time)
  p|param restype delete|update;          │ Set recovery type (Delete/Update)
  p|param exmode csv|sql;                 │ Set export format (default CSV)
  p|param encoding utf8|gbk;              │ Set character encoding (default utf8)
  reset <parameter>|all;                      │ Reset specified parameter|all parameter
  show;                                   │ Display all parameters
  t;                                      │ Display all supported datatypes 
└──────────────────────────────────────────────────────────────────────────────────────────────────┘

Syntax Rules
◈ All commands must end with `;`
```

## Instructions for Different Data Recover Scenario 
PDU supports full Scenarios of Postgresql data recovery which can not be concluded in the ReadMe,so I've compiled **a series of documents in this repository wiki**, which you can refer to.

## Contact Me
For any questions regarding the use of PDU software or if the program encounters bugs/core dumps, you are welcome to contact me directly for assistance.
```bash
  • WeChat: x1987LJ2020929
  • Email:  1109315180@qq.com
  • Tel:    15251853831
```