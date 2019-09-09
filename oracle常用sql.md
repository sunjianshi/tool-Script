## 1、遍历执行储存过程

```sql
declare
  lv_sql varchar2(1000);
begin
  for x in(select classes from DM_CAR_ANALYZE_SCORE_2 WHERE SCORE = 0) loop
	  lv_sql:='UPDATE DM_CAR_ANALYZE_SCORE_2 set SCORE = (SELECT round(a/b, 0) from (select SUM(SCORE) a, COUNT(1) b from DM_CAR_ANALYZE_SCORE_2 WHERE CLASSES in (select CAR_NAME from CAR_INFO WHERE CAR_TYPE = (select CAR_TYPE from CAR_INFO WHERE CAR_NAME='''||x.classes||''') and CAR_LEVEL = (select CAR_LEVEL from CAR_INFO WHERE CAR_NAME = '''||x.classes||''')))) WHERE classes = '''||x.classes||'''';
		execute immediate lv_sql;
	end loop;
end;
```

##  2、 查询表空间使用率

```sql
SELECT total.tablespace_name,
       Round(total.MB, 2)           AS Total_MB,
       Round(total.MB - free.MB, 2) AS Used_MB,
       Round(( 1 - free.MB / total.MB ) * 100, 2)
       || '%'                       AS Used_Pct
FROM   (SELECT tablespace_name,
               Sum(bytes) / 1024 / 1024 AS MB
        FROM   dba_free_space
        GROUP  BY tablespace_name) free,
       (SELECT tablespace_name,
               Sum(bytes) / 1024 / 1024 AS MB
        FROM   dba_data_files
        GROUP  BY tablespace_name) total
WHERE  free.tablespace_name = total.tablespace_name;
```

##  3、查询表空间数据文件

```sql
--查询表空间数据文件
select FILE_NAME,Sum(bytes) / 1024 / 1024/ 1024 AS GB from dba_data_files where tablespace_name ='BEIQIVMDA' group by FILE_NAME ;

--增加表空间容量
alter tablespace BEIQIVMDA add datafile '/oradata/database/beiqivmda_02.dbf' size 2048M autoextend on maxsize 30720M; 	
```

##  4、查询连接数

```sql
--查询数据库当前进程的连接数：
select count(*) from v$process;

--查看数据库当前会话的连接数：
select count(*) from v$session;

--查看数据库的并发连接数：
select count(*) from v$session where status='ACTIVE';

--查看当前数据库建立的会话情况：
select sid,serial#,username,program,machine,status from v$session;

--查询数据库允许的最大连接数：
select value from v$parameter where name = 'processes';
--或者：show parameter processes;

--修改数据库允许的最大连接数：
alter system set sessions = 500 scope = spfile;
alter system set processes = 350 scope = spfile;
```

##  5、更改用户密码

```sql
---更改用户密码
ALTER USER 用户名 IDENTIFIED BY "密码";
--(需要重启数据库才能实现连接数的修改)
```

##  6、快速恢复数据

```sql
---oracle delete、insert、update可恢复  truncate清空的不可恢复
--被闪回的表必须启用行移动功能
alter table DM_CAR_CLHALLCARSALES enable row movement;
--闪回到10分钟之前
 flashback table beiqi_vmda.DM_CAR_CLHALLCARSALES to timestamp(systimestamp-interval '10' minute);
 
 --闪回到具体时间
 flashback table 表名 to timestamp to_timestamp('2019-01-01 00:00:00','yyyy-mm-dd hh24:mi:ss');

--闪回被drop的表
flashback table 表名 to before drop;
```

## 7、相似度判断

```sql
select cartype s,classes s2 from (select distinct cartype from DM_SJZL) ,
(select distinct classes from DM_CAR_ANALYZE_SCORE_2)
where SYS.UTL_MATCH.edit_distance_similarity(cartype,classes) >80 and 
cartype <> classes
```

