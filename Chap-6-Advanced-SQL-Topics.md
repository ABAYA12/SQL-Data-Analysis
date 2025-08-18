# Chap 6: Advanced SQL Topics - Power User Techniques

*By Ishmael Abayateye*
* Remember explanatory words like scenarios are for Learning purposes*

Today we reach the summit of SQL mastery. These are the techniques that separate SQL experts from everyone else - the skills that make you the go-to person when complex analytical challenges arise.

## The Query That Saved a $50M Deal

Two years ago, I received an urgent call at 11 PM from the VP of Sales at a Fortune 500 company. They were presenting to their biggest potential client the next morning - a deal worth $50 million over three years. The client had asked a seemingly simple question during the final negotiation:

"Can you show us month-over-month customer retention rates by product category, including cohort analysis showing how long customers typically stay with each product line, segmented by their initial purchase size?"

Their existing reports couldn't answer this. Their data team estimated it would take 2-3 weeks to build the analysis. The deal would be dead by then.

I spent 4 hours that night crafting a single SQL query using window functions, recursive CTEs, and advanced analytical techniques. The query:

```sql
WITH customer_cohorts AS (
  SELECT 
    customer_id,
    product_category,
    MIN(order_date) as first_purchase_date,
    DATE_TRUNC('month', MIN(order_date)) as cohort_month,
    CASE 
      WHEN SUM(order_value) <= 100 THEN 'Small'
      WHEN SUM(order_value) <= 500 THEN 'Medium'
      ELSE 'Large'
    END as initial_purchase_size
  FROM orders o
  JOIN products p ON o.product_id = p.product_id
  GROUP BY customer_id, product_category
),
monthly_activity AS (
  SELECT 
    cc.customer_id,
    cc.product_category,
    cc.cohort_month,
    cc.initial_purchase_size,
    DATE_TRUNC('month', o.order_date) as activity_month,
    MONTHS_BETWEEN(DATE_TRUNC('month', o.order_date), cc.cohort_month) as months_since_first_purchase
  FROM customer_cohorts cc
  JOIN orders o ON cc.customer_id = o.customer_id
  JOIN products p ON o.product_id = p.product_id AND p.category = cc.product_category
),
retention_analysis AS (
  SELECT 
    product_category,
    initial_purchase_size,
    cohort_month,
    months_since_first_purchase,
    COUNT(DISTINCT customer_id) as retained_customers,
    FIRST_VALUE(COUNT(DISTINCT customer_id)) OVER (
      PARTITION BY product_category, initial_purchase_size, cohort_month 
      ORDER BY months_since_first_purchase
    ) as cohort_size
  FROM monthly_activity
  GROUP BY product_category, initial_purchase_size, cohort_month, months_since_first_purchase
)
SELECT 
  product_category,
  initial_purchase_size,
  months_since_first_purchase,
  AVG(retained_customers * 100.0 / cohort_size) as avg_retention_rate,
  COUNT(DISTINCT cohort_month) as cohort_count
FROM retention_analysis
WHERE months_since_first_purchase <= 24  -- First 2 years
GROUP BY product_category, initial_purchase_size, months_since_first_purchase
ORDER BY product_category, initial_purchase_size, months_since_first_purchase;
```

At 7 AM, they had comprehensive retention analytics that impressed their client so much that they not only signed the $50M deal but asked to license the analytics capability as a separate product.

That's the power of advanced SQL - it doesn't just answer questions, it creates competitive advantages.

## Window Functions: Analytical Superpowers

Window functions perform calculations across a set of rows related to the current row, without collapsing the result set like GROUP BY does.

### Basic Window Function Syntax

```sql
SELECT 
  column1,
  column2,
  WINDOW_FUNCTION() OVER (
    [PARTITION BY column3]
    [ORDER BY column4]
    [ROWS/RANGE frame_specification]
  ) as result
FROM table_name;
```

### Ranking Functions

**ROW_NUMBER(): Unique sequential numbers**
```sql
-- Rank donations by amount within each member
SELECT 
  m.first_name || ' ' || m.last_name as member_name,
  d.amount,
  d.donation_date,
  ROW_NUMBER() OVER (
    PARTITION BY d.member_id 
    ORDER BY d.amount DESC
  ) as donation_rank
FROM members m
JOIN donations d ON m.member_id = d.member_id
ORDER BY member_name, donation_rank;
```

**RANK(): Rankings with ties**
```sql
-- Find top 3 donors by total giving (ties get same rank)
SELECT 
  member_name,
  total_donated,
  RANK() OVER (ORDER BY total_donated DESC) as donor_rank
FROM (
  SELECT 
    m.first_name || ' ' || m.last_name as member_name,
    SUM(d.amount) as total_donated
  FROM members m
  JOIN donations d ON m.member_id = d.member_id
  GROUP BY m.member_id, member_name
) donor_totals
WHERE RANK() OVER (ORDER BY total_donated DESC) <= 3;
```

