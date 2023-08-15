---
id: g2kxvtxs2jyvpeawosoqtp1
title: Draft Capture File(DCF GEN4 file)
desc: ''
updated: 1691134578864
created: 1691133256160
---
### DCF কী? 
মাস্টারকার্ড **MPGS**-এর মাধ্যমে ট্রানজ্যাকশন করা সব মেম্বারের জন্য প্রতিদিন কমপক্ষে একটা করে **Draft Capture File** জেনারেট করে। এই ফাইলটাতে যত **Acquirer** কাছ থেকে ট্রানজ্যাকশন আসে সবগুলো একটার পর একটা দেয়া থাকে। 

### DCF GEN4 ফাইলের লেআউট
```
Header for acquirer 1
    Detail Record for acquirer 1
Trailer for acquirer 1 record
Header for acquirer 2
    Detail Record for acquirer 2
Trailer for acquirer 2 record
...
...
Header for acquirer n
    Detail Record for acquirer n
Trailer for acquirer n record
```

