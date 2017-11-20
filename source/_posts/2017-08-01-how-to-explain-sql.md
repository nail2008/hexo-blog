---
title: Oracle执行计划
layout: post
date: 2017-08-01 17:54:33
categories:
- 数据库
tags:
- oracle
- 执行计划
---


```sql

--set autotrace on;

--Session级别启用收集
alter session set STATISTICS_LEVEL = ALL; -- 捕捉下一次执行SQL的执行信息

select d.dept_name,u.user_name from sy_org_user u ,sy_org_dept d
where u.dept_code = d.dept_code;

-- 参数:SQL_ID,Child number,Format,
--select * from table(SYS.DBMS_XPLAN.DISPLAY_CURSOR(null,null,'ADVANCED ALLSTATS LAST PEEKED_BINDS'));
select * from table(SYS.DBMS_XPLAN.DISPLAY_CURSOR('bkx0s41y98z0c',null,'ADVANCED ALLSTATS LAST PEEKED_BINDS'));

-- 下面讲解这三个参数的获取

    -- 获取SQL_ID,通过sql_text,获得相应的SQL_ID,这里为bkx0s41y98z0c
    select * from v$SQLAREA where sql_text like 'select d.dept_name,u.user_name from sy_org_user u ,sy_org_dept d%';
    
    -- 获取Child number,一般指定为null,获取所有的子游标.如果要特别获取,使用以下方式:
        -- 父游标
        select * from v$SQLAREA where sql_id = 'bkx0s41y98z0c';
        
        -- 子游标:执行计划和优化环境
        select * from v$SQL where SQL_id = 'bkx0s41y98z0c';
        -- 计划:
        select * from v$SQL_PLAN where SQL_id = 'bkx0s41y98z0c' ;
        -- 优化环境:
        select * from v$SQL_OPTIMIZER_ENV where SQL_id = 'bkx0s41y98z0c';
    
    -- 设置Format: 可以选ALLSTATS=IOSTATS+MEMSTATS
       -- IOSTATS 显示该游标累计执行的IO统计信息(Buffers, Reads)
       -- MEMSTATS 累计执行的PGA使用信息(Omem 1Mem Used-Mem)
       -- LAST 仅显示最后一次执行的统计信息
       -- Advanced 显示outline\Query Block Name\Column Projection等信息
       -- PEEKED_BINDS 打印解析时使用的绑定变量
       -- Typical 不打印PROJECTION,ALIAS
       
       
-- 语句级别收集
select /*+ gather_plan_statistics*/ d.dept_name,u.user_name from sy_org_user u ,sy_org_dept d
where u.dept_code = d.dept_code;
--收集all stats有额外的负载,结束后需要回设成默认值:
alter session set STATISTICS_LEVEL=TYPICAL;
       
```