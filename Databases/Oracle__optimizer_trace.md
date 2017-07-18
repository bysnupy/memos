## SQL and Optimizer trace on the Oracle Database

### Introduction
It's the snippet note for SQL trace on session level in Oracle DB to take the execution plans, wait events and so on.

### Environment

Item|Value
-|-
  DBMS| Oracle Database Standard Edition 12.1.0.2.0
  Client| SQL*Plus
  Requirements| SYSDBA or ALTER SESSION system privilege is needed

### Configuration steps

#### Step1: Extended SQL trace by using 10046 event

The 10046 is a event code for tracing SQL_TRACE type of internal Oracle, it break into the several levels.

Level|Description
-|-
level 1 | Default value, standard sql trace information (resonponse time, physical/logical reads and so on).
level 4 | As Level 1 + trace bind values, the same syntax is 'bind=true'
level 8 | As Level 1 + trace waits, the same syntax is 'wait=true'<br/>e.g.> latch wait, full scan, index scan and so on.

As of 11g as follows.

Level|Description
-|-
level 16 | Generate traces for each execution, the same syntax is 'plan_stat=all_executions'
level 32 | Not to trace about execution statistics, the same syntax is 'plan_stat=never'

As of 11.2.0.2 as follows.

Level|Description
-|-
level 64 | Generate extensive traces for slow SQL, the same syntax is 'plan_stat=adaptive'<br/>e.g.>1 additional minute of DB time.

And the level is used by following format:

Level|Description
-|-
level 12 = level 4 + level 8 | level 4 and 8 are enabled at the same time.
level 28 = level 4 + level 8 + level 16 | level 4, 8 and 16 are enabled at the same time.

Additionally the resulting trace files need to be translated by tkprof utility for checking SQL trace results.

The settings for enabling to sql trace on the session level as follows.

```sql
SQL> ALTER SESSION SET TRACEFILE_IDENTIFIER = '10046';
SQL> ALTER SESSION SET TIMED_STATISTICS = true;
SQL> ALTER SESSION SET STATITICS_LEVEL = all;
SQL> ALTER SESSION SET MAX_DUMP_FILE_SIZE = unlimited;
SQL> -- Pattern1
SQL> ALTER SESSION SET EVENTS '10046 trace name context forever, level 28';
SQL> -- Pattern2: The alternative is as follows at 11g or more than it.
SQL> ALTER SESSION SET EVENTS 'sql_trace bind=true, wait=true, plan_stat=all_executions';
```

And then execute the sql statements for check to trace.

The statement for disabling to sql trace as follows.

```sql
SQL> -- Pattern1
SQL> ALTER SESSION SET EVENTS '10046 trace name context off';
SQL> -- Pattern2: The alternative is as follows at 11g or more than it.
SQL> ALTER SESSION SET EVENTS 'sql_trace off';
```

Display the resulting tracefile path.

```sql
SQL> SELECT value FROM v$diag_info WHERE name = 'Default Trace File';
VALUE
-------------------------------------------------------------
/u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_18830_10046.trc
```

Finally the trace file is translated into the human readable report file with the following command format.

```bash
$ tkprof [trace file path] [output file as report]
```

The output executed actually is as follows.

```bash
$ tkprof /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_18830_10046.trc /tmp/orcl_18830_10046.txt
TKPROF: Release 12.1.0.2.0 - Development on Tue Dec 6 15:56:20 2016

Copyright (c) 1982, 2015, Oracle and/or its affiliates.  All rights reserved.

$ cat /tmp/orcl_18830_10046.txt
TKPROF: Release 12.1.0.2.0 - Development on Tue Dec 6 15:56:20 2016

Copyright (c) 1982, 2015, Oracle and/or its affiliates.  All rights reserved.

Trace file: /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_18830_10046.trc
Sort options: default

********************************************************************************
count    = number of times OCI procedure was executed
cpu      = cpu time in seconds executing
elapsed  = elapsed time in seconds executing
disk     = number of physical reads of buffers from disk
query    = number of buffers gotten for consistent read
current  = number of buffers gotten in current mode (usually for update)
rows     = number of rows processed by the fetch or execute call
********************************************************************************

SQL ID: 1p5grz1gs7fjq Plan Hash: 813480514

select obj#,type#,ctime,mtime,stime, status, dataobj#, flags, oid$, spare1,
  spare2, spare3, signature, spare7, spare8, spare9
from
 obj$ where owner#=:1 and name=:2 and namespace=:3 and remoteowner is null
  and linkname is null and subname is null


call     count       cpu    elapsed       disk      query    current        rows
------- ------  -------- ---------- ---------- ---------- ----------  ----------
Parse        1      0.00       0.00          0          0          0           0
Execute      7      0.00       0.02          0          0          0           0
Fetch        7      0.00       0.00          0         28          0           7
------- ------  -------- ---------- ---------- ---------- ----------  ----------
total       15      0.00       0.02          0         28          0           7

Misses in library cache during parse: 1
Misses in library cache during execute: 1
Optimizer mode: CHOOSE
Parsing user id: SYS   (recursive depth: 1)
Number of plan statistics captured: 7

Rows (1st) Rows (avg) Rows (max)  Row Source Operation
---------- ---------- ----------  ---------------------------------------------------
         1          1          1  TABLE ACCESS BY INDEX ROWID BATCHED OBJ$ (cr=4 pr=0 pw=0 time=44 us cost=4 size=100 card=1)
         1          1          1   INDEX RANGE SCAN I_OBJ2 (cr=3 pr=0 pw=0 time=36 us cost=3 size=0 card=1)(object id 37)

********************************************************************************
...snip...
Trace file: /u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_18830_10046.trc
Trace file compatibility: 11.1.0.7
Sort options: default

       1  session in tracefile.
       9  user  SQL statements in trace file.
      33  internal SQL statements in trace file.
      42  SQL statements in trace file.
      38  unique SQL statements in trace file.
    2131  lines in trace file.
    9180  elapsed seconds in trace file.
```

#### Step2: Optimizer trace by using 10053 event

This event code is used of optimizer tracing such as query transformation process, query optimization process to diagnose optimizer decisions and behaviour.

```sql
SQL> ALTER SESSION SET TRACEFILE_IDENTIFIER = '10053';
SQL> ALTER SESSION SET MAX_DUMP_FILE_SIZE = unlimited;
SQL> ALTER SESSION SET EVENTS '10053 trace name context forever, level 1';
```

Execute the sql statements to diagnose.

Stop the tracing and take the trace file name and path.

```sql
SQL> -- stop the tracing with 10053 event.
SQL> ALTER SESSION SET EVENTS '10053 trace name context off';
SQL> -- get the trace file path from v$diag_info view.
SQL> SELECT value FROM v$diag_info WHERE name = 'Default Trace File';
VALUE
--------------------------------------------------------------------------------
/u01/app/oracle/diag/rdbms/orcl/orcl/trace/orcl_ora_33842_10053.trc
```

If you can't get the valid trace information, you should execute the sql statements with a meaningless spaces or hint comment again.
The 10053 event code needs the hard parse of the sql statements for getting the valid trace informations.

Done.