**DENSE_RANK(): Rankings without gaps**
```sql
-- Event attendance rankings by location
SELECT 
  l.location_name,
  e.event_name,
  COUNT(er.registration_id) as attendance,
  DENSE_RANK() OVER (
    PARTITION BY l.location_id 
    ORDER BY COUNT(er.registration_id) DESC
  ) as event_rank
FROM locations l
JOIN events e ON l.location_id = e.location_id
LEFT JOIN event_registrations er ON e.event_id = er.event_id
GROUP BY l.location_id, l.location_name, e.event_id, e.event_name;
```

### Aggregate Window Functions

**Running Totals and Moving Averages**
```sql
-- Monthly donation trends with running totals
SELECT 
  donation_month,
  monthly_total,
  SUM(monthly_total) OVER (
    ORDER BY donation_month 
    ROWS UNBOUNDED PRECEDING
  ) as running_total,
  AVG(monthly_total) OVER (
    ORDER BY donation_month 
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
  ) as three_month_avg
FROM (
  SELECT 
    TO_CHAR(donation_date, 'YYYY-MM') as donation_month,
    SUM(amount) as monthly_total
  FROM donations
  GROUP BY TO_CHAR(donation_date, 'YYYY-MM')
) monthly_donations
ORDER BY donation_month;
```

**Lead and Lag Functions**
```sql
-- Member activity patterns - time between donations
SELECT 
  m.first_name || ' ' || m.last_name as member_name,
  d.donation_date,
  d.amount,
  LAG(d.donation_date) OVER (
    PARTITION BY d.member_id 
    ORDER BY d.donation_date
  ) as previous_donation_date,
  julianday(d.donation_date) - julianday(LAG(d.donation_date) OVER (
    PARTITION BY d.member_id 
    ORDER BY d.donation_date
  )) as days_since_last_donation,
  LEAD(d.donation_date) OVER (
    PARTITION BY d.member_id 
    ORDER BY d.donation_date
  ) as next_donation_date
FROM members m
JOIN donations d ON m.member_id = d.member_id
ORDER BY m.member_id, d.donation_date;
```

### Advanced Window Function Applications

**Percentile Analysis**
```sql
-- Donation amount percentiles for member segmentation
SELECT 
  m.member_id,
  m.first_name || ' ' || m.last_name as member_name,
  SUM(d.amount) as total_donated,
  NTILE(4) OVER (ORDER BY SUM(d.amount)) as quartile,
  PERCENT_RANK() OVER (ORDER BY SUM(d.amount)) as percentile_rank,
  CUME_DIST() OVER (ORDER BY SUM(d.amount)) as cumulative_distribution
FROM members m
JOIN donations d ON m.member_id = d.member_id
GROUP BY m.member_id, member_name
ORDER BY total_donated DESC;
```

**First and Last Value Analysis**
```sql
-- Compare each member's donations to their first and highest
SELECT 
  m.first_name || ' ' || m.last_name as member_name,
  d.donation_date,
  d.amount,
  FIRST_VALUE(d.amount) OVER (
    PARTITION BY d.member_id 
    ORDER BY d.donation_date 
    ROWS UNBOUNDED PRECEDING
  ) as first_donation_amount,
  MAX(d.amount) OVER (
    PARTITION BY d.member_id
  ) as highest_donation_amount,
  d.amount - FIRST_VALUE(d.amount) OVER (
    PARTITION BY d.member_id 
    ORDER BY d.donation_date 
    ROWS UNBOUNDED PRECEDING
  ) as growth_from_first
FROM members m
JOIN donations d ON m.member_id = d.member_id
ORDER BY m.member_id, d.donation_date;
```

## Common Table Expressions (CTEs): Building Complex Logic

CTEs make complex queries readable and maintainable by breaking them into logical steps.

### Basic CTE Structure

```sql
WITH cte_name AS (
  -- CTE query definition
  SELECT columns FROM tables WHERE conditions
)
SELECT columns
FROM cte_name
WHERE additional_conditions;
```

### Business Intelligence with CTEs

