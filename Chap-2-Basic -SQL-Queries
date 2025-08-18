# Chap 2: Basic SQL Queries - Getting Data Out Like a Pro

*By Ishmael Abayateye*
* Remeber are explanatory words like scenarios are for Learning purposes*

Yesterday, you built your first database. Today, we're going to turn you into a data detective - someone who can extract exactly the insights businesses need from raw data.

## The Reality of Data Analysis Jobs

Let me share what happened during my first week as a data analyst at a marketing agency.

My manager, Jessica, walked over to my desk with a coffee in hand and said, "Ishmael, our client wants to know which customers from California made purchases over $500 in the last quarter, but only if they're active email subscribers. Can you have that by 2 PM?"

My heart sank. I had a database with 50,000 customer records, and I needed to filter by:
- Geographic location (California)
- Purchase amount (over $500)
- Time period (last quarter)
- Subscription status (active email subscribers)

In Excel, this would have taken me hours. With SQL, it took me 2 minutes:

```sql
SELECT customer_name, email, purchase_amount, purchase_date
FROM customers 
WHERE state = 'California' 
  AND purchase_amount > 500
  AND purchase_date >= '2024-07-01'
  AND email_status = 'active'
ORDER BY purchase_amount DESC;
```

That query launched my reputation as the "data wizard" in the office. Today, you'll learn to write queries like this with confidence.

## The WHERE Clause: Your Filtering Superpower

In data analysis, you rarely want ALL the data. You want specific subsets that answer business questions. The WHERE clause is how you filter data to get exactly what you need.

**Basic Syntax:**
```sql
SELECT columns FROM table WHERE condition;
```

Let's practice with our church database using scenarios you'll encounter in real jobs.

### Scenario 1: Customer Segmentation Analysis

**Business Question:** "Show me all members who joined in 2023 so we can analyze our recent growth."

```sql
SELECT first_name, last_name, email, join_date
FROM members
WHERE join_date >= '2023-01-01' AND join_date <= '2023-12-31';
```

**Alternative (more efficient):**
```sql
SELECT first_name, last_name, email, join_date
FROM members
WHERE join_date BETWEEN '2023-01-01' AND '2023-12-31';
```

### Scenario 2: Contact List Validation

**Business Question:** "Find all members missing phone numbers for our upcoming call campaign."

```sql
SELECT first_name, last_name, email
FROM members
WHERE phone IS NULL OR phone = '';
```

**Analysis Note:** This query identifies data quality issues - crucial for any data analyst role.

### Scenario 3: Status-Based Reporting

**Business Question:** "List all inactive members for our reactivation campaign."

```sql
SELECT first_name, last_name, email, join_date
FROM members
WHERE membership_status = 'Inactive';
```

## Text Pattern Matching: Finding Needles in Haystacks

Real-world databases are messy. People enter data inconsistently. As an analyst, you need to find patterns and variations in text data.

### The LIKE Operator

**Find all members whose first name starts with 'J':**
```sql
SELECT first_name, last_name, email
FROM members
WHERE first_name LIKE 'J%';
```

**Find emails from Gmail accounts:**
```sql
SELECT first_name, last_name, email
FROM members
WHERE email LIKE '%@gmail.com';
```

**Find phone numbers missing area codes:**
```sql
SELECT first_name, last_name, phone
FROM members
WHERE phone NOT LIKE '(%)%';
```

### Wildcards in Action

- `%` = any number of characters
- `_` = exactly one character

**Real Business Example:**
"Find all customers whose names might be misspelled variations of 'Johnson'"

```sql
SELECT first_name, last_name, email
FROM members
WHERE last_name LIKE 'John%'  -- Johnson, Johns, Johnsen
   OR last_name LIKE 'Joh_son';  -- Johnson, Johason
```

## Working with Numbers: Financial Analysis Skills

Every business needs financial analysis. Let's create a donations table and practice number-based queries that employers value.

### Setting Up Financial Data

