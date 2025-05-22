# Data Analytics Assessment

##  Assessment_Q1.sql

```
--- Select user info along with savings count, investment count, and total deposits
SELECT 
    adashi_staging.users_customuser.owner_id AS owner_id,  -- Unique identifier for each user
    adashi_staging.users_customuser.email AS name,         -- Display user email as their name
    COALESCE(savings_summary.savings_count, 0) AS savings_count,  -- Number of savings entries (0 if none)
    COALESCE(investment_summary.investment_count, 0) AS investment_count,  -- Number of investment plans (0 if none)
    COALESCE(savings_summary.total_savings, 0) + COALESCE(investment_summary.total_investment, 0) AS total_deposits
    -- Sum of savings and investments (replace nulls with 0 to avoid NULL results)

FROM 
    adashi_staging.users_customuser  -- Main user table

-- LEFT JOIN with a subquery that calculates savings count and total confirmed savings per user
LEFT JOIN (
    SELECT 
        owner_id,  -- Grouped user reference from the savings table
        COUNT(*) AS savings_count,  -- Count of savings records
        SUM(confirmed_amount) AS total_savings  -- Total confirmed savings amount
    FROM adashi_staging.savings_savingsaccount
    GROUP BY owner_id  -- Aggregate by user
) AS savings_summary
ON adashi_staging.users_customuser.owner_id = savings_summary.owner_id
-- Join condition: match user ID with savings records

-- LEFT JOIN with a subquery that calculates investment count and total amount per user
LEFT JOIN (
    SELECT 
        owner_id,  -- Grouped user reference from the investment table
        COUNT(*) AS investment_count,  -- Count of investment records
        SUM(amount) AS total_investment  -- Total investment amount
    FROM adashi_staging.plans_plan
    WHERE is_fixed_investment = 1  -- Filter only fixed investment plans
    GROUP BY owner_id  -- Aggregate by user
) AS investment_summary
ON adashi_staging.users_customuser.owner_id = investment_summary.owner_id;
-- Join condition: match user ID with investment records