**Multi-Step Customer Analysis**
```sql
-- Complex member engagement analysis
WITH member_stats AS (
  -- Step 1: Calculate member statistics
  SELECT 
    m.member_id,
    m.first_name || ' ' || m.last_name as member_name,
    m.join_date,
    COUNT(DISTINCT d.donation_id) as donation_count,
    COALESCE(SUM(d.amount), 0) as total_donated,
    COUNT(DISTINCT er.registration_id) as event_registrations,
    MAX(d.donation_date) as last_donation_date,
    MAX(er.registration_date) as last_event_registration
  FROM members m
  LEFT JOIN donations d ON m.member_id = d.member_id
  LEFT JOIN event_registrations er ON m.member_id = er.member_id
  GROUP BY m.member_id, member_name, m.join_date
),
engagement_scores AS (
  -- Step 2: Calculate engagement scores
  SELECT 
    *,
    CASE 
      WHEN donation_count = 0 AND event_registrations = 0 THEN 'Inactive'
      WHEN total_donated < 100 AND event_registrations <= 1 THEN 'Low'
      WHEN total_donated < 500 AND event_registrations <= 3 THEN 'Medium'
      ELSE 'High'
    END as engagement_level,
    julianday('now') - julianday(COALESCE(last_donation_date, last_event_registration, join_date)) as days_since_activity
  FROM member_stats
),
member_segments AS (
  -- Step 3: Create actionable segments
  SELECT 
    *,
    CASE 
      WHEN engagement_level = 'High' AND days_since_activity <= 30 THEN 'Champions'
      WHEN engagement_level = 'High' AND days_since_activity > 30 THEN 'At Risk VIP'
      WHEN engagement_level = 'Medium' AND days_since_activity <= 60 THEN 'Regular Members'
      WHEN engagement_level = 'Low' AND days_since_activity <= 90 THEN 'New Members'
      WHEN days_since_activity > 180 THEN 'Needs Reactivation'
      ELSE 'Monitor'
    END as action_segment
  FROM engagement_scores
)
-- Step 4: Generate actionable insights
SELECT 
  action_segment,
  COUNT(*) as member_count,
  AVG(total_donated) as avg_lifetime_value,
  AVG(days_since_activity) as avg_days_inactive,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM member_segments), 2) as percentage_of_base
FROM member_segments
GROUP BY action_segment
ORDER BY COUNT(*) DESC;
```

### Recursive CTEs: Hierarchical Data

**Organizational Hierarchy Analysis**
```sql
-- Add staff hierarchy to our database
ALTER TABLE staff ADD COLUMN manager_id INTEGER;
ALTER TABLE staff ADD FOREIGN KEY (manager_id) REFERENCES staff(staff_id);

-- Insert hierarchical data
UPDATE staff SET manager_id = NULL WHERE position = 'General Manager';
UPDATE staff SET manager_id = (SELECT staff_id FROM staff WHERE position = 'General Manager') 
WHERE position IN ('Assistant Manager', 'Head Trainer');

-- Recursive CTE to show organizational structure
WITH RECURSIVE org_hierarchy AS (
  -- Base case: Top-level managers
  SELECT 
    staff_id,
    first_name || ' ' || last_name as staff_name,
    position,
    manager_id,
    0 as level,
    first_name || ' ' || last_name as hierarchy_path
  FROM staff 
  WHERE manager_id IS NULL
  
  UNION ALL
  
  -- Recursive case: Employees with managers
  SELECT 
    s.staff_id,
    s.first_name || ' ' || s.last_name as staff_name,
    s.position,
    s.manager_id,
    oh.level + 1,
    oh.hierarchy_path || ' -> ' || s.first_name || ' ' || s.last_name
  FROM staff s
  INNER JOIN org_hierarchy oh ON s.manager_id = oh.staff_id
)
SELECT 
  REPEAT('  ', level) || staff_name as indented_name,
  position,
  level,
  hierarchy_path
FROM org_hierarchy
ORDER BY hierarchy_path;
```

**Category Hierarchy with Sales Data**
```sql
WITH RECURSIVE category_hierarchy AS (
  -- Root categories
  SELECT 
    category_id,
    category_name,
    parent_category_id,
    0 as level,
    category_name as path
  FROM categories 
  WHERE parent_category_id IS NULL
  
  UNION ALL
  
  -- Child categories
  SELECT 
    c.category_id,
    c.category_name,
    c.parent_category_id,
    ch.level + 1,
    ch.path || ' > ' || c.category_name
  FROM categories c
  INNER JOIN category_hierarchy ch ON c.parent_category_id = ch.category_id
),
category_sales AS (
  SELECT 
    ch.category_id,
    ch.category_name,
    ch.level,
    ch.path,
    COUNT(DISTINCT o.order_id) as orders,
    SUM(oi.quantity * oi.unit_price) as revenue
  FROM category_hierarchy ch
  LEFT JOIN products p ON ch.category_id = p.category_id
  LEFT JOIN order_items oi ON p.product_id = oi.product_id
  LEFT JOIN orders o ON oi.order_id = o.order_id
  GROUP BY ch.category_id, ch.category_name, ch.level, ch.path
)
SELECT 
  REPEAT('  ', level) || category_name as category_hierarchy,
  orders,
  COALESCE(revenue, 0) as revenue,
  path
FROM category_sales
ORDER BY path;
```

## Advanced Aggregation Techniques

### GROUPING SETS and ROLLUP

