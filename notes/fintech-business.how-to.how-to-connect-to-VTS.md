---
id: 5ipagftzt6wdubxzp4ipts9
title: How-to-Connect-to VTS
desc: ''
updated: 1693899124127
created: 1693803531053
---
### VTS-এ লগইন করা
1. **VTS**-এর জন্য আগে থেকে RD(remote desktop) এসাইন করা আছে। সেগুলোতেই শুধু ঢুকতে হবে। **চাপ বেশি থাকলে আগে থেকে বুক করে রাখতে হতে পারে**। (**VISA certification-এর জন্য আলাদা RD আছে)।
2. **VTS**-এ ঢুকতে হবে আমাদেরকে এসাইন করা **RD** থেকে। অর্থাৎ RD থেকে RD-তে।

### শুরুতে যা করতে হবে
1. টেস্ট ফাইল লোড করতে হবে।
2. কার্ড সেটআপ করতে হবে।
2. VTS-এর সঙ্গে Database Server কানেক্ট করতে হবে।
3. **VTS**-এর **Line Up** করতে হবে।

**টেস্ট ফাইল লোড করা**  
 * প্রথমে **test case**-গুলো লোড করতে হবে। 
 * **program files**-এর **VTS** ফোল্ডার থেকে **Transaction** অনুযায়ী ফোল্ডার থেকে **Test case(.stf ফাইল)** লোড করবো।
 * Test Case না থাকলে নতুন করে বানিয়ে নিতে হবে।

  > **যেভাবে নতুন টেস্ট কেস বানাতে হবে**
  > 
  > 1. **Insert New Case**(Menu bard থেকে) সিলেক্ট করতে হবে।
  > 2. ডান পাশের **sample test case** উইন্ডো থেকে মানানসই test case পিক করতে হবে।
  > 3. সব ক্ষেত্রেই **General Services - Sign On - Incoming Request** থেকে **sign on** মেসেজ পিক করতে হবে।
  > 


2. **কার্ড সেটআপ**


 **VTS-এর সঙ্গে Database Server কানেক্ট করা**
1. যে ব্যাংকের জন্য টেস্ট, সেটার **SSH Server** আর **DB Server**-এ যেতে হবে। 
3. **DB Server** - এ গিয়ে **VTS**-এর **IP**-টা রেজিস্টার করতে হবে। 
    > **DB Server** অংশের সেটআপ
    > 1. **VTS**এর **TCP/IP Dialog**-এ গিয়ে **IP** আর **Port** দেখে নিতে হবে।
    > 1. নিচের query রান করতে হবে - 
        ```sql
        select * from resources;
        ```
    > 2. এরপর যতগুলো সার্ভিস আমার লাগবে সেগুলোর **PRIS_REMOTE_APPLICATION_NAME**-ফিল্ডে **VTS**-এর **IP** আর **PRIS_APPLICATION_ID** ফিল্ডে **VTS**-এর **Port** বসাতে হবে।
    > 3. এরপর কমিট করতে হবে।

2. এবারে **SSH Server** থেকে প্রয়োজনীয় কিছু সার্ভিস up করতে হবে(কিছু কিছু ক্ষেত্রে **down দিয়ে তারপর আবার up করতে হবে**)।
    * প্রথমে `lui` কমান্ড দিয়ে কোন সার্ভিস আপ আছে সেগুলো দেখে নিতে হবে।
    * যদি প্রয়োজনীয় সার্ভিস আপ না থাকে তাহলে সেটা আপ করতে হবে।

    > **যেভাবে সার্ভিস আপ/ডাউন করতে হবে**
    > 1. প্রথমে **DB Server**-এ ঢুকতে হবে।
    > 2. নিচের query রান করতে হবে - 
        ```sql
        select * from resources;
        ```
    > 3. এটা যে সব সার্ভিস আপ/ডাউন করতে হবে(যেমন - sms সার্ভিস), সেটার **PRIS_START_SCRIPT_NAME** আর **PRIS_STOP_SCRIPT_NAME** কলাম থেকে সার্ভিস আপ/সার্ভিস ডাউন করার কমান্ড দেখে নিতে হবে।
    > 4. এবারে **SSH server**-এ গিয়ে সেই কমান্ড রান করতে হবে।
    > 

3. **VTS** এর লাইন আপ করতে হবে।
    * Menu bar-থেকে **Start Communication** এ যেতে হবে।
    * এটার মেনু থেকে **start line**-এ যেতে হবে।

4. এবারে **network sign on** করতে হবে। 
    * test case থেকে **sign on case** execute করবো।
    * আগে থেকে **SSH server** চালু করে সার্ভিসটা(**ATM** হলে **sms service**, **অন্য যে কোন কিছু** হলে **BASEI**) ডাউন দিয়ে রাখতে হবে।
    * **sign on test** ফায়ার করার সর্বোচ্চ পাঁচ সেকন্ডের মধ্যে **SSH server** থেকে ঐ সার্ভিসটা আপ দিতে হবে।
    * **sign on** সফল হলে ডান দিকের নিচের কোণায় **LINE** লেখার পাশে সবুজ চিহ্ন দেখাবে।

5. এরপর যে কোন **test case** পারফর্ম করা যাবে।
6. Transaction-এর ট্রেস চেক করতে হবে।

    > **কীভাবে যে কোন transaction-এর **Trace** চেক করা যাবে**
    > 1. প্রথমে **SSH Server**-এ যেতে হবে।
    > 2. সেখান থেকে **Trace**-এর ফোল্ডারে যেতে হবে। এজন্যে `cd $TRACE` রান দেয়া যেতে পারে।
    > 3. এবারে `ls -lart` কমান্ড রান দিলে সবগুলো **Trace** লিস্ট হবে।
    > 4. এখান থেকে `grep` কমান্ড রান করে আমরা সার্চ দিতে পারি। 
    > 5. এরপর আমরা চাইলে পুরো trace বা সেটার অংশ কপি করতে পারি, সেইটার 3টা ওয়ে আছে - 
    >       * আমরা চাইলে vi দিয়ে কপি করতে পারি, তবে সেটা একটু কঠিন।
    >       * আংশিক লাগলে আমরা লাইন সিলেক্ট করে **vi** দিয়ে **yank** করতে পারি।
    >       * যে trace ফাইল লাগবে সেটার নাম দেখে নিয়ে **WinSCP** দিয়ে ঢুকে আমরা সেই ফাইলটা পুরোটা নিয়ে নিতে পারি।
### Extras

> **Issue**: Connection Refused in **Start Communication**
> Change the **current port** to something else.



---
### Questions
> ****SSH** থেকে কোন কোন সার্ভিস আপ দিতে হবে সেটা কীভাবে বুঝা যাবে?**
> 
> **resource_services** টেবিলটা দেখা যেতে পারে। 
> 



