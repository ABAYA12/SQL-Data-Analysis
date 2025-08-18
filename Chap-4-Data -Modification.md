# Chap 4: Data Modification - Making Changes Safely

*By Ishmael Abayateye*
* Remeber are explanatory words like scenarios are for Learning purposes*

Today we shift from analyzing data to actively managing it. These skills separate data analysts from data engineers and database administrators. In most companies, senior analysts are trusted with data modification responsibilities - and this trust comes with serious responsibility.

## The $50,000 DELETE Statement

Let me tell you about the most expensive mistake I almost made early in my career.

I was working as a database analyst for a retail company when my manager asked me to "clean up the old promotional codes table - remove anything older than 2 years." Simple enough, right?

I wrote what seemed like a straightforward DELETE statement:

```sql
DELETE FROM promotional_codes 
WHERE created_date < '2022-01-01';
```

I was about to execute it on the production database when my senior colleague Sarah grabbed my wrist. "Wait! Show me that query first."

She pointed to a critical flaw: I hadn't included a WHERE clause that preserved active codes that customers might still be using. My query would have deleted promotional codes worth $50,000 in pending customer orders.

We rewrote it properly:

```sql
DELETE FROM promotional_codes 
WHERE created_date < '2022-01-01' 
  AND status = 'expired'
  AND promo_code NOT IN (
    SELECT DISTINCT promo_code 
    FROM pending_orders 
    WHERE promo_code IS NOT NULL
  );
```

That day taught me the fundamental rule of data modification: **Always think about the business impact before changing a single record.**

Today, you'll learn to modify data with the precision and safety practices that make you trusted with production systems.

## Understanding Data Modification Impact

Before we dive into syntax, let's understand why data modification skills are crucial for career advancement:

### Why Employers Value These Skills

**Operational Efficiency:** Senior analysts who can safely update data save companies from hiring additional database administrators.

**Data Quality Management:** You'll be able to fix data quality issues directly rather than filing tickets and waiting for IT.

**Automation Capabilities:** Data modification skills enable you to automate routine data maintenance tasks.

**Emergency Response:** When data issues occur, you can respond immediately rather than escalating to others.

**Cost Savings:** Companies save thousands by having analysts who can handle both analysis and data management.

## INSERT: Adding New Data

INSERT statements add new records to tables. In business environments, you'll use these for data imports, creating test data, and adding records from analysis results.

### Basic INSERT Syntax

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

### Real-World Scenario: New Member Registration

**Business Context:** You're analyzing membership growth and need to add new members from a registration form.

```sql
-- Single member registration
INSERT INTO members (first_name, last_name, email, phone, join_date, membership_status)
VALUES ('Alex', 'Rodriguez', 'alex.rodriguez@email.com', '(555) 901-2345', '2024-08-18', 'Active');
```

### Bulk Data Import

**Business Context:** You receive a CSV file with 500 new member registrations from an event.

```sql
-- Multiple member registrations
INSERT INTO members (first_name, last_name, email, phone, join_date, membership_status) VALUES
('Maria', 'Garcia', 'maria.garcia@email.com', '(555) 111-2222', '2024-08-18', 'Active'),
('James', 'Chen', 'james.chen@email.com', '(555) 333-4444', '2024-08-18', 'Active'),
('Sarah', 'Williams', 'sarah.williams@email.com', '(555) 555-6666', '2024-08-18', 'Active'),
('Mohammed', 'Ali', 'mohammed.ali@email.com', '(555) 777-8888', '2024-08-18', 'Active'),
('Emma', 'Johnson', 'emma.johnson@email.com', '(555) 999-0000', '2024-08-18', 'Active');
```

### INSERT with SELECT: Data Transformation

**Business Context:** Create a summary table of high-value donors for the marketing team.

```sql
-- First, create the summary table
CREATE TABLE high_value_donors (
    member_id INTEGER,
    member_name TEXT,
    total_donated REAL,
    donation_count INTEGER,
    first_donation_date DATE,
    last_donation_date DATE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Insert data from analysis
INSERT INTO high_value_donors (member_id, member_name, total_donated, donation_count, first_donation_date, last_donation_date)
SELECT 
    m.member_id,
    m.first_name || ' ' || m.last_name,
    SUM(d.amount),
    COUNT(d.donation_id),
    MIN(d.donation_date),
    MAX(d.donation_date)
FROM members m
INNER JOIN donations d ON m.member_id = d.member_id
GROUP BY m.member_id, m.first_name, m.last_name
HAVING SUM(d.amount) > 500;
```