**Multi-dimensional Analysis**
```sql
-- Revenue analysis by multiple dimensions
SELECT 
  COALESCE(l.location_name, 'All Locations') as location,
  COALESCE(e.event_type, 'All Event Types') as event_type,
  COALESCE(EXTRACT(YEAR FROM e.event_date), 'All Years') as year,
  COUNT(er.registration_id) as registrations,
  SUM(e.registration_fee) as revenue
FROM locations l
CROSS JOIN events e
LEFT JOIN event_registrations er ON e.event_id = er.event_id 
  AND er.payment_status = 'Paid'
GROUP BY GROUPING SETS (
  (l.location_name, e.event_type, EXTRACT(YEAR FROM e.event_date)),  -- All dimensions
  (l.location_name, e.event_type),                               -- By location and type
  (l.location_name, EXTRACT(YEAR FROM e.event_date)),               -- By location and year
  (e.event_type, EXTRACT(YEAR FROM e.event_date)),                  -- By type and year
  (l.location_name),                                             -- By location only
  (e.event_type),                                                -- By type only
  (EXTRACT(YEAR FROM e.event_date)),                                -- By year only
  ()                                                             -- Grand total
)
ORDER BY 
  CASE WHEN location = 'All Locations' THEN 1 ELSE 0 END,
  location,
  CASE WHEN event_type = 'All Event Types' THEN 1 ELSE 0 END,
  event_type,
  year;
```

### Pivot Tables with SQL

**Member Activity Matrix**
```sql
-- Create a pivot showing member activity by month
SELECT 
  member_name,
  SUM(CASE WHEN activity_month = '2024-01' THEN activity_count ELSE 0 END) as Jan_2024,
  SUM(CASE WHEN activity_month = '2024-02' THEN activity_count ELSE 0 END) as Feb_2024,
  SUM(CASE WHEN activity_month = '2024-03' THEN activity_count ELSE 0 END) as Mar_2024,
  SUM(CASE WHEN activity_month = '2024-04' THEN activity_count ELSE 0 END) as Apr_2024,
  SUM(CASE WHEN activity_month = '2024-05' THEN activity_count ELSE 0 END) as May_2024,
  SUM(CASE WHEN activity_month = '2024-06' THEN activity_count ELSE 0 END) as Jun_2024,
  SUM(activity_count) as total_activity
FROM (
  SELECT 
    m.first_name || ' ' || m.last_name as member_name,
    TO_CHAR(d.donation_date, 'YYYY-MM') as activity_month,
    COUNT(*) as activity_count
  FROM members m
  JOIN donations d ON m.member_id = d.member_id
  WHERE d.donation_date >= '2024-01-01'
  GROUP BY m.member_id, member_name, TO_CHAR(d.donation_date, 'YYYY-MM')
  
  UNION ALL
  
  SELECT 
    m.first_name || ' ' || m.last_name as member_name,
    TO_CHAR(er.registration_date, 'YYYY-MM') as activity_month,
    COUNT(*) as activity_count
  FROM members m
  JOIN event_registrations er ON m.member_id = er.member_id
  WHERE er.registration_date >= '2024-01-01'
  GROUP BY m.member_id, member_name, TO_CHAR(er.registration_date, 'YYYY-MM')
) member_activities
GROUP BY member_name
HAVING total_activity > 0
ORDER BY total_activity DESC;
```

## Stored Procedures and Functions

While SQLite has limited stored procedure support, understanding the concepts is crucial for other database systems.

### User-Defined Functions (Conceptual)

```sql
-- Conceptual function for calculating member engagement score
-- (Implementation varies by database system)

/*
CREATE FUNCTION calculate_engagement_score(
    @member_id INT
) RETURNS INT
AS
BEGIN
    DECLARE @score INT = 0;
    DECLARE @donations INT;
    DECLARE @events INT;
    DECLARE @recent_activity_days INT;
    
    SELECT 
        @donations = COUNT(DISTINCT d.donation_id),
        @events = COUNT(DISTINCT er.registration_id),
        @recent_activity_days = MIN(
            julianday('now') - julianday(COALESCE(MAX(d.donation_date), MAX(er.registration_date)))
        )
    FROM members m
    LEFT JOIN donations d ON m.member_id = d.member_id
    LEFT JOIN event_registrations er ON m.member_id = er.member_id
    WHERE m.member_id = @member_id;
    
    -- Calculate score based on business rules
    SET @score = @donations * 10 + @events * 5;
    
    -- Reduce score based on inactivity
    IF @recent_activity_days > 90
        SET @score = @score * 0.5;
    
    RETURN @score;
END;
*/
```

### Trigger-Based Automation

```sql
-- Automatic member status updates based on activity
CREATE TRIGGER update_member_status
    AFTER INSERT ON donations
    FOR EACH ROW
BEGIN
    -- Update member status to VIP if total donations exceed $1000
    UPDATE members 
    SET membership_status = 'VIP'
    WHERE member_id = NEW.member_id 
      AND membership_status != 'VIP'
      AND (
        SELECT SUM(amount) 
        FROM donations 
        WHERE member_id = NEW.member_id
      ) >= 1000;
      
    -- Update last activity date
    UPDATE members 
    SET last_activity_date = NEW.donation_date
    WHERE member_id = NEW.member_id;
END;

-- Maintain running totals for performance
CREATE TRIGGER maintain_donation_totals
    AFTER INSERT ON donations
    FOR EACH ROW
BEGIN
    UPDATE members 
    SET total_lifetime_giving = (
        SELECT COALESCE(SUM(amount), 0)
        FROM donations 
        WHERE member_id = NEW.member_id
    )
    WHERE member_id = NEW.member_id;
END;
```

