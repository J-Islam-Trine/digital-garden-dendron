---
id: b2ws5bluuzq2i6h4v56gna5
title: Functions
desc: ''
updated: 1683099897684
created: 1683099085743
---
Functions in PL/SQL are same old function from any other language.

Here's how a function is structued:
```pl/sql
CREATE OR REPLACE 
FUNCTION function_name
(parameter [IN] [OUT] data_type,
...,
...,
...) 
IS
//declare vars or cursor here
BEGIN
//add your code here
RETURN variable_name
END;
```