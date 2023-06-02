---
id: tajxkvdaqqjmhl7pmruphsb
title: CLOB-for-large-string
desc: ''
updated: 1685521760295
created: 1685447336371
---
### CLOB
Oracle-এ বড় আকারের স্ট্রিং রাখতে সাধারণতঃ **CLOB** ইউজ করা হয়। 

### How To Parse CLOB Data
```sql
dbms_lob.substr(clob_column, 4000);
```

**এই পদ্ধতির একটা লিমিটেশন হলো যে, substr দিয়ে সর্বোচ্চ 32 KB পর্যন্ত সাইজের স্ট্রিং পার্স করা যায়।**