## Performance Optimization Strategies

### Query Execution Plan Analysis

```sql
-- Analyze query performance
EXPLAIN QUERY PLAN
SELECT 
    m.first_name || ' ' || m.last_name as member_name,
    SUM(d.amount) as total_donated,
    COUNT(er.registration_id) as events_attended
FROM members m
LEFT JOIN donations d ON m.member_id = d.member_id
LEFT JOIN event_registrations er ON m.member_id = er.member_id
WHERE m.membership_status = 'Active'
GROUP BY m.member_id, member_name
HAVING total_donated > 500
ORDER BY total_donated DESC;
```

### Index Optimization for Complex Queries

```sql
-- Composite indexes for common query patterns
CREATE INDEX idx_donations_member_date_amount ON donations(member_id, donation_date, amount);
CREATE INDEX idx_registrations_member_date ON event_registrations(member_id, registration_date);
CREATE INDEX idx_members_status_join ON members(membership_status, join_date);

-- Covering indexes (include all needed columns)
CREATE INDEX idx_member_donation_summary ON donations(member_id, amount) 
WHERE amount > 0;

-- Partial indexes for specific conditions
CREATE INDEX idx_active_members ON members(member_id, first_name, last_name) 
WHERE membership_status = 'Active';
```

### Query Optimization Techniques

```sql
-- Inefficient: Correlated subquery
SELECT 
    m.first_name,
    m.last_name,
    (SELECT SUM(amount) FROM donations d WHERE d.member_id = m.member_id) as total_donated
FROM members m
WHERE membership_status = 'Active';

-- Efficient: JOIN with aggregation
SELECT 
    m.first_name,
    m.last_name,
    COALESCE(d.total_donated, 0) as total_donated
FROM members m
LEFT JOIN (
    SELECT member_id, SUM(amount) as total_donated
    FROM donations
    GROUP BY member_id
) d ON m.member_id = d.member_id
WHERE m.membership_status = 'Active';

-- Most efficient: Materialized view approach
CREATE VIEW member_donation_summary AS
SELECT 
    m.member_id,
    m.first_name,
    m.last_name,
    m.membership_status,
    COALESCE(SUM(d.amount), 0) as total_donated,
    COUNT(d.donation_id) as donation_count,
    MAX(d.donation_date) as last_donation_date
FROM members m
LEFT JOIN donations d ON m.member_id = d.member_id
GROUP BY m.member_id, m.first_name, m.last_name, m.membership_status;
```

---

## Practical Session: Advanced Analytics Implementation

*Build a comprehensive business intelligence system using advanced SQL techniques.*

### Scenario: Executive Dashboard for Multi-Location Fitness Chain

**Business Requirement:** Create real-time analytics for C-level executives to monitor business performance across all locations.

### Step 1: Member Lifecycle Analytics

```sql
-- Member acquisition and retention funnel
WITH monthly_cohorts AS (
  SELECT 
    DATE_TRUNC('month', join_date) as cohort_month,
    location_id,
    membership_type_id,
    COUNT(*) as cohort_size
  FROM members
  GROUP BY DATE_TRUNC('month', join_date), location_id, membership_type_id
),
member_activity_by_month AS (
  SELECT 
    m.member_id,
    DATE_TRUNC('month', m.join_date) as cohort_month,
    m.location_id,
    m.membership_type_id,
    DATE_TRUNC('month', activity.activity_date) as activity_month,
    MONTHS_BETWEEN(DATE_TRUNC('month', activity.activity_date), DATE_TRUNC('month', m.join_date)) as months_since_join
  FROM members m
  CROSS JOIN (
    SELECT checkin_time as activity_date, member_id FROM member_checkins
    UNION 
    SELECT registration_date as activity_date, member_id FROM class_registrations
    UNION
    SELECT session_date as activity_date, member_id FROM personal_training_sessions
  ) activity
  WHERE activity.member_id = m.member_id
),
retention_by_cohort AS (
  SELECT 
    cohort_month,
    location_id,
    membership_type_id,
    months_since_join,
    COUNT(DISTINCT member_id) as active_members
  FROM member_activity_by_month
  WHERE months_since_join >= 0
  GROUP BY cohort_month, location_id, membership_type_id, months_since_join
)
SELECT 
  c.cohort_month,
  l.location_name,
  mt.type_name as membership_type,
  c.cohort_size,
  r.months_since_join,
  r.active_members,
  ROUND(r.active_members * 100.0 / c.cohort_size, 2) as retention_rate,
  ROUND(
    AVG(r.active_members * 100.0 / c.cohort_size) OVER (
      PARTITION BY l.location_id, mt.membership_type_id, r.months_since_join
      ORDER BY c.cohort_month
      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2
  ) as avg_retention_rate
FROM monthly_cohorts c
JOIN retention_by_cohort r ON c.cohort_month = r.cohort_month 
  AND c.location_id = r.location_id 
  AND c.membership_type_id = r.membership_type_id
JOIN locations l ON c.location_id = l.location_id
JOIN membership_types mt ON c.membership_type_id = mt.membership_type_id
WHERE r.months_since_join <= 12  -- First year retention
ORDER BY c.cohort_month, l.location_name, mt.type_name, r.months_since_join;
```

