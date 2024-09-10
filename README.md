# Neobank
Group project bootcamp data analytics Le Wagon

# CHURN ANALYSIS

In this Neo Bank we determined the period of churn to be 35 days without transactions.

```sql
WITH user_transactions AS (
    SELECT 
        user_id,
        created_date
    FROM 
        `neobank-425308.neo_bank_clean.transactions_clean`
),

-- Calculate the difference in days between each transaction and the next one for each user
transaction_gaps AS (
    SELECT
        user_id,
        created_date,
        LEAD(created_date) OVER (PARTITION BY user_id ORDER BY created_date) AS next_transaction_date,
        DATE_DIFF(LEAD(created_date) OVER (PARTITION BY user_id ORDER BY created_date), created_date, DAY) AS days_between
    FROM 
        user_transactions
),

-- Identify churn events
churn_events AS (
    SELECT
        user_id,
        created_date,
        next_transaction_date,
        days_between,
        DATE_TRUNC(created_date, MONTH) AS churn_month
    FROM
        transaction_gaps
    WHERE
        days_between >= 30 OR next_transaction_date IS NULL
)

-- Count the number of churners per month
SELECT
    churn_month,
    COUNT(DISTINCT user_id) AS churn_count
FROM
    churn_events
GROUP BY
    churn_month
ORDER BY
    churn_month
```


We then wanted to point out if our clients were active churners (latest perdiod) or churned before:

```sql
SELECT
user_id,
IF(SUM(IF(active_or_churner='churner',1,0))=0,'active','once_churner') as churner
FROM `neobank-425308.neo_bank_clean.info_churners_all_Q_`
GROUP BY user_id
ORDER BY user_id
```


