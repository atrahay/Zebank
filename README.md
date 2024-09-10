# Neobank retention analysis
## Group projet - Bootcamp Data Analytics - Le Wagon #Batch1671

For this project we receive data from a neobank. We have decided to rename the projet Zebank.
We wanted to collaborate with DBT. After spending two days trying to solve a problem on the website, we had to try different way. We worked all on the same new clean view and then table on GCP to collaborate.
Here we would like to present the code we used on Google BigQuery to conduct our analysis and solve the problem: 
### How to Increase User Engagement?

=> Let's have a look on the data we had at the beginning :
- 4 tables : devices, users, notifications, transactions
- january 2018 to april 2019
- 18K of users
- 2,4M of transactions
- 155M$ of transactions
- 191K$ bank fees

<img src="./lucidchart.png" alt="Description de l'image" height="300"/>

![lucidchart](https://github.com/user-attachments/assets/32a9cd8e-5521-4d27-b3a8-9fbf5e41e31e)


=> First, we have cleaned the data with SQL in Google BigQuery

- devices table : Supprimer la ligne device= “brand”
```sql
select
distinct( brand) as brand,
string_field_1 as user_id
FROM `neobank-425308.neo_bank.devices`
WHERE brand<> "brand"
```

- notifications table :
```sql
SELECT 
reason,
channel,
status,
CAST(regexp_replace (user_id,"user_","" ) AS INT64) as user_id,
created_date,
extract (year from created_date) as year,
extract (month from created_date) as month,
extract (day from created_date) as day,
 FROM `data-analytics-bootcamp-363212.neo_bank.notifications`
```

- 3 plans : standard, premium, metal
- 4 generations : Gen Z, Millenial, Gen x, Baby boomer

Some quick analysis before to begin : 
 - crypto
 - cashback
 - devices
 - transactions
   
=> We have make a churn analysis with this cleaned data

### 1) Churn analysis
 > See Aramis's branch to explore code about churn : (https://github.com/atrahay/Zebank/tree/Aramis)

 > See Maria's branch to explore code about cohort : (https://github.com/atrahay/Zebank/tree/Maria)

### 2) Notification analysis
   
### 3) Potential upgraders turnover