### Step 2: Revenue Analytics with Forecasting

```sql
-- Revenue trends with predictive modeling
WITH daily_revenue AS (
  SELECT 
    DATE(payment_date) as revenue_date,
    location_id,
    SUM(payment_amount) as daily_revenue
  FROM payments p
  JOIN member_billing mb ON p.billing_id = mb.billing_id
  JOIN members m ON mb.member_id = m.member_id
  WHERE payment_status = 'completed'
  GROUP BY DATE(payment_date), location_id
),
revenue_with_trends AS (
  SELECT 
    revenue_date,
    location_id,
    daily_revenue,
    AVG(daily_revenue) OVER (
      PARTITION BY location_id
      ORDER BY revenue_date
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as seven_day_avg,
    AVG(daily_revenue) OVER (
      PARTITION BY location_id
      ORDER BY revenue_date
      ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) as thirty_day_avg,
    LAG(daily_revenue, 7) OVER (
      PARTITION BY location_id
      ORDER BY revenue_date
    ) as revenue_week_ago,
    LAG(daily_revenue, 30) OVER (
      PARTITION BY location_id
      ORDER BY revenue_date
    ) as revenue_month_ago
  FROM daily_revenue
),
growth_metrics AS (
  SELECT 
    *,
    CASE 
      WHEN revenue_week_ago > 0 
      THEN ROUND((daily_revenue - revenue_week_ago) * 100.0 / revenue_week_ago, 2)
      ELSE NULL
    END as week_over_week_growth,
    CASE 
      WHEN revenue_month_ago > 0 
      THEN ROUND((daily_revenue - revenue_month_ago) * 100.0 / revenue_month_ago, 2)
      ELSE NULL
    END as month_over_month_growth,
    CASE 
      WHEN seven_day_avg > thirty_day_avg THEN 'Growing'
      WHEN seven_day_avg < thirty_day_avg * 0.95 THEN 'Declining'
      ELSE 'Stable'
    END as trend_direction
  FROM revenue_with_trends
)
SELECT 
  l.location_name,
  gm.revenue_date,
  gm.daily_revenue,
  gm.seven_day_avg,
  gm.thirty_day_avg,
  gm.week_over_week_growth,
  gm.month_over_month_growth,
  gm.trend_direction,
  -- Simple linear regression for next 7 days forecast
  ROUND(
    gm.thirty_day_avg + (gm.seven_day_avg - gm.thirty_day_avg) * 1.2, 2
  ) as forecasted_revenue
FROM growth_metrics gm
JOIN locations l ON gm.location_id = l.location_id
WHERE gm.revenue_date >= DATE('now', '-90 days')
ORDER BY l.location_name, gm.revenue_date DESC;
```

### Step 3: Operational Efficiency Analytics

```sql
-- Staff productivity and utilization analysis
WITH staff_performance AS (
  SELECT 
    s.staff_id,
    s.first_name || ' ' || s.last_name as staff_name,
    s.position,
    l.location_name,
    
    -- Class instruction metrics
    COUNT(DISTINCT ci.instance_id) as classes_taught,
    AVG(ci.enrolled_count) as avg_class_size,
    SUM(ci.enrolled_count * e.registration_fee) as class_revenue_generated,
    
    -- Personal training metrics
    COUNT(DISTINCT pts.session_id) as training_sessions,
    SUM(pts.price) as training_revenue_generated,
    AVG(pts.price) as avg_session_price,
    
    -- Overall metrics
    COUNT(DISTINCT ci.instance_id) + COUNT(DISTINCT pts.session_id) as total_sessions,
    SUM(ci.enrolled_count * e.registration_fee) + SUM(pts.price) as total_revenue_generated
    
  FROM staff s
  JOIN locations l ON s.location_id = l.location_id
  LEFT JOIN class_instances ci ON s.staff_id = ci.instructor_id
    AND ci.class_date >= DATE('now', '-30 days')
  LEFT JOIN class_schedules cs ON ci.schedule_id = cs.schedule_id
  LEFT JOIN events e ON cs.class_type_id = e.event_id
  LEFT JOIN personal_training_sessions pts ON s.staff_id = pts.trainer_id
    AND pts.session_date >= DATE('now', '-30 days')
  WHERE s.is_active = 1
  GROUP BY s.staff_id, staff_name, s.position, l.location_name
),
staff_rankings AS (
  SELECT 
    *,
    RANK() OVER (PARTITION BY location_name ORDER BY total_revenue_generated DESC) as revenue_rank,
    RANK() OVER (PARTITION BY location_name ORDER BY total_sessions DESC) as session_rank,
    NTILE(4) OVER (ORDER BY total_revenue_generated) as performance_quartile
  FROM staff_performance
)
SELECT 
  location_name,
  staff_name,
  position,
  classes_taught,
  training_sessions,
  total_sessions,
  ROUND(total_revenue_generated, 2) as revenue_generated,
  ROUND(total_revenue_generated / NULLIF(total_sessions, 0), 2) as revenue_per_session,
  revenue_rank,
  CASE performance_quartile
    WHEN 4 THEN 'Top Performer'
    WHEN 3 THEN 'Above Average'
    WHEN 2 THEN 'Average'
    ELSE 'Needs Improvement'
  END as performance_category
FROM staff_rankings
ORDER BY location_name, revenue_rank;
```

