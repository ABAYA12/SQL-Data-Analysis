# Day 5: Database Design - Building the Foundation

*By Ishmael Abayateye*
* Remeber are explanatory words like scenarios are for Learning purposes*

Today we move from using databases to designing them. This is the skill that transforms you from a data analyst into a data architect - someone who shapes how organizations store, organize, and access their most valuable asset: information.

## The Database That Changed Everything

Three years ago, I was hired by a fast-growing startup to solve their "data problem." They had grown from 5 employees to 150 in just 18 months, and their data infrastructure was falling apart.

Their "database" was actually 47 different Excel files, 12 Google Sheets, 3 different SaaS tools that didn't talk to each other, and a MySQL database that someone had cobbled together with no clear structure. The CEO told me, "Ishmael, we're spending $50,000 a month on data analysts, but we can't get basic answers like 'How many customers did we acquire last month?' without a week of manual work."

The problem wasn't the tools or the people - it was the foundation. No one had designed a proper database structure that could grow with the business.

I spent two weeks analyzing their business processes, understanding their data relationships, and designing a normalized database structure. The result? A system that:

- Reduced report generation time from days to minutes
- Eliminated duplicate data entry across 5 departments
- Enabled real-time dashboards for executives
- Scaled to handle 10x more data without performance issues
- Saved the company $200,000 annually in manual data processing

That project taught me the fundamental truth: **Good database design isn't just about organizing data - it's about enabling business success.**

Today, you'll learn to design databases that can scale, perform, and adapt to changing business needs.

## Understanding Business Requirements

Before writing a single CREATE TABLE statement, successful database designers spend time understanding the business. Every table, column, and relationship should serve a specific business purpose.

### The Business Analysis Framework

**1. Identify Core Business Entities**
What are the "things" your business cares about?
- Customers, Products, Orders, Employees, Locations

**2. Define Business Processes**
How do these entities interact?
- Customer places Order containing Products
- Employee processes Order at Location

**3. Determine Information Needs**
What questions does the business need to answer?
- Which products sell best in each location?
- What's the average customer lifetime value?
- Which employees are most productive?

**4. Plan for Growth**
How will these needs change as the business scales?
- More products, more customers, more locations
- New business processes and requirements
- Integration with additional systems

### Real-World Example: E-commerce Database Design

Let's design a database for an online retail company. I'll walk you through my actual process from a recent consulting project.

**Business Requirements Discovery:**

The client told me: "We sell products online. We need to track customers, orders, and inventory."

But through deeper questioning, I discovered:

- Products have variants (size, color, material)
- Customers can have multiple shipping addresses  
- Orders can be split into multiple shipments
- They plan to add marketplace selling (Amazon, eBay)
- They want to implement loyalty programs
- They need to track supplier information
- They plan international expansion

This expanded a "simple" 3-table database into a comprehensive 15-table system.

## Entity-Relationship Modeling

ER modeling is how we visualize and plan database relationships before building them.

### Core Entity Types

**Strong Entities:** Can exist independently
- Customer, Product, Employee, Location

**Weak Entities:** Depend on other entities
- Order Item (depends on Order)
- Product Variant (depends on Product)

**Associative Entities:** Represent relationships with attributes
- Customer Order (Customer + Order + order details)
- Product Category Assignment (Product + Category + assignment date)

### Relationship Types

**One-to-One (1:1)**
- Customer ↔ Customer Profile
- Employee ↔ Employee Details

**One-to-Many (1:M)**
- Customer → Orders (one customer can have many orders)
- Category → Products (one category contains many products)

**Many-to-Many (M:M)**
- Products ↔ Orders (products can be in many orders, orders can contain many products)
- Students ↔ Courses (students can take many courses, courses can have many students)

### Designing Our E-commerce Database

Let's build this step by step:

```sql
-- 1. Core Business Entities

-- Customers
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone TEXT,
    date_registered DATE NOT NULL DEFAULT CURRENT_DATE,
    customer_status TEXT DEFAULT 'active' CHECK (customer_status IN ('active', 'inactive', 'suspended')),
    loyalty_points INTEGER DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Product Categories
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    parent_category_id INTEGER,
    category_description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INTEGER DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);

-- Products
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    product_description TEXT,
    category_id INTEGER NOT NULL,
    brand TEXT,
    base_price DECIMAL(10,2) NOT NULL,
    cost_price DECIMAL(10,2),
    weight_grams INTEGER,
    dimensions_cm TEXT, -- JSON: {"length": 10, "width": 5, "height": 3}
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

-- Product Variants (sizes, colors, etc.)
CREATE TABLE product_variants (
    variant_id SERIAL PRIMARY KEY,
    product_id INTEGER NOT NULL,
    variant_name VARCHAR(100) NOT NULL, -- "Large Red" or "XL Blue"
    sku TEXT UNIQUE NOT NULL,
    price_adjustment DECIMAL(10,2) DEFAULT FALSE.00,
    stock_quantity INTEGER DEFAULT FALSE,
    variant_attributes TEXT, -- JSON: {"size": "L", "color": "red"}
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Customer Addresses
CREATE TABLE customer_addresses (
    address_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    address_type TEXT DEFAULT 'shipping' CHECK (address_type IN ('shipping', 'billing')),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    company TEXT,
    address_line1 VARCHAR(100) NOT NULL,
    address_line2 TEXT,
    city VARCHAR(100) NOT NULL,
    state_province VARCHAR(100) NOT NULL,
    postal_code VARCHAR(100) NOT NULL,
    country VARCHAR(100) NOT NULL DEFAULT 'US',
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Orders
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_number TEXT UNIQUE NOT NULL, -- Human-readable order number
    order_status TEXT DEFAULT 'pending' CHECK (order_status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded')),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    shipping_address_id INTEGER NOT NULL,
    billing_address_id INTEGER NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    tax_amount DECIMAL(10,2) DEFAULT FALSE.00,
    shipping_amount DECIMAL(10,2) DEFAULT FALSE.00,
    discount_amount DECIMAL(10,2) DEFAULT FALSE.00,
    total_amount DECIMAL(10,2) NOT NULL,
    payment_status TEXT DEFAULT 'pending' CHECK (payment_status IN ('pending', 'paid', 'failed', 'refunded')),
    shipping_method TEXT,
    tracking_number TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (shipping_address_id) REFERENCES customer_addresses(address_id),
    FOREIGN KEY (billing_address_id) REFERENCES customer_addresses(address_id)
);

-- Order Items (resolves many-to-many between orders and products)
CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL,
    variant_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (variant_id) REFERENCES product_variants(variant_id)
);
```

## Normalization: Eliminating Data Redundancy

Normalization is the process of organizing data to reduce redundancy and improve data integrity. It's based on a series of "normal forms."

### First Normal Form (1NF): Atomic Values

**Rule:** Each column should contain only atomic (indivisible) values.

**Violation Example:**
```sql
-- DON'T DO THIS
CREATE TABLE bad_customers (
    customer_id INTEGER,
    name TEXT,
    phones TEXT  -- "555-1234, 555-5678, 555-9012"
);
```

**Correct 1NF Design:**
```sql
-- DO THIS
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    first_name TEXT,
    last_name TEXT
);

CREATE TABLE customer_phones (
    phone_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    phone_type TEXT, -- 'home', 'work', 'mobile'
    phone_number TEXT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

### Second Normal Form (2NF): No Partial Dependencies

**Rule:** Remove partial dependencies on composite keys.

**Violation Example:**
```sql
-- DON'T DO THIS
CREATE TABLE bad_order_items (
    order_id INTEGER,
    product_id INTEGER,
    product_name TEXT,  -- Depends only on product_id, not the full key
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
```

**Correct 2NF Design:**
```sql
-- DO THIS
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,
    product_name TEXT
);

CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

### Third Normal Form (3NF): No Transitive Dependencies

**Rule:** Remove columns that depend on other non-key columns.

**Violation Example:**
```sql
-- DON'T DO THIS
CREATE TABLE bad_orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    customer_email TEXT,  -- Depends on customer_id, not order_id
    order_date DATE
);
```

**Correct 3NF Design:**
```sql
-- DO THIS
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY,
    customer_email TEXT
);

CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

### When to Denormalize for Performance

Sometimes, strict normalization hurts performance. Strategic denormalization can improve query speed:

```sql
-- Normalized (slower queries)
SELECT c.customer_name, SUM(oi.total_price)
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id  
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.customer_name;