**Business Value:** This creates a dedicated table for marketing campaigns targeting high-value donors.

## UPDATE: Modifying Existing Data

UPDATE statements change existing records. This is where most data modification errors occur, so we'll focus heavily on safety practices.

### Basic UPDATE Syntax

```sql
UPDATE table_name 
SET column1 = value1, column2 = value2
WHERE condition;
```

### ⚠️ Critical Safety Rule: Always Use WHERE Clauses

**NEVER do this in production:**
```sql
UPDATE members SET membership_status = 'Inactive';  -- Updates ALL records!
```

**Always do this:**
```sql
UPDATE members 
SET membership_status = 'Inactive' 
WHERE last_activity_date < '2022-01-01';
```

### Real-World Update Scenarios

#### Scenario 1: Data Quality Improvement

**Business Context:** Phone numbers were imported without consistent formatting. Standardize them for better analysis.

```sql
-- First, let's see what we're working with
SELECT DISTINCT phone 
FROM members 
WHERE phone IS NOT NULL 
ORDER BY phone;

-- Update phone numbers to standard format
UPDATE members 
SET phone = '(' || substr(phone, 1, 3) || ') ' || substr(phone, 4, 3) || '-' || substr(phone, 7, 4)
WHERE phone LIKE '##########' AND length(phone) = 10;

-- Handle different formats
UPDATE members 
SET phone = '(' || substr(phone, 1, 3) || ') ' || substr(phone, 5, 3) || '-' || substr(phone, 9, 4)
WHERE phone LIKE '###-###-####';
```

#### Scenario 2: Status Updates Based on Activity

**Business Context:** Automatically update member status based on recent activity.

```sql
-- Mark members as inactive if no donations or event registrations in 18 months
UPDATE members 
SET membership_status = 'Inactive'
WHERE member_id IN (
    SELECT m.member_id 
    FROM members m
    LEFT JOIN donations d ON m.member_id = d.member_id 
        AND d.donation_date > date('now', '-18 months')
    LEFT JOIN event_registrations er ON m.member_id = er.member_id 
        AND er.registration_date > date('now', '-18 months')
    WHERE d.member_id IS NULL 
      AND er.member_id IS NULL
      AND m.membership_status = 'Active'
);
```

#### Scenario 3: Bulk Price Updates

**Business Context:** Increase all event registration fees by 10% for inflation adjustment.

```sql
-- Update event fees with proper rounding
UPDATE events 
SET registration_fee = ROUND(registration_fee * 1.10, 2)
WHERE event_date > date('now')  -- Only future events
  AND registration_fee > 0;     -- Only paid events
```

### Advanced UPDATE Techniques

#### Conditional Updates with CASE

**Business Context:** Apply different membership discounts based on donation history.

```sql
UPDATE members 
SET discount_percentage = 
    CASE 
        WHEN member_id IN (
            SELECT member_id FROM donations 
            GROUP BY member_id 
            HAVING SUM(amount) > 1000
        ) THEN 20  -- VIP discount
        WHEN member_id IN (
            SELECT member_id FROM donations 
            GROUP BY member_id 
            HAVING SUM(amount) > 500
        ) THEN 15  -- Premium discount
        WHEN member_id IN (
            SELECT member_id FROM donations 
            GROUP BY member_id 
            HAVING COUNT(*) > 5
        ) THEN 10  -- Loyal member discount
        ELSE 5     -- Standard discount
    END
WHERE membership_status = 'Active';
```

## DELETE: Removing Data Safely

DELETE is the most dangerous data modification operation. In production environments, deletions are often replaced with "soft deletes" (marking records as deleted rather than removing them).

### Basic DELETE Syntax

```sql
DELETE FROM table_name WHERE condition;
```

### Safe Deletion Practices

#### 1. Always Test with SELECT First

**Before deleting:**
```sql
-- See what will be deleted
SELECT * FROM donations 
WHERE amount < 0;  -- Identify erroneous negative donations
```

**Then delete:**
```sql
-- Remove erroneous records
DELETE FROM donations 
WHERE amount < 0;
```

