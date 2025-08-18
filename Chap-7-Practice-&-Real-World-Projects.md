# Chap 7: Practice & Real-World Projects - Your SQL Mastery Portfolio

*By Ishmael Abayateye*
* Remember explanatory words like scenarios are for Learning purposes*

Today is your graduation day. After six days of intensive learning, you're ready to tackle the kind of complex, real-world projects that define senior data professionals. This isn't just practice - you're building a portfolio that demonstrates your complete SQL mastery.

## The Project That Launched My Consulting Career

Five years ago, I was presented with what seemed like an impossible challenge. A rapidly growing e-commerce company had just acquired three smaller competitors and needed to integrate their data systems within 60 days to meet investor reporting requirements.

The complexity was staggering:
- Four different database schemas
- Inconsistent customer identification across systems
- Overlapping product catalogs with different naming conventions
- Three years of historical data that needed reconciliation
- Real-time reporting requirements for the board of directors

The previous consultant had quoted 6 months and $500,000. I proposed to deliver it in 60 days for $150,000 using advanced SQL techniques.

Here's what I delivered:

**Week 1:** Complete data mapping and normalization strategy  
**Week 2:** Automated data quality assessment and cleanup  
**Week 3:** Real-time ETL processes using SQL-based procedures  
**Week 4:** Comprehensive business intelligence dashboard  
**Week 5:** Predictive analytics and customer segmentation  
**Week 6:** Performance optimization and documentation  
**Week 7:** Training and knowledge transfer  
**Week 8:** Go-live support and final adjustments  

The project was completed on time, under budget, and became the foundation for their IPO six months later. More importantly, it established my reputation as someone who could solve "impossible" data challenges.

Today, you'll build a similar comprehensive solution that showcases every SQL skill you've learned.

## Project Overview: Comprehensive Business Intelligence System

**Scenario:** You're the Senior Data Analyst for "HealthFirst," a chain of medical clinics that has grown from 3 locations to 25 locations in 18 months. They need a complete business intelligence system to:

1. **Track patient care quality** across all locations
2. **Monitor financial performance** and optimize pricing
3. **Analyze staff productivity** and scheduling efficiency  
4. **Predict patient demand** for resource planning
5. **Ensure regulatory compliance** with healthcare standards
6. **Identify growth opportunities** for expansion

## Phase 1: Database Design & Architecture

### Step 1: Requirements Analysis

Based on stakeholder interviews, we need to track:

**Clinical Operations:**
- Patient demographics and medical history
- Appointments and scheduling
- Medical procedures and treatments
- Staff specializations and availability
- Equipment and facility utilization

**Financial Management:**
- Patient billing and insurance processing
- Revenue by service type and location
- Cost analysis and profitability
- Payment processing and collections

**Regulatory Compliance:**
- Treatment outcome tracking
- Staff certification monitoring
- Patient satisfaction metrics
- Quality assurance reporting

### Step 2: Complete Database Schema

```sql
-- Location Management
CREATE TABLE locations (
    location_id SERIAL PRIMARY KEY,
    location_name VARCHAR(100) NOT NULL,
    address VARCHAR(100) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    zip_code VARCHAR(100) NOT NULL,
    phone TEXT,
    email TEXT,
    manager_id INTEGER,
    opening_date DATE,
    total_capacity INTEGER,
    specialties TEXT, -- JSON array
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Staff Management
CREATE TABLE staff (
    staff_id SERIAL PRIMARY KEY,
    employee_id TEXT UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email TEXT UNIQUE NOT NULL,
    phone TEXT,
    role VARCHAR(100) NOT NULL CHECK (role IN ('Doctor', 'Nurse', 'Technician', 'Administrator', 'Support')),
    specialization TEXT,
    location_id INTEGER NOT NULL,
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2),
    certification_expiry DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
);

-- Patient Management
CREATE TABLE patients (
    patient_id SERIAL PRIMARY KEY,
    patient_number TEXT UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender TEXT CHECK (gender IN ('M', 'F', 'Other', 'Prefer not to say')),
    email TEXT,
    phone TEXT,
    address TEXT,
    city TEXT,
    state TEXT,
    zip_code TEXT,
    emergency_contact_name TEXT,
    emergency_contact_phone TEXT,
    insurance_provider TEXT,
    insurance_policy_number TEXT,
    primary_location_id INTEGER,
    registration_date DATE DEFAULT CURRENT_DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (primary_location_id) REFERENCES locations(location_id)
);

-- Medical Services
CREATE TABLE medical_services (
    service_id SERIAL PRIMARY KEY,
    service_code TEXT UNIQUE NOT NULL,
    service_name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(100) NOT NULL,
    base_price DECIMAL(8,2) NOT NULL,
    duration_minutes INTEGER DEFAULT 30,
    requires_specialist BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Appointments
CREATE TABLE appointments (
    appointment_id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL,
    staff_id INTEGER NOT NULL,
    location_id INTEGER NOT NULL,
    appointment_date DATE NOT NULL,
    appointment_time TIME NOT NULL,
    duration_minutes INTEGER DEFAULT 30,
    appointment_type TEXT DEFAULT 'Regular' CHECK (appointment_type IN ('Regular', 'Follow-up', 'Emergency', 'Consultation')),
    status TEXT DEFAULT 'Scheduled' CHECK (status IN ('Scheduled', 'Confirmed', 'In Progress', 'Completed', 'Cancelled', 'No Show')),
    chief_complaint TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (staff_id) REFERENCES staff(staff_id),
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
);

-- Treatment Records
CREATE TABLE treatments (
    treatment_id SERIAL PRIMARY KEY,
    appointment_id INTEGER NOT NULL,
    service_id INTEGER NOT NULL,
    quantity INTEGER DEFAULT TRUE,
    unit_price DECIMAL(8,2) NOT NULL,
    total_price DECIMAL(8,2) NOT NULL,
    treatment_date DATE NOT NULL,
    performed_by INTEGER NOT NULL,
    diagnosis_code TEXT,
    treatment_notes TEXT,
    outcome TEXT,
    follow_up_required BOOLEAN DEFAULT FALSE,
    follow_up_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (appointment_id) REFERENCES appointments(appointment_id),
    FOREIGN KEY (service_id) REFERENCES medical_services(service_id),
    FOREIGN KEY (performed_by) REFERENCES staff(staff_id)
);

-- Billing and Payments
CREATE TABLE patient_billing (
    billing_id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL,
    treatment_id INTEGER,
    billing_date DATE NOT NULL,
    service_charges DECIMAL(10,2) NOT NULL,
    insurance_coverage DECIMAL(10,2) DEFAULT FALSE.00,
    patient_responsibility DECIMAL(10,2) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    payment_status TEXT DEFAULT 'Pending' CHECK (payment_status IN ('Pending', 'Paid', 'Partial', 'Overdue', 'Cancelled')),
    due_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (treatment_id) REFERENCES treatments(treatment_id)
);

CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    billing_id INTEGER NOT NULL,
    payment_amount DECIMAL(10,2) NOT NULL,
    payment_date DATE NOT NULL,
    payment_method TEXT CHECK (payment_method IN ('Cash', 'Credit Card', 'Debit Card', 'Check', 'Insurance', 'Online')),
    transaction_reference TEXT,
    processed_by INTEGER,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (billing_id) REFERENCES patient_billing(billing_id),
    FOREIGN KEY (processed_by) REFERENCES staff(staff_id)
);

-- Patient Satisfaction
CREATE TABLE patient_feedback (
    feedback_id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL,
    appointment_id INTEGER,
    location_id INTEGER NOT NULL,
    overall_satisfaction INTEGER CHECK (overall_satisfaction BETWEEN 1 AND 5),
    staff_rating INTEGER CHECK (staff_rating BETWEEN 1 AND 5),
    facility_rating INTEGER CHECK (facility_rating BETWEEN 1 AND 5),
    wait_time_rating INTEGER CHECK (wait_time_rating BETWEEN 1 AND 5),
    would_recommend BOOLEAN,
    comments TEXT,
    feedback_date DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (patient_id) REFERENCES patients(patient_id),
    FOREIGN KEY (appointment_id) REFERENCES appointments(appointment_id),
    FOREIGN KEY (location_id) REFERENCES locations(location_id)
);
```

