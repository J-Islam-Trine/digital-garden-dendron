---
id: ctiw4roaft273epexqn9fl6
title: SQL Exercises
desc: ''
updated: 1678775200844
created: 1678775200844
---
1. Print a message in following format -
>In "SCHEMA_NAME" , TABLE "TABLE_NAME" , table status "ENABLE/DISABLE" number of rows "ROWS" last analyzed "LAST_ANALYZED_DATE_TIME" completed successfully.

```sql


if(dateBetween(end(prop("Deadline")), start(now()), "days") + 1 > 4 && dateBetween(end(prop("Deadline")), start(now()), "days") + 1 < 8, 5, if(dateBetween(end(prop("Deadline")), start(now()), "days") + 1 > 2 && dateBetween(end(prop("Deadline")), start(now()), "days") + 1 < 5 && dateBetween(end(prop("Deadline")), start(now()), "days") + 1 > 2, 3, if(dateBetween(end(prop("Deadline")), start(now()), "days") + 1 < 3, 1, 0))) 

 