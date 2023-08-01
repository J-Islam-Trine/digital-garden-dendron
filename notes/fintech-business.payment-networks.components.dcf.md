---
id: gs1w1d1pi60m3edwlmbbbw0
title: Digital Card Facilitator(DCF)
desc: ''
updated: 1690880037008
created: 1690877547428
---
### DCF কী?
DCF-টা হচ্ছে এমন একটা সার্ভিস যেটা **payment network**গুলো দিয়ে থাকে। প্রত্যেকটা নেটওয়ার্কেরই আলাদা আলাদা **DCF** থাকে।

### DCF যেভাবে কাজ করে
![DCF](./assets/images/dcf.png)

* **DCF** কোন কার্ড সিলেক্ট করা হচ্ছে সেটার স্ক্রিনটা দেখায়। 
* যদি আগে থেকে **DCF**-এ কার্ড এড করা থাকে তাহলে শুধু মাত্র কোড ভেরিফিকেশন করলেই সবগুলো কার্ডের লিস্ট দেখাবে। 

![DCF screen-1](./assets/images/dcf-screen-added.png){max-width: 40vw} 
![DCF screen-2](./assets/images/src-card_list.png){max-width: 40vw} 

* যদি আগে থেকে কার্ড এড করা না থাকে তাহলে কার্ড দিয়ে ট্রানজ্যাকশন করার সময় **DCF**-এ এড করার অপশান দেখাবে।
![DCF screen-add-card](./assets/images/dcf-add-card-screen.png){max-width: 50vw} 