---
id: 7tyin4pftl24s11e7v94zhd
title: architecture
desc: ''
updated: 1685016952135
created: 1685014888699
---
Dynamic SQL লেখার দুইটা ওয়ে আছে - 
* Placeholder ব্যবহার করা (: দিয়ে, recommended, safe),
* || দিয়ে concat করে করে (সহজ, কিন্তু SQL injection এর রিস্ক আছে)। 

|| দিয়ে concat করার সময় dbms_assert প্যাকেজ ইউজ করাটাই সেইফ।


### Architecture
Dynamic SQLএর চারটা স্টেপ থাকেঃ
- Parse
– Bind
– Execute
– Fetch

## Tools
সাধারণতঃ দুইটা টুল ইউজ করে Dynamic SQL লেখা যায় 
* NDS (execute immediate যেটা থেকে আসে)
* dbms_sql

### Read Later
1. [DBMS_ASSERT](https://www.dba-oracle.com/t_dbms_assert.htm)