### Step 3: Performance Indexes

```sql
-- Critical performance indexes
CREATE INDEX idx_appointments_date_location ON appointments(appointment_date, location_id);
CREATE INDEX idx_appointments_staff_date ON appointments(staff_id, appointment_date);
CREATE INDEX idx_appointments_patient_date ON appointments(patient_id, appointment_date);
CREATE INDEX idx_treatments_date_service ON treatments(treatment_date, service_id);
CREATE INDEX idx_billing_patient_status ON patient_billing(patient_id, payment_status);
CREATE INDEX idx_billing_date_amount ON patient_billing(billing_date, total_amount);
CREATE INDEX idx_payments_date_method ON payments(payment_date, payment_method);
CREATE INDEX idx_feedback_location_date ON patient_feedback(location_id, feedback_date);
CREATE INDEX idx_staff_location_role ON staff(location_id, role, is_active);

-- Composite indexes for complex queries
CREATE INDEX idx_appointments_complete ON appointments(location_id, appointment_date, status, staff_id);
CREATE INDEX idx_treatments_analysis ON treatments(treatment_date, service_id, performed_by, total_price);
```

## Phase 2: Sample Data Generation

### Step 4: Realistic Test Data

```sql
-- Insert sample locations
INSERT INTO locations (location_name, address, city, state, zip_code, phone, opening_date, total_capacity, specialties) VALUES
('HealthFirst Downtown', '123 Main St', 'Atlanta', 'GA', '30309', '(404) 555-0101', '2022-01-15', 50, '["General Practice", "Cardiology", "Dermatology"]'),
('HealthFirst Midtown', '456 Peachtree St', 'Atlanta', 'GA', '30308', '(404) 555-0102', '2022-03-20', 75, '["General Practice", "Pediatrics", "Orthopedics"]'),
('HealthFirst Buckhead', '789 Lenox Ave', 'Atlanta', 'GA', '30326', '(404) 555-0103', '2022-06-10', 100, '["General Practice", "Surgery", "Radiology"]'),
('HealthFirst Alpharetta', '321 North Point Dr', 'Alpharetta', 'GA', '30022', '(770) 555-0104', '2023-01-05', 60, '["General Practice", "Internal Medicine"]'),
('HealthFirst Marietta', '654 Cobb Pkwy', 'Marietta', 'GA', '30060', '(770) 555-0105', '2023-04-15', 80, '["General Practice", "Family Medicine", "Urgent Care"]');

-- Insert sample staff
INSERT INTO staff (employee_id, first_name, last_name, email, role, specialization, location_id, hire_date, salary, certification_expiry) VALUES
('EMP001', 'Dr. Sarah', 'Johnson', 'sarah.johnson@healthfirst.com', 'Doctor', 'General Practice', 1, '2022-01-20', 180000.00, '2025-12-31'),
('EMP002', 'Dr. Michael', 'Chen', 'michael.chen@healthfirst.com', 'Doctor', 'Cardiology', 1, '2022-02-15', 220000.00, '2025-12-31'),
('EMP003', 'Nurse Lisa', 'Williams', 'lisa.williams@healthfirst.com', 'Nurse', 'General', 1, '2022-01-25', 75000.00, '2024-12-31'),
('EMP004', 'Dr. David', 'Brown', 'david.brown@healthfirst.com', 'Doctor', 'Pediatrics', 2, '2022-03-25', 190000.00, '2025-12-31'),
('EMP005', 'Nurse Jennifer', 'Davis', 'jennifer.davis@healthfirst.com', 'Nurse', 'Pediatric', 2, '2022-04-01', 78000.00, '2024-12-31');

-- Insert sample medical services
INSERT INTO medical_services (service_code, service_name, category, base_price, duration_minutes, requires_specialist) VALUES
('CONSULT001', 'Initial Consultation', 'General', 150.00, 45, 0),
('PHYSICAL001', 'Annual Physical Exam', 'Preventive', 200.00, 60, 0),
('CARDIO001', 'Cardiac Consultation', 'Cardiology', 300.00, 60, 1),
('XRAY001', 'Chest X-Ray', 'Radiology', 125.00, 30, 0),
('BLOOD001', 'Basic Blood Panel', 'Laboratory', 85.00, 15, 0),
('VACCINE001', 'Adult Vaccination', 'Preventive', 45.00, 15, 0),
('URGENT001', 'Urgent Care Visit', 'Urgent Care', 175.00, 30, 0);

-- Generate sample patients (first 10 of many)
INSERT INTO patients (patient_number, first_name, last_name, date_of_birth, gender, email, phone, city, state, insurance_provider, primary_location_id, registration_date) VALUES
('PAT0001', 'John', 'Smith', '1985-03-15', 'M', 'john.smith@email.com', '(404) 123-4567', 'Atlanta', 'GA', 'Blue Cross Blue Shield', 1, '2022-02-01'),
('PAT0002', 'Mary', 'Johnson', '1978-07-22', 'F', 'mary.johnson@email.com', '(404) 234-5678', 'Atlanta', 'GA', 'Aetna', 1, '2022-02-03'),
('PAT0003', 'Robert', 'Williams', '1992-11-08', 'M', 'robert.williams@email.com', '(404) 345-6789', 'Atlanta', 'GA', 'Cigna', 2, '2022-02-05'),
('PAT0004', 'Lisa', 'Brown', '1988-04-12', 'F', 'lisa.brown@email.com', '(770) 456-7890', 'Alpharetta', 'GA', 'United Healthcare', 4, '2022-02-10'),
('PAT0005', 'David', 'Davis', '1975-09-30', 'M', 'david.davis@email.com', '(770) 567-8901', 'Marietta', 'GA', 'Kaiser Permanente', 5, '2022-02-15');

-- Generate comprehensive appointment and treatment data
-- (This would typically be much larger in a real system)
INSERT INTO appointments (patient_id, staff_id, location_id, appointment_date, appointment_time, appointment_type, status, chief_complaint) VALUES
(1, 1, 1, '2024-01-15', '09:00', 'Regular', 'Completed', 'Annual checkup'),
(1, 2, 1, '2024-02-20', '14:30', 'Follow-up', 'Completed', 'Cardiac follow-up'),
(2, 1, 1, '2024-01-18', '10:30', 'Regular', 'Completed', 'Flu symptoms'),
(3, 4, 2, '2024-01-20', '11:00', 'Regular', 'Completed', 'Child wellness visit'),
(4, 1, 1, '2024-01-25', '15:00', 'Regular', 'Completed', 'Back pain'),
(5, 1, 1, '2024-02-01', '09:30', 'Regular', 'Completed', 'Hypertension monitoring');
```