### Step 4: Predictive Equipment Maintenance

```sql
-- Equipment maintenance scheduling with predictive analytics
WITH equipment_usage AS (
  SELECT 
    e.equipment_id,
    e.equipment_name,
    e.category_id,
    ec.category_name,
    e.location_id,
    l.location_name,
    e.purchase_date,
    e.last_maintenance_date,
    e.next_maintenance_date,
    
    -- Calculate usage intensity based on gym check-ins
    COUNT(ci.checkin_id) as location_traffic_30days,
    julianday('now') - julianday(e.last_maintenance_date) as days_since_maintenance,
    julianday(e.next_maintenance_date) - julianday('now') as days_until_maintenance,
    
    -- Age of equipment
    julianday('now') - julianday(e.purchase_date) as equipment_age_days,
    
    -- Estimate usage based on equipment category and location traffic
    CASE e.category_id
      WHEN 1 THEN COUNT(ci.checkin_id) * 0.8  -- Cardio equipment (high usage)
      WHEN 2 THEN COUNT(ci.checkin_id) * 0.6  -- Strength equipment (medium usage)
      WHEN 3 THEN COUNT(ci.checkin_id) * 0.3  -- Free weights (lower individual usage)
      ELSE COUNT(ci.checkin_id) * 0.4
    END as estimated_usage_sessions
    
  FROM equipment e
  JOIN equipment_categories ec ON e.category_id = ec.category_id
  JOIN locations l ON e.location_id = l.location_id
  LEFT JOIN member_checkins ci ON e.location_id = ci.location_id
    AND ci.checkin_time >= DATE('now', '-30 days')
  WHERE e.status IN ('operational', 'maintenance')
  GROUP BY e.equipment_id, e.equipment_name, e.category_id, ec.category_name, 
           e.location_id, l.location_name, e.purchase_date, e.last_maintenance_date, e.next_maintenance_date
),
maintenance_priority AS (
  SELECT 
    *,
    -- Calculate maintenance priority score
    CASE 
      WHEN days_until_maintenance < 0 THEN 100  -- Overdue
      WHEN days_until_maintenance <= 7 THEN 90  -- Due this week
      WHEN days_until_maintenance <= 14 THEN 70 -- Due in 2 weeks
      WHEN days_until_maintenance <= 30 THEN 50 -- Due this month
      ELSE 30  -- Not due soon
    END +
    CASE 
      WHEN estimated_usage_sessions > 1000 THEN 20  -- High usage
      WHEN estimated_usage_sessions > 500 THEN 10   -- Medium usage
      ELSE 5  -- Low usage
    END +
    CASE 
      WHEN equipment_age_days > 2190 THEN 15  -- Over 6 years old
      WHEN equipment_age_days > 1095 THEN 10  -- Over 3 years old
      ELSE 5  -- Newer equipment
    END as priority_score,
    
    -- Predict optimal maintenance window
    CASE 
      WHEN estimated_usage_sessions < 200 THEN 'Low-traffic period recommended'
      WHEN estimated_usage_sessions < 500 THEN 'Schedule during off-peak hours'
      ELSE 'Consider temporary replacement during maintenance'
    END as maintenance_recommendation
    
  FROM equipment_usage
)
SELECT 
  location_name,
  equipment_name,
  category_name,
  days_since_maintenance,
  days_until_maintenance,
  estimated_usage_sessions,
  priority_score,
  maintenance_recommendation,
  CASE 
    WHEN priority_score >= 90 THEN 'Critical'
    WHEN priority_score >= 70 THEN 'High'
    WHEN priority_score >= 50 THEN 'Medium'
    ELSE 'Low'
  END as priority_level,
  CASE 
    WHEN days_until_maintenance < 0 THEN 'Schedule immediately'
    WHEN days_until_maintenance <= 7 THEN 'Schedule this week'
    WHEN days_until_maintenance <= 14 THEN 'Schedule next week'
    ELSE 'Monitor'
  END as action_required
FROM maintenance_priority
ORDER BY priority_score DESC, location_name, equipment_name;
```

## Real-World Interview Scenarios

### Senior Data Analyst Interview Question

**"Design a system to detect unusual patterns in customer behavior that might indicate fraud or data quality issues."**