#### 2. Implement Soft Deletes

Instead of permanent deletion, add a `deleted_at` column:

```sql
-- Add deletion tracking column
ALTER TABLE members ADD COLUMN deleted_at DATETIME;

-- "Delete" a member (soft delete)
UPDATE members 
SET deleted_at = CURRENT_TIMESTAMP,
    membership_status = 'Deleted'
WHERE member_id = 999;

-- Query active members (excluding soft-deleted)
SELECT * FROM members 
WHERE deleted_at IS NULL;
```

#### 3. Cascading Deletes with Business Logic

**Business Context:** When removing a member, handle their related data appropriately.

```sql
-- First, archive their donations to a history table
INSERT INTO donation_history 
SELECT *, CURRENT_TIMESTAMP as archived_at 
FROM donations 
WHERE member_id = 999;

-- Cancel their future event registrations
UPDATE event_registrations 
SET payment_status = 'Cancelled'
WHERE member_id = 999 
  AND event_id IN (
    SELECT event_id FROM events WHERE event_date > date('now')
  );

-- Finally, soft delete the member
UPDATE members 
SET deleted_at = CURRENT_TIMESTAMP,
    membership_status = 'Deleted'
WHERE member_id = 999;
```

## Transaction Management: All or Nothing

Transactions ensure that multiple related changes either all succeed or all fail together. This is crucial for data integrity.

### Basic Transaction Syntax

```sql
BEGIN TRANSACTION;
-- Multiple related operations
COMMIT;  -- Save changes
-- OR
ROLLBACK;  -- Undo changes
```

### Real-World Transaction Example

**Business Context:** Process a donation that includes updating multiple tables and sending a thank-you email flag.

```sql
BEGIN TRANSACTION;

-- Insert the donation
INSERT INTO donations (member_id, amount, donation_date, purpose, payment_method)
VALUES (5, 250.00, '2024-08-18', 'Building Fund', 'Credit Card');

-- Update member's total giving (cached field for performance)
UPDATE members 
SET total_lifetime_giving = total_lifetime_giving + 250.00,
    last_donation_date = '2024-08-18'
WHERE member_id = 5;

-- Check if member qualifies for VIP status
UPDATE members 
SET membership_status = 'VIP'
WHERE member_id = 5 
  AND total_lifetime_giving >= 1000
  AND membership_status != 'VIP';

-- Flag for thank-you email
INSERT INTO email_queue (member_id, email_type, scheduled_date)
VALUES (5, 'donation_thank_you', '2024-08-18');

-- If all operations succeed, commit
COMMIT;
```

If any operation fails, the entire transaction rolls back, maintaining data consistency.

## Data Validation and Constraints

Proper data validation prevents many modification errors before they occur.

### Adding Constraints to Existing Tables

```sql
-- Ensure donation amounts are positive
ALTER TABLE donations ADD CONSTRAINT check_positive_amount 
CHECK (amount > 0);

-- Ensure email uniqueness
CREATE UNIQUE INDEX idx_unique_member_email ON members(email);

-- Ensure valid membership status
ALTER TABLE members ADD CONSTRAINT check_valid_status 
CHECK (membership_status IN ('Active', 'Inactive', 'VIP', 'Deleted'));
```

### Validation Before Modification

**Always validate data before bulk operations:**

```sql
-- Check for data quality issues before bulk update
SELECT 
    'Invalid emails' as issue_type,
    COUNT(*) as count
FROM members 
WHERE email NOT LIKE '%@%.%'

UNION ALL

SELECT 
    'Duplicate emails' as issue_type,
    COUNT(*) - COUNT(DISTINCT email) as count
FROM members

UNION ALL

SELECT 
    'Missing required fields' as issue_type,
    COUNT(*) as count
FROM members 
WHERE first_name IS NULL OR last_name IS NULL;
```

---

## Practical Session: Data Management Operations

*Real-world scenarios that test your data modification skills.*

### Scenario 1: Monthly Data Maintenance

**Business Requirement:** Implement monthly data cleanup procedures.

