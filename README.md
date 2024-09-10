# NEOBANK

Group project bootcamp data analytics Le Wagon

# COHORT ANALYSIS

Here is the code to see the cohorts per months and determine if any patterns arise:

```sql
WITH a AS(
SELECT 
t.user_id,
DATE(u.created_date) AS creation_date,
u.year AS creation_year,
u.month AS creation_month,
t.transaction_id,
DATE(t.created_date) AS trans_date, ##cohort date,
t.year AS transaction_year,
t.month AS transaction_month

FROM `neobank-425308.neo_bank_clean.transaction_clean_vf` AS t
inner join `neobank-425308.neo_bank_clean.users_clean` AS u
USING (user_id)
order by user_id, transaction_id
),
b AS (
SELECT
a.*,
a.transaction_year -creation_year AS year_diff,
transaction_month-creation_month AS month_diff
FROM a
),


c AS (
SELECT
b.*,
(year_diff * 12) + month_diff + 1 as cohort_index
from b
order by transaction_year,transaction_month
)

SELECT
*
FROM(
  SELECT DISTINCT
    user_id,
    trans_date,
    cohort_index
 FROM `neobank-425308.neo_bank_clean.cohorts_retention` 
)tbl 
PIVOT(
COUNT (user_id)
FOR cohort_index IN (1,2,3,4,5,6,7,8,9,10,11,12,13,14)
) AS Pivot_table
order by cohort_index
```

We then visualised it in a tab on Looker to draw recommendations from the behavioral patterns illustrated.