```sql
CREATE TABLE donations (
    donation_id SERIAL PRIMARY KEY,
    member_id INTEGER,
    amount NUMERIC(10,2),
    donation_date DATE,
    purpose VARCHAR(100),
    payment_method VARCHAR(50),
    FOREIGN KEY (member_id) REFERENCES members(member_id)
);

INSERT INTO donations (member_id, amount, donation_date, purpose, payment_method) VALUES
(1, 100.00, '2024-01-15', 'General Fund', 'Credit Card'),
(2, 250.50, '2024-01-20', 'Building Fund', 'Check'),
(1, 75.00, '2024-02-15', 'General Fund', 'Credit Card'),
(3, 500.00, '2024-02-28', 'Special Project', 'Bank Transfer'),
(4, 50.00, '2024-03-10', 'General Fund', 'Cash'),
(2, 1000.00, '2024-03-15', 'Building Fund', 'Check'),
(5, 25.00, '2024-04-05', 'General Fund', 'Credit Card'),
(1, 150.00, '2024-04-20', 'Special Project', 'Credit Card');
```

### High-Value Donor Analysis

**Business Question:** "Identify major donors (donations over $200) for VIP treatment."

```sql
SELECT m.first_name, m.last_name, d.amount, d.donation_date, d.purpose
FROM members m
JOIN donations d ON m.member_id = d.member_id
WHERE d.amount > 200
ORDER BY d.amount DESC;
```

**Analysis Output:**
```
Jennifer|Taylor|1000.00|2024-03-15|Building Fund
David|Brown|500.00|2024-02-28|Special Project
Mary|Johnson|250.50|2024-01-20|Building Fund
```

### Donation Range Analysis

**Business Question:** "Categorize donors by giving levels for targeted communications."

```sql
SELECT 
    first_name, 
    last_name, 
    amount,
    CASE 
        WHEN amount >= 500 THEN 'Major Donor'
        WHEN amount >= 100 THEN 'Regular Donor'
        WHEN amount >= 50 THEN 'Supporter'
        ELSE 'Small Gift'
    END AS donor_category
FROM members m
JOIN donations d ON m.member_id = d.member_id
ORDER BY amount DESC;
```

This type of categorization analysis is essential in customer segmentation roles.

## Date and Time Analysis: Temporal Intelligence

Every data analyst needs to work with dates. Businesses want to understand trends over time, seasonal patterns, and time-based comparisons.

### Recent Activity Analysis

**Business Question:** "Show donation activity in the last 30 days."

```sql
SELECT m.first_name, m.last_name, d.amount, d.donation_date
FROM members m
JOIN donations d ON m.member_id = d.member_id
WHERE d.donation_date >= date('now', '-30 days')
ORDER BY d.donation_date DESC;
```

### Quarterly Analysis

**Business Question:** "Compare Q1 vs Q2 donation patterns."

```sql
-- Q1 Donations (Jan-Mar)
SELECT 'Q1' as quarter, COUNT(*) as donation_count, SUM(amount) as total_amount
FROM donations
WHERE donation_date BETWEEN '2024-01-01' AND '2024-03-31'

UNION

-- Q2 Donations (Apr-Jun)
SELECT 'Q2' as quarter, COUNT(*) as donation_count, SUM(amount) as total_amount
FROM donations
WHERE donation_date BETWEEN '2024-04-01' AND '2024-06-30';
```

### Month-over-Month Growth

**Business Question:** "Show monthly donation trends for board presentation."

```sql
SELECT 
    TO_CHAR(donation_date, 'YYYY-MM') as month,
    COUNT(*) as donation_count,
    SUM(amount) as total_amount,
    AVG(amount) as average_donation
FROM donations
GROUP BY TO_CHAR(donation_date, 'YYYY-MM')
ORDER BY month;
```

## Combining Conditions: Complex Business Logic

Real business questions are rarely simple. You need to combine multiple conditions to answer complex analytical questions.

### Advanced Filtering Example

**Business Question:** "Find active members who joined in 2023 and have made donations over $100, sorted by most recent donation."

```sql
SELECT DISTINCT
    m.first_name,
    m.last_name,
    m.email,
    m.join_date,
    MAX(d.donation_date) as last_donation,
    MAX(d.amount) as highest_donation
FROM members m
JOIN donations d ON m.member_id = d.member_id
WHERE m.membership_status = 'Active'
  AND m.join_date >= '2023-01-01'
  AND d.amount > 100
GROUP BY m.member_id
ORDER BY last_donation DESC;
```

This query demonstrates several job-ready skills:
- Complex joins
- Multiple WHERE conditions
- Grouping and aggregation
- Date handling
- Result sorting

## Sorting and Organizing Results: Presentation Skills

Data without proper organization is useless. Employers want analysts who can present data clearly.

### Multi-Level Sorting

