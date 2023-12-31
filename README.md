# Case Study #3 Foodie Fi

*Note: All information and data related to the case study were obtained from [here](https://8weeksqlchallenge.com/case-study-3/).*
![Screenshot 2023-08-13 at 8 37 39 pm](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/96128d5d-ca8e-4509-8e10-4215e4108337)

## Business Task
Spotting a niche in the market, Danny launched "Foodie-Fi" in 2020, a streaming service dedicated solely to food-related content. Similar to Netflix, but exclusively for cooking enthusiasts, it offers monthly and annual subscriptions for unlimited access to unique food videos globally.
Danny's vision for Foodie-Fi is data-driven. He wants to make informed decisions about investments and feature additions based on data. He has provided the data design and descriptions for the Foodie-Fi database. Our objective is to focus on two specific tables and also craft a new table for the team. When exploring the data and addressing the study's questions, it's essential to reference the "foodie_fi" database schema in the SQL scripts.

## Entity Relationship Diagram
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/3800b43d-1dc0-4e8e-8d12-829dc5bbcc0f)

## General Insights
- *Customer Journey:* The queries provided a detailed onboarding journey of 10 sample customers, showcasing their plan subscriptions and respective start dates. This helps in understanding the customer's lifecycle and their interactions with the Foodie-Fi platform.
- *Subscription Analysis:* Insights were derived about the total number of unique customers of Foodie-Fi, the monthly distribution of trial plan start dates, and a breakdown of plan subscriptions after 2020. This provides a clear picture of the platform's growth and customer preferences.
- *Churn Analysis:* Detailed insights were provided into customer churn, both overall and post-free trial. This is crucial for understanding customer retention and identifying potential areas of improvement in the customer experience.
- *Plan Transition:* The analysis provided a snapshot of the plan distribution at the end of 2020, the number of customers who upgraded to an annual plan in 2020, and the average duration for customers to upgrade to an annual plan. This helps in understanding customer loyalty and preferences.
- *Payment Analysis:* A comprehensive payments table for 2020 was created, capturing payment dates, amounts, and the sequence of payments for each customer. This is essential for revenue tracking and financial analysis.

## Key SQL Syntax and Functions
- Joins (`INNER JOIN`)
- Aggregate Functions (`COUNT`, `SUM`, `ROUND`)
- Window Functions (`LAG()`, `LEAD()`, `ROW_NUMBER()`)
- Common Table Expressions (CTE)
- Conditional Logic (`CASE WHEN`)
- Date Functions (`DATE_TRUNC`, `DATE_PART`, `AGE`, `EXTRACT`)
- String Functions (`STRING_AGG`, `TO_CHAR`)
- Set Operation (`UNION ALL`)

## Questions and Solutions
### Part A: Customer Journey
> Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

There are two approaches here. The first is the debugging technique provided by the course, and the second is an alternative approach I found.
*First Approach*
```sql
SELECT
  customer_id,
  subscriptions.plan_id,
  plan_name,
  start_date
FROM foodie_fi.subscriptions
INNER JOIN foodie_fi.plans
ON subscriptions.plan_id = plans.plan_id
WHERE customer_id IN (1, 2, 13, 15, 16, 18, 19, 25, 39, 42);
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/f01dbc77-1400-4f1f-a1c7-13246d139097)

*Second Approach*
```sql
SELECT
    subs.customer_id
  , STRING_AGG(plans.plan_name || ' on ' || TO_CHAR(subs.start_date, 'YYYY-MM-DD'), ', ') AS plan_journey
FROM foodie_fi.subscriptions AS subs 
INNER JOIN foodie_fi.plans 
 ON subs.plan_id = plans.plan_id
WHERE customer_id IN (1, 3, 9, 15, 16, 18, 21, 25, 39, 41)
GROUP BY 
    subs.customer_id
ORDER BY subs.customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/105801af-4911-4dc6-bec3-ef64ab0f6aee)