-- Denormalized (faster queries)
CREATE TABLE customer_summary (
    customer_id INTEGER PRIMARY KEY,
    customer_name TEXT,
    total_lifetime_value DECIMAL(10,2),
    last_order_date DATE,
    order_count INTEGER,
    -- Updated via triggers or scheduled jobs
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

## Index Strategy for Performance

Indexes are like the index in a book - they help find information quickly without scanning everything.

### Types of Indexes

**Primary Index:** Automatically created for primary keys
```sql
-- Automatic when you define PRIMARY KEY
CREATE TABLE customers (
    customer_id INTEGER PRIMARY KEY  -- Index created automatically
);
```

**Unique Index:** Ensures uniqueness and speeds up lookups
```sql
CREATE UNIQUE INDEX idx_customer_email ON customers(email);
CREATE UNIQUE INDEX idx_product_sku ON product_variants(sku);
```

**Composite Index:** Covers multiple columns
```sql
-- For queries filtering by customer_id and order_date
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- For queries filtering by status and date
CREATE INDEX idx_orders_status_date ON orders(order_status, order_date);
```

**Partial Index:** Only indexes rows meeting certain conditions
```sql
-- Only index active products
CREATE INDEX idx_active_products ON products(category_id) WHERE is_active = 1;

-- Only index recent orders
CREATE INDEX idx_recent_orders ON orders(customer_id) WHERE order_date > '2024-01-01';
```

### Index Strategy Guidelines

**Always Index:**
- Primary keys (automatic)
- Foreign keys (manual)
- Columns used in WHERE clauses
- Columns used in ORDER BY
- Columns used in JOINs

**Consider Indexing:**
- Columns used in GROUP BY
- Columns with high selectivity (many unique values)
- Frequently searched text fields

**Avoid Indexing:**
- Columns with low selectivity (gender, boolean)
- Frequently updated columns
- Very wide columns
- Tables with high INSERT/UPDATE volume

### Real-World Index Examples

```sql
-- E-commerce performance indexes
CREATE INDEX idx_products_category_active ON products(category_id, is_active);
CREATE INDEX idx_orders_customer_status ON orders(customer_id, order_status);
CREATE INDEX idx_orders_date_status ON orders(order_date, order_status);
CREATE INDEX idx_order_items_variant ON order_items(variant_id);
CREATE INDEX idx_variants_product_active ON product_variants(product_id, is_active);
CREATE INDEX idx_addresses_customer_type ON customer_addresses(customer_id, address_type);

-- Text search indexes
CREATE INDEX idx_products_name ON products(product_name);
CREATE INDEX idx_customers_name ON customers(last_name, first_name);
```

## Advanced Database Design Patterns

### Audit Trail Pattern

Track all changes for compliance and debugging:

```sql
-- Generic audit table
CREATE TABLE audit_log (
    audit_id SERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    record_id INTEGER NOT NULL,
    operation VARCHAR(100) NOT NULL CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
    old_values TEXT, -- JSON
    new_values TEXT, -- JSON
    changed_by TEXT,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger example for products table
CREATE TRIGGER audit_products_changes
    AFTER UPDATE ON products
    FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, record_id, operation, old_values, new_values, changed_by)
    VALUES (
        'products',
        NEW.product_id,
        'UPDATE',
        json_object('name', OLD.product_name, 'price', OLD.base_price),
        json_object('name', NEW.product_name, 'price', NEW.base_price),
        'system'
    );
END;
```

### Soft Delete Pattern

Mark records as deleted instead of removing them:

```sql
-- Add deletion tracking to tables
ALTER TABLE products ADD COLUMN deleted_at DATETIME;
ALTER TABLE customers ADD COLUMN deleted_at DATETIME;

-- Create view for active records
CREATE VIEW active_products AS
SELECT * FROM products WHERE deleted_at IS NULL;

-- Soft delete function
UPDATE products 
SET deleted_at = CURRENT_TIMESTAMP, is_active = 0
WHERE product_id = 123;
```

### Hierarchy Pattern

Handle tree-like data structures:

```sql
-- Categories with unlimited nesting
CREATE TABLE categories (
    category_id INTEGER PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    parent_category_id INTEGER,
    level INTEGER DEFAULT FALSE,
    path TEXT, -- "/electronics/computers/laptops/"
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);

-- Query all subcategories
WITH RECURSIVE category_tree AS (
    -- Root categories
    SELECT category_id, category_name, parent_category_id, 0 as level
    FROM categories 
    WHERE parent_category_id IS NULL
    
    UNION ALL
    
    -- Child categories
    SELECT c.category_id, c.category_name, c.parent_category_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_category_id = ct.category_id
)
SELECT * FROM category_tree ORDER BY level, category_name;
```

### Polymorphic Association Pattern

When an entity can belong to multiple types:

```sql
-- Comments that can belong to products, orders, or customers
CREATE TABLE comments (
    comment_id INTEGER PRIMARY KEY,
    commentable_type VARCHAR(100) NOT NULL, -- 'product', 'order', 'customer'
    commentable_id INTEGER NOT NULL,
    comment_text VARCHAR(100) NOT NULL,
    created_by INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(user_id)
);

-- Query comments for a specific product
SELECT * FROM comments 
WHERE commentable_type = 'product' AND commentable_id = 123;
```

---

## Practical Session: Complete Database Design

*Design a comprehensive database for a real business scenario.*

### Scenario: Fitness Center Management System

**Business Requirements:**
You're designing a database for a chain of fitness centers. They need to track:

- Multiple gym locations
- Members with different membership types
- Classes and personal training sessions
- Equipment and maintenance
- Staff and their specializations
- Member check-ins and activity tracking
- Billing and payments

### Step 1: Entity Identification

```sql
-- Locations
CREATE TABLE locations (
    location_id SERIAL PRIMARY KEY,
    location_name VARCHAR(100) NOT NULL,
    address VARCHAR(100) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    zip_code VARCHAR(100) NOT NULL,
    phone TEXT,
    manager_id INTEGER,
    opening_hours TEXT, -- JSON
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Membership Types
CREATE TABLE membership_types (
    membership_type_id SERIAL PRIMARY KEY,
    type_name VARCHAR(100) NOT NULL,
    description TEXT,
    monthly_price DECIMAL(8,2) NOT NULL,
    annual_price DECIMAL(8,2),
    benefits TEXT, -- JSON array of benefits
    max_locations INTEGER, -- NULL for unlimited
    max_guests INTEGER DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE
);

-- Members
CREATE TABLE members (
    member_id SERIAL PRIMARY KEY,
    membership_number TEXT UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email TEXT UNIQUE NOT NULL,
    phone TEXT,
    date_of_birth DATE,
    gender TEXT CHECK (gender IN ('M', 'F', 'Other', 'Prefer not to say')),
    emergency_contact_name TEXT,
    emergency_contact_phone TEXT,
    membership_type_id INTEGER NOT NULL,
    home_location_id INTEGER NOT NULL,
    join_date DATE NOT NULL DEFAULT CURRENT_DATE,
    membership_status TEXT DEFAULT 'active' CHECK (membership_status IN ('active', 'frozen', 'cancelled', 'expired')),
    billing_day INTEGER DEFAULT TRUE CHECK (billing_day BETWEEN 1 AND 28),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (membership_type_id) REFERENCES membership_types(membership_type_id),
    FOREIGN KEY (home_location_id) REFERENCES locations(location_id)
);

-- Staff
CREATE TABLE staff (
    staff_id SERIAL PRIMARY KEY,
    employee_id TEXT UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email TEXT UNIQUE NOT NULL,
    phone TEXT,
    position VARCHAR(100) NOT NULL,
    location_id INTEGER NOT NULL,
    hire_date DATE NOT NULL,
    hourly_rate DECIMAL(8,2),
    is_active BOOLEAN DEFAULT TRUE,
    certifications TEXT, -- JSON array
    specializations TEXT, -- JSON array
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
);

-- Equipment Categories
CREATE TABLE equipment_categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL,
    description TEXT
);

-- Equipment
CREATE TABLE equipment (
    equipment_id SERIAL PRIMARY KEY,
    equipment_name VARCHAR(100) NOT NULL,
    category_id INTEGER NOT NULL,
    location_id INTEGER NOT NULL,
    manufacturer TEXT,
    model TEXT,
    serial_number TEXT,
    purchase_date DATE,
    purchase_price DECIMAL(10,2),
    warranty_expiry DATE,
    status TEXT DEFAULT 'operational' CHECK (status IN ('operational', 'maintenance', 'out_of_order', 'retired')),
    last_maintenance_date DATE,
    next_maintenance_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES equipment_categories(category_id),
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
);
```

### Step 2: Activity and Class Management

```sql
-- Class Types
CREATE TABLE class_types (
    class_type_id SERIAL PRIMARY KEY,
    class_name VARCHAR(100) NOT NULL,
    description TEXT,
    duration_minutes INTEGER NOT NULL,
    max_capacity INTEGER NOT NULL,
    difficulty_level TEXT CHECK (difficulty_level IN ('Beginner', 'Intermediate', 'Advanced', 'All Levels')),
    equipment_required TEXT, -- JSON array
    is_active BOOLEAN DEFAULT TRUE
);

-- Class Schedules
CREATE TABLE class_schedules (
    schedule_id SERIAL PRIMARY KEY,
    class_type_id INTEGER NOT NULL,
    location_id INTEGER NOT NULL,
    instructor_id INTEGER NOT NULL,
    day_of_week VARCHAR(100) NOT NULL CHECK (day_of_week IN ('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday')),
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    room_number TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    effective_date DATE NOT NULL,
    expiry_date DATE,
    FOREIGN KEY (class_type_id) REFERENCES class_types(class_type_id),
    FOREIGN KEY (location_id) REFERENCES locations(location_id),
    FOREIGN KEY (instructor_id) REFERENCES staff(staff_id)
);

-- Class Instances (specific class occurrences)
CREATE TABLE class_instances (
    instance_id SERIAL PRIMARY KEY,
    schedule_id INTEGER NOT NULL,
    class_date DATE NOT NULL,
    actual_start_time DATETIME,
    actual_end_time DATETIME,
    instructor_id INTEGER NOT NULL,
    substitute_instructor_id INTEGER,
    capacity INTEGER NOT NULL,
    enrolled_count INTEGER DEFAULT FALSE,
    status TEXT DEFAULT 'scheduled' CHECK (status IN ('scheduled', 'in_progress', 'completed', 'cancelled')),
    cancellation_reason TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (schedule_id) REFERENCES class_schedules(schedule_id),
    FOREIGN KEY (instructor_id) REFERENCES staff(staff_id),
    FOREIGN KEY (substitute_instructor_id) REFERENCES staff(staff_id)
);

-- Class Registrations
CREATE TABLE class_registrations (
    registration_id SERIAL PRIMARY KEY,
    instance_id INTEGER NOT NULL,
    member_id INTEGER NOT NULL,
    registration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    attendance_status TEXT DEFAULT 'registered' CHECK (attendance_status IN ('registered', 'attended', 'no_show', 'cancelled')),
    registration_source TEXT DEFAULT 'app' CHECK (registration_source IN ('app', 'website', 'front_desk', 'phone')),
    waitlist_position INTEGER,
    checked_in_at DATETIME,
    FOREIGN KEY (instance_id) REFERENCES class_instances(instance_id),
    FOREIGN KEY (member_id) REFERENCES members(member_id),
    UNIQUE(instance_id, member_id)
);
```

### Step 3: Check-ins and Activity Tracking

```sql
-- Member Check-ins
CREATE TABLE member_checkins (
    checkin_id SERIAL PRIMARY KEY,
    member_id INTEGER NOT NULL,
    location_id INTEGER NOT NULL,
    checkin_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    checkout_time DATETIME,
    entry_method TEXT DEFAULT 'card' CHECK (entry_method IN ('card', 'app', 'manual', 'guest_pass')),
    guest_count INTEGER DEFAULT FALSE,
    staff_id INTEGER, -- Who processed the check-in
    notes TEXT,
    FOREIGN KEY (member_id) REFERENCES members(member_id),
    FOREIGN KEY (location_id) REFERENCES locations(location_id),
    FOREIGN KEY (staff_id) REFERENCES staff(staff_id)
);

-- Personal Training Sessions
CREATE TABLE personal_training_sessions (
    session_id SERIAL PRIMARY KEY,
    member_id INTEGER NOT NULL,
    trainer_id INTEGER NOT NULL,
    location_id INTEGER NOT NULL,
    session_date DATE NOT NULL,
    start_time TIME NOT NULL,
    end_time TIME NOT NULL,
    session_type TEXT DEFAULT 'personal' CHECK (session_type IN ('personal', 'semi_private', 'assessment')),
    status TEXT DEFAULT 'scheduled' CHECK (status IN ('scheduled', 'completed', 'cancelled', 'no_show')),
    price DECIMAL(8,2) NOT NULL,
    notes TEXT,
    workout_plan TEXT, -- JSON or text
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (member_id) REFERENCES members(member_id),
    FOREIGN KEY (trainer_id) REFERENCES staff(staff_id),
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
);
```

### Step 4: Billing and Payments

```sql
-- Member Billing
CREATE TABLE member_billing (
    billing_id SERIAL PRIMARY KEY,
    member_id INTEGER NOT NULL,
    billing_period_start DATE NOT NULL,
    billing_period_end DATE NOT NULL,
    membership_fee DECIMAL(8,2) NOT NULL,
    personal_training_fee DECIMAL(8,2) DEFAULT FALSE.00,
    other_charges DECIMAL(8,2) DEFAULT FALSE.00,
    discounts DECIMAL(8,2) DEFAULT FALSE.00,
    tax_amount DECIMAL(8,2) DEFAULT FALSE.00,
    total_amount DECIMAL(8,2) NOT NULL,
    due_date DATE NOT NULL,
    billing_status TEXT DEFAULT 'pending' CHECK (billing_status IN ('pending', 'paid', 'overdue', 'cancelled')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (member_id) REFERENCES members(member_id)
);

-- Payments
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    billing_id INTEGER NOT NULL,
    payment_amount DECIMAL(8,2) NOT NULL,
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    payment_method VARCHAR(100) NOT NULL CHECK (payment_method IN ('credit_card', 'debit_card', 'bank_transfer', 'cash', 'check')),
    transaction_id TEXT,
    payment_status TEXT DEFAULT 'completed' CHECK (payment_status IN ('pending', 'completed', 'failed', 'refunded')),
    processed_by INTEGER,
    notes TEXT,
    FOREIGN KEY (billing_id) REFERENCES member_billing(billing_id),
    FOREIGN KEY (processed_by) REFERENCES staff(staff_id)
);
```

### Step 5: Performance Indexes

```sql
-- Essential indexes for performance
CREATE INDEX idx_members_status ON members(membership_status);
CREATE INDEX idx_members_location ON members(home_location_id);
CREATE INDEX idx_members_type ON members(membership_type_id);
CREATE INDEX idx_members_email ON members(email);

CREATE INDEX idx_checkins_member_date ON member_checkins(member_id, checkin_time);
CREATE INDEX idx_checkins_location_date ON member_checkins(location_id, checkin_time);

CREATE INDEX idx_class_instances_date ON class_instances(class_date);
CREATE INDEX idx_class_instances_schedule ON class_instances(schedule_id, class_date);

CREATE INDEX idx_registrations_member ON class_registrations(member_id);
CREATE INDEX idx_registrations_instance ON class_registrations(instance_id);

CREATE INDEX idx_billing_member_period ON member_billing(member_id, billing_period_start);
CREATE INDEX idx_billing_status_due ON member_billing(billing_status, due_date);

CREATE INDEX idx_payments_billing ON payments(billing_id);
CREATE INDEX idx_payments_date ON payments(payment_date);

CREATE INDEX idx_equipment_location_status ON equipment(location_id, status);
CREATE INDEX idx_staff_location_active ON staff(location_id, is_active);
```

### Step 6: Business Intelligence Views

```sql
-- Member activity summary
CREATE VIEW member_activity_summary AS
SELECT 
    m.member_id,
    m.first_name || ' ' || m.last_name AS member_name,
    m.membership_status,
    mt.type_name AS membership_type,
    COUNT(DISTINCT ci.checkin_id) AS total_checkins,
    COUNT(DISTINCT cr.registration_id) AS total_class_registrations,
    COUNT(DISTINCT pts.session_id) AS total_training_sessions,
    MAX(ci.checkin_time) AS last_checkin,
    AVG(CASE WHEN ci.checkout_time IS NOT NULL 
        THEN (julianday(ci.checkout_time) - julianday(ci.checkin_time)) * 24 
        ELSE NULL END) AS avg_workout_hours
FROM members m
LEFT JOIN membership_types mt ON m.membership_type_id = mt.membership_type_id
LEFT JOIN member_checkins ci ON m.member_id = ci.member_id
LEFT JOIN class_registrations cr ON m.member_id = cr.member_id
LEFT JOIN personal_training_sessions pts ON m.member_id = pts.member_id
GROUP BY m.member_id, m.first_name, m.last_name, m.membership_status, mt.type_name;

-- Location performance dashboard
CREATE VIEW location_performance AS
SELECT 
    l.location_id,
    l.location_name,
    COUNT(DISTINCT m.member_id) AS total_members,
    COUNT(DISTINCT CASE WHEN m.membership_status = 'active' THEN m.member_id END) AS active_members,
    COUNT(DISTINCT ci.checkin_id) AS total_checkins_today,
    COUNT(DISTINCT cls.instance_id) AS classes_today,
    COUNT(DISTINCT pts.session_id) AS training_sessions_today,
    COUNT(DISTINCT e.equipment_id) AS total_equipment,
    COUNT(DISTINCT CASE WHEN e.status = 'operational' THEN e.equipment_id END) AS operational_equipment
FROM locations l
LEFT JOIN members m ON l.location_id = m.home_location_id
LEFT JOIN member_checkins ci ON l.location_id = ci.location_id AND date(ci.checkin_time) = date('now')
LEFT JOIN class_instances cls ON l.location_id = (SELECT location_id FROM class_schedules WHERE schedule_id = cls.schedule_id) AND cls.class_date = date('now')
LEFT JOIN personal_training_sessions pts ON l.location_id = pts.location_id AND pts.session_date = date('now')
LEFT JOIN equipment e ON l.location_id = e.location_id
GROUP BY l.location_id, l.location_name;
```

## Database Design Interview Questions

### Question 1: Scalability Planning

"How would you design a database that can handle 10x growth in users and data?"

**Answer Framework:**
```sql
-- 1. Horizontal partitioning strategy
CREATE TABLE members_2024 (
    -- Same structure as members
    CHECK (join_date >= '2024-01-01' AND join_date < '2025-01-01')
) INHERITS (members);

-- 2. Archive strategy
CREATE TABLE archived_checkins AS
SELECT * FROM member_checkins WHERE 1=0;

-- 3. Read replicas simulation
CREATE VIEW member_checkins_readonly AS
SELECT * FROM member_checkins;

-- 4. Caching layer tables
CREATE TABLE member_cache (
    member_id INTEGER PRIMARY KEY,
    cached_data TEXT, -- JSON
    cache_timestamp DATETIME,
    expires_at DATETIME
);
```

### Question 2: Data Migration Strategy

"Design a migration plan from a legacy system to your new database."

**Answer:**
```sql
-- 1. Create staging tables
CREATE TABLE legacy_member_import (
    old_id INTEGER,
    full_name TEXT,
    contact_info TEXT,
    membership_info TEXT,
    import_status TEXT DEFAULT 'pending'
);

-- 2. Data transformation queries
INSERT INTO members (first_name, last_name, email, membership_type_id)
SELECT 
    substr(full_name, 1, instr(full_name, ' ') - 1) as first_name,
    substr(full_name, instr(full_name, ' ') + 1) as last_name,
    json_extract(contact_info, '$.email') as email,
    (SELECT membership_type_id FROM membership_types WHERE type_name = json_extract(membership_info, '$.type'))
FROM legacy_member_import
WHERE import_status = 'pending'
  AND json_valid(contact_info)
  AND json_valid(membership_info);

-- 3. Validation and rollback plan
CREATE TABLE migration_log (
    migration_id INTEGER PRIMARY KEY,
    table_name TEXT,
    records_processed INTEGER,
    records_successful INTEGER,
    records_failed INTEGER,
    migration_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## What Makes You Architecture-Ready

After mastering today's content, you can:

✅ **Design normalized database schemas** that eliminate redundancy  
✅ **Plan for scalability** from day one  
✅ **Implement proper indexing strategies** for optimal performance  
✅ **Design audit trails and compliance features** for enterprise requirements  
✅ **Handle complex business relationships** with appropriate data models  
✅ **Create migration strategies** for legacy system modernization  
✅ **Implement advanced patterns** like soft deletes and hierarchical data  
✅ **Design for multiple environments** (dev, staging, production)  

These skills qualify you for roles like:
- Senior Data Architect
- Database Design Consultant
- Lead Data Engineer  
- Principal Data Analyst
- Technical Lead for data projects

## Chap 6 Focus

In Chap 6, we'll explore advanced SQL techniques that push the boundaries of what's possible with database queries:

- Window functions and analytical queries
- Recursive CTEs for complex hierarchies
- Advanced aggregation techniques
- Performance optimization strategies
- Stored procedures and functions

These advanced techniques will cement your position as a SQL expert capable of solving the most complex analytical challenges.

---

*Day 5 Complete! You now have the database design skills that make you indispensable to any organization building or modernizing their data infrastructure.*

**Ishmael Abayateye**
