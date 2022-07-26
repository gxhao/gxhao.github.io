---
title: SQL Server 死锁问题排查
id: '32'
tags: ['sql','lock']
categories:
  - - deployment
date: 2022-05-12 15:59:16
url: /sql-lock
---

参考：
[微软官方api](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-lock-transact-sql?view=sql-server-ver15)

## 问题日志描述

> 事务(进程 ID 140)与另一个进程被死锁在 锁 | 通信缓冲区 资源上，并且已被选作死锁牺牲品。请重新运行该事务 

```
        at com.microsoft.sqlserver.jdbc.SQLServerException.makeFromDatabaseError(SQLServerException.java:196)
        at com.microsoft.sqlserver.jdbc.SQLServerStatement.getNextResult(SQLServerStatement.java:1454)
        at com.microsoft.sqlserver.jdbc.SQLServerPreparedStatement.doExecutePreparedStatement(SQLServerPreparedStatement.java:388)        
        at com.microsoft.sqlserver.jdbc.SQLServerPreparedStatement$PrepStmtExecCmd.doExecute(SQLServerPreparedStatement.java:338)
        at com.microsoft.sqlserver.jdbc.TDSCommand.ex
```

## 问题排查
### 1、查询哪些进程导致的死锁
```Sql
select request_session_id spid, object_name( resource_associated_entity_id )
tablename from sys.dm_tran_locks where resource_type = 'object'
```
### 2、查询死锁的原因
```SQL
CREATE Table #Who(spid int,
    ecid int,
    status nvarchar(50),
    loginname nvarchar(50),
    hostname nvarchar(50),
    blk int,
    dbname nvarchar(50),
    cmd nvarchar(50),
    request_ID int);

CREATE Table #Lock(spid int,
    dpid int,
    objid int,
    indld int,
    [Type]  nvarchar(20),
    Resource nvarchar(50),
    Mode nvarchar(10),
    Status nvarchar(10));

INSERT INTO #Who    EXEC sp_who active  
--看哪个引起的阻塞，blk \
 INSERT INTO #Lock     EXEC sp_lock  
--看锁住了那个资源id，objid 
 
 
 
DECLARE @DBName nvarchar(20);
SET @DBName='czggzy'

SELECT #Who.* FROM #Who WHERE dbname=@DBName
SELECT #Lock.* FROM #Lock
    JOIN #Who
        ON #Who.spid=#Lock.spid
            AND dbname=@DBName;

--最后发送到SQL Server的语句
DECLARE crsr Cursor FOR     SELECT blk FROM #Who WHERE dbname = @DBName AND blk <> 0;
DECLARE @blk int;
open crsr;
FETCH NEXT FROM crsr INTO @blk;
WHILE (@@FETCH_STATUS  =  0)BEGIN;

    dbcc inputbuffer(@blk);

    FETCH NEXT FROM crsr INTO @blk;
END;

close crsr;

DEALLOCATE crsr;
--锁定的资源
 SELECT #Who.spid,
hostname,
objid,
[type],
mode,
object_name(objid) as objName FROM #Lock    JOIN #Who        ON #Who.spid =#Lock.spid             AND dbname = @DBName    WHERE objid <> 0;

DROP Table #Who;

DROP Table #Lock;
```




## 问题处理

### 杀死进程
```
kill spid  --杀死进程
```
