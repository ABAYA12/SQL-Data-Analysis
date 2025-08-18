# Chap 3: Advanced Querying - Joining the Dots

*By Ishmael Abayateye*
* Remeber explanatory words like scenarios are for Learning purposes*

This Chap. marks the transition from basic SQL to professional-level data analysis. JOINs are what separate entry-level analysts from senior data professionals. Master this, and you'll be able to answer complex business questions that require combining data from multiple sources.

## The Million-Dollar JOIN

Let me tell you about the query that saved my company $1.2 million.

I was working as a senior data analyst at an e-commerce company when our CMO burst into the analytics team meeting. "We're spending $100,000 monthly on email marketing, but I have no idea which campaigns actually drive sales. Marketing says their open rates are great, but sales are declining. I need answers by Friday, or we're cutting the entire email budget."

The problem? Our data was scattered across four different tables:
- Customer information in one table
- Email campaign data in another
- Purchase transactions in a third
- Product details in a fourth

No single table could answer the CMO's question. But with strategic JOINs, I created a comprehensive analysis that revealed:

- Email campaigns were generating $300,000 in monthly revenue
- Certain customer segments had 15x higher purchase rates
- The timing of campaigns directly impacted conversion
- Product recommendations needed optimization

That analysis not only saved the email marketing budget but helped optimize it for a 40% increase in ROI. Today, you'll learn to write those million-dollar queries.

## Understanding Table Relationships

Before mastering JOINs, you need to understand how business data connects. Every successful data analyst thinks in terms of relationships.

### Real-World Business Relationships

**Customers → Orders → Products**
- One customer can place many orders
- One order can contain many products
- One product can appear in many orders

**Students → Enrollments → Courses**
- One student can enroll in many courses
- One course can have many students
- Enrollments track the relationship

**Members → Donations → Campaigns**
- One member can make many donations
- One donation belongs to one member
- Donations can be linked to specific campaigns

Let's expand our church database to include these relationships:

```sql
-- Events table
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    event_name VARCHAR(200) NOT NULL,
    event_date DATE,
    location VARCHAR(150),
    max_capacity INTEGER,
    registration_fee NUMERIC(8,2) DEFAULT 0.00,
    event_type VARCHAR(50)
);

-- Event registrations (many-to-many relationship)
CREATE TABLE event_registrations (
    registration_id SERIAL PRIMARY KEY,
    member_id INTEGER,
    event_id INTEGER,
    registration_date DATE,
    payment_status VARCHAR(20) DEFAULT 'Pending',
    special_requirements TEXT,
    FOREIGN KEY (member_id) REFERENCES members(member_id),
    FOREIGN KEY (event_id) REFERENCES events(event_id)
);

-- Insert sample events
INSERT INTO events (event_name, event_date, location, max_capacity, registration_fee, event_type) VALUES
('Annual Gala', '2024-06-15', 'Grand Ballroom', 200, 75.00, 'Fundraising'),
('Youth Conference', '2024-07-20', 'Community Center', 100, 25.00, 'Educational'),
('Christmas Service', '2024-12-24', 'Main Sanctuary', 500, 0.00, 'Service'),
('Leadership Retreat', '2024-09-10', 'Mountain Lodge', 50, 150.00, 'Training'),
('Community Outreach', '2024-08-05', 'Downtown Park', 1000, 0.00, 'Service');

-- Insert sample registrations
INSERT INTO event_registrations (member_id, event_id, registration_date, payment_status) VALUES
(1, 1, '2024-05-01', 'Paid'),
(2, 1, '2024-05-03', 'Paid'),
(3, 2, '2024-06-01', 'Paid'),
(1, 2, '2024-06-02', 'Pending'),
(4, 3, '2024-11-01', 'Confirmed'),
(5, 3, '2024-11-02', 'Confirmed'),
(2, 4, '2024-08-01', 'Paid'),
(1, 5, '2024-07-15', 'Confirmed');
```

## INNER JOIN: The Foundation of Data Analysis

INNER JOIN returns only records that have matching values in both tables. This is your most-used JOIN type in business analysis.

### Basic INNER JOIN Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.common_column = table2.common_column;
```

### Business Scenario: Customer Purchase Analysis

**Question:** "Show me all members who have made donations, along with their donation details."

```sql
SELECT 
    m.first_name,
    m.last_name,
    m.email,
    d.amount,
    d.donation_date,
    d.purpose
