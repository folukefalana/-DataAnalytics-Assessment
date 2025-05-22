# Data Analytics Assessment
---
## Assessment_Q1.sql
---
## Introduction

This business is a financial institution that helps people save and invest money. There are three big database that stores the list of users (like a contacts list),
list of their savings accounts, and list of their investment plans. This report identifies customers who have both a savings and an investment plan (cross-selling opportunity).
This project generates a complete financial summary report for users based on their savings and investment records stored across multiple relational database tables.

---

## Objective
The objective is to make a single report that shows for each user:
- Their unique ID and email
- Number of savings accounts
- Number of investment plans
- Total amount of confirmed savings and investments

---

## Dataset Distribution
In modern fintech systems, customer data is often distributed across multiple database tables. The database stores:
- User data in `users_customuser`
- Savings transactions in `savings_savingsaccount`
- Investment plans in `plans_plan`
---

## 🛠️ Methodology

1. **Start with the users table,`users_customuser`**  
Step 1: Start with a list of all users 'adashi_staging.users_customuser'
I begin with the user table. This gives me: A unique ID (called owner_id).Their email (since there is no name column in the three tables)

```
SELECT 
    adashi_staging.users_customuser.owner_id AS owner_id;
```

```
adashi_staging.users_customuser.email AS name;
```

2. **Build a savings summary subquery**
Step 2: Select the savings table 'adashi_staging.savings_savingsaccount'
I counted how many savings accounts each person has and how much they’ve saved in total
I summarize that info using a small temporary report just for savings.

```
SELECT 
    owner_id,
    COUNT(*) AS savings_count,
    SUM(confirmed_amount) AS total_savings
FROM adashi_staging.savings_savingsaccount
GROUP BY owner_id
```

3. **Build an investment summary subquery**  
Step 3: Select the investment table 'plans_plan'
I counted how many fixed investment plans they have and how much total money they’ve invested
Again, I build a small summary for this.

```
SELECT 
    owner_id,
    COUNT(*) AS investment_count,
    SUM(amount) AS total_investment
FROM plans_plan
WHERE is_fixed_investment = 1
GROUP BY owner_id;
```


4. **Join all pieces together**
I join the users_customuser with the savings summary and the investment summary.
This creates one complete row per user with all the information I need. I Left-joined the savings and investment summaries to the users_customuser 
   - Used `COALESCE` to convert NULLs into 0 for users without records.

---

## 🧾 SQL Query

```sql
SELECT 
    adashi_staging.users_customuser.owner_id AS owner_id,
    adashi_staging.users_customuser.email AS name,
    COALESCE(savings_summary.savings_count, 0) AS savings_count,
    COALESCE(investment_summary.investment_count, 0) AS investment_count,
    COALESCE(savings_summary.total_savings, 0) + COALESCE(investment_summary.total_investment, 0) AS total_deposits
FROM 
    adashi_staging.users_customuser

LEFT JOIN (
    SELECT 
        owner_id,
        COUNT(*) AS savings_count,
        SUM(confirmed_amount) AS total_savings
    FROM adashi_staging.savings_savingsaccount
    GROUP BY owner_id
) AS savings_summary
ON adashi_staging.users_customuser.owner_id = savings_summary.owner_id

LEFT JOIN (
    SELECT 
        owner_id,
        COUNT(*) AS investment_count,
        SUM(amount) AS total_investment
    FROM adashi_staging.plans_plan
    WHERE is_fixed_investment = 1
    GROUP BY owner_id
) AS investment_summary
ON adashi_staging.users_customuser.owner_id = investment_summary.owner_id;






```
--- Select user info along with savings count, investment count, and total deposits
SELECT 
    adashi_staging.users_customuser.owner_id AS owner_id,  -- The correct column to use for identifying users for the three tables is owner_id. I selected owner_id from the users_customuser 
                                                              table as a unique identifier for each user.
    adashi_staging.users_customuser.email AS name,   -- There is no first_name or last_name column across the three tables except the customuser. Since there's no name column, I'm using 
                                                        the user’s email as a way to identify them. I labelled this column as 'name'. Email is used in place of name to represent the user.
    COALESCE(savings_summary.savings_count, 0) AS savings_count,  -- Number of savings entries (0 if none). Ensures that users with no savings or investments show 0 instead of NULL
    COALESCE(investment_summary.investment_count, 0) AS investment_count,  -- Number of investment plans (If this user has no savings, show 0 instead of NULL). Counts and sums savings and 
                                                                              investment data per user.
    COALESCE(savings_summary.total_savings, 0) + COALESCE(investment_summary.total_investment, 0) AS total_deposits  -- This calculates the total money the user has deposited:
                                                                    It adds total_savings and total_investment. If either one is missing (NULL), COALESCE replaces it with 0 so the math works.
FROM 
    adashi_staging.users_customuser  -- Main user table


LEFT JOIN (        -- I am now joining savings data to each user even if they have no savings.
    SELECT 
        owner_id,  -- Grouped user reference from the savings table
        COUNT(*) AS savings_count,  -- Counts how many savings records each user has.
        SUM(confirmed_amount) AS total_savings  -- Total confirmed savings amount. Adds up all the confirmed savings
    FROM adashi_staging.savings_savingsaccount
    GROUP BY owner_id  --  GROUP BY owner_id groups the results so we get one row per user.
) AS savings_summary
ON adashi_staging.users_customuser.owner_id = savings_summary.owner_id  --  I name the summary savings_summary, and match it to users by comparing their owner_id.


LEFT JOIN (  --- I join investment data to users — again using LEFT JOIN so users without investments still appear.

```
SELECT 
        owner_id,  -- Grouped user reference from the investment table. I name the investment summary table and match it to each user by owner_id.
        COUNT(*) AS investment_count,  -- Count of investment records
        SUM(amount) AS total_investment  -- Total investment amount
    FROM adashi_staging.plans_plan
    WHERE is_fixed_investment = 1  -- Filter only fixed investment plans
    GROUP BY owner_id  -- Aggregate by user
) AS investment_summary
ON adashi_staging.users_customuser.owner_id = investment_summary.owner_id;  -- Join condition: match user ID with investment records

```
LEFT JOIN (
    SELECT 
        owner_id,
        COUNT(*) AS savings_count,
        SUM(confirmed_amount) AS total_savings
    FROM adashi_staging.savings_savingsaccount
    GROUP BY owner_id
) AS savings_summary
ON adashi_staging.users_customuser.owner_id = savings_summary.owner_id