## Phase 3: Business Intelligence Dashboard Queries

### Step 5: Executive Summary Dashboard

```sql
-- Executive KPI Dashboard
CREATE VIEW executive_dashboard AS
WITH monthly_metrics AS (
  SELECT 
    TO_CHAR(appointment_date, 'YYYY-MM') as month,
    l.location_name,
    
    -- Patient Volume Metrics
    COUNT(DISTINCT a.appointment_id) as total_appointments,
    COUNT(DISTINCT a.patient_id) as unique_patients,
    COUNT(DISTINCT CASE WHEN a.status = 'Completed' THEN a.appointment_id END) as completed_appointments,
    COUNT(DISTINCT CASE WHEN a.status = 'No Show' THEN a.appointment_id END) as no_shows,
    
    -- Revenue Metrics
    SUM(t.total_price) as gross_revenue,
    AVG(t.total_price) as avg_revenue_per_treatment,
    
    -- Efficiency Metrics
    AVG(CASE WHEN a.status = 'Completed' THEN a.duration_minutes END) as avg_appointment_duration,
    COUNT(DISTINCT a.staff_id) as active_staff_count
    
  FROM appointments a
  JOIN locations l ON a.location_id = l.location_id
  LEFT JOIN treatments t ON a.appointment_id = t.appointment_id
  WHERE a.appointment_date >= DATE('now', '-12 months')
  GROUP BY TO_CHAR(appointment_date, 'YYYY-MM'), l.location_id, l.location_name
),
current_vs_previous AS (
  SELECT 
    *,
    LAG(total_appointments) OVER (PARTITION BY location_name ORDER BY month) as prev_appointments,
    LAG(gross_revenue) OVER (PARTITION BY location_name ORDER BY month) as prev_revenue,
    LAG(unique_patients) OVER (PARTITION BY location_name ORDER BY month) as prev_patients
  FROM monthly_metrics
)
SELECT 
  month,
  location_name,
  total_appointments,
  unique_patients,
  completed_appointments,
  no_shows,
  ROUND(no_shows * 100.0 / NULLIF(total_appointments, 0), 2) as no_show_rate,
  ROUND(gross_revenue, 2) as revenue,
  ROUND(avg_revenue_per_treatment, 2) as avg_revenue_per_treatment,
  avg_appointment_duration,
  
  -- Growth Calculations
  CASE 
    WHEN prev_appointments > 0 
    THEN ROUND((total_appointments - prev_appointments) * 100.0 / prev_appointments, 2)
    ELSE NULL 
  END as appointment_growth_pct,
  
  CASE 
    WHEN prev_revenue > 0 
    THEN ROUND((gross_revenue - prev_revenue) * 100.0 / prev_revenue, 2)
    ELSE NULL 
  END as revenue_growth_pct,
  
  -- Efficiency Ratios
  ROUND(gross_revenue / NULLIF(total_appointments, 0), 2) as revenue_per_appointment,
  ROUND(total_appointments / NULLIF(active_staff_count, 0), 2) as appointments_per_staff_member

FROM current_vs_previous
ORDER BY month DESC, location_name;
```

### Step 6: Patient Analytics & Segmentation