```sql
-- Monthly data maintenance script
BEGIN TRANSACTION;

-- 1. Update member statuses based on activity
UPDATE members 
SET membership_status = 'Inactive'
WHERE membership_status = 'Active'
  AND member_id NOT IN (
    SELECT DISTINCT member_id 
    FROM donations 
    WHERE donation_date > date('now', '-12 months')
    UNION
    SELECT DISTINCT member_id 
    FROM event_registrations 
    WHERE registration_date > date('now', '-12 months')
  );

-- 2. Clean up cancelled event registrations older than 1 year
DELETE FROM event_registrations 
WHERE payment_status = 'Cancelled' 
  AND registration_date < date('now', '-1 year');

-- 3. Archive old donation records to history table
INSERT INTO donation_archive 
SELECT *, CURRENT_TIMESTAMP as archived_at 
FROM donations 
WHERE donation_date < date('now', '-3 years');

-- 4. Update cached totals for performance
UPDATE members 
SET total_lifetime_giving = COALESCE((
    SELECT SUM(amount) 
    FROM donations 
    WHERE donations.member_id = members.member_id
), 0);

-- 5. Log maintenance completion
INSERT INTO maintenance_log (operation_type, completion_date, records_affected)
VALUES ('monthly_cleanup', CURRENT_TIMESTAMP, changes());

COMMIT;
```

### Scenario 2: Data Import Processing

**Business Context:** Process a batch import of 1,000 new members from a marketing event.

```sql
-- Create staging table for import validation
CREATE TEMP TABLE member_staging (
    first_name TEXT,
    last_name TEXT,
    email TEXT,
    phone TEXT,
    source_event TEXT
);

-- Insert imported data (normally from CSV)
INSERT INTO member_staging VALUES
('John', 'Doe', 'john.doe@email.com', '5551234567', 'Tech Conference 2024'),
('Jane', 'Smith', 'jane.smith@email.com', '5552345678', 'Tech Conference 2024');
-- ... (would be 1,000 records in real scenario)

-- Validate data quality
SELECT 
    'Total records' as metric,
    COUNT(*) as count
FROM member_staging

UNION ALL

SELECT 
    'Invalid emails' as metric,
    COUNT(*) as count
FROM member_staging 
WHERE email NOT LIKE '%@%.%'

UNION ALL

SELECT 
    'Duplicate emails in import' as metric,
    COUNT(*) - COUNT(DISTINCT email) as count
FROM member_staging

UNION ALL

SELECT 
    'Emails already exist' as metric,
    COUNT(*) as count
FROM member_staging s
INNER JOIN members m ON s.email = m.email;

-- Process valid records only
BEGIN TRANSACTION;

INSERT INTO members (first_name, last_name, email, phone, join_date, membership_status, source)
SELECT 
    first_name,
    last_name,
    email,
    '(' || substr(phone, 1, 3) || ') ' || substr(phone, 4, 3) || '-' || substr(phone, 7, 4),
    date('now'),
    'Active',
    source_event
FROM member_staging
WHERE email LIKE '%@%.%'  -- Valid email format
  AND email NOT IN (SELECT email FROM members WHERE email IS NOT NULL)  -- Not duplicate
  AND first_name IS NOT NULL 
  AND last_name IS NOT NULL;

-- Log import results
INSERT INTO import_log (import_date, source, records_imported, records_rejected)
SELECT 
    date('now'),
    'Tech Conference 2024',
    changes(),
    (SELECT COUNT(*) FROM member_staging) - changes();

COMMIT;
```

### Scenario 3: Event Registration Processing

**Business Context:** Process event registrations and handle payment confirmations.

```sql
-- Process pending registrations when payments are confirmed
CREATE TEMP TABLE payment_confirmations (
    registration_id INTEGER,
    payment_amount REAL,
    payment_date DATE,
    transaction_id TEXT
);

-- Sample payment confirmations
INSERT INTO payment_confirmations VALUES
(1, 75.00, '2024-08-18', 'TXN001'),
(2, 75.00, '2024-08-18', 'TXN002');

BEGIN TRANSACTION;

-- Update registration status
UPDATE event_registrations 
SET payment_status = 'Paid',
    payment_date = pc.payment_date,
    transaction_id = pc.transaction_id
FROM payment_confirmations pc
WHERE event_registrations.registration_id = pc.registration_id;

-- Create donation records for registration fees
INSERT INTO donations (member_id, amount, donation_date, purpose, payment_method, reference_id)
SELECT 
    er.member_id,
    pc.payment_amount,
    pc.payment_date,
    'Event Registration: ' || e.event_name,
    'Credit Card',
    'REG-' || er.registration_id
FROM event_registrations er
INNER JOIN payment_confirmations pc ON er.registration_id = pc.registration_id
INNER JOIN events e ON er.event_id = e.event_id;

-- Update member last activity date
UPDATE members 
SET last_activity_date = date('now')
WHERE member_id IN (
    SELECT er.member_id 
    FROM event_registrations er
    INNER JOIN payment_confirmations pc ON er.registration_id = pc.registration_id
);

-- Queue confirmation emails
INSERT INTO email_queue (member_id, email_type, event_id, scheduled_date)
SELECT 
    er.member_id,
    'event_registration_confirmation',
    er.event_id,
    date('now')
FROM event_registrations er
INNER JOIN payment_confirmations pc ON er.registration_id = pc.registration_id;

COMMIT;
```