Finally we wanted to know the right number of churners and active users for each quarter:
```sql
-- Je récupère toutes les infos de transactions + la user creation date
WITH all_infos AS (
SELECT
t.transaction_id,
t.transactions_type,
t.transactions_currency,
t.amount_usd,
t.transactions_state,
t.ea_cardholderpresence,
t.ea_merchant_mcc,
t.ea_merchant_city,
t.ea_merchant_country,
t.direction,
t.user_id,
t.created_date as t_created_date,
t.year,
t.month,
t.day,
t.generation,
t.plan_clean,
t.brand,
t.user_settings_crypto_unlocked,
u.created_date as u_created_date
FROM `neobank-425308.neo_bank_clean.transaction_clean_vf` as t
LEFT JOIN neobank-425308.neo_bank_clean.users_clean as u
USING (user_id)
),

-- Je filtre pour chaque Quarter pour n'avoir que les user créés jusqu'à la fin du Q ainsi que les transactions de cette période.
filters_Q1 AS (
SELECT *
FROM all_infos
WHERE (u_created_date BETWEEN '2018-01-01' AND '2018-03-31') AND (t_created_date BETWEEN '2018-01-01' AND '2018-03-31')
),

filters_Q2 AS (
SELECT *
FROM all_infos
WHERE (u_created_date BETWEEN '2018-01-01' AND '2018-06-30') AND (t_created_date BETWEEN '2018-01-01' AND '2018-06-30')
),

filters_Q3 AS (
SELECT *
FROM all_infos
WHERE (u_created_date BETWEEN '2018-01-01' AND '2018-09-30') AND (t_created_date BETWEEN '2018-01-01' AND '2018-09-30')
),

filters_Q4 AS (
SELECT *
FROM all_infos
WHERE (u_created_date BETWEEN '2018-01-01' AND '2018-12-31') AND (t_created_date BETWEEN '2018-01-01' AND '2018-12-31')
),

filters_Q5 AS (
SELECT *
FROM all_infos
WHERE (u_created_date BETWEEN '2018-01-01' AND '2019-03-31') AND (t_created_date BETWEEN '2018-01-01' AND '2019-03-31')
),

filters_Q6 AS (
SELECT *
FROM all_infos
WHERE (u_created_date BETWEEN '2018-01-01' AND '2019-05-16') AND (t_created_date BETWEEN '2018-01-01' AND '2019-05-16')
),

-- Je regarde si la dernière transaction du user est faite pendant la periode dite de churn (fin de Q - 35 jours), si oui alors il est actif, sinon il est churner.
Q1 AS (
SELECT
user_id,
MAX(t_created_date) AS last_transaction,
u_created_date,
CASE 
  WHEN MAX(t_created_date) BETWEEN '2018-02-24' AND '2018-03-31' THEN 'active'
  ELSE 'churner'
  END AS active_or_churner
FROM filters_Q1
GROUP BY user_id, u_created_date
),

Q2 AS (
SELECT
user_id,
MAX(t_created_date) AS last_transaction,
u_created_date,
CASE 
  WHEN MAX(t_created_date) BETWEEN '2018-05-27' AND '2018-06-30' THEN 'active'
  ELSE 'churner'
  END AS active_or_churner
FROM filters_Q2
GROUP BY user_id, u_created_date
),

Q3 AS (
SELECT
user_id,
MAX(t_created_date) AS last_transaction,
u_created_date,
CASE 
  WHEN MAX(t_created_date) BETWEEN '2018-08-27' AND '2018-09-30' THEN 'active'
  ELSE 'churner'
  END AS active_or_churner
FROM filters_Q3
GROUP BY user_id, u_created_date
),

Q4 AS (
SELECT
user_id,
MAX(t_created_date) AS last_transaction,
u_created_date,
CASE 
  WHEN MAX(t_created_date) BETWEEN '2018-11-27' AND '2018-12-31' THEN 'active'
  ELSE 'churner'
  END AS active_or_churner
FROM filters_Q4
GROUP BY user_id, u_created_date
),

Q5 AS (
SELECT
user_id,
MAX(t_created_date) AS last_transaction,
u_created_date,
CASE 
  WHEN MAX(t_created_date) BETWEEN '2019-02-24' AND '2019-03-31' THEN 'active'
  ELSE 'churner'
  END AS active_or_churner
FROM filters_Q5
GROUP BY user_id, u_created_date
),

Q6 AS (
SELECT
user_id,
MAX(t_created_date) AS last_transaction,
u_created_date,
CASE 
  WHEN MAX(t_created_date) BETWEEN '2019-04-12' AND '2019-05-16' THEN 'active'
  ELSE 'churner'
  END AS active_or_churner
FROM filters_Q6
GROUP BY user_id, u_created_date
)

-- Je compte le nombre d'active users et de churners pour chaque Quarter.
SELECT
SUM(CASE WHEN active_or_churner = 'active' THEN 1 ELSE 0 END) AS nb_active_users_Q1,
SUM(CASE WHEN active_or_churner = 'churner' THEN 1 ELSE 0 END) AS nb_churners_Q1
FROM Q1
UNION ALL
SELECT
  SUM(CASE WHEN active_or_churner = 'active' THEN 1 ELSE 0 END) AS nb_active_users_Q2,
  SUM(CASE WHEN active_or_churner = 'churner' THEN 1 ELSE 0 END) AS nb_churners_Q2
  FROM Q2
UNION ALL
SELECT
  SUM(CASE WHEN active_or_churner = 'active' THEN 1 ELSE 0 END) AS nb_active_users_Q3,
  SUM(CASE WHEN active_or_churner = 'churner' THEN 1 ELSE 0 END) AS nb_churners_Q3
  FROM Q3
  UNION ALL
SELECT
  SUM(CASE WHEN active_or_churner = 'active' THEN 1 ELSE 0 END) AS nb_active_users_Q4,
  SUM(CASE WHEN active_or_churner = 'churner' THEN 1 ELSE 0 END) AS nb_churners_Q4
  FROM Q4
UNION ALL
SELECT
  SUM(CASE WHEN active_or_churner = 'active' THEN 1 ELSE 0 END) AS nb_active_users_Q5,
  SUM(CASE WHEN active_or_churner = 'churner' THEN 1 ELSE 0 END) AS nb_churners_Q5
  FROM Q5
  UNION ALL
SELECT
  SUM(CASE WHEN active_or_churner = 'active' THEN 1 ELSE 0 END) AS nb_active_users_Q6,
  SUM(CASE WHEN active_or_churner = 'churner' THEN 1 ELSE 0 END) AS nb_churners_Q6
  FROM Q6
```


# COHORT ANALYSIS


