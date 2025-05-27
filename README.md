# PostgreSQL灾难恢复工具PDU使用指南

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-10--17-336791?logo=postgresql) ![Version](https://img.shields.io/badge/Version-2.5-success?style=flat&color=2ea44f) ![License](https://img.shields.io/badge/License-Apache-green?logo=open-source-initiative)

## 项目介绍
数据拯救是专业DBA绕不开的一个话题，当数据库遇到正常途径无法开库、备份失效或干脆没有备份、常规方式已无法恢复等极端情况时，想要获取原数据库中的数据，唯一可行的手段就是直接对数据文件进行抽取。

极端场景下的数据拯救也是各种数据库的生态中重要的一环。在此类场景中，Oracle可以用odu/dul直接对数据文件或者ASM磁盘进行数据提取；Postgresql也有生态中的pg_filedump工具可以在知道表结构的情况下对单表进行挖掘。


经过很长一段时间的开发与验证，我终于可以向各位介绍这款面向Postgresql系数据库的数据拯救工具——pdu。

首先让我们明确pdu的使用场景：在无法使用备份进行恢复，且数据文件的一致性被破坏，无法通过gs_ctl start起库的情况下，可使用pdu工具直接从数据文件中进行数据抽取，是用于极端场景下的数据恢复手段，为用户的数据安全提供最后一道屏障。

pdu工具由两部分组成，pdu可执行文件+PGDATA.ini配置文件，整体的设计理念就是降低使用者的学习成本。
## 核心功能
**PostgreSQL Data Unloader (PDU)** 是针对PostgreSQL 10-17版本的灾难恢复工具，主要功能：
- 从归档WAL中恢复DELETE/UPDATE的原数据
- 在数据库无法启动时直接从数据文件中提取数据
- 支持单表/整库/自定义文件恢复
- 提供事务级/时间区间级数据恢复

## 快速部署
![EL-7/8/9](https://img.shields.io/badge/EL-7/8/9-red?style=flat&logo=redhat&logoColor=red) ![LINUX ARM64](https://img.shields.io/badge/LINUX-ARM-%23FCC624?style=flat&logo=linux&logoColor=black&labelColor=FCC624) ![LINUX X86](https://img.shields.io/badge/LINUX-X86-%23FCC624?style=flat&logo=linux&logoColor=black&labelColor=FCC624)
### 环境准备
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

#### 填写配置文件，写入数据目录和归档目录
```bash
vim pdu.ini

PGDATA=/home/postgres/data
ARCHIVE_DEST=/home/postgres/wal_arch
```

### 启动pdu


### 初始化