Comparing the two table results, the second table provides a clearer picture of the 'Customer's Journey'. No need to individually inspect each customer separately.

Throughout this case study, I provided alternative solutions to some questions. In SQL, there are a lot of approaches you can tackle a given problem; the key is identifying the most optimised query for your SQL environment.

### Part B: Data Analysis Questions
> 1. How many customers has Foodie-Fi ever had?
```sql
SELECT 
  COUNT(DISTINCT customer_id) AS total_customers
FROM foodie_fi.subscriptions;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/72c10f9b-94a3-4725-8ecf-f431a61f1b8a)

> 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT
    DATE_TRUNC('MONTH', start_date) AS month_start
  , COUNT(*) AS trial_customers
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY month_start
ORDER BY month_start;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/442dcfc8-8661-4277-ae56-7be9bed4be87)

> 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.
```sql
SELECT
    subs.plan_id
  , plans.plan_name
  , COUNT(*) AS events
FROM foodie_fi.subscriptions AS subs 
INNER JOIN foodie_fi.plans
  ON subs.plan_id = plans.plan_id
WHERE start_date > '2020-12-31'
GROUP BY 
    subs.plan_id
  , plans.plan_name
ORDER BY subs.plan_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/51da92db-7d81-4f7b-8d39-f111202ae076)

> 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
SELECT
    SUM(
      CASE
        WHEN plan_id = 4 THEN 1
        ELSE 0
        END) AS churn_customers
  , ROUND(100 *
      SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END)::NUMERIC /
        COUNT(DISTINCT customer_id)
      , 1) AS percentage
FROM foodie_fi.subscriptions;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/e20e83ea-90e7-41e2-853c-312a7e0fa0bf)

> 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to 1 decimal place?
```sql
WITH ranked_plans AS(
  SELECT
      customer_id
    , plan_id
    , ROW_NUMBER() OVER(
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS plan_rank
  FROM foodie_fi.subscriptions
)
SELECT
    SUM(
      CASE
        WHEN plan_id = 4 THEN 1
        ELSE 0
        END) AS churn_customers_within_freetrial
  , ROUND(100 *
      SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END)::NUMERIC / 
        COUNT(DISTINCT customer_id)
      , 1) AS percentage
FROM ranked_plans
WHERE plan_rank = 2;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/48e75522-d7ca-427e-8998-b9a8f4ae6811)

> 6. What is the number and percentage of customer plans after their initial free trial?
```sql
WITH ranked_plans AS (
  SELECT
      customer_id
    , plan_id
    , ROW_NUMBER() OVER(
            PARTITION BY customer_id
            ORDER BY start_date
          ) AS plan_rank
  FROM foodie_fi.subscriptions
)
SELECT
    plans.plan_id
  , plans.plan_name
  , COUNT(*) AS number_of_customers
  , ROUND(100 *
      COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER()
    ) AS percentage
FROM ranked_plans 
INNER JOIN foodie_fi.plans
  ON ranked_plans.plan_id = plans.plan_id
WHERE plan_rank = 2
GROUP BY 
    plans.plan_id
  , plans.plan_name
ORDER BY plan_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/26c7a2b1-fa5c-4482-bc8d-b7fae3b12e1e)

