---
id: cy1fk0ap0btrz7x7mekivcz
title: Exercises
desc: ''
updated: 1683105442070
created: 1683100597159
---

> Always enable **SERVEROUTPUT** env variable before you do a output.
> SET SERVEROUTPUT ON
> Now you can print script output using the following function:
> dbms_output.put_line('Hello!');

1. write a plsql block to find odd/even number.
```pl/sql
SET SERVEROUTPUT ON //trun on the server output

CREATE OR REPLACE 
FUNCTION oddEvenChecker
IS
BEGIN
FOR i in 1..10 LOOP
IF MOD(i,2) = 0 THEN
dbms_ouput.put_line(i || ' is an even number.')
ELSE dbms_ouput.put_line(i || ' is an odd number.')
END LOOP;
END;
```

2. Write a plsql block to print employee full name, salary. department's full salary.
```pl/sql
CREATE OR REPLACE
PROCEDURE employeeDetailCheckP 
IS
emp_first_name employees.first_name%TYPE;
emp_last_name employees.last_name%TYPE;
emp_id employees.employee_id%TYPE;
emp_salary employees.salary%TYPE;
emp_dept_id employees.department_id%TYPE;
emp_dept_salary_sum employees.salary%TYPE;
BEGIN
select employee_id, first_name, last_name, salary, department_id 
INTO emp_id, emp_first_name, emp_last_name, emp_salary, emp_dept_id
FROM employees
where employee_id = 104;
select sum(salary)
INTO 
emp_dept_salary_sum
FROM employees
where department_id = emp_dept_id
group by department_id;
dbms_output.put_line('The name of employee with ' || emp_id || ' is ' || emp_first_name || ' ' || emp_last_name);
dbms_output.put_line('The total salary for his/her dept is ' || emp_dept_salary_sum);
END;

BEGIN
employeeDetailCheckP;
END;

```