FROM members m
INNER JOIN donations d ON m.member_id = d.member_id
ORDER BY d.donation_date DESC;
```

**Business Value:** This query identifies your active donors - critical for donor retention strategies and personalized communication.

### Advanced INNER JOIN: Multiple Tables

**Question:** "Show member details for all paid event registrations, including event information."

```sql
SELECT 
    m.first_name || ' ' || m.last_name AS member_name,
    m.email,
    e.event_name,
    e.event_date,
    e.registration_fee,
    er.registration_date,
    er.payment_status
FROM members m
INNER JOIN event_registrations er ON m.member_id = er.member_id
INNER JOIN events e ON er.event_id = e.event_id
WHERE er.payment_status = 'Paid'
ORDER BY e.event_date, er.registration_date;
```

**Analysis Insight:** This gives you a complete picture of engaged members who financially commit to events - your most valuable community members.

## LEFT JOIN: Finding the Missing Pieces

LEFT JOIN returns all records from the left table and matching records from the right table. This is crucial for identifying gaps in your data.

### Business Scenario: Member Engagement Analysis

**Question:** "Show all members and their donation history, including members who haven't donated yet."

```sql
SELECT 
    m.first_name,
    m.last_name,
    m.email,
    m.join_date,
    d.amount,
    d.donation_date,
    d.purpose
FROM members m
LEFT JOIN donations d ON m.member_id = d.member_id
ORDER BY m.join_date DESC;
```

**Key Business Insight:** Members with NULL donation values are prospects for first-time donor campaigns.

### Finding Non-Participants

**Question:** "Identify members who haven't registered for any events - potential outreach targets."

```sql
SELECT 
    m.first_name,
    m.last_name,
    m.email,
    m.join_date
FROM members m
LEFT JOIN event_registrations er ON m.member_id = er.member_id
WHERE er.member_id IS NULL
ORDER BY m.join_date DESC;
```

**Marketing Application:** These members need targeted engagement to increase participation.

## RIGHT JOIN and FULL OUTER JOIN

While less common in business analysis, these JOINs have specific use cases:

**RIGHT JOIN:** Returns all records from the right table (rarely used, can be rewritten as LEFT JOIN)

**FULL OUTER JOIN:** Returns all records from both tables (SQLite doesn't support this, but other databases do)

## Aggregate Functions with JOINs: The Power Combination

This is where data analysis gets exciting. Combining JOINs with aggregate functions answers complex business questions.

### Revenue Analysis by Member

**Question:** "Calculate total lifetime value for each member."

```sql
SELECT 
    m.first_name || ' ' || m.last_name AS member_name,
    m.email,
    COUNT(d.donation_id) AS total_donations,
    SUM(d.amount) AS lifetime_value,
    AVG(d.amount) AS average_donation,
    MIN(d.donation_date) AS first_donation,
    MAX(d.donation_date) AS latest_donation
FROM members m
LEFT JOIN donations d ON m.member_id = d.member_id
GROUP BY m.member_id, m.first_name, m.last_name, m.email
ORDER BY lifetime_value DESC NULLS LAST;
```

**Business Impact:** This analysis identifies your most valuable members for VIP treatment and helps segment your donor base.

### Event Performance Metrics

**Question:** "Analyze event profitability and attendance rates."

```sql
SELECT 
    e.event_name,
    e.event_date,
    e.max_capacity,
    e.registration_fee,
    COUNT(er.registration_id) AS total_registrations,
    COUNT(CASE WHEN er.payment_status = 'Paid' THEN 1 END) AS paid_registrations,
    SUM(CASE WHEN er.payment_status = 'Paid' THEN e.registration_fee ELSE 0 END) AS total_revenue,
    ROUND(COUNT(er.registration_id) * 100.0 / e.max_capacity, 2) AS attendance_rate,
    ROUND(COUNT(CASE WHEN er.payment_status = 'Paid' THEN 1 END) * 100.0 / COUNT(er.registration_id), 2) AS payment_rate
FROM events e
LEFT JOIN event_registrations er ON e.event_id = er.event_id
GROUP BY e.event_id, e.event_name, e.event_date, e.max_capacity, e.registration_fee
ORDER BY total_revenue DESC;
```

**Executive Dashboard Value:** This single query provides all key metrics for event ROI analysis.

## Advanced JOIN Techniques

### Self JOINs: Comparing Records Within the Same Table

**Question:** "Find members who joined on the same date."

```sql
SELECT 
    m1.first_name || ' ' || m1.last_name AS member1,
    m2.first_name || ' ' || m2.last_name AS member2,
    m1.join_date
FROM members m1
INNER JOIN members m2 ON m1.join_date = m2.join_date
WHERE m1.member_id < m2.member_id  -- Avoid duplicates
ORDER BY m1.join_date;
```

### Conditional JOINs

**Question:** "Show high-value donors and their event participation."

```sql
SELECT 
    m.first_name || ' ' || m.last_name AS member_name,
    total_donated,
    event_name,
    registration_date