*Alternative Solution*
```sql
WITH ranked_plans AS (
  SELECT
      customer_id
    , plan_id
    , ROW_NUMBER() OVER(
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS plan_rank
  FROM foodie_fi.subscriptions
),
total_customers AS (
  SELECT
    COUNT(DISTINCT customer_id) AS total
  FROM ranked_plans
  WHERE plan_rank = 2
)
SELECT
    plans.plan_id
  , plans.plan_name
  , COUNT(ranked_plans.*) AS number_of_customers
  , ROUND(100 * COUNT(ranked_plans.*)::NUMERIC / total) AS PERCENTAGE
FROM ranked_plans
INNER JOIN foodie_fi.plans
    ON ranked_plans.plan_id = plans.plan_id
  , total_customers
WHERE plan_rank = 2
GROUP BY
    plans.plan_id
  , plans.plan_name
  , total
ORDER BY plan_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/8c4d266c-0e1d-4edc-9252-e8ab00ae231a)

> 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```sql
WITH valid_subscription AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , ROW_NUMBER() OVER(
          PARTITION BY customer_id
          ORDER BY start_date DESC
        ) AS plan_rank
  FROM foodie_fi.subscriptions
  WHERE start_date <= '2020-12-31'
),
summarised_plan AS (
  SELECT
      plan_id
    , COUNT(DISTINCT customer_id) AS customers
  FROM valid_subscription
  WHERE plan_rank = 1
  GROUP BY plan_id
)
SELECT
    plans.plan_id
  , plans.plan_name
  , customers
  , ROUND(100 *
      customers::NUMERIC / SUM(customers) OVER()
    , 1) AS percentage
FROM summarised_plan 
INNER JOIN foodie_fi.plans
 ON summarised_plan.plan_id = plans.plan_id
GROUP BY
    plans.plan_id
  , plans.plan_name
  , customers
ORDER BY plan_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/68841f90-7a8b-49cf-bd5f-d3abab41eb06)

> 8. How many customers have upgraded to an annual plan in 2020?
```sql
SELECT
  COUNT(DISTINCT customer_id) AS annual_plan_customers
FROM foodie_fi.subscriptions
WHERE plan_id = 3
  AND start_date BETWEEN '2020-01-01' AND '2020-12-31';
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/5ccd6bbf-8204-4335-bdc1-c2b8532b48a7)

*Alternative Solution*
```sql
SELECT 
  COUNT(DISTINCT customer_id) as annual_plan_customers
FROM foodie_fi.subscriptions
WHERE plan_id = 3 
  AND EXTRACT(YEAR FROM start_date) = 2020;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/04f0ffab-19d9-4e0e-9b9a-fe3e3560045a)

> 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
WITH annual_start_date AS (
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
),
trial_start_date AS (
  SELECT
    customer_id,
    start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
)
SELECT
  ROUND(  
    AVG(
      DATE_PART('day',
        asd.start_date::TIMESTAMP - tsd.start_date::TIMESTAMP)
      ) 
    ) AS avg_days_to_annual
FROM annual_start_date AS asd 
INNER JOIN trial_start_date AS tsd
  ON asd.customer_id = tsd.customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/cd6f3d66-ab1c-431e-b81e-a12f9bb80c9f)

