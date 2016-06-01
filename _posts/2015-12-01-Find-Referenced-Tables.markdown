---
layout: post
title:  "Find all tables and its child tables referencing a given table "
date:   2015-11-30 15:53:00
categories: Oracle
---

Scenario: *An requirement states "to delete the data from a table which is referenced by different tables".*

Well, to delete records in Database is believed to be an simple job by *"Delete FROM table_name"*. But the prerequisite is that the table should not be referenced by any other tables otherwise it may cause the "foreign key violation" while trying to delete the table. Particularly, a given tables with lots of child tables and so does these child tables, though it's a bad design. Then it would be a nightmare to delete specific records in the given table.

Before going thru, we should have an clear view on how the reference constraints is maintained in Oracle. The key system table is ***ALL_CONSTRAINTS***.

>**ALL_CONSTRAINTS** describes constraint definitions on tables accessible to the current user.

|   Column           |         Descriptions                                         |
|:-------------------|:-------------------------------------------------------------|
|  CONSTRAINT_TYPE   | * C (check constraint on a table)                            |
|                    | * P (primary key)                                            |
|                    | * U (unique key)                                             |
|                    | * R (referential integrity)                                  |
|                    | * V (with check option, on a view)                           |  
|                    | * O (with read only, on a view)                              |
|  R_CONSTRAINT_NAME | Name of the unique constraint definition for referenced table|
|  CONSTRAINT_NAME   | Name of the constraint definition                            |

There is a hierarchical structure in this table that *R_CONSTRAINT_NAME* indicating the foreign key constraint of the table would be the *CONSTRAINT_NAME* of a table it refers to (note, foreign key must refer to a primary key). By this relationship, we can create a hierarchical query to find out the referenced tables of a given table.

~~~sql
  SELECT CONSTRAINT_NAME,
         TABLE_NAME,
         CONSTRAINT_TYPE,
         R_CONSTRAINT_NAME,
         LEVEL (LVL)
  FROM ALL_CONSTRAINTS
  START WITH TABLE_NAME = 'szTableName' AND CONSTRAINT_TYPE = 'P'
  CONNECT BY PRIOR CONSTRAINT_NAME = R_CONSTRAINT_NAME
~~~

To further find out the corresponding primary and foreign key columns, we can use the above result to join with table ***ALL_IND_COLUMNS*** and ***ALL_CONS_COLUMNS***.

~~~sql
SELECT TREE.CONSTRAINT_NAME,
       TREE.TABLE_NAME,
       ACC.COLUMN_NAME,
       TREE.R_CONSTRAINT_NAME,
       AIC.TABLE_NAME,
       AIC.COLUMN_NAME,
       TREE.LVL
FROM
      ALL_IND_COLUMNS AIC,
      ALL_CONS_COLUMNS ACC,
      (
      SELECT CONSTRAINT_NAME, TABLE_NAME, CONSTRAINT_TYPE, R_CONSTRAINT_NAME, LEVEL LVL
      FROM ALL_CONSTRAINTS
      START WITH TABLE_NAME = 'szTableName' AND CONSTRAINT_TYPE = 'P'
      CONNECT BY PRIOR CONSTRAINT_NAME = R_CONSTRAINT_NAME
      )Tree
WHERE ACC.CONSTRAINT_NAME = TREE.CONSTRAINT_NAME
AND   AIC.INDEX_NAME = TREE.R_CONSTRAINT_NAME
AND   AIC.COLUMN_POSITION = ACC.POSITION
ORDER BY TREE.TABLE_NAME, TREE.LVL;
~~~

As shown, the hierarchical query result is nested as a Tree table for join operation. This query provides a better performance compared with another solution mentioned in the the following section.

Another solution for retrieving referenced tables of a given table.

~~~sql
  SELECT	AC.Table_Name,		--Source
  	      ACC.Column_Name,	--Source
  	      AIC.Table_Name,		--Ref
  	      AIC.Column_Name,	--Ref
  	      AC.Constraint_Name,	--Source
  	      AC.R_Constraint_Name	--Ref
  FROM	  All_Constraints AC,
        	All_Cons_Columns ACC,
        	All_Ind_Columns AIC,
        	(
        	SELECT	Constraint_Name
        	FROM	All_Constraints
        	WHERE	Table_Name = UPPER(szTableName)
        	AND	Constraint_Type = 'P'
        	) TMP
  WHERE	AC.Constraint_Name = ACC.Constraint_Name
  AND	  AC.Table_Name = ACC.Table_Name
  AND	  AC.R_Constraint_Name = AIC.Index_Name
  AND	  AC.R_Constraint_Name = TMP.Constraint_Name
  AND	  AC.Constraint_Type = 'R'
  AND	  ACC.Position = AIC.Column_Position
  ORDER BY 1, 2, 3, 4, 5, 6;
~~~

But these two solutions are buggy if there is **self referential constraint** among those tables. Therefore, make sure there is no tables has foreign keys refer to itself or using ***"ON DELETE CASCADE"*** to avoid the problem.


***Reference***

> <https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_1037.htm#i1576022>