```sql
-- Advanced Patient Segmentation Analysis
CREATE VIEW patient_segmentation AS
WITH patient_metrics AS (
  SELECT 
    p.patient_id,
    p.first_name || ' ' || p.last_name as patient_name,
    p.date_of_birth,
    ROUND((CURRENT_DATE - p.date_of_birth) / 365.25) as age,
    p.gender,
    p.insurance_provider,
    l.location_name as primary_location,
    
    -- Appointment History
    COUNT(DISTINCT a.appointment_id) as total_appointments,
    MIN(a.appointment_date) as first_appointment,
    MAX(a.appointment_date) as last_appointment,
    CURRENT_DATE - MAX(a.appointment_date) as days_since_last_visit,
    
    -- Financial Metrics
    SUM(pb.total_amount) as total_billed,
    SUM(py.payment_amount) as total_paid,
    SUM(pb.total_amount) - SUM(COALESCE(py.payment_amount, 0)) as outstanding_balance,
    AVG(pb.total_amount) as avg_bill_amount,
    
    -- Service Utilization
    COUNT(DISTINCT t.service_id) as unique_services_used,
    COUNT(DISTINCT CASE WHEN ms.category = 'Preventive' THEN t.treatment_id END) as preventive_treatments,
    COUNT(DISTINCT CASE WHEN ms.category = 'Urgent Care' THEN t.treatment_id END) as urgent_treatments,
    
    -- Satisfaction
    AVG(pf.overall_satisfaction) as avg_satisfaction,
    COUNT(pf.feedback_id) as feedback_count
    
  FROM patients p
  LEFT JOIN locations l ON p.primary_location_id = l.location_id
  LEFT JOIN appointments a ON p.patient_id = a.patient_id
  LEFT JOIN treatments t ON a.appointment_id = t.appointment_id
  LEFT JOIN medical_services ms ON t.service_id = ms.service_id
  LEFT JOIN patient_billing pb ON p.patient_id = pb.patient_id
  LEFT JOIN payments py ON pb.billing_id = py.billing_id
  LEFT JOIN patient_feedback pf ON p.patient_id = pf.patient_id
  GROUP BY p.patient_id, patient_name, p.date_of_birth, p.gender, p.insurance_provider, l.location_name
),
patient_segments AS (
  SELECT 
    *,
    -- Age Segmentation
    CASE 
      WHEN age < 18 THEN 'Pediatric'
      WHEN age BETWEEN 18 AND 35 THEN 'Young Adult'
      WHEN age BETWEEN 36 AND 55 THEN 'Middle Age'
      WHEN age BETWEEN 56 AND 70 THEN 'Senior'
      ELSE 'Elderly'
    END as age_segment,
    
    -- Value Segmentation
    CASE 
      WHEN total_billed >= 2000 THEN 'High Value'
      WHEN total_billed >= 1000 THEN 'Medium Value'
      WHEN total_billed >= 500 THEN 'Low Value'
      ELSE 'Minimal Value'
    END as value_segment,
    
    -- Engagement Segmentation
    CASE 
      WHEN total_appointments >= 10 AND days_since_last_visit <= 90 THEN 'Highly Engaged'
      WHEN total_appointments >= 5 AND days_since_last_visit <= 180 THEN 'Engaged'
      WHEN total_appointments >= 2 AND days_since_last_visit <= 365 THEN 'Moderately Engaged'
      WHEN days_since_last_visit > 365 THEN 'At Risk'
      ELSE 'New/Inactive'
    END as engagement_segment,
    
    -- Health Behavior Segmentation
    CASE 
      WHEN preventive_treatments > urgent_treatments AND preventive_treatments >= 2 THEN 'Proactive'
      WHEN urgent_treatments > preventive_treatments THEN 'Reactive'
      WHEN preventive_treatments > 0 AND urgent_treatments = 0 THEN 'Preventive-Focused'
      ELSE 'Occasional'
    END as health_behavior_segment
    
  FROM patient_metrics
)
SELECT 
  patient_name,
  age,
  age_segment,
  gender,
  primary_location,
  total_appointments,
  days_since_last_visit,
  ROUND(total_billed, 2) as total_billed,
  ROUND(outstanding_balance, 2) as outstanding_balance,
  value_segment,
  engagement_segment,
  health_behavior_segment,
  ROUND(avg_satisfaction, 2) as avg_satisfaction,
  
  -- Risk Scoring
  CASE 
    WHEN engagement_segment = 'At Risk' AND outstanding_balance > 500 THEN 'High Risk'
    WHEN engagement_segment = 'At Risk' OR outstanding_balance > 1000 THEN 'Medium Risk'
    ELSE 'Low Risk'
  END as risk_level,
  
  -- Recommended Actions
  CASE 
    WHEN engagement_segment = 'At Risk' THEN 'Schedule wellness check-in call'
    WHEN outstanding_balance > 500 THEN 'Contact for payment arrangement'
    WHEN avg_satisfaction < 3.5 THEN 'Follow up on service quality'
    WHEN health_behavior_segment = 'Reactive' THEN 'Promote preventive care programs'
    ELSE 'Continue current care plan'
  END as recommended_action

FROM patient_segments
ORDER BY 
  CASE risk_level WHEN 'High Risk' THEN 1 WHEN 'Medium Risk' THEN 2 ELSE 3 END,
  total_billed DESC;
```

### Step 7: Operational Efficiency Analysis

