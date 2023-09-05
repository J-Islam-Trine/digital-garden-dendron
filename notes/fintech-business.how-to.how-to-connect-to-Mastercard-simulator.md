---
id: 7sur2gtka37e1f9pj2i17ui
title: How To Connect To Mastercard Simulator & Run Tests
desc: ''
updated: 1693892959989
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

### Performing the actual test
1. Pick the card from **Models(Bottom right section)** and pick the created card.
2. Click on **link card to this message**.
3. Change the **RRN** to ..... .
4. Log in to **XXXX Bank**SSH server.
5. List all services with `lui` command.
6. Stop the intended servers.
7. Log into Oracle server.
8. Get the card details from **card** table.
9. Get the details from accounts_link table.






---
**Question**
* Do we need to setup keys when creating cards only?
* What is BankNet ref?
* Do we need to fill up **EMV/other params**?


