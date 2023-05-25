---
id: u21vypufllwwkas2wd2bjin
title: Env
desc: ''
updated: 1683101274845
created: 1683101212632
---
Oracle uses **SERVEROUTPUT** to control the printing of script's output.

Always enable **SERVEROUTPUT** env variable before you do a output.
```pl/sql
SET SERVEROUTPUT ON
```

Now you can print script output using the following function:
```pl/sql
dbms_output.put_line('Hello!');
```