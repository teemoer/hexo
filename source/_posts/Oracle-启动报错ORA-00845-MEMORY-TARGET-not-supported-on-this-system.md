---
title: 'Oracle 启动报错ORA-00845: MEMORY_TARGET not supported on this system'
date: 2016-07-27 23:33:32
tags: Oracle
---
##  启动Oracle报 ORA-00845: MEMORY_TARGET not supported on this system
今天晚上新装一台Oracle 11g的数据库，打算将SGA设大一点，知道 11g 中有一个新特新 MEMORY_TARGET，于是尝一下鲜，谁知报了个 ORA-00845，报错比较容易迷惑人，不借助Google真得想半天：
``` sql
SQL> alter system set memory_max_target=3G scope=spfile ;
System altered.
SQL> alter system set memory_target=2G scope=spfile ;
System altered.
SQL>
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup ;
ORA-00845: MEMORY_TARGET not supported on this system
```

###来自Oracle的官方解析是：
> Starting with Oracle Database 11g, the Automatic Memory Management feature requires more shared memory (/dev/shm)and file descriptors. The size of the shared memory should be at least the greater of MEMORY_MAX_TARGET and MEMORY_TARGET for each Oracle instance on the computer. If MEMORY_MAX_TARGET or MEMORY_TARGET is set to a non zero value, and an incorrect size is assigned to the shared memory, it will result in an ORA-00845 error at startup.


### 错误原因：
简单来说就是 MEMORY_MAX_TARGET 的设置不能超过 /dev/shm 的大小：
``` sql
[oracle@FWDB FWDB]$ df -h | grep shm
tmpfs 2.0G 0 2.0G 0% /dev/shm
```
还真是撞到这个枪口上了,马上把它加大：
``` sql
[root@FWDB ~]# cat /etc/fstab | grep tmpfs
tmpfs /dev/shm tmpfs defaults,size=4G 0 0
```
现在可以通过重启使这个配置生效，也可以通过重新挂载来修改其大小：

``` sql
[root@FWDB ~]# mount -o remount,size=4G /dev/shm
[root@FWDB ~]# df -h | grep shm
tmpfs 4.0G 0 4.0G 0% /dev/shm
```

再次启动数据库，没有报错了:
``` sh
su - oracle
lsnrctl stop
ps -ef|grep $ORACLE_SID|grep -v ora_|grep LOCAL=NO|awk '{print $2}'|xargs kill
sqlplus / as sysdba
shutdown immediate
exit
lsnrctl start
sqlplus / as sysbda
startup
```