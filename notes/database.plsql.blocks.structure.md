---
id: mtsxl1hqgoonq3bfsk6vus3
title: Structure
desc: ''
updated: 1683098486906
created: 1683097410857
---

Generally a pl/sql block has two part:
* Declaration
* Body


### Declaration
It contains **cursors** and **variables**.

It has the following structure:
```pl/sql
DECLARE
//place variable and cursors here.
```

### Body 
The body contains the **SQL** and **PL/SQL** statements. This is the only **mandatory** part of a block 

It may **optionally, but most of the time** contain **exceptions**.

The body has following structure:
```pl/sql
BEGIN
//place your code here

//EXCEPTION
//optional, add if you want to.
END;
```
> #### The body block must end with a semicolon.