*Alternative Solution*
```sql
WITH annual_start_date AS (
  SELECT
      customer_id
    , start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
), 
trial_start_date AS (
  SELECT
      customer_id
    , start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
)
SELECT
  ROUND(AVG(asd.start_date - tsd.start_date)) AS avg_days_to_annual_plan
FROM annual_start_date AS asd 
INNER JOIN trial_start_date AS tsd
  ON asd.customer_id = tsd.customer_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/bdda5889-9a02-4fe9-8167-95702d20d5e4)

> 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
WITH annual_start_date AS (
  SELECT
      customer_id
    , start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3
), 
trial_start_date AS (
  SELECT
      customer_id
    , start_date
  FROM foodie_fi.subscriptions
  WHERE plan_id = 0
),
annual_days AS (
  SELECT
      asd.customer_id
    , DATE_PART('day',
          asd.start_date::TIMESTAMP - tsd.start_date::TIMESTAMP
        )::INT AS duration
  FROM annual_start_date AS asd 
  INNER JOIN trial_start_date AS tsd
    ON asd.customer_id = tsd.customer_id
)
SELECT
    30 * (annual_days.duration / 30) || ' - ' || (30 * (annual_days.duration / 30) + 30) || ' days' AS breakdown_period
  , COUNT(*) AS customers
FROM annual_days
GROUP BY breakdown_period
ORDER BY min(annual_days.duration);
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/2d65116f-f5b6-40ee-9e46-ef46664e673d)

> 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
WITH ranked_plans AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , LAG(plan_id) OVER(
            PARTITION BY customer_id
            ORDER BY start_date
        ) AS lag_plan_id
  FROM foodie_fi.subscriptions
  WHERE DATE_PART('year', start_date) = 2020
)
SELECT
  COUNT(*)
FROM ranked_plans
WHERE lag_plan_id = 1 AND plan_id = 2;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/b6ad9e0d-5f0c-4e56-9867-8a7ce8506188)

### Part C: Challenge Pauyment Question
* With this part, I followed along the explanation and debugged the given queries.*
> The Foodie-Fi team wants you to create a new `payments` table for the year 2020 that includes amounts paid by each customer in the `subscriptions` table with the following requirements:
> - monthly payments always occur on the same day of month as the original `start_date` of any monthly paid plan
> - upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
> - upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
> - once a customer churns they will no longer make payments

Example outputs for this table might look like the following:
| customer_id	| plan_id	| plan_name	| payment_date	| amount	| payment_order | 
| ------------| --------| ----------| --------------| --------| --------------| 
| 1	| 1	| basic monthly	| 2020-08-08	| 9.90	| 1| 
| 1	| 1	| basic monthly	| 2020-09-08	| 9.90	| 2| 
| 1	| 1	| basic monthly	| 2020-10-08	| 9.90	| 3| 
| 1	| 1	| basic monthly	| 2020-11-08	| 9.90	| 4| 
| 1	| 1	| basic monthly	| 2020-12-08	| 9.90	| 5| 
| 2	| 3	| pro annual	| 2020-09-27	| 199.00	| 1| 
| 7	| 1	| basic monthly	| 2020-02-12	| 9.90	| 1| 
| 7	| 1	| basic monthly	| 2020-03-12	| 9.90	| 2| 
| 7	| 1	| basic monthly	| 2020-04-12	| 9.90	| 3| 
| 7	| 1	| basic monthly	| 2020-05-12	| 9.90	| 4| 
| 7	| 2	| pro monthly	| 2020-05-22	| 10.00	| 5| 
| 7	| 2	| pro monthly	| 2020-06-22	| 19.90	| 6| 
| 7	| 2	| pro monthly	| 2020-07-22	| 19.90	| 7| 
| 7	| 2	| pro monthly	| 2020-08-22	| 19.90	| 8| 
| 7	| 2	| pro monthly	| 2020-09-22	| 19.90	| 9| 
| 7	| 2	| pro monthly	| 2020-10-22	| 19.90	| 10| 
| 7	| 2	| pro monthly	| 2020-11-22	| 19.90	| 11| 
| 7	| 2	| pro monthly	| 2020-12-22	| 19.90	| 12| 

*First Explanation*
```sql
SELECT
    customer_id
  , plan_id
  , start_date
  , LEAD(plan_id) OVER(
        PARTITION BY customer_id
        ORDER BY start_date
      ) AS lead_plan_id
  , LEAD(start_date) OVER (
        PARTITION BY customer_id
        ORDER BY start_date
      ) AS lead_start_date
FROM foodie_fi.subscriptions
WHERE DATE_PART('year', start_date) = 2020
  AND plan_id != 0
  AND customer_id IN (1, 2, 7, 11, 13, 15, 16, 18, 19, 25, 39);
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/7021340f-bc31-428b-a7a4-b7715c24daad)

