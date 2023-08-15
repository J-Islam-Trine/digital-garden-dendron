---
id: bd8pypqo4pxiczdzvno4330
title: Card Security Code(CVV, CVV2, iCVV)
desc: ''
updated: 1692098541981
created: 1692095675071
---
### Card Security Code কী?
**Card security Code** হচ্ছে একটা সিকিউরিটি কোড যা কার্ডে লেখা থাকে। ভিসা, মাস্টারকার্ড সহ বিভিন্ন ইস্যুয়ার বিভিন্ন ধরণের **CSC** ব্যবহার করে থাকে।

### বিভিন্ন ইস্যুয়ারদের CSC
| Acronym | Definition | Issuer |
| -- | -- | -- | -- | 
| CID | Card Identification Number | Discover
| CVC2 | Card Validation Code | Mastercard
| CSC | Card Security Code | Debit Card
| CVV2 | Card Verification Value 2 | Visa

### কেন **CSC** ব্যবহার করা হয়?
**card not present(CNP)**এর ক্ষেত্রে সিকিউরিটির জন্য **CSC** ব্যবহার করা হয়।

### **CSC**এর ধরণগুলো 
#### **CVV1**
* **CVV1** কার্ডের ম্যাগস্ট্রিপে থাকে।
* **CVV1** হচ্ছে স্ট্যাটিক, যেহেতু এটা ম্যাগস্ট্রিপে দেয়া থাকে। সেজন্য কার্ড ক্লোন করা হলে সেই কার্ডটাতেও **CVV1** কাজ করবে। 
* এই কোডটি কাজে লাগে যদি কখনো **POS**-এ ম্যাগস্ট্রিপ সোয়াইপ করে ট্রানজ্যাকশন করতে হয়।
* PAN নাম্বার, 4-ডিজিটের Expiration Date, 3-ডিজিটের সার্ভিস কোড আর একজোড়া DES ব্যবহার করে **CVV** জেনারেট করা হয়।

#### **CVV2**
* **VISA** **CVV2** ব্যবহার করে থাকে।
* **CVV2** মূলতঃ **CNP** ট্রানজ্যাকশনের জন্যই ব্যবহার করা হয়। 


#### **iCVV**
* **iCVV** মূলতঃ **contactless** আর **chip EMV**-তে ব্যবহার করা হয়। 


### Read More
* [IBM Doc](https://www.ibm.com/docs/en/linux-on-systems?topic=services-how-visa-card-verification-values-are-used)
* [VISA Dynamic CVV](https://developer.visa.com/capabilities/visa-dcvv2-generate)