**Business Question:** "Create a donor report sorted by donation amount (highest first), then by date (most recent first)."

```sql
SELECT 
    m.first_name || ' ' || m.last_name as donor_name,
    d.amount,
    d.donation_date,
    d.purpose,
    d.payment_method
FROM members m
JOIN donations d ON m.member_id = d.member_id
ORDER BY d.amount DESC, d.donation_date DESC;
```

### Top N Analysis

**Business Question:** "Show the top 5 largest donations for executive summary."

```sql
SELECT 
    m.first_name || ' ' || m.last_name as donor_name,
    d.amount,
    d.donation_date,
    d.purpose
FROM members m
JOIN donations d ON m.member_id = d.member_id
ORDER BY d.amount DESC
LIMIT 5;
```

## Job Interview Scenarios

Here are real questions I've been asked in data analyst interviews, with solutions:

### Interview Question 1: Data Quality Assessment

"How would you identify potential data quality issues in a customer database?"

**Answer with SQL:**
```sql
-- Find duplicate emails
SELECT email, COUNT(*) as duplicate_count
FROM members
GROUP BY email
HAVING COUNT(*) > 1;

-- Find invalid email formats
SELECT first_name, last_name, email
FROM members
WHERE email NOT LIKE '%@%.%'
   OR email IS NULL;

-- Find inconsistent phone formats
SELECT DISTINCT phone
FROM members
WHERE phone IS NOT NULL
ORDER BY phone;
```

### Interview Question 2: Business Performance Analysis

"Calculate the average donation amount by payment method to optimize payment processing."

**Answer:**
```sql
SELECT 
    payment_method,
    COUNT(*) as transaction_count,
    AVG(amount) as average_amount,
    SUM(amount) as total_amount,
    MIN(amount) as smallest_donation,
    MAX(amount) as largest_donation
FROM donations
GROUP BY payment_method
ORDER BY average_amount DESC;
```

---

## Practical Session: Real-World Data Analysis Tasks

*Time to practice skills that employers actually test in interviews.*

### Task 1: Customer Acquisition Analysis

You're a data analyst at a nonprofit. The director asks: "We need to understand our member acquisition patterns to plan next year's marketing budget."

**Step 1: Create a comprehensive member analysis query**

```sql
SELECT 
    EXTRACT(YEAR FROM join_date) as join_year,
    EXTRACT(MONTH FROM join_date) as join_month,
    COUNT(*) as new_members,
    COUNT(*) * 1.0 / (SELECT COUNT(*) FROM members) * 100 as percentage_of_total
FROM members
WHERE join_date IS NOT NULL
GROUP BY EXTRACT(YEAR FROM join_date), EXTRACT(MONTH FROM join_date)
ORDER BY join_year, join_month;
```

**Analysis Questions:**
1. Which month had the highest member acquisition?
2. What percentage of total members joined in 2023?
3. Are there any seasonal patterns?

### Task 2: Revenue Analysis by Customer Segment

The CFO needs donor segmentation analysis for the annual report.

**Step 2: Create donor value segments**

```sql
SELECT 
    CASE 
        WHEN total_donated >= 1000 THEN 'Platinum'
        WHEN total_donated >= 500 THEN 'Gold'
        WHEN total_donated >= 100 THEN 'Silver'
        ELSE 'Bronze'
    END as donor_tier,
    COUNT(*) as donor_count,
    AVG(total_donated) as avg_donation_per_donor,
    SUM(total_donated) as tier_total
FROM (
    SELECT 
        m.member_id,
        m.first_name,
        m.last_name,
        SUM(d.amount) as total_donated
    FROM members m
    LEFT JOIN donations d ON m.member_id = d.member_id
    GROUP BY m.member_id
) donor_summary
GROUP BY donor_tier
ORDER BY AVG(total_donated) DESC;
```

### Task 3: Retention Analysis

Marketing wants to identify at-risk members for a retention campaign.

**Step 3: Find members who haven't donated recently**

```sql
SELECT 
    m.first_name,
    m.last_name,
    m.email,
    m.join_date,
    MAX(d.donation_date) as last_donation,
    julianday('now') - julianday(MAX(d.donation_date)) as days_since_last_donation
FROM members m
LEFT JOIN donations d ON m.member_id = d.member_id
WHERE m.membership_status = 'Active'
GROUP BY m.member_id
HAVING days_since_last_donation > 90 OR last_donation IS NULL
ORDER BY days_since_last_donation DESC;
```

