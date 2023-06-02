---
id: na8p0ey547r4fr149thdlwr
title: Transaction Capture Process
desc: ''
updated: 1685526497712
created: 1685525988431
---


* ট্রানজ্যাকশনগুলো **controlled** হয়ে **Trans_hist** টেবিলে যায়।
* **Trans_hist** টেবিলে ঢুকার সময় **Origin code** টেবিলে ট্রানজ্যাকশনের ধরণ রেজিস্টার করা হয়। Possible values are:
* ‘0’ = ONUS card / local merchant,
* ‘1’ = ONUS card / local other bank
merchant,
* ‘2’ = ONUS card / foreign merchant,
* ‘3’ = Local other bank card / local
merchant,
* ‘4’ = Foreign card / local merchant

* **Trans_hist** টেবিল থেকে **IPM_Outgoing** টেবিলে যায়। 