*Second Explanation*
```sql
WITH lead_plan_start AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , LEAD(plan_id) OVER(
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_plan_id
    , LEAD(start_date) OVER (
          PARTITION BY customer_id
          ORDER BY start_date
        ) AS lead_start_date
  FROM foodie_fi.subscriptions
  WHERE DATE_PART('year', start_date) = 2020
    AND plan_id != 0
)
SELECT
    plan_id
  , lead_plan_id
  , COUNT(*) AS transition_count
FROM lead_plan_start
GROUP BY
    plan_id
  , lead_plan_id
ORDER BY 
    plan_id
  , lead_plan_id;
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/310e36bd-58fa-4a3b-a6bf-438f2b0b8306)

*Third Explanation*
The next interim step is to start interpreting the above dataset as follows:

- Basic monthly plan customers can move freely to all states

| plan_id | lead_plan_id | transition_count |
| --- | --- | --- |
| 1 | 2 | 163 |
| 1 | 3 | 88 |
| 1 | 4 | 63 |
| 1 | null | 224 |
- Monthly pro customers only seem to move to annual plan or churn states

| plan_id | lead_plan_id | transition_count |
| --- | --- | --- |
| 2 | 3 | 70 |
| 2 | 4 | 83 |
| 2 | null | 326 |
- No annual customers or churn customers move to another plan in 2020

*Fourth Explanation*
```sql
WITH lead_plans AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , LEAD(plan_id) OVER(
          PARTITION BY customer_id
          ORDER BY start_date
          ) AS lead_plan_id
    , LEAD(start_date) OVER(
          PARTITION BY customer_id
          ORDER BY start_date
          ) AS lead_start_date
  FROM foodie_fi.subscriptions
  WHERE DATE_PART('year', start_date) = 2020
    AND plan_id != 0
),
case_1 AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , DATE_PART('month', AGE('2020-12-31'::DATE, start_date))::INT AS month_diff
  FROM lead_plans
  WHERE lead_plan_id IS NULL
    AND plan_id NOT IN (3, 4)
),
case_1_payments AS (
  SELECT
      customer_id
    , plan_id
    , (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE as start_date
  FROM case_1
),
case_2 AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , DATE_PART('month', AGE(lead_start_date - 1, start_date))::INT AS month_diff
  FROM lead_plans
  WHERE lead_plan_id = 4
),
case_2_payments AS (
  SELECT
      customer_id
    , plan_id
    , (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month') AS start_date
  FROM case_2
),
case_3 AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , DATE_PART('month', AGE(lead_start_date - 1, start_date))::INT AS month_diff
  FROM lead_plans
  WHERE plan_id = 1 
    AND lead_plan_id IN (2,3)
),
case_3_payments AS (
  SELECT
      customer_id
    , plan_id
    , (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month') AS start_date
  FROM case_3
),
case_4 AS (
  SELECT
      customer_id
    , plan_id
    , start_date
    , DATE_PART('month', AGE(lead_start_date - 1, start_date))::INT AS month_diff
  FROM lead_plans
  WHERE plan_id = 2
    AND lead_plan_id = 3
),
case_4_payments AS (
  SELECT
      customer_id
    , plan_id
    , (start_date + GENERATE_SERIES(0, month_diff) * INTERVAL '1 month')::DATE AS start_date
  FROM case_4
),
case_5_payments AS (
  SELECT
      customer_id
    , plan_id
    , start_date
  FROM lead_plans
  WHERE plan_id = 3
),
union_output AS (
  SELECT *
  FROM case_1_payments
UNION ALL 
  SELECT *
  FROM case_2_payments
UNION ALL 
  SELECT *
  FROM case_3_payments
UNION ALL 
  SELECT *
  FROM case_4_payments
UNION ALL
  SELECT *
  FROM case_5_payments
)
SELECT
    customer_id
  , plans.plan_id
  , plans.plan_name
  , start_date AS payment_date
  , CASE
      WHEN union_output.plan_id IN (2, 3) AND LAG(union_output.plan_id) OVER w = 1
      THEN plans.price - 9.90
      ELSE plans.price
      END AS amount
  , RANK() OVER w AS payment_order
FROM union_output

INNER JOIN foodie_fi.plans
  ON union_output.plan_id = plans.plan_id

WHERE customer_id IN (1, 2, 7, 11, 13, 15, 16, 18, 19, 25, 39)

WINDOW w AS (
    PARTITION BY customer_id
    ORDER BY start_date
);
```
![image](https://github.com/jef-fortunahamid/CaseStudy3_FoodieFi/assets/125134025/fae93c17-ef70-4a4b-982a-ac955cef6aa5)