```sql
-- Staff Productivity and Operational Analysis
CREATE VIEW operational_efficiency AS
WITH staff_productivity AS (
  SELECT 
    s.staff_id,
    s.first_name || ' ' || s.last_name as staff_name,
    s.role,
    s.specialization,
    l.location_name,
    
    -- Appointment Metrics
    COUNT(DISTINCT a.appointment_id) as total_appointments,
    COUNT(DISTINCT CASE WHEN a.status = 'Completed' THEN a.appointment_id END) as completed_appointments,
    COUNT(DISTINCT CASE WHEN a.status = 'No Show' THEN a.appointment_id END) as no_shows,
    COUNT(DISTINCT CASE WHEN a.status = 'Cancelled' THEN a.appointment_id END) as cancellations,
    
    -- Time Utilization
    SUM(CASE WHEN a.status = 'Completed' THEN a.duration_minutes ELSE 0 END) as productive_minutes,
    AVG(CASE WHEN a.status = 'Completed' THEN a.duration_minutes END) as avg_appointment_duration,
    
    -- Revenue Generation
    SUM(t.total_price) as revenue_generated,
    AVG(t.total_price) as avg_revenue_per_treatment,
    
    -- Patient Satisfaction
    AVG(pf.staff_rating) as avg_staff_rating,
    COUNT(DISTINCT pf.feedback_id) as feedback_count,
    
    -- Working Days (approximation)
    COUNT(DISTINCT DATE(a.appointment_date)) as working_days
    
  FROM staff s
  JOIN locations l ON s.location_id = l.location_id
  LEFT JOIN appointments a ON s.staff_id = a.staff_id
    AND a.appointment_date >= DATE('now', '-90 days')
  LEFT JOIN treatments t ON a.appointment_id = t.appointment_id
  LEFT JOIN patient_feedback pf ON a.appointment_id = pf.appointment_id
  WHERE s.is_active = 1
  GROUP BY s.staff_id, staff_name, s.role, s.specialization, l.location_name
),
efficiency_metrics AS (
  SELECT 
    *,
    -- Efficiency Calculations
    ROUND(completed_appointments * 100.0 / NULLIF(total_appointments, 0), 2) as completion_rate,
    ROUND(no_shows * 100.0 / NULLIF(total_appointments, 0), 2) as no_show_rate,
    ROUND(revenue_generated / NULLIF(completed_appointments, 0), 2) as revenue_per_completed_appointment,
    ROUND(productive_minutes / NULLIF(working_days, 0), 2) as avg_productive_minutes_per_day,
    ROUND(completed_appointments / NULLIF(working_days, 0), 2) as appointments_per_working_day,
    
    -- Performance Rankings within Role
    RANK() OVER (PARTITION BY role ORDER BY revenue_generated DESC) as revenue_rank_in_role,
    RANK() OVER (PARTITION BY role ORDER BY completed_appointments DESC) as volume_rank_in_role,
    RANK() OVER (PARTITION BY role ORDER BY avg_staff_rating DESC) as satisfaction_rank_in_role
    
  FROM staff_productivity
  WHERE total_appointments > 0  -- Only include staff with appointments
),
performance_categories AS (
  SELECT 
    *,
    -- Overall Performance Score (weighted)
    ROUND(
      (completion_rate * 0.3) +
      (CASE WHEN avg_staff_rating IS NOT NULL THEN avg_staff_rating * 20 ELSE 80 END * 0.25) +
      (CASE WHEN revenue_per_completed_appointment > 0 THEN LEAST(revenue_per_completed_appointment / 10, 100) ELSE 0 END * 0.25) +
      (CASE WHEN appointments_per_working_day > 0 THEN LEAST(appointments_per_working_day * 10, 100) ELSE 0 END * 0.2), 2
    ) as performance_score,
    
    -- Performance Category
    CASE 
      WHEN revenue_rank_in_role <= 2 AND volume_rank_in_role <= 3 AND completion_rate >= 85 THEN 'Top Performer'
      WHEN revenue_rank_in_role <= 3 AND completion_rate >= 80 THEN 'High Performer'
      WHEN completion_rate >= 75 AND no_show_rate <= 15 THEN 'Good Performer'
      WHEN completion_rate >= 65 THEN 'Average Performer'
      ELSE 'Needs Improvement'
    END as performance_category
    
  FROM efficiency_metrics
)
SELECT 
  staff_name,
  role,
  specialization,
  location_name,
  total_appointments,
  completed_appointments,
  completion_rate,
  no_show_rate,
  ROUND(revenue_generated, 2) as revenue_generated,
  revenue_per_completed_appointment,
  appointments_per_working_day,
  avg_productive_minutes_per_day,
  ROUND(avg_staff_rating, 2) as avg_staff_rating,
  performance_score,
  performance_category,
  
  -- Recommendations
  CASE 
    WHEN no_show_rate > 20 THEN 'Focus on appointment confirmation process'
    WHEN completion_rate < 70 THEN 'Review scheduling and time management'
    WHEN avg_staff_rating < 3.5 THEN 'Consider patient communication training'
    WHEN revenue_per_completed_appointment < 100 THEN 'Review service mix and pricing'
    WHEN performance_category = 'Top Performer' THEN 'Consider for mentoring role'
    ELSE 'Continue current performance level'
  END as recommendation

FROM performance_categories
ORDER BY performance_score DESC, revenue_generated DESC;
```

### Step 8: Predictive Analytics for Resource Planning

```sql
-- Demand Forecasting and Resource Planning
CREATE VIEW demand_forecasting AS
WITH historical_demand AS (
  SELECT 
    l.location_id,
    l.location_name,
    DATE(a.appointment_date) as appointment_date,
    strftime('%w', a.appointment_date) as day_of_week, -- 0=Sunday, 6=Saturday
    strftime('%H', a.appointment_time) as hour_of_day,
    COUNT(a.appointment_id) as appointment_count,
    COUNT(CASE WHEN a.status = 'Completed' THEN 1 END) as completed_count,
    COUNT(CASE WHEN a.status = 'No Show' THEN 1 END) as no_show_count,
    AVG(a.duration_minutes) as avg_duration
  FROM appointments a
  JOIN locations l ON a.location_id = l.location_id
  WHERE a.appointment_date >= DATE('now', '-180 days')
    AND a.appointment_date <= DATE('now')
  GROUP BY l.location_id, l.location_name, DATE(a.appointment_date), day_of_week, hour_of_day
),
demand_patterns AS (
  SELECT 
    location_id,
    location_name,
    day_of_week,
    hour_of_day,
    AVG(appointment_count) as avg_appointments,
    STDDEV(appointment_count) as stddev_appointments,
    AVG(completed_count) as avg_completed,
    AVG(no_show_count) as avg_no_shows,
    AVG(avg_duration) as avg_appointment_duration,
    COUNT(*) as data_points
  FROM historical_demand
  GROUP BY location_id, location_name, day_of_week, hour_of_day
  HAVING COUNT(*) >= 5  -- Minimum data points for reliable forecast
),
capacity_analysis AS (
  SELECT 
    dp.*,
    l.total_capacity,
    
    -- Calculate predicted demand with confidence intervals
    ROUND(avg_appointments + (stddev_appointments * 1.96), 0) as upper_demand_forecast,
    ROUND(avg_appointments, 0) as expected_demand_forecast,
    ROUND(GREATEST(avg_appointments - (stddev_appointments * 1.96), 0), 0) as lower_demand_forecast,
    
    -- Capacity utilization
    ROUND(avg_appointments * 100.0 / l.total_capacity, 2) as current_utilization_pct,
    
    -- Staffing recommendations
    CEIL(avg_appointments / 8.0) as recommended_staff_count, -- Assuming 8 appointments per staff per hour
    
    -- Day of week labels
    CASE dp.day_of_week
      WHEN '0' THEN 'Sunday'
      WHEN '1' THEN 'Monday'
      WHEN '2' THEN 'Tuesday'
      WHEN '3' THEN 'Wednesday'
      WHEN '4' THEN 'Thursday'
      WHEN '5' THEN 'Friday'
      WHEN '6' THEN 'Saturday'
    END as day_name
    
  FROM demand_patterns dp
  JOIN locations l ON dp.location_id = l.location_id
),
optimization_recommendations AS (
  SELECT 
    *,
    -- Optimization flags
    CASE 
      WHEN current_utilization_pct > 90 THEN 'High Demand - Consider Expansion'
      WHEN current_utilization_pct > 75 THEN 'Good Utilization'
      WHEN current_utilization_pct > 50 THEN 'Moderate Utilization'
      WHEN current_utilization_pct > 25 THEN 'Low Utilization - Optimize Schedule'
      ELSE 'Very Low Utilization - Investigate'
    END as utilization_status,
    
    CASE 
      WHEN recommended_staff_count > 5 THEN 'Peak Period - Full Staffing'
      WHEN recommended_staff_count > 3 THEN 'Busy Period - Standard Staffing'
      WHEN recommended_staff_count > 1 THEN 'Moderate Period - Reduced Staffing'
      ELSE 'Low Period - Minimum Staffing'
    END as staffing_recommendation
    
  FROM capacity_analysis
)
SELECT 
  location_name,
  day_name,
  CASE 
    WHEN hour_of_day = '08' THEN '8:00 AM'
    WHEN hour_of_day = '09' THEN '9:00 AM'
    WHEN hour_of_day = '10' THEN '10:00 AM'
    WHEN hour_of_day = '11' THEN '11:00 AM'
    WHEN hour_of_day = '12' THEN '12:00 PM'
    WHEN hour_of_day = '13' THEN '1:00 PM'
    WHEN hour_of_day = '14' THEN '2:00 PM'
    WHEN hour_of_day = '15' THEN '3:00 PM'
    WHEN hour_of_day = '16' THEN '4:00 PM'
    WHEN hour_of_day = '17' THEN '5:00 PM'
    ELSE hour_of_day || ':00'
  END as time_slot,
  expected_demand_forecast,
  lower_demand_forecast,
  upper_demand_forecast,
  current_utilization_pct,
  recommended_staff_count,
  utilization_status,
  staffing_recommendation,
  
  -- Weekly pattern insights
  AVG(expected_demand_forecast) OVER (
    PARTITION BY location_id, day_of_week
  ) as daily_avg_demand,
  
  -- Peak hour identification
  CASE 
    WHEN expected_demand_forecast = MAX(expected_demand_forecast) OVER (
      PARTITION BY location_id, day_of_week
    ) THEN 'Peak Hour'
    WHEN expected_demand_forecast >= AVG(expected_demand_forecast) OVER (
      PARTITION BY location_id, day_of_week
    ) THEN 'Above Average'
    ELSE 'Below Average'
  END as demand_level

FROM optimization_recommendations
WHERE hour_of_day BETWEEN '08' AND '17'  -- Business hours
ORDER BY location_name, day_of_week, hour_of_day;
```

