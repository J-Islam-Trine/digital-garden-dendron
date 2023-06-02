---
id: g5qhd6u8caeknj6pyz0bv14
title: What and Why
desc: ''
updated: 1685446547396
created: 1685441618658
---
## Auditing In Oracle
Oracle-এ Audit-এর মাধ্যমে নিচের কাজগুলা করা যায় - 
* ডেটাবেজ-এর নানান একটিভিটি মনিটর
* ডেটাবেজ একটিভিটি রেকর্ড।

## Unified Audit Trail
Oracle 12C থেকে অডিটের সব ডেটা আলাদা ভাবে **audsys** schema-এর **AUD$UNIFIED** টেবিলে থাকে।  **এখানে statement-টা CLOB আকারে থাকে।**

