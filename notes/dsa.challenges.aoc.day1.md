---
id: 0jjgq32vpkvq1aqqo6mgwdu
title: Day1
desc: ''
updated: 1670668615956
created: 1670668580022
---
### Problem
> --- Day 1: Calorie Counting ---
> Santa's reindeer typically eat regular reindeer food, but they need a lot of magical energy to deliver presents on Christmas. For that, their favorite snack is a special type of star fruit that only grows deep in the jungle. The Elves have brought you on their annual expedition to the grove where the fruit grows.
> 
> The jungle must be too overgrown and difficult to navigate in vehicles or access from the air; the Elves' expedition traditionally goes on foot. As your boats approach land, the Elves begin taking inventory of their supplies. One important consideration is food - in particular, the number of Calories each Elf is carrying (your puzzle input).
> 
> The Elves take turns writing down the number of Calories contained by the various meals, snacks, rations, etc. that they've brought with them, one item per line. Each Elf separates their own inventory from the previous Elf's inventory (if any) by a blank line.
> 
> For example, suppose the Elves finish writing their items' Calories and end up with the following list:
> 
> 1000  
> 2000  
> 3000
> 
> 4000
> 
> 5000  
> 6000
> 
> 7000  
> 8000  
> 9000  
>  
> 10000  
> This list represents the Calories of the food carried by five Elves:
> The first Elf is carrying food with 1000, 2000, and 3000 Calories, a total of 6000 Calories.
> The second Elf is carrying one food item with 4000 Calories.
> The third Elf is carrying food with 5000 and 6000 Calories, a total of 11000 Calories.
> The fourth Elf is carrying food with 7000, 8000, and 9000 Calories, a total of 24000 Calories.
> The fifth Elf is carrying one food item with 10000 Calories.
> In case the Elves get hungry and need extra snacks, they need to know which Elf to ask: they'd like to know how many Calories are being carried by the Elf carrying the most Calories. In the example above, this is 24000 (carried by the fourth Elf).
> 
> Find the Elf carrying the most Calories. How many total Calories is that Elf carrying?
---
### Pseudocode
* গ্রুপগুলোতে থাকা সংখ্যাগুলোর যোগফল রাখার জন্য দুইটা ভ্যারিয়েবল নিলাম।
* এবারে i=0 থেকে শুরু করে সংখ্যাগুলোতে ট্র্যাভার্স করলাম।
* যদি i পজিশনে সংখ্যা থাকেঃ তাহলে সেটাকে টেম্পোরারি যোগফলের ভ্যারিয়েবলের সাথে যোগ করলাম।
* যদি i পজিশন ফাঁকা থাকেঃ তাহলে টেম্পোরারি যোগফলের ভ্যারিয়েবলে থাকা ভ্যালুটাকে স্থায়ী ভ্যারিয়বেলে দিয়ে দিলাম আর টেম্পোরারি ভ্যারিয়েবল রিসেট করলাম।
* এবারে ফাঁকা ভ্যালুর পরের পজিশন থেকে আবারও কন্টিনিউ করলাম।