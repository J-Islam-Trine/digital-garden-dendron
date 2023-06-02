---
id: 3vyl7w8vk1u1sb8bmonerdf
title: Structures
desc: ''
updated: 1685424890240
created: 1685364555893
---
## Trigger-এর জেনারেল স্ট্রাকচার
একটা ট্রিগারের সাধারণত চারটা পার্ট থাকেঃ

* নাম, 
* স্টেটমেন্ট,
* শর্ত, 
* একশন।

নাম থেকে কন্ডিশন পর্যন্ত হচ্ছে ডিক্লারেশনের অংশ। আর একশন থেকে আসে ট্রিগারের বডি। সাধারণতঃ ট্রিগারের বডি এননিমাস ব্লক হিসাবে আলাদা ভাবে স্টোরেজ করা থাকে।


### Elaborate Structure of a Trigger
```sql
CREATE [OR REPLACE] [EDITIONABLE | NONEDITIONABLE] TRIGGER trigger_name
{BEFORE | AFTER}
{INSERT | UPDATE | UPDATE OF column [, column [, ... ]] | DELETE}
ON table_name
[FOR EACH ROW]
[REFERENCING {old | new} [ROW] AS something_else
[{old | new} [ROW] AS something_else]]
[ENABLE | DISABLE]
[FOLLOWS table_name]
[WHEN (logical_expression)]
[DECLARE]
[PRAGMA AUTONOMOUS_TRANSACTION;]
declaration_statements
BEGIN
execution_statements
END [trigger_name];
/

```

### ধাপে ধাপে ট্রিগার লেখা
* প্রথমে থাকবে ট্রিগারের নাম।
```sql
create or replace trigger trigger_name
```

* এরপর আমাদেরকে টাইমিং ইভেন্টটা স্পেসিফাই করতে হবে।
```sql
create or replace trigger trigger_name
BEFORE -- rest goes here
```

* এরপর আমরা স্পেসিফাই করবো ট্রিগার কন্ডিশন। এটা সাধারণত দুইটা জিনিস নিয়ে বিল্ড হয় - ১) statement আর ২) যে object-এর চেঞ্জের উপর লক্ষ রাখতে হবে সেইটা। 

```sql
create or replace trigger trigger_name
BEFORE 
INSERT 
ON employees
```
* তারপর যাবে আমাদের ট্রিগারের বডি যেটাতে আমরা ট্রিগারটা এক্সিকিউট করবো।
```sql
create or replace trigger trigger_name
BEFORE 
INSERT 
ON employees
BEGIN
-- put your statments here
END;
---

### A simple Example
```sql
CREATE OR REPLACE TRIGGER student_bi
BEFORE INSERT ON STUDENT
FOR EACH ROW
BEGIN
:NEW.student_id := STUDENT_ID_SEQ.NEXTVAL;
:NEW.created_by := USER;
:NEW.created_date := SYSDATE;
:NEW.modified_by := USER;
:NEW.modified_date := SYSDATE;
END;
```



### Read Later 
* Editionables and noneditionables
* Trigger Invalidation
* Follows