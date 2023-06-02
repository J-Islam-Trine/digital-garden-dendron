---
id: ltiukoj8fw6x177etpyp1ri
title: Sequence
desc: ''
updated: 1685611434762
created: 1685611309594
---
**Oracle ডাটাবেজ Auto-increment সাপোর্ট করে না।** তবে একই কাজটা আমরা **Identity Column**** দিয়েও করতে পারি।


### উদাহরণ
```sql
create table t1 (
    c1 NUMBER GENERATED ALWAYS as IDENTITY(START with 1 INCREMENT by 1),
    c2 VARCHAR2(10)
    );
```