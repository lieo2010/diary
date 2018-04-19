goldengate 12.3配置
---
DDL支持
编辑my.cnf，加入：

log-bin=/opt/app/mysql/var_3306/binlog

binlog_format=row

binlog-ignore-db=oggddl

---
```
ln -s /opt/app/mysql/tmp/mysql3306.sock /tmp/mysql.sock
```
进入ogg目录

```
[root@IDC-BX-DBtest-master og]# ./ddl_install.sh install root "test123" 3306
```
---
ogg主程设置
---
`vim  ~/.bash_profile`
加入：
```
export GGHOME=/opt/app/og

export PATH=$PATH:$GGHOME:$HOME/bin

export LD_LIBRARY_PATH=/opt/app/og:$LD_LIBRARY_PATH

export PATH
```
---
创建ogg相关目录
---
```
[root@IDC-BX-DBtest-master og]# ./ggsci
```

```
GGSCI (IDC-BX-DBtest-172-26-3-242) 1> CREATE subdirs
```

```
GGSCI (IDC-BX-DBtest-172-26-3-242) 2> edit params mgr
```
```
PORT 7809
DYNAMICPORTLIST 7810-7850
ACCESSRULE, PROG *,  IPADDR *,  ALLOW
```

---
配置SOURCE端
---
```
GGSCI (IDC-BX-DBtest-172-26-3-242) 3> dblogin sourcedb ogtest@172.26.3.242:3306,userid ogg,PASSWORD oggtest
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 4> ADD EXTRACT e1,tranlog,BEGIN NOW
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 5> ADD exttrail /og/dirdat/e1,EXTRACT e1
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 6> ADD EXTTRAIL /opt/app/og/dirdat/e1, EXTRACT E1
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 7> ADD EXTTRAIL /opt/app/og/dirdat/r1, EXTRACT P1
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 8> edit params e1
```
```
EXTRACT e1

setenv (MYSQL_HOME="/opt/app/mysql")

tranlogoptions altlogdest /opt/app/mysql/var_3306/binlog.index

sourcedb ogtest@master:3306,userid ogg,PASSWORD oggtest

exttrail /opt/app/og/dirdat/e1

dynamicresolution

gettruncates

TABLE ogtest.*;
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 8> ADD EXTRACT p2,exttrailsource /og/dirdat/e1
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 9> ADD rmttrail /og/dirdat/r1,EXTRACT p1
```
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 10> edit param p1
```
```
EXTRACT p1

rmthost slave,mgrport 7809

rmttrail /opt/app/og/dirdat/r1

passthru

gettruncates

TABLE ogtest.*;
```
---
配置目标端
---
创建ogg相关目录
```
[root@IDC-BX-DBtest-172-26-3-94 og]# ./ggsci
```
```
GGSCI (IDC-BX-DBtest-172-26-3-94) 1> CREATE subdirs
```
```
GGSCI (IDC-BX-DBtest-172-26-3-94) 2> edit params mgr
```
```
PORT 7809
DYNAMICPORTLIST 7810-7850
ACCESSRULE, PROG *,  IPADDR *,  ALLOW
```
```
GGSCI (IDC-BX-DBtest-172-26-3-94) 7> dblogin sourcedb ogtest@172.26.3.94:3306,userid ogg,PASSWORD oggtest
```
```
GGSCI (IDC-BX-DBtest-172-26-3-94 DBLOGIN as ogg) 8> ADD checkpointtable ogtest.checkpoint
```
```
GGSCI (IDC-BX-DBtest-172-26-3-94 DBLOGIN as ogg) 9> ADD replicat r1,exttrail /og/dirdat/r1,checkpointtable ogtest.checkpoint
```
```
GGSCI (IDC-BX-DBtest-172-26-3-94 DBLOGIN as ogg) 10> edit params r1
```
```
replicat r1
TARGETDB ogtest@slave:3306, userid ogg, PASSWORD oggtest
assumetargetdefs
DISCARDFILE /opt/app/og/dirrpt/r1.dsc,append,megabytes 50
MAP ogtest.*,target ogtest.*;
REPERROR DEFAULT, IGNORE
```
---
初始化数据
---
1. `dump数据出来`
```
/opt/app/mysql/bin/mysqldump -u ogg -poggtest -h master --master-data ogtest > ogtest.sql
```
2. `找到位置信息`
```mysql
CHANGE MASTER TO MASTER_LOG_FILE='binlog.000024', MASTER_LOG_POS=1660;
```
3. `改主库extract e1 从导出的时的binlog开始抽取`
```
GGSCI (IDC-BX-DBtest-172-26-3-242 DBLOGIN as ogg) 12> ALTER EXTRACT e1,VAM,lognum 24,logpos 1660
```
---
开启同步
---

1. ``主库``
```
start mgr
start e1
start p1
```
2. ``从库``
```
start mgr
start r1
```
---
观察状态
---
```
info all
lag e1
lag p1
lag r1
```