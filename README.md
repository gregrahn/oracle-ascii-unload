# oracle-ascii-unload

## Overview

`oracle-ascii-unload` is an Oracle Pro*C program to efficiently unload data from an Oracle database table into a delimited ASCII text file delimited by control-A (also known as `^A` or `\x01` or `\001`).  It natively supports columns of type DATE, NUMBER, CHAR, VARCHAR2.  It does not currently support TIMESTAMP directly (via `select * from...` )), but it will support a TIMESTAMP column cast to a string like such: `select to_char(ts_column, 'YYYY-MM-DD HH24:MI:SSxFF') ts_column from...`  Thus if the table contains TIMESTAMP column(s), the query used to extract the data will have to explicitly list all the columns and cast TIMESTAMP columns to strings.

Fields are not quoted, so if it is possible that a text field could contain any `^A `values, then a different delimiter value should be chosen (see `oracle-ascii-unload.pc` line 150)

Tested on Oracle 11.2.0 only but should work on any version.

If you are using a version other than 11.2.0, you will likely need copy the `$ORACLE_HOME/rdbms/demo/demo_rdbms.mk` file to this directory.  If you don't have this file, then you will need to install the demos/examples.

## Prerequisites

* gcc
* make
* proc (from Oracle)

```
$ which proc
/home/oracle/app/oracle/product/11.2.0/dbhome_1/bin/proc
```

## Compile

Example on Linux:

```
$ ./oracle-ascii-unload-mk.sh

Pro*C/C++: Release 11.2.0.4.0 - Production on Wed Mar 5 11:30:00 2014

Copyright (c) 1982, 2011, Oracle and/or its affiliates.  All rights reserved.

System default option values taken from: /home/oracle/app/oracle/product/11.2.0/dbhome_1/precomp/admin/pcscfg.cfg

/usr/bin/gcc -fPIC -c -I/home/oracle/app/oracle/product/11.2.0/dbhome_1/rdbms/demo -I/home/oracle/app/oracle/product/11.2.0/dbhome_1/rdbms/public -I/home/oracle/app/oracle/product/11.2.0/dbhome_1/plsql/public -I/home/oracle/app/oracle/product/11.2.0/dbhome_1/network/public -I/home/oracle/app/oracle/product/11.2.0/dbhome_1/precomp/public oracle-ascii-unload.c -DLINUX -DORAX86_64 -D_GNU_SOURCE -D_LARGEFILE64_SOURCE=1 -D_LARGEFILE_SOURCE=1 -DSLTS_ENABLE -DSLMXMX_ENABLE -D_REENTRANT -DNS_THREADS -DLONG_IS_64 -DSS_64BIT_SERVER -DLDAP_CM
/usr/bin/gcc -L/home/oracle/app/oracle/product/11.2.0/dbhome_1/lib/ -L/home/oracle/app/oracle/product/11.2.0/dbhome_1/rdbms/lib/ -o oracle-ascii-unload oracle-ascii-unload.o -lclntsh   `cat /home/oracle/app/oracle/product/11.2.0/dbhome_1/lib/sysliblist` -ldl -lm  -lpthread -m64
```

## Exporting Data

To unload the data, simply specify the `userid` and the `sqlstmt` options and direct stderr to a log and stdout (the actual table rows) to a file or pipe to gzip to compress the output like such:

```
$ ./oracle-ascii-unload userid=scott/tiger sqlstmt="select * from emp" 2>emp.log | gzip > emp.txt.gz
```

If the table is partitioned, you can use a predicate in the `sqlstmt` query to export by partition like such:

```
$ ./oracle-ascii-unload userid=system/oracle sqlstmt="select * from partitioned_table where partition_key between date '2014-01-01' and date '2014-01-31' "2>partitioned_table_2014-01.log | gzip > partitioned_table_2014-01.txt.gz
```

Alternatively you can use the partition extension clause:

```
$ ./oracle-ascii-unload userid=system/oracle sqlstmt="select * from partitioned_table partition(P201401)" 2>partitioned_table_201401.log | gzip > partitioned_table_201401.txt.gz
```
More examples of [Oracle's partition extension](http://docs.oracle.com/cd/E16655_01/server.121/e17209/sql_elements009.htm#SQLRF51143) syntax:

```
{ PARTITION (partition)
| PARTITION FOR (partition_key_value [, partition_key_value]..)
| SUBPARTITION (subpartition)
| SUBPARTITION FOR (subpartition_key_value [, subpartition_key_value]..)
}
```

## Example Logfile Contents

The log file contains the query used, the column names, number of rows extracted and the elapsed time.

```
$ cat emp.log
Unloading 'select * from emp'
Array size = 100
EMPNO|ENAME|JOB|MGR|HIREDATE|SAL|COMM|DEPTNO
14 rows extracted
Elapsed: 00:00:00
```

## Example Datafile Contents

```
$ gunzip emp.txt.gz
$ vi emp.txt
7369^ASMITH^ACLERK^A7902^A1980-12-17 00:00:00^A800^A^A20
7499^AALLEN^ASALESMAN^A7698^A1981-02-20 00:00:00^A1600^A300^A30
7521^AWARD^ASALESMAN^A7698^A1981-02-22 00:00:00^A1250^A500^A30
7566^AJONES^AMANAGER^A7839^A1981-04-02 00:00:00^A2975^A^A20
7654^AMARTIN^ASALESMAN^A7698^A1981-09-28 00:00:00^A1250^A1400^A30
7698^ABLAKE^AMANAGER^A7839^A1981-05-01 00:00:00^A2850^A^A30
7782^ACLARK^AMANAGER^A7839^A1981-06-09 00:00:00^A2450^A^A10
7788^ASCOTT^AANALYST^A7566^A1987-04-19 00:00:00^A3000^A^A20
7839^AKING^APRESIDENT^A^A1981-11-17 00:00:00^A5000^A^A10
7844^ATURNER^ASALESMAN^A7698^A1981-09-08 00:00:00^A1500^A0^A30
7876^AADAMS^ACLERK^A7788^A1987-05-23 00:00:00^A1100^A^A20
7900^AJAMES^ACLERK^A7698^A1981-12-03 00:00:00^A950^A^A30
7902^AFORD^AANALYST^A7566^A1981-12-03 00:00:00^A3000^A^A20
7934^AMILLER^ACLERK^A7782^A1982-01-23 00:00:00^A1300^A^A10
```

## Authors

* Tom Kyte <https://twitter.com/OracleAskTom>
* Greg Rahn <https://twitter.com/GregRahn>

## License

Copyright 2014 Greg Rahn

Licensed under the Apache License, Version 2.0: http://www.apache.org/licenses/LICENSE-2.0