## Phase 4: Advanced Analytics & Machine Learning Features

### Step 9: Patient Risk Stratification

```sql
-- Advanced Patient Risk Assessment and Stratification
CREATE VIEW patient_risk_assessment AS
WITH patient_risk_factors AS (
  SELECT 
    p.patient_id,
    p.first_name || ' ' || p.last_name as patient_name,
    ROUND((CURRENT_DATE - p.date_of_birth) / 365.25) as age,
    p.gender,
    
    -- Appointment Pattern Analysis
    COUNT(DISTINCT a.appointment_id) as total_visits,
    COUNT(DISTINCT CASE WHEN a.appointment_type = 'Emergency' THEN a.appointment_id END) as emergency_visits,
    COUNT(DISTINCT CASE WHEN a.status = 'No Show' THEN a.appointment_id END) as no_show_count,
    CURRENT_DATE - MAX(a.appointment_date) as days_since_last_visit,
    
    -- Clinical Indicators
    COUNT(DISTINCT CASE WHEN ms.category = 'Urgent Care' THEN t.treatment_id END) as urgent_treatments,
    COUNT(DISTINCT CASE WHEN ms.category = 'Preventive' THEN t.treatment_id END) as preventive_treatments,
    COUNT(DISTINCT t.diagnosis_code) as unique_diagnoses,
    
    -- Follow-up Compliance
    COUNT(CASE WHEN t.follow_up_required = 1 THEN 1 END) as follow_ups_required,
    COUNT(CASE WHEN t.follow_up_required = 1 AND t.follow_up_date <= DATE('now') THEN 1 END) as overdue_follow_ups,
    
    -- Financial Risk Factors
    SUM(pb.total_amount) as total_charges,
    SUM(pb.patient_responsibility) as patient_responsibility,
    SUM(py.payment_amount) as total_payments,
    COUNT(CASE WHEN pb.payment_status = 'Overdue' THEN 1 END) as overdue_bills,
    
    -- Satisfaction Indicators
    AVG(pf.overall_satisfaction) as avg_satisfaction,
    COUNT(CASE WHEN pf.overall_satisfaction <= 2 THEN 1 END) as poor_satisfaction_count
    
  FROM patients p
  LEFT JOIN appointments a ON p.patient_id = a.patient_id
  LEFT JOIN treatments t ON a.appointment_id = t.appointment_id
  LEFT JOIN medical_services ms ON t.service_id = ms.service_id
  LEFT JOIN patient_billing pb ON p.patient_id = pb.patient_id
  LEFT JOIN payments py ON pb.billing_id = py.billing_id
  LEFT JOIN patient_feedback pf ON p.patient_id = pf.patient_id
  GROUP BY p.patient_id, patient_name, age, p.gender
),
risk_scoring AS (
  SELECT 
    *,
    
    -- Calculate individual risk scores (0-100 scale)
    CASE 
      WHEN age >= 75 THEN 25
      WHEN age >= 65 THEN 20
      WHEN age >= 55 THEN 15
      WHEN age >= 45 THEN 10
      ELSE 5
    END as age_risk_score,
    
    CASE 
      WHEN emergency_visits >= 3 THEN 30
      WHEN emergency_visits >= 2 THEN 20
      WHEN emergency_visits >= 1 THEN 10
      ELSE 0
    END as emergency_risk_score,
    
    CASE 
      WHEN urgent_treatments > preventive_treatments * 2 THEN 25
      WHEN urgent_treatments > preventive_treatments THEN 15
      WHEN preventive_treatments = 0 AND urgent_treatments > 0 THEN 20
      ELSE 0
    END as care_pattern_risk_score,
    
    CASE 
      WHEN days_since_last_visit > 730 THEN 20  -- 2+ years
      WHEN days_since_last_visit > 365 THEN 15  -- 1+ year
      WHEN days_since_last_visit > 180 THEN 10  -- 6+ months
      ELSE 0
    END as engagement_risk_score,
    
    CASE 
      WHEN overdue_follow_ups > 0 THEN overdue_follow_ups * 10
      ELSE 0
    END as compliance_risk_score,
    
    CASE 
      WHEN overdue_bills > 2 THEN 20
      WHEN overdue_bills > 0 THEN 10
      WHEN (total_charges - total_payments) > 1000 THEN 15
      ELSE 0
    END as financial_risk_score,
    
    CASE 
      WHEN poor_satisfaction_count > 1 THEN 15
      WHEN poor_satisfaction_count > 0 THEN 10
      WHEN avg_satisfaction < 3.0 THEN 10
      ELSE 0
    END as satisfaction_risk_score
    
  FROM patient_risk_factors
),
comprehensive_risk AS (
  SELECT 
    *,
    
    -- Composite Risk Score
    age_risk_score + 
    emergency_risk_score + 
    care_pattern_risk_score + 
    engagement_risk_score + 
    compliance_risk_score + 
    financial_risk_score + 
    satisfaction_risk_score as total_risk_score,
    
    -- Outstanding Balance
    ROUND(COALESCE(total_charges - total_payments, 0), 2) as outstanding_balance
    
  FROM risk_scoring
)
SELECT 
  patient_name,
  age,
  gender,
  total_visits,
  emergency_visits,
  days_since_last_visit,
  overdue_follow_ups,
  outstanding_balance,
  total_risk_score,
  
  -- Risk Stratification
  CASE 
    WHEN total_risk_score >= 80 THEN 'Critical Risk'
    WHEN total_risk_score >= 60 THEN 'High Risk'
    WHEN total_risk_score >= 40 THEN 'Medium Risk'
    WHEN total_risk_score >= 20 THEN 'Low Risk'
    ELSE 'Minimal Risk'
  END as risk_category,
  
  -- Specific Risk Factors
  CASE 
    WHEN age_risk_score >= 20 THEN 'Age, '
    ELSE ''
  END ||
  CASE 
    WHEN emergency_risk_score >= 20 THEN 'Emergency Pattern, '
    ELSE ''
  END ||
  CASE 
    WHEN care_pattern_risk_score >= 15 THEN 'Poor Care Patterns, '
    ELSE ''
  END ||
  CASE 
    WHEN engagement_risk_score >= 15 THEN 'Poor Engagement, '
    ELSE ''
  END ||
  CASE 
    WHEN compliance_risk_score >= 10 THEN 'Non-Compliance, '
    ELSE ''
  END ||
  CASE 
    WHEN financial_risk_score >= 15 THEN 'Financial Issues, '
    ELSE ''
  END ||
  CASE 
    WHEN satisfaction_risk_score >= 10 THEN 'Satisfaction Issues'
    ELSE ''
  END as primary_risk_factors,
  
  -- Intervention Recommendations
  CASE 
    WHEN total_risk_score >= 80 THEN 'Immediate intervention required - Assign care coordinator'
    WHEN total_risk_score >= 60 THEN 'Proactive outreach - Schedule comprehensive review'
    WHEN total_risk_score >= 40 THEN 'Enhanced monitoring - Quarterly check-ins'
    WHEN total_risk_score >= 20 THEN 'Standard monitoring - Annual wellness visits'
    ELSE 'Continue routine care'
  END as recommended_intervention,
  
  -- Priority Score for Care Management
  CASE 
    WHEN total_risk_score >= 80 THEN 1
    WHEN total_risk_score >= 60 THEN 2
    WHEN total_risk_score >= 40 THEN 3
    ELSE 4
  END as intervention_priority

FROM comprehensive_risk
WHERE total_visits > 0  -- Only include patients with visit history
ORDER BY total_risk_score DESC, outstanding_balance DESC;
```