### Task 4: Payment Method Performance

Finance wants to optimize payment processing costs.

**Step 4: Analyze payment method efficiency**

```sql
SELECT 
    payment_method,
    COUNT(*) as transaction_count,
    SUM(amount) as total_revenue,
    AVG(amount) as average_transaction,
    COUNT(*) * 100.0 / (SELECT COUNT(*) FROM donations) as percentage_of_transactions
FROM donations
GROUP BY payment_method
ORDER BY total_revenue DESC;
```

## Interview Preparation Exercises

Practice these scenarios that commonly appear in data analyst interviews:

### Exercise 1: Time-Series Analysis
"Show month-over-month growth in donation revenue."

```sql
SELECT 
    TO_CHAR(donation_date, 'YYYY-MM') as month,
    SUM(amount) as monthly_revenue,
    LAG(SUM(amount)) OVER (ORDER BY TO_CHAR(donation_date, 'YYYY-MM')) as previous_month,
    ((SUM(amount) - LAG(SUM(amount)) OVER (ORDER BY TO_CHAR(donation_date, 'YYYY-MM'))) / 
     LAG(SUM(amount)) OVER (ORDER BY TO_CHAR(donation_date, 'YYYY-MM')) * 100) as growth_percentage
FROM donations
GROUP BY TO_CHAR(donation_date, 'YYYY-MM')
ORDER BY month;
```

### Exercise 2: Cohort Analysis
"Analyze member behavior by join month."

```sql
SELECT 
    TO_CHAR(m.join_date, 'YYYY-MM') as join_cohort,
    COUNT(DISTINCT m.member_id) as cohort_size,
    COUNT(DISTINCT d.member_id) as donors_in_cohort,
    COUNT(DISTINCT d.member_id) * 100.0 / COUNT(DISTINCT m.member_id) as donation_rate
FROM members m
LEFT JOIN donations d ON m.member_id = d.member_id
GROUP BY TO_CHAR(m.join_date, 'YYYY-MM')
ORDER BY join_cohort;
```

## Common Interview Questions and SQL Solutions

**Q: "How would you find duplicate records in a database?"**
```sql
SELECT email, COUNT(*) as duplicate_count
FROM members
GROUP BY email
HAVING COUNT(*) > 1;
```

**Q: "Show me the top 10% of customers by revenue."**
```sql
SELECT m.first_name, m.last_name, SUM(d.amount) as total_revenue
FROM members m
JOIN donations d ON m.member_id = d.member_id
GROUP BY m.member_id
ORDER BY total_revenue DESC
LIMIT (SELECT COUNT(DISTINCT member_id) * 0.1 FROM donations);
```

**Q: "Calculate a rolling 3-month average."**
```sql
SELECT 
    month,
    monthly_revenue,
    AVG(monthly_revenue) OVER (
        ORDER BY month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as rolling_3month_avg
FROM (
    SELECT 
        TO_CHAR(donation_date, 'YYYY-MM') as month,
        SUM(amount) as monthly_revenue
    FROM donations
    GROUP BY TO_CHAR(donation_date, 'YYYY-MM')
) monthly_data
ORDER BY month;
```

## What Makes You Job-Ready

After today's practice, you can now:

✅ **Filter data precisely** using WHERE clauses  
✅ **Handle text pattern matching** for messy real-world data  
✅ **Analyze financial data** with numerical comparisons  
✅ **Work with dates and time periods** for trend analysis  
✅ **Combine multiple conditions** for complex business logic  
✅ **Present data professionally** with proper sorting  
✅ **Solve common interview problems** with confidence  

These skills directly translate to:
- Customer segmentation analysis
- Financial reporting and KPI tracking
- Data quality assessment
- Trend analysis and forecasting
- Operational reporting

## Chap 3 Preview

In Chap 3, we'll master JOINs - the skill that separates junior analysts from senior ones. You'll learn to combine data from multiple tables to answer complex business questions like:

- "Which customers bought Product A but never bought Product B?"
- "Show me sales performance by region and sales representative"
- "Calculate customer lifetime value across all touchpoints"

These multi-table analysis skills are essential for any serious data analyst role.

---

*Chap 2 Complete! You're now equipped with filtering and analysis skills that employers actively seek. Tomorrow, we'll tackle the advanced techniques that will make you indispensable.*

**Ishmael Abayateye**
