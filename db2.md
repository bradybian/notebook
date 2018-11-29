> ### db2 查询语句

```
SELECT SERVICE_LEVEL FROM SYSIBMADM.ENV_INST_INFO     #查看db2版本

call get_dbsize_info (?,?,?,-1)   #获取数据库容量和大小


SELECT (db_size)/1024/1024, (db_capacity)/1024/1024 FROM systools.stmg_dbsize_info  #查看数据库大小



select sum(TBSP_USED_SIZE_KB) as DATABASE_SIZE from sysibmadm.TBSP_UTILIZATION  #查看表空间
```

### AIX下查看db2端口

```
$db2 get dbm cfg |grep SVCENAME
TCP/IP Service name                          (SVCENAME) = DB2_db2inst1

$grep  DB2_db2inst1 /etc/services
DB2_db2inst1    64000/tcp
DB2_db2inst1_1  64001/tcp
DB2_db2inst1_2  64002/tcp
DB2_db2inst1_END        64003/tcp

SVCENAME 也有可能直接是个端口号

若是grep为空
去/etc /services
查找DB2
DB2_datamart    60000/tcp
DB2_datamart_1  60001/tcp
DB2_datamart_2  60002/tcp
DB2_datamart_END        60003/tcp
```

### 



