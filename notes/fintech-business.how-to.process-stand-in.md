---
id: gjexrpvcudbtuv8sy01k6ae
title: Process Stand In
desc: ''
updated: 1693816623839
created: 1693816540119
---
### Scenario
* Core bank is down - 
    * Do Stand-IN
* Core bank is taking too long to response 
    * Reverse the transaction
    * Do Stand-IN.

### Steps
1. Log IN to XXXX Bank DB.
2. Find the **load_balance** file from **pcard_task**.
---
1. Go to WINSCP preprod for the same bank. 
2. From the pre-prod folder find the **standin.dat** file.
---
1. Go to SSH  
2. Run **load_balance_file.sh**.
3. Print the trace using the command load_balance_file.sh balance.trc
4. You can find the trace from **pcard_trace**.
---
5. Check if the stand-in file has been process by selecting all from **pcrd_file_processing** table.
6. Check the **balance** table.