FROM members m
INNER JOIN (
    SELECT member_id, SUM(amount) AS total_donated
    FROM donations
    GROUP BY member_id
    HAVING SUM(amount) > 200
) high_donors ON m.member_id = high_donors.member_id
LEFT JOIN event_registrations er ON m.member_id = er.member_id
LEFT JOIN events e ON er.event_id = e.event_id
ORDER BY total_donated DESC, registration_date DESC;
```

## Data Analysis Interview Scenarios

Here are real interview questions that test JOIN knowledge:

### Interview Question 1: Customer Acquisition Cost

"Calculate the cost per acquired customer by marketing channel."

```sql
-- Assuming we have marketing_campaigns and acquisitions tables
SELECT 
    mc.channel_name,
    mc.campaign_cost,
    COUNT(a.customer_id) AS customers_acquired,
    ROUND(mc.campaign_cost / COUNT(a.customer_id), 2) AS cost_per_acquisition
FROM marketing_campaigns mc
LEFT JOIN acquisitions a ON mc.campaign_id = a.campaign_id
GROUP BY mc.campaign_id, mc.channel_name, mc.campaign_cost
ORDER BY cost_per_acquisition ASC;
```

### Interview Question 2: Product Cross-Sell Analysis

"Identify products frequently bought together."

```sql
-- Products bought in the same order
SELECT 
    p1.product_name AS product_a,
    p2.product_name AS product_b,
    COUNT(*) AS times_bought_together
FROM order_items oi1
INNER JOIN order_items oi2 ON oi1.order_id = oi2.order_id
INNER JOIN products p1 ON oi1.product_id = p1.product_id
INNER JOIN products p2 ON oi2.product_id = p2.product_id
WHERE oi1.product_id < oi2.product_id  -- Avoid duplicates
GROUP BY p1.product_id, p2.product_id, p1.product_name, p2.product_name
HAVING COUNT(*) > 5
ORDER BY times_bought_together DESC;
```

### Interview Question 3: Churn Analysis

"Identify customers at risk of churning based on activity patterns."

```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.registration_date,
    MAX(o.order_date) AS last_order_date,
    COUNT(o.order_id) AS total_orders,
    AVG(o.order_value) AS average_order_value,
    julianday('now') - julianday(MAX(o.order_date)) AS days_since_last_order
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.registration_date
HAVING days_since_last_order > 90 OR last_order_date IS NULL
ORDER BY days_since_last_order DESC;
```

## Performance Optimization for JOINs

As datasets grow, JOIN performance becomes critical. Here are professional optimization techniques:

### 1. Use Indexes on JOIN Columns

```sql
CREATE INDEX idx_donations_member_id ON donations(member_id);
CREATE INDEX idx_event_registrations_member_id ON event_registrations(member_id);
CREATE INDEX idx_event_registrations_event_id ON event_registrations(event_id);
```

### 2. Filter Early with WHERE Clauses

**Inefficient:**
```sql
SELECT *
FROM members m
INNER JOIN donations d ON m.member_id = d.member_id
WHERE d.donation_date > '2024-01-01';
```

**Efficient:**
```sql
SELECT *
FROM members m
INNER JOIN (
    SELECT * FROM donations 
    WHERE donation_date > '2024-01-01'
) d ON m.member_id = d.member_id;
```

### 3. Use EXPLAIN QUERY PLAN

```sql
EXPLAIN QUERY PLAN
SELECT m.first_name, d.amount
FROM members m
INNER JOIN donations d ON m.member_id = d.member_id
WHERE d.amount > 100;
```

This shows you how SQLite executes your query and helps identify bottlenecks.

---

## Practical Session: Multi-Table Analysis

*Real-world scenarios that test your JOIN mastery.*

### Scenario 1: Donor Engagement Dashboard

**Business Requirement:** Create a comprehensive donor engagement report for the board meeting.

```sql
-- Complete donor analysis
SELECT 
    member_profile.member_name,
    member_profile.email,
    member_profile.join_date,
    member_profile.total_donated,
    member_profile.donation_count,
    member_profile.avg_donation,
    member_profile.last_donation_date,
    event_stats.events_attended,
    event_stats.total_event_fees,
    CASE 
        WHEN member_profile.total_donated >= 1000 AND event_stats.events_attended >= 3 THEN 'VIP'
        WHEN member_profile.total_donated >= 500 OR event_stats.events_attended >= 2 THEN 'Engaged'
        WHEN member_profile.total_donated > 0 OR event_stats.events_attended > 0 THEN 'Active'
        ELSE 'Inactive'
    END AS engagement_level
