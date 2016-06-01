---
layout: post
title:  "Query Tuning"
date:   2016-05-19 11:08:00
categories: Java
---

In the post ["Find all tables and its child tables referencing a given table"](http://kelvinleong.github.io/oracle/2015/11/30/Find-Referenced-Tables.html), I have mentioned two solutions to find out all tables that referencing to a given table. Also, I have concluded that the solution which used the hierarchical query performed better.

In this chapter, I would explain deeply why the hierarchical solution perform a better solution. First of first, let's take a look at the elapsed time that two solutions spent on find out all children tables and its child tables of table 'BARG'. In oracle sqlplus terminal, use the following command to enable elapsed time display. If you don't want the time information later then use the same command but changing "ON" to "OFF"

```sql
set timing on   -- enable time information display
set timing off  -- disable time information display
```

Elapsed time of solution using hierarchical query

```sql
57 rows selected.

Elapsed: 00:00:05.53
```

Elapsed time of solution using native join operations

```sql
57 rows selected.

Elapsed: 00:00:14.17
```
From the result, we see that the first solution outweigh the second solution. To explain the difference, we would have a look at the execution plan table of these two query.

* What is execution plan ?

>A query plan (or query execution plan) is an ordered set of steps used to access data in a SQL relational database management system. This is a specific case of the relational model concept of access plans

When a query is submitted to the database, the query optimizer evaluates some of the different, correct possible plans for executing the query and returns what it considers the best option. That says we would fine tune a query with the power of execution plan table.

To enable sqlplus display the execution plan, use the following command to turn the autotrace on.

```sql
  set autotrace only
```

Reminded, if the autotrace is set, the query submitted to sql server will NOT be executed but analyzed and sql server returns the analyzed result as a plan table.

The following results are the plan table of solution using hierarchical query.

```sql
---------------------------------------------------------------------------------------------------------------------------------
| Id  | Operation                                                | Name                 | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                                         |                      |     1 |   364 |   189   (3)| 00:00:03 |
|   1 |  NESTED LOOPS OUTER                                      |                      |     1 |    67 |     3   (0)| 00:00:01 |
|   2 |   TABLE ACCESS BY INDEX ROWID                            | COL$                 |     1 |    24 |     2   (0)| 00:00:01 |
|*  3 |    INDEX UNIQUE SCAN                                     | I_COL3               |     1 |       |     1   (0)| 00:00:01 |
|*  4 |   TABLE ACCESS CLUSTER                                   | ATTRCOL$             |     1 |    43 |     1   (0)| 00:00:01 |
|   5 |  SORT ORDER BY                                           |                      |     1 |   364 |   189   (3)| 00:00:03 |
|*  6 |   FILTER                                                 |                      |       |       |            |          |
|   7 |    NESTED LOOPS OUTER                                    |                      |     1 |   364 |   188   (2)| 00:00:03 |
|   8 |     NESTED LOOPS                                         |                      |     1 |   321 |   187   (2)| 00:00:03 |
|   9 |      NESTED LOOPS                                        |                      |     1 |   297 |   186   (2)| 00:00:03 |
|  10 |       NESTED LOOPS                                       |                      |     1 |   294 |   185   (2)| 00:00:03 |
|  11 |        NESTED LOOPS                                      |                      |     1 |   291 |   184   (2)| 00:00:03 |
|  12 |         NESTED LOOPS                                     |                      |     1 |   264 |   182   (2)| 00:00:03 |
|  13 |          NESTED LOOPS                                    |                      |     1 |   252 |   181   (2)| 00:00:03 |
|* 14 |           HASH JOIN                                      |                      |     1 |   233 |   179   (2)| 00:00:03 |
|  15 |            NESTED LOOPS                                  |                      |     1 |   206 |   146   (3)| 00:00:02 |
|  16 |             NESTED LOOPS                                 |                      |     1 |   203 |   145   (3)| 00:00:02 |
|  17 |              NESTED LOOPS                                |                      |     1 |   182 |   144   (3)| 00:00:02 |
|  18 |               NESTED LOOPS OUTER                         |                      |     1 |   170 |   143   (3)| 00:00:02 |
|  19 |                NESTED LOOPS                              |                      |     1 |   127 |   142   (3)| 00:00:02 |
|  20 |                 NESTED LOOPS                             |                      |     1 |   106 |   141   (3)| 00:00:02 |
|  21 |                  NESTED LOOPS                            |                      |     2 |   182 |   137   (3)| 00:00:02 |
|* 22 |                   HASH JOIN                              |                      |     2 |   158 |   135   (3)| 00:00:02 |
|  23 |                    VIEW                                  |                      |     2 |   116 |   130   (4)| 00:00:02 |
|* 24 |                     CONNECT BY WITH FILTERING            |                      |       |       |            |          |
|* 25 |                      FILTER                              |                      |       |       |            |          |
|  26 |                       NESTED LOOPS OUTER                 |                      |     1 |   136 |    42   (0)| 00:00:01 |
|  27 |                        NESTED LOOPS                      |                      |     1 |   131 |    41   (0)| 00:00:01 |
|  28 |                         NESTED LOOPS OUTER               |                      |     1 |   110 |    40   (0)| 00:00:01 |
|  29 |                          NESTED LOOPS OUTER              |                      |     1 |   107 |    39   (0)| 00:00:01 |
|  30 |                           NESTED LOOPS                   |                      |     1 |   102 |    38   (0)| 00:00:01 |
|  31 |                            NESTED LOOPS OUTER            |                      |     1 |    99 |    37   (0)| 00:00:01 |
|  32 |                             NESTED LOOPS                 |                      |     1 |    96 |    36   (0)| 00:00:01 |
|  33 |                              NESTED LOOPS OUTER          |                      |     1 |    75 |    35   (0)| 00:00:01 |
|  34 |                               NESTED LOOPS               |                      |     1 |    54 |    34   (0)| 00:00:01 |
|* 35 |                                INDEX FAST FULL SCAN      | I_OBJ2               |     1 |    34 |    33   (0)| 00:00:01 |
|* 36 |                                TABLE ACCESS CLUSTER      | CDEF$                |     1 |    20 |     1   (0)| 00:00:01 |
|* 37 |                                 INDEX UNIQUE SCAN        | I_COBJ#              |     1 |       |     0   (0)| 00:00:01 |
|  38 |                               TABLE ACCESS BY INDEX ROWID| CON$                 |     1 |    21 |     1   (0)| 00:00:01 |
|* 39 |                                INDEX UNIQUE SCAN         | I_CON2               |     1 |       |     0   (0)| 00:00:01 |
|  40 |                              TABLE ACCESS BY INDEX ROWID | CON$                 |     1 |    21 |     1   (0)| 00:00:01 |
|* 41 |                               INDEX UNIQUE SCAN          | I_CON2               |     1 |       |     0   (0)| 00:00:01 |
|* 42 |                             INDEX RANGE SCAN             | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|* 43 |                            INDEX RANGE SCAN              | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|* 44 |                           INDEX RANGE SCAN               | I_OBJ1               |     1 |     5 |     1   (0)| 00:00:01 |
|* 45 |                          INDEX RANGE SCAN                | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|* 46 |                         INDEX RANGE SCAN                 | I_USER2              |     1 |    21 |     1   (0)| 00:00:01 |
|  47 |                        VIEW PUSHED PREDICATE             | _CURRENT_EDITION_OBJ |     1 |     5 |     1   (0)| 00:00:01 |
|* 48 |                         FILTER                           |                      |       |       |            |          |
|  49 |                          NESTED LOOPS                    |                      |     1 |    33 |     3   (0)| 00:00:01 |
|* 50 |                           INDEX RANGE SCAN               | I_OBJ1               |     1 |    12 |     2   (0)| 00:00:01 |
|* 51 |                           INDEX RANGE SCAN               | I_USER2              |     1 |    21 |     1   (0)| 00:00:01 |
|  52 |                          NESTED LOOPS                    |                      |     1 |    28 |     2   (0)| 00:00:01 |
|* 53 |                           INDEX FULL SCAN                | I_USER2              |     1 |    19 |     1   (0)| 00:00:01 |
|* 54 |                           INDEX RANGE SCAN               | I_OBJ4               |     1 |     9 |     1   (0)| 00:00:01 |
|  55 |                       NESTED LOOPS                       |                      |     1 |    21 |     2   (0)| 00:00:01 |
|* 56 |                        INDEX RANGE SCAN                  | I_OBJAUTH1           |     1 |     8 |     2   (0)| 00:00:01 |
|* 57 |                        FIXED TABLE FULL                  | X$KZSRO              |     1 |    13 |     0   (0)| 00:00:01 |
|* 58 |                       FIXED TABLE FULL                   | X$KZSPR              |     1 |    26 |     0   (0)| 00:00:01 |
|  59 |                       NESTED LOOPS                       |                      |     1 |    28 |     2   (0)| 00:00:01 |
|* 60 |                        INDEX FULL SCAN                   | I_USER2              |     1 |    19 |     1   (0)| 00:00:01 |
|* 61 |                        INDEX RANGE SCAN                  | I_OBJ4               |     1 |     9 |     1   (0)| 00:00:01 |
|* 62 |                      FILTER                              |                      |       |       |            |          |
|  63 |                       NESTED LOOPS OUTER                 |                      |     1 |   153 |    86   (3)| 00:00:02 |
|  64 |                        NESTED LOOPS OUTER                |                      |     1 |   150 |    85   (3)| 00:00:02 |
|  65 |                         NESTED LOOPS                     |                      |     1 |   145 |    84   (3)| 00:00:02 |
|  66 |                          NESTED LOOPS                    |                      |     1 |   124 |    83   (3)| 00:00:01 |
|* 67 |                           HASH JOIN OUTER                |                      |     1 |    90 |    81   (3)| 00:00:01 |
|  68 |                            NESTED LOOPS                  |                      |     1 |    85 |    68   (2)| 00:00:01 |
|  69 |                             NESTED LOOPS                 |                      |     1 |    82 |    67   (2)| 00:00:01 |
|* 70 |                              HASH JOIN                   |                      |     1 |    61 |    66   (2)| 00:00:01 |
|  71 |                               NESTED LOOPS OUTER         |                      |     1 |    41 |    50   (2)| 00:00:01 |
|* 72 |                                HASH JOIN                 |                      |     1 |    38 |    49   (3)| 00:00:01 |
|  73 |                                 CONNECT BY PUMP          |                      |       |       |            |          |
|  74 |                                 TABLE ACCESS FULL        | CON$                 |  3584 | 75264 |     6   (0)| 00:00:01 |
|* 75 |                                INDEX RANGE SCAN          | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|* 76 |                               TABLE ACCESS FULL          | CDEF$                |    82 |  1640 |    16   (0)| 00:00:01 |
|  77 |                              TABLE ACCESS BY INDEX ROWID | CON$                 |     1 |    21 |     1   (0)| 00:00:01 |
|* 78 |                               INDEX UNIQUE SCAN          | I_CON2               |     1 |       |     0   (0)| 00:00:01 |
|* 79 |                             INDEX RANGE SCAN             | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|  80 |                            VIEW                          | _CURRENT_EDITION_OBJ | 12815 | 64075 |    13   (8)| 00:00:01 |
|* 81 |                             FILTER                       |                      |       |       |            |          |
|* 82 |                              HASH JOIN                   |                      | 12815 |   412K|    13   (8)| 00:00:01 |
|  83 |                               INDEX FULL SCAN            | I_USER2              |    33 |   693 |     1   (0)| 00:00:01 |
|  84 |                               INDEX FAST FULL SCAN       | I_OBJ1               | 12815 |   150K|    11   (0)| 00:00:01 |
|  85 |                              NESTED LOOPS                |                      |     1 |    28 |     2   (0)| 00:00:01 |
|* 86 |                               INDEX FULL SCAN            | I_USER2              |     1 |    19 |     1   (0)| 00:00:01 |
|* 87 |                               INDEX RANGE SCAN           | I_OBJ4               |     1 |     9 |     1   (0)| 00:00:01 |
|  88 |                           TABLE ACCESS BY INDEX ROWID    | OBJ$                 |     1 |    34 |     2   (0)| 00:00:01 |
|* 89 |                            INDEX RANGE SCAN              | I_OBJ1               |     1 |       |     1   (0)| 00:00:01 |
|* 90 |                          INDEX RANGE SCAN                | I_USER2              |     1 |    21 |     1   (0)| 00:00:01 |
|* 91 |                         INDEX RANGE SCAN                 | I_OBJ1               |     1 |     5 |     1   (0)| 00:00:01 |
|* 92 |                        INDEX RANGE SCAN                  | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|  93 |                       NESTED LOOPS                       |                      |     1 |    21 |     2   (0)| 00:00:01 |
|* 94 |                        INDEX RANGE SCAN                  | I_OBJAUTH1           |     1 |     8 |     2   (0)| 00:00:01 |
|* 95 |                        FIXED TABLE FULL                  | X$KZSRO              |     1 |    13 |     0   (0)| 00:00:01 |
|* 96 |                       FIXED TABLE FULL                   | X$KZSPR              |     1 |    26 |     0   (0)| 00:00:01 |
|  97 |                       NESTED LOOPS                       |                      |     1 |    28 |     2   (0)| 00:00:01 |
|* 98 |                        INDEX FULL SCAN                   | I_USER2              |     1 |    19 |     1   (0)| 00:00:01 |
|* 99 |                        INDEX RANGE SCAN                  | I_OBJ4               |     1 |     9 |     1   (0)| 00:00:01 |
| 100 |                    TABLE ACCESS FULL                     | CON$                 |  3584 | 75264 |     6   (0)| 00:00:01 |
|*101 |                   TABLE ACCESS BY INDEX ROWID            | CDEF$                |     1 |    12 |     1   (0)| 00:00:01 |
|*102 |                    INDEX UNIQUE SCAN                     | I_CDEF1              |     1 |       |     0   (0)| 00:00:01 |
|*103 |                  TABLE ACCESS BY INDEX ROWID             | CCOL$                |     1 |    15 |     2   (0)| 00:00:01 |
|*104 |                   INDEX RANGE SCAN                       | I_CCOL1              |     1 |       |     1   (0)| 00:00:01 |
| 105 |                 TABLE ACCESS BY INDEX ROWID              | COL$                 |     1 |    21 |     1   (0)| 00:00:01 |
|*106 |                  INDEX UNIQUE SCAN                       | I_COL3               |     1 |       |     0   (0)| 00:00:01 |
|*107 |                TABLE ACCESS CLUSTER                      | ATTRCOL$             |     1 |    43 |     1   (0)| 00:00:01 |
|*108 |               INDEX RANGE SCAN                           | I_OBJ1               |     1 |    12 |     1   (0)| 00:00:01 |
|*109 |              INDEX RANGE SCAN                            | I_USER2              |     1 |    21 |     1   (0)| 00:00:01 |
|*110 |             INDEX RANGE SCAN                             | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
| 111 |            INDEX FAST FULL SCAN                          | I_OBJ2               | 12815 |   337K|    33   (0)| 00:00:01 |
|*112 |           TABLE ACCESS BY INDEX ROWID                    | ICOL$                |     1 |    19 |     2   (0)| 00:00:01 |
|*113 |            INDEX RANGE SCAN                              | I_ICOL1              |     2 |       |     1   (0)| 00:00:01 |
|*114 |          TABLE ACCESS BY INDEX ROWID                     | IND$                 |     1 |    12 |     1   (0)| 00:00:01 |
|*115 |           INDEX UNIQUE SCAN                              | I_IND1               |     1 |       |     0   (0)| 00:00:01 |
| 116 |         TABLE ACCESS BY INDEX ROWID                      | OBJ$                 |     1 |    27 |     2   (0)| 00:00:01 |
|*117 |          INDEX RANGE SCAN                                | I_OBJ1               |     1 |       |     1   (0)| 00:00:01 |
|*118 |        INDEX RANGE SCAN                                  | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|*119 |       INDEX RANGE SCAN                                   | I_USER2              |     1 |     3 |     1   (0)| 00:00:01 |
|*120 |      TABLE ACCESS CLUSTER                                | COL$                 |     1 |    24 |     1   (0)| 00:00:01 |
|*121 |     TABLE ACCESS CLUSTER                                 | ATTRCOL$             |     1 |    43 |     1   (0)| 00:00:01 |
| 122 |    NESTED LOOPS                                          |                      |     1 |    21 |     2   (0)| 00:00:01 |
|*123 |     INDEX RANGE SCAN                                     | I_OBJAUTH1           |     1 |     8 |     2   (0)| 00:00:01 |
|*124 |     FIXED TABLE FULL                                     | X$KZSRO              |     1 |    13 |     0   (0)| 00:00:01 |
|*125 |    FIXED TABLE FULL                                      | X$KZSPR              |     1 |    26 |     0   (0)| 00:00:01 |
| 126 |    NESTED LOOPS                                          |                      |     1 |    21 |     2   (0)| 00:00:01 |
|*127 |     INDEX RANGE SCAN                                     | I_OBJAUTH1           |     1 |     8 |     2   (0)| 00:00:01 |
|*128 |     FIXED TABLE FULL                                     | X$KZSRO              |     1 |    13 |     0   (0)| 00:00:01 |
|*129 |    FIXED TABLE FULL                                      | X$KZSPR              |     1 |    26 |     0   (0)| 00:00:01 |
| 130 |    NESTED LOOPS                                          |                      |     1 |    28 |     2   (0)| 00:00:01 |
|*131 |     INDEX FULL SCAN                                      | I_USER2              |     1 |    19 |     1   (0)| 00:00:01 |
|*132 |     INDEX RANGE SCAN                                     | I_OBJ4               |     1 |     9 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------------------------------------
```
Frankly speaking, the plan table is not easy to understand. The following summaries give you some notes on query plan.

FTS or Full Table Scan
  * Whole table is read upto high water mark
  * Uses multiblock input/output
  * Buffer from FTS operation is stored in LRU end of buffer cache

Index Unique Scan
  * Single block input/output

Index Fast Full Scan
  * Multi block i/o possible
  * Returned rows may not be in sorted order

Index Full Scan
  * Single block i/o
  * Returned rows generally will be in sorted order

To further understand the execution plan, please refer to [execution plan](https://docs.oracle.com/cd/B10501_01/server.920/a96533/ex_plan.htm) & [Understand oracle query plan](http://dwbi.org/database/oracle/38-oracle-query-plan-a-10-minutes-guide)

Except for the plan table, there are also some *Statistics information* that may help you to understand the plan.

Solution using hierarchical query

```sql
Statistics
----------------------------------------------------------
        127  recursive calls
          0  db block gets
       5379  consistent gets
          1  physical reads
          0  redo size
       2807  bytes sent via SQL*Net to client
        393  bytes received via SQL*Net from client
          5  SQL*Net roundtrips to/from client
          5  sorts (memory)
          0  sorts (disk)
         57  rows processed
```

Solution using native join operations

```sql
Statistics
----------------------------------------------------------
         92  recursive calls
          0  db block gets
      21305  consistent gets
        580  physical reads
          0  redo size
       2859  bytes sent via SQL*Net to client
        393  bytes received via SQL*Net from client
          5  SQL*Net roundtrips to/from client
          1  sorts (memory)
          0  sorts (disk)
         57  rows processed
```

From the Statistics, we can see that first solution has advantages on reducing consistent gets and physical read, which is due to that the nested hierarchy query dramatically narrowed the scope for data intersection thus reducing the operations while joining different tables.