```sql
-- Anomaly detection using statistical methods
WITH member_behavior_baseline AS (
  SELECT 
    member_id,
    AVG(amount) as avg_donation,
    STDDEV(amount) as stddev_donation,
    COUNT(*) as donation_frequency,
    AVG(julianday(donation_date) - julianday(LAG(donation_date) OVER (PARTITION BY member_id ORDER BY donation_date))) as avg_days_between_donations
  FROM donations
  WHERE donation_date >= DATE('now', '-1 year')
  GROUP BY member_id
  HAVING COUNT(*) >= 3  -- Need at least 3 donations for statistical analysis
),
recent_activity AS (
  SELECT 
    d.member_id,
    d.donation_id,
    d.amount,
    d.donation_date,
    julianday(d.donation_date) - julianday(LAG(d.donation_date) OVER (PARTITION BY d.member_id ORDER BY d.donation_date)) as days_since_last
  FROM donations d
  WHERE d.donation_date >= DATE('now', '-30 days')
),
anomaly_detection AS (
  SELECT 
    ra.member_id,
    m.first_name || ' ' || m.last_name as member_name,
    ra.donation_id,
    ra.amount,
    ra.donation_date,
    bb.avg_donation,
    bb.stddev_donation,
    bb.avg_days_between_donations,
    ra.days_since_last,
    
    -- Z-score for donation amount
    CASE 
      WHEN bb.stddev_donation > 0 
      THEN ABS(ra.amount - bb.avg_donation) / bb.stddev_donation
      ELSE 0
    END as amount_z_score,
    
    -- Flag unusual timing
    CASE 
      WHEN ra.days_since_last IS NOT NULL AND bb.avg_days_between_donations > 0
      THEN ABS(ra.days_since_last - bb.avg_days_between_donations) / bb.avg_days_between_donations
      ELSE 0
    END as timing_anomaly_ratio,
    
    -- Multiple donations on same day
    COUNT(*) OVER (PARTITION BY ra.member_id, DATE(ra.donation_date)) as donations_same_day
    
  FROM recent_activity ra
  JOIN member_behavior_baseline bb ON ra.member_id = bb.member_id
  JOIN members m ON ra.member_id = m.member_id
)
SELECT 
  member_name,
  donation_date,
  amount,
  ROUND(avg_donation, 2) as typical_amount,
  ROUND(amount_z_score, 2) as amount_z_score,
  days_since_last,
  ROUND(avg_days_between_donations, 1) as typical_days_between,
  donations_same_day,
  
  -- Composite anomaly score
  CASE 
    WHEN amount_z_score > 3 OR timing_anomaly_ratio > 2 OR donations_same_day > 3 THEN 'High'
    WHEN amount_z_score > 2 OR timing_anomaly_ratio > 1.5 OR donations_same_day > 2 THEN 'Medium'
    WHEN amount_z_score > 1.5 OR timing_anomaly_ratio > 1 OR donations_same_day > 1 THEN 'Low'
    ELSE 'Normal'
  END as anomaly_level,
  
  -- Specific flags
  CASE 
    WHEN amount_z_score > 3 THEN 'Unusual amount'
    WHEN timing_anomaly_ratio > 2 THEN 'Unusual timing'
    WHEN donations_same_day > 3 THEN 'Multiple same-day donations'
    ELSE 'No specific flags'
  END as anomaly_type

FROM anomaly_detection
WHERE amount_z_score > 1.5 OR timing_anomaly_ratio > 1 OR donations_same_day > 1
ORDER BY 
  CASE anomaly_level
    WHEN 'High' THEN 1
    WHEN 'Medium' THEN 2
    WHEN 'Low' THEN 3
    ELSE 4
  END,
  amount_z_score DESC;
```

## What Makes You a SQL Expert

After mastering today's advanced techniques, you can:

✅ **Implement sophisticated analytical models** using window functions  
✅ **Build complex hierarchical queries** with recursive CTEs  
✅ **Create predictive analytics** using SQL-based statistical methods  
✅ **Optimize query performance** for enterprise-scale data  
✅ **Design anomaly detection systems** for data quality and fraud prevention  
✅ **Build real-time business intelligence** dashboards  
✅ **Implement advanced aggregation** and pivot table functionality  
✅ **Handle multi-dimensional analysis** with GROUPING SETS  

These skills position you for roles like:
- Senior Data Scientist  
- Principal Business Intelligence Developer
- Data Architecture Consultant
- Advanced Analytics Manager
- Technical Lead for complex data projects

## Capstone IS NEXT

In Chap 7, we'll put everything together in a comprehensive real-world project that demonstrates your complete SQL mastery. You'll design and implement a complete business intelligence system that showcases all the skills you've learned across the seven days.

---

*Day 6 Complete! You've now mastered the advanced SQL techniques that separate experts from practitioners. Tomorrow, we'll demonstrate your complete mastery through a comprehensive capstone project.*

**Ishmael Abayateye**