FROM (
    -- Member donation summary
    SELECT 
        m.member_id,
        m.first_name || ' ' || m.last_name AS member_name,
        m.email,
        m.join_date,
        COALESCE(SUM(d.amount), 0) AS total_donated,
        COUNT(d.donation_id) AS donation_count,
        COALESCE(AVG(d.amount), 0) AS avg_donation,
        MAX(d.donation_date) AS last_donation_date
    FROM members m
    LEFT JOIN donations d ON m.member_id = d.member_id
    GROUP BY m.member_id, m.first_name, m.last_name, m.email, m.join_date
) member_profile
LEFT JOIN (
    -- Member event summary
    SELECT 
        m.member_id,
        COUNT(er.registration_id) AS events_attended,
        SUM(CASE WHEN er.payment_status = 'Paid' THEN e.registration_fee ELSE 0 END) AS total_event_fees
    FROM members m
    LEFT JOIN event_registrations er ON m.member_id = er.member_id
    LEFT JOIN events e ON er.event_id = e.event_id
    GROUP BY m.member_id
) event_stats ON member_profile.member_id = event_stats.member_id
ORDER BY 
    CASE engagement_level
        WHEN 'VIP' THEN 1
        WHEN 'Engaged' THEN 2
        WHEN 'Active' THEN 3
        ELSE 4
    END,
    member_profile.total_donated DESC;
```

### Scenario 2: Event ROI Analysis

**Business Question:** "Which events generate the best return on investment?"

```sql
SELECT 
    e.event_name,
    e.event_type,
    e.event_date,
    e.max_capacity,
    COUNT(er.registration_id) AS total_registrations,
    COUNT(CASE WHEN er.payment_status = 'Paid' THEN 1 END) AS paid_registrations,
    SUM(CASE WHEN er.payment_status = 'Paid' THEN e.registration_fee ELSE 0 END) AS direct_revenue,
    
    -- Calculate related donations (donations made within 30 days after event)
    COALESCE(post_event_donations.donation_revenue, 0) AS related_donations,
    
    SUM(CASE WHEN er.payment_status = 'Paid' THEN e.registration_fee ELSE 0 END) + 
    COALESCE(post_event_donations.donation_revenue, 0) AS total_revenue,
    
    ROUND(
        (COUNT(er.registration_id) * 100.0 / e.max_capacity), 2
    ) AS capacity_utilization,
    
    ROUND(
        (COUNT(CASE WHEN er.payment_status = 'Paid' THEN 1 END) * 100.0 / 
         NULLIF(COUNT(er.registration_id), 0)), 2
    ) AS payment_conversion_rate

FROM events e
LEFT JOIN event_registrations er ON e.event_id = er.event_id
LEFT JOIN (
    -- Donations within 30 days after each event
    SELECT 
        e.event_id,
        SUM(d.amount) AS donation_revenue
    FROM events e
    INNER JOIN event_registrations er ON e.event_id = er.event_id
    INNER JOIN donations d ON er.member_id = d.member_id
    WHERE d.donation_date BETWEEN e.event_date AND date(e.event_date, '+30 days')
    GROUP BY e.event_id
) post_event_donations ON e.event_id = post_event_donations.event_id

GROUP BY e.event_id, e.event_name, e.event_type, e.event_date, e.max_capacity
ORDER BY total_revenue DESC;
```

### Scenario 3: Member Lifecycle Analysis

**Marketing Question:** "Track member journey from registration through engagement milestones."

```sql
SELECT 
    member_journey.member_name,
    member_journey.join_date,
    member_journey.days_as_member,
    member_journey.first_donation_days,
    member_journey.first_event_days,
    member_journey.total_interactions,
    
    CASE 
        WHEN member_journey.first_donation_days <= 30 THEN 'Fast Converter'
        WHEN member_journey.first_donation_days <= 90 THEN 'Regular Converter'
        WHEN member_journey.first_donation_days IS NOT NULL THEN 'Slow Converter'
        WHEN member_journey.first_event_days <= 60 THEN 'Event Engaged'
        ELSE 'Needs Attention'
    END AS member_type

FROM (
    SELECT 
        m.member_id,
        m.first_name || ' ' || m.last_name AS member_name,
        m.join_date,
        julianday('now') - julianday(m.join_date) AS days_as_member,
        
        -- Days until first donation
        MIN(julianday(d.donation_date) - julianday(m.join_date)) AS first_donation_days,
        
        -- Days until first event registration
        MIN(julianday(er.registration_date) - julianday(m.join_date)) AS first_event_days,
        
        -- Total interactions
        COUNT(DISTINCT d.donation_id) + COUNT(DISTINCT er.registration_id) AS total_interactions
        
    FROM members m
    LEFT JOIN donations d ON m.member_id = d.member_id
    LEFT JOIN event_registrations er ON m.member_id = er.member_id
    GROUP BY m.member_id, m.first_name, m.last_name, m.join_date
) member_journey

