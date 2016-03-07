---
layout: post
title:  "Find tables that reference a particular column in a table "
date:   2015-11-30 15:53:00
categories: Oracle
---

Scenario: *An requirement states "to delete the data from a table which is referenced by different tables".*

Well, to delete records in Oracle Database is believed to be an simple job by the following command *"Delete FROM table_name"*. But the prerequisite is that the table should not be referenced by any other tables otherwise it may cause the "foreign key violation" while trying to delete the table.

Before that, we should have an explicit view on how the reference constraints is maintained in Oracle. The key system table is ***ALL_CONSTRAINTS***.

>**ALL_CONSTRAINTS** describes constraint definitions on tables accessible to the current user.

|   Column           |         Descriptions         |
|:-------------------|:-----------------------------|
|  CONSTRAINT_TYPE   | <ul><li>C (check constraint on a table) </li><li>P (primary key)</li><li>U (unique key) </li><li>* R (referential integrity)</li><li>V (with check option, on a view)</li><li>O (with read only, on a view)</li></ul>|
|  R_CONSTRAINT_NAME | Name of the unique constraint definition for referenced table|
|  CONSTRAINT_NAME   | Name of the constraint definition                            |

There is a hierarchical structure in this table that *R_CONSTRAINT_NAME* indicating the foreign key constraint of the table would be the *CONSTRAINT_NAME* of a table it refers to (note, foreign key must refer to a primary key). By this relationship, we can create a hierarchical query to find out the table name, primary constraint name and its referenced tables name and the foreign key constraint name.

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
SELECT AIC.TABLE_NAME,
       AIC.COLUMN_NAME,
       ACC.COLUMN_NAME,
       TREE.TABLE_NAME,
       TREE.CONSTRAINT_NAME,
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
AND   AIC.INDEX_NAME = TREE.R_CONSTRAINT_NAME;
~~~

As shown, the hierarchical query result is nested as a Tree table for join operation.

I find another slow solution from the other, it's easy to understand but with less efficiency as too many join operations to perform for getting the result.

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