## Error Handling and Debugging

### Common Modification Errors

**1. Foreign Key Violations**
```sql
-- This will fail if member_id 999 doesn't exist
INSERT INTO donations (member_id, amount, donation_date, purpose)
VALUES (999, 100.00, '2024-08-18', 'General Fund');

-- Solution: Validate first
INSERT INTO donations (member_id, amount, donation_date, purpose)
SELECT 999, 100.00, '2024-08-18', 'General Fund'
WHERE EXISTS (SELECT 1 FROM members WHERE member_id = 999);
```

**2. Data Type Mismatches**
```sql
-- This will fail: trying to insert text into numeric field
UPDATE donations SET amount = 'One Hundred' WHERE donation_id = 1;

-- Solution: Validate data types
UPDATE donations 
SET amount = CASE 
    WHEN CAST(new_amount_text AS REAL) IS NOT NULL 
    THEN CAST(new_amount_text AS REAL)
    ELSE amount  -- Keep original if conversion fails
END;
```

**3. Constraint Violations**
```sql
-- This will fail if email already exists
INSERT INTO members (first_name, last_name, email)
VALUES ('John', 'Smith', 'existing@email.com');

-- Solution: Use INSERT OR REPLACE or INSERT OR IGNORE
INSERT OR IGNORE INTO members (first_name, last_name, email)
VALUES ('John', 'Smith', 'existing@email.com');
```

## Performance Considerations

### Optimizing Bulk Operations

**Slow: Row-by-row updates**
```sql
-- Don't do this for large datasets
UPDATE members SET last_activity_date = '2024-08-18' WHERE member_id = 1;
UPDATE members SET last_activity_date = '2024-08-18' WHERE member_id = 2;
-- ... thousands of individual updates
```

**Fast: Batch updates**
```sql
-- Do this instead
UPDATE members 
SET last_activity_date = '2024-08-18'
WHERE member_id IN (1, 2, 3, 4, 5, /* ... list all IDs */);
```

**Fastest: JOIN-based updates**
```sql
UPDATE members 
SET last_activity_date = '2024-08-18'
WHERE member_id IN (
    SELECT member_id 
    FROM recent_activity_list
);
```

### Using Indexes for Faster Modifications

```sql
-- Create indexes on commonly updated columns
CREATE INDEX idx_members_status ON members(membership_status);
CREATE INDEX idx_donations_member_date ON donations(member_id, donation_date);
CREATE INDEX idx_events_date ON events(event_date);
```

## Audit Trail Implementation

Professional systems track all data changes for compliance and debugging.

```sql
-- Create audit table
CREATE TABLE audit_log (
    audit_id SERIAL PRIMARY KEY,
    table_name VARCHAR(50),
    operation_type VARCHAR(10),  -- INSERT, UPDATE, DELETE
    record_id INTEGER,
    old_values TEXT,      -- JSON of old values
    new_values TEXT,      -- JSON of new values
    changed_by VARCHAR(100),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- PostgreSQL uses functions instead of triggers for audit logging
-- Example audit function (more advanced than SQLite triggers)
CREATE OR REPLACE FUNCTION audit_donation_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation_type, record_id, old_values, new_values, changed_by)
        VALUES ('donations', 'UPDATE', NEW.donation_id, 
                row_to_json(OLD), row_to_json(NEW), current_user);
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Create trigger using the function
CREATE TRIGGER audit_donations_update
    AFTER UPDATE ON donations
    FOR EACH ROW
    EXECUTE FUNCTION audit_donation_changes();
BEGIN
    INSERT INTO audit_log (table_name, operation_type, record_id, old_values, new_values, changed_by)
    VALUES (
        'donations',
        'UPDATE',
        NEW.donation_id,
        json_object('amount', OLD.amount, 'purpose', OLD.purpose),
        json_object('amount', NEW.amount, 'purpose', NEW.purpose),
        'system_user'
    );
END;
```