## Phase 5: Executive Reporting Suite

### Step 10: Comprehensive Business Performance Report

```sql
-- Master Business Performance Report
CREATE VIEW master_business_report AS
WITH location_performance AS (
  SELECT 
    l.location_id,
    l.location_name,
    l.opening_date,
    
    -- Patient Metrics
    COUNT(DISTINCT p.patient_id) as total_patients,
    COUNT(DISTINCT CASE WHEN p.registration_date >= DATE('now', '-30 days') THEN p.patient_id END) as new_patients_30d,
    COUNT(DISTINCT a.appointment_id) as total_appointments,
    COUNT(DISTINCT CASE WHEN a.appointment_date >= DATE('now', '-30 days') THEN a.appointment_id END) as appointments_30d,
    
    -- Revenue Metrics
    SUM(pb.total_amount) as total_revenue,
    SUM(CASE WHEN pb.billing_date >= DATE('now', '-30 days') THEN pb.total_amount ELSE 0 END) as revenue_30d,
    SUM(py.payment_amount) as total_collections,
    SUM(pb.total_amount) - SUM(COALESCE(py.payment_amount, 0)) as outstanding_ar,
    
    -- Quality Metrics
    AVG(pf.overall_satisfaction) as avg_satisfaction,
    COUNT(CASE WHEN pf.overall_satisfaction >= 4 THEN 1 END) * 100.0 / NULLIF(COUNT(pf.feedback_id), 0) as satisfaction_rate,
    
    -- Efficiency Metrics
    COUNT(CASE WHEN a.status = 'No Show' THEN 1 END) * 100.0 / NULLIF(COUNT(a.appointment_id), 0) as no_show_rate,
    AVG(CASE WHEN a.status = 'Completed' THEN a.duration_minutes END) as avg_appointment_duration,
    
    -- Staff Metrics
    COUNT(DISTINCT s.staff_id) as total_staff,
    COUNT(DISTINCT CASE WHEN s.role = 'Doctor' THEN s.staff_id END) as doctor_count,
    COUNT(DISTINCT CASE WHEN s.role = 'Nurse' THEN s.staff_id END) as nurse_count
    
  FROM locations l
  LEFT JOIN patients p ON l.location_id = p.primary_location_id
  LEFT JOIN appointments a ON l.location_id = a.location_id
  LEFT JOIN patient_billing pb ON p.patient_id = pb.patient_id
  LEFT JOIN payments py ON pb.billing_id = py.billing_id
  LEFT JOIN patient_feedback pf ON l.location_id = pf.location_id
  LEFT JOIN staff s ON l.location_id = s.location_id AND s.is_active = 1
  GROUP BY l.location_id, l.location_name, l.opening_date
),
benchmarks AS (
  SELECT 
    AVG(total_revenue / NULLIF(total_patients, 0)) as avg_revenue_per_patient,
    AVG(satisfaction_rate) as avg_satisfaction_rate,
    AVG(no_show_rate) as avg_no_show_rate,
    AVG(total_appointments / NULLIF(total_staff, 0)) as avg_appointments_per_staff
  FROM location_performance
)
SELECT 
  lp.location_name,
  lp.opening_date,
  CURRENT_DATE - lp.opening_date as days_operational,
  
  -- Patient Metrics
  lp.total_patients,
  lp.new_patients_30d,
  lp.total_appointments,
  lp.appointments_30d,
  
  -- Revenue Metrics
  ROUND(lp.total_revenue, 2) as total_revenue,
  ROUND(lp.revenue_30d, 2) as revenue_30d,
  ROUND(lp.total_collections, 2) as total_collections,
  ROUND(lp.outstanding_ar, 2) as outstanding_ar,
  ROUND(lp.outstanding_ar * 100.0 / NULLIF(lp.total_revenue, 0), 2) as ar_percentage,
  
  -- Per-Patient Metrics
  ROUND(lp.total_revenue / NULLIF(lp.total_patients, 0), 2) as revenue_per_patient,
  ROUND(lp.total_appointments / NULLIF(lp.total_patients, 0), 2) as visits_per_patient,
  
  -- Quality Metrics
  ROUND(lp.avg_satisfaction, 2) as avg_satisfaction,
  ROUND(lp.satisfaction_rate, 2) as satisfaction_rate,
  
  -- Efficiency Metrics
  ROUND(lp.no_show_rate, 2) as no_show_rate,
  ROUND(lp.avg_appointment_duration, 1) as avg_appointment_duration,
  
  -- Staffing Metrics
  lp.total_staff,
  lp.doctor_count,
  lp.nurse_count,
  ROUND(lp.total_appointments / NULLIF(lp.total_staff, 0), 2) as appointments_per_staff,
  
  -- Performance vs Benchmarks
  CASE 
    WHEN (lp.total_revenue / NULLIF(lp.total_patients, 0)) > b.avg_revenue_per_patient * 1.1 THEN 'Above Average'
    WHEN (lp.total_revenue / NULLIF(lp.total_patients, 0)) < b.avg_revenue_per_patient * 0.9 THEN 'Below Average'
    ELSE 'Average'
  END as revenue_performance,
  
  CASE 
    WHEN lp.satisfaction_rate > b.avg_satisfaction_rate * 1.05 THEN 'Above Average'
    WHEN lp.satisfaction_rate < b.avg_satisfaction_rate * 0.95 THEN 'Below Average'
    ELSE 'Average'
  END as satisfaction_performance,
  
  CASE 
    WHEN lp.no_show_rate < b.avg_no_show_rate * 0.8 THEN 'Above Average'
    WHEN lp.no_show_rate > b.avg_no_show_rate * 1.2 THEN 'Below Average'
    ELSE 'Average'
  END as efficiency_performance,
  
  -- Overall Performance Score
  ROUND(
    CASE WHEN (lp.total_revenue / NULLIF(lp.total_patients, 0)) > b.avg_revenue_per_patient * 1.1 THEN 25 
         WHEN (lp.total_revenue / NULLIF(lp.total_patients, 0)) > b.avg_revenue_per_patient * 0.9 THEN 20 
         ELSE 15 END +
    CASE WHEN lp.satisfaction_rate > b.avg_satisfaction_rate * 1.05 THEN 25 
         WHEN lp.satisfaction_rate > b.avg_satisfaction_rate * 0.95 THEN 20 
         ELSE 15 END +
    CASE WHEN lp.no_show_rate < b.avg_no_show_rate * 0.8 THEN 25 
         WHEN lp.no_show_rate < b.avg_no_show_rate * 1.2 THEN 20 
         ELSE 15 END +
    CASE WHEN (lp.total_appointments / NULLIF(lp.total_staff, 0)) > b.avg_appointments_per_staff * 1.1 THEN 25 
         WHEN (lp.total_appointments / NULLIF(lp.total_staff, 0)) > b.avg_appointments_per_staff * 0.9 THEN 20 
         ELSE 15 END, 2
  ) as overall_performance_score

FROM location_performance lp
CROSS JOIN benchmarks b
ORDER BY overall_performance_score DESC, total_revenue DESC;
```

