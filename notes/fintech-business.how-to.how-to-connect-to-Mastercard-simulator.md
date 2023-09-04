---
id: 7sur2gtka37e1f9pj2i17ui
title: How To Connect To Mastercard Simulator
desc: ''
updated: 1693817648765
created: 1693810357513
---
### Setting Up A Project
1. Open Mastercard sim.
2. Open/create a new project(**new projects are saved in XLS format**).


### Creating New Crypto Keys
3. Go to **simulation data** section and find **crypto keys**.  You have some examples there for example, **PIN Key set double length for ZICB**.
4. Create a new one by right-click on ** examples**.
5. Set both **key 1 & key 2** for **ZMK_double & ZPK-double**.
6. Set this one as the **default keys**.

### Adding New Issuer
2. Set any other issuer data as default.
2. Check one of the example and create a new Issuer Data from it.
3. You put respective values in, (values such as **STAN**, **CountryCode**, **BankNetRef**, appropriate **PIN block format**).
    * For **STAN**, any value(Numeric, 6) will do.
    * You can skip **Concentrator ID**.
    * **PIN key set** is the name for key pair from previous step as **PINKeySetRef**.
4. Set this as default if everything is okay.

### Adding new card
2. Either create or reuse cards from **CardList** section for **Sim Data**.
3. You'll find the datails about the card from **Card Production Data(Embossing file + DEC file)** file you receive earlier.
4. Copy the **line for the card** from **DEC** file and put it inside the splitter file.
5. The **splitter** will split the line into seperate values.
6. Put the individual data in the **card details** in simulator.
    * You have to find the **PIN** using **Crypto Cal**/**DES/MDS Cal** tool.
    > ### Calculating PIN From clear PIN BLOCK
    > Tool - [PIN Extractor](https://paymentcardtools.com/pin-extract/pin-from-pinblock)
    > 1. Put the **clear PIN Block** in PIN BLOCK field.
    > 2. Put the**PAN** in PAN field.
    > 3. You can skip other fields.
    > 4. That's all!
    * Put any date before today as **effectiveDate**.
4. Make this card **default** one.


**Question**
* What does Load Balance file do?
* What are we doing in step 6?
* Do we need to setup keys when creating cards only?
* Does setting up one time work for others time?
* What is BankNet ref?


