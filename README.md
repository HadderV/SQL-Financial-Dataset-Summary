# SQL-Financial-Dataset-Summary

# Financial Dataset Summary for Power BI Reports

## 📌 Overview
-- Description: This query generates a summarized dataset combining user demographic and financial data with card information.
-- It masks user IDs, categorizes income and credit score, and provides insights into users' financial behavior.

## 🛠️ Technologies Used
- MySQL Workbench
- SQL Queries
- Data Cleaning & Aggregation

## 📊 Key Features
- Masked User IDs for privacy
- Income and credit score categorization
- Debt-to-income ratio calculation
- Card fraud risk assessment

## 🚀 Usage
Run `financial_summary.sql` in MySQL to generate the summary dataset.

Code

      SELECT 
        -- Masked User ID for Privacy
        CONCAT('USR-', RIGHT(SHA2(u.id, 256), 6)) AS masked_user_id,
        
        -- User Demographics
        u.gender,
        u.current_age,
        CASE
           WHEN u.current_age < 30 THEN 'Under 30'
           WHEN u.current_age BETWEEN 30 AND 39 THEN '30-39'
           WHEN u.current_age BETWEEN 40 AND 49 THEN '40-49'
           WHEN u.current_age BETWEEN 50 AND 59 THEN '50-59'
           WHEN u.current_age > 59 THEN 'Over 60'
        END AS age_group,
        
        -- Income and Financial Metrics
        CASE 
            WHEN CAST(REPLACE(u.yearly_income, '$', '') AS UNSIGNED) < 30000 THEN 'Low Income'
            WHEN CAST(REPLACE(u.yearly_income, '$', '') AS UNSIGNED) BETWEEN 30000 AND 60000 THEN 'Middle Income'
            ELSE 'High Income'
        END AS income_group,
        CAST(REPLACE(u.yearly_income, '$', '') AS UNSIGNED) AS yearly_income,
        CAST(REPLACE(u.total_debt, '$', '') AS UNSIGNED) AS total_debt,
        ROUND(CAST(REPLACE(u.total_debt, '$', '') AS UNSIGNED) * 100.0 / CAST(REPLACE(u.yearly_income, '$', '') AS UNSIGNED), 2) AS debt_to_income_ratio,
        u.credit_score,
        CASE 
            WHEN u.credit_score < 600 THEN 'Poor'
            WHEN u.credit_score BETWEEN 600 AND 749 THEN 'Fair'
            ELSE 'Good'
        END AS credit_score_category,
    
        -- Card Metrics
        c.card_type,
        c.card_brand,
        COUNT(c.id) AS num_cards_issued,
        MAX(STR_TO_DATE(CONCAT('01/', c.acct_open_date), '%d/%m/%Y')) AS latest_acct_open_date,
        MIN(STR_TO_DATE(CONCAT('01/', c.acct_open_date), '%d/%m/%Y')) AS earliest_acct_open_date,
        SUM(CAST(REPLACE(c.credit_limit, '$', '') AS UNSIGNED)) AS total_credit_limit,
        CASE 
            WHEN c.card_on_dark_web = 'Yes' THEN 1
            ELSE 0
        END AS flagged_card,
        CASE 
            WHEN c.has_chip = 'Yes' THEN 'Chip Card'
            ELSE 'Non-Chip Card'
        END AS chip_status
        
    FROM keto.users_data u
    LEFT JOIN keto.cards_data c 
        ON u.id = c.client_id
    
    GROUP BY 
        masked_user_id, 
        u.gender, 
        u.current_age, 
        age_group, 
        income_group, 
        yearly_income, 
        total_debt, 
        debt_to_income_ratio, 
        u.credit_score, 
        credit_score_category, 
        c.card_type, 
        c.card_brand, 
        flagged_card, 
        chip_status;