## Project Deliverables & Portfolio Showcase

### Your Complete SQL Mastery Portfolio

**1. Database Architecture Document**
- Complete ERD with business justification
- Normalization analysis and design decisions
- Index strategy and performance considerations
- Scalability and maintenance plans

**2. Business Intelligence Dashboard Suite**
- Executive summary KPIs
- Operational efficiency metrics
- Patient analytics and segmentation
- Staff productivity analysis
- Financial performance tracking

**3. Advanced Analytics Implementation**
- Predictive demand forecasting
- Risk stratification algorithms
- Anomaly detection systems
- Statistical analysis models

**4. Performance Optimization Report**
- Query execution plan analysis
- Index utilization assessment
- Performance tuning recommendations
- Scalability projections

**5. Documentation Package**
- Complete SQL code library
- Business logic documentation
- User guides and training materials
- Implementation timeline and methodology

## Real-World Application Examples

### For Job Interviews

**Senior Data Analyst Position:**
"Walk me through your approach to building a comprehensive BI system for a healthcare organization."

**Your Response:** Present the HealthFirst project, emphasizing:
- Requirements gathering and stakeholder analysis
- Database design methodology and normalization
- Advanced SQL techniques for complex analytics
- Performance optimization and scalability planning
- Business impact and ROI measurement

### For Consulting Proposals

**Client Challenge:** "We need better visibility into our operational performance."

**Your Solution:** Demonstrate with your portfolio:
- Executive dashboard with real-time KPIs
- Predictive analytics for resource planning
- Risk assessment and patient segmentation
- Staff productivity optimization
- Financial performance analysis

### For Team Leadership

**Technical Architecture Decisions:**
- Database design patterns and best practices
- Query optimization strategies
- Performance monitoring and tuning
- Data quality and governance frameworks
- Scalability and maintenance planning

## What You've Accomplished

**🎯 Complete SQL Mastery:** From basic queries to advanced analytics  
**🏗️ Database Architecture:** Design and optimization expertise  
**📊 Business Intelligence:** Executive-level reporting and dashboards  
**🔮 Predictive Analytics:** Statistical modeling and forecasting  
**⚡ Performance Optimization:** Enterprise-scale query tuning  
**🎯 Problem Solving:** Complex business challenge resolution  

## Your Next Steps

**Immediate Actions:**
1. **Build Your GitHub Portfolio:** Upload your complete project code
2. **Create Case Studies:** Document your methodology and results
3. **Practice Presentations:** Prepare to demonstrate your work
4. **Network and Apply:** Target senior analyst and architect roles
5. **Continue Learning:** Stay current with new SQL features and tools

**Career Progression Path:**
- **Data Analyst** → **Principal Data Architect** → **Director of Analytics** → **Chief Data Officer**

Your SQL journey is complete, but your career adventure is just beginning. You now possess the skills that command premium salaries and solve million-dollar business problems.

---

*Day 7 Complete! Congratulations on completing your SQL mastery journey. You're now equipped with the expertise to tackle any data challenge and advance your career to the highest levels.*

**Welcome to the ranks of SQL experts. The data world is yours to command.**

**Ishmael Abayateye**