## Job Interview Scenarios

### Interview Question 1: Data Migration

"How would you migrate 100,000 customer records from an old system to a new one while ensuring data integrity?"

**Answer:**
```sql
-- 1. Create staging table with validation
CREATE TABLE customer_migration_staging AS 
SELECT * FROM old_system_customers WHERE 1=0;  -- Copy structure only

-- 2. Add validation columns
ALTER TABLE customer_migration_staging ADD COLUMN validation_status TEXT;
ALTER TABLE customer_migration_staging ADD COLUMN validation_errors TEXT;

-- 3. Load and validate data
INSERT INTO customer_migration_staging 
SELECT *, 'pending', NULL FROM old_system_customers;

-- 4. Mark validation status
UPDATE customer_migration_staging 
SET validation_status = 'invalid',
    validation_errors = 'Missing email'
WHERE email IS NULL OR email = '';

UPDATE customer_migration_staging 
SET validation_status = 'invalid',
    validation_errors = COALESCE(validation_errors || '; ', '') || 'Invalid email format'
WHERE email NOT LIKE '%@%.%' AND validation_status != 'invalid';

-- 5. Migrate valid records in batches
BEGIN TRANSACTION;
INSERT INTO customers (customer_name, email, phone, address)
SELECT customer_name, email, phone, address
FROM customer_migration_staging 
WHERE validation_status != 'invalid'
LIMIT 1000;  -- Process in batches
COMMIT;
```

### Interview Question 2: Data Quality Cleanup

"Clean up a database where customer names have inconsistent capitalization and formatting."

**Answer:**
```sql
-- Analyze current state
SELECT 
    'Mixed case names' as issue,
    COUNT(*) as count
FROM customers 
WHERE customer_name != upper(substr(customer_name,1,1)) || lower(substr(customer_name,2))

UNION ALL

SELECT 
    'Multiple spaces' as issue,
    COUNT(*) as count
FROM customers 
WHERE customer_name LIKE '%  %'

UNION ALL

SELECT 
    'Leading/trailing spaces' as issue,
    COUNT(*) as count
FROM customers 
WHERE customer_name != trim(customer_name);

-- Fix formatting issues
BEGIN TRANSACTION;

-- Remove extra spaces
UPDATE customers 
SET customer_name = trim(replace(replace(customer_name, '  ', ' '), '  ', ' '))
WHERE customer_name LIKE '%  %' OR customer_name != trim(customer_name);

-- Fix capitalization (proper case)
UPDATE customers 
SET customer_name = upper(substr(customer_name,1,1)) || lower(substr(customer_name,2))
WHERE customer_name != upper(substr(customer_name,1,1)) || lower(substr(customer_name,2));

COMMIT;
```

## What Makes You Production-Ready

After mastering today's content, you can:

✅ **Safely modify production data** without causing system failures  
✅ **Implement proper transaction management** for data integrity  
✅ **Handle bulk data operations** efficiently  
✅ **Create audit trails** for compliance requirements  
✅ **Validate data quality** before and after modifications  
✅ **Troubleshoot data modification errors** quickly  
✅ **Optimize modification performance** for large datasets  
✅ **Implement soft deletes** and data archiving strategies  

These skills qualify you for roles like:
- Senior Data Analyst with database responsibilities
- Data Operations Specialist  
- Business Intelligence Developer
- Database Administrator (entry-level)
- Data Quality Manager

## Chap 5 Focus

Tomorrow, we'll design databases from scratch. You'll learn:

- Entity-relationship modeling
- Normalization principles
- Index strategy design
- Performance optimization planning
- Scalability considerations

These database design skills are essential for senior positions where you're not just using existing systems, but architecting new ones.

---

*Day 4 Complete! You now have the data modification skills that make you trusted with production systems. Tomorrow, we'll add database design expertise to make you indispensable.*

**Ishmael Abayateye**