ORDER BY member_journey.join_date DESC;
```

## Advanced Business Intelligence Queries

### Customer Segmentation with RFM Analysis

RFM (Recency, Frequency, Monetary) analysis is a standard technique in customer analytics:

```sql
WITH rfm_calculation AS (
    SELECT 
        m.member_id,
        m.first_name || ' ' || m.last_name AS member_name,
        
        -- Recency: Days since last donation
        julianday('now') - julianday(MAX(d.donation_date)) AS recency_days,
        
        -- Frequency: Number of donations
        COUNT(d.donation_id) AS frequency,
        
        -- Monetary: Total donation amount
        SUM(d.amount) AS monetary
        
    FROM members m
    LEFT JOIN donations d ON m.member_id = d.member_id
    GROUP BY m.member_id, m.first_name, m.last_name
),

rfm_scores AS (
    SELECT *,
        -- Score recency (1-5, where 5 is most recent)
        CASE 
            WHEN recency_days IS NULL THEN 1
            WHEN recency_days <= 30 THEN 5
            WHEN recency_days <= 90 THEN 4
            WHEN recency_days <= 180 THEN 3
            WHEN recency_days <= 365 THEN 2
            ELSE 1
        END AS recency_score,
        
        -- Score frequency (1-5, where 5 is highest frequency)
        CASE 
            WHEN frequency >= 10 THEN 5
            WHEN frequency >= 5 THEN 4
            WHEN frequency >= 3 THEN 3
            WHEN frequency >= 1 THEN 2
            ELSE 1
        END AS frequency_score,
        
        -- Score monetary (1-5, where 5 is highest monetary)
        CASE 
            WHEN monetary >= 1000 THEN 5
            WHEN monetary >= 500 THEN 4
            WHEN monetary >= 200 THEN 3
            WHEN monetary >= 50 THEN 2
            WHEN monetary > 0 THEN 1
            ELSE 0
        END AS monetary_score
        
    FROM rfm_calculation
)

SELECT 
    member_name,
    recency_days,
    frequency,
    monetary,
    recency_score,
    frequency_score,
    monetary_score,
    recency_score + frequency_score + monetary_score AS total_rfm_score,
    
    CASE 
        WHEN recency_score >= 4 AND frequency_score >= 4 AND monetary_score >= 4 THEN 'Champions'
        WHEN recency_score >= 3 AND frequency_score >= 3 AND monetary_score >= 3 THEN 'Loyal Customers'
        WHEN recency_score >= 3 AND frequency_score <= 2 THEN 'Potential Loyalists'
        WHEN recency_score <= 2 AND frequency_score >= 3 THEN 'At Risk'
        WHEN recency_score <= 2 AND frequency_score <= 2 AND monetary_score >= 3 THEN 'Cannot Lose Them'
        ELSE 'Others'
    END AS customer_segment

FROM rfm_scores
ORDER BY total_rfm_score DESC;
```

## What Makes You Hire-Ready

After mastering today's content, you can:

✅ **Combine data from multiple tables** to answer complex business questions  
✅ **Perform customer segmentation analysis** using advanced JOIN techniques  
✅ **Calculate customer lifetime value** and other key business metrics  
✅ **Analyze product performance** and cross-selling opportunities  
✅ **Identify at-risk customers** using churn analysis  
✅ **Optimize query performance** for large datasets  
✅ **Create executive dashboards** with comprehensive KPIs  
✅ **Handle real interview scenarios** with confidence  

These skills position you for roles like:
- Senior Data Analyst
- Business Intelligence Analyst
- Customer Analytics Specialist
- Marketing Data Analyst
- Revenue Operations Analyst

## Chap 4 Focus

In Chap 4, we'll dive into data modification - INSERT, UPDATE, and DELETE operations. You'll learn to:

- Safely modify production data
- Implement data validation rules
- Handle batch operations efficiently
- Maintain data integrity during updates
- Create audit trails for compliance

These skills are crucial for operational roles where you're not just analyzing data, but actively managing and maintaining business-critical databases.

---

*Chap 3 Complete! You've now mastered the JOIN techniques that separate junior analysts from senior professionals. Tomorrow, we'll add data management skills to your toolkit.*

**Ishmael Abayateye**
