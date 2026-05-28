# 📊 Customer 360 & RFM Analytics Platform

An analytical platform centered around customers to evaluate behaviors, perform segmentation, and optimize end-of-year marketing campaigns for maximizing business revenue.

---

## 📌 Project Overview
In a business scenario where the Marketing Team is launching an end-of-year sales drive to push revenue, the primary objective is to segment a base of **over 114k customers** into distinct, actionable groups. 

This project utilizes the **Customer 360** approach and **RFM Analysis** to partition the customer base. The core deliverable enables the Customer Relationship Management (CRM) and Marketing teams to design personalized care policies and highly effective target marketing plans.

- **Data Scope:** Integrated transactions gathered from `2022-06-01` to `2022-08-31`.
- **Target Metrics:** Total Customers (~114,082), Total Monetary (~1111M GMV), Metrics normalized by Contract Age to ensure unbiased evaluation between long-term and new users.

---

## 🧭 Theoretical Framework

### 1. Customer 360 Approach
A comprehensive methodology encompassing 4 major data pillars around the user journey:
* **Transaction Data:** Purchase history, payment styles, and GMV metrics.
* **Interaction Data:** Customer service notes, communication logs (emails, chats, calls).
* **Demographic Data:** Baseline user profiles (location, age, status).
* **Behavioral Data:** Detailed usage frequency, preferences, and platform habits.

### 2. RFM Analysis Methodology
Evaluating core purchasing actions based on three variables:
* **Recency (R):** Days elapsed since the customer's last transaction relative to the report index date (`2022-09-01`).
* **Frequency (F):** Average number of monthly transactions, normalized by Contract Age.
* **Monetary (M):** Average monthly gross monetary value (GMV) contribution, normalized by Contract Age.

### 3. Segmentation Strategy (Interquartile Range & BCG Matrix Adaptation)
* **Scoring:** Utilizing the Interquartile Range method (Q1, Q2, Q3) to divide the customer distributions evenly into 4 quartiles (Scores from 1 to 4).
* **Grouping:** Combining 64 potential RFM combinations into 4 main tiers inspired by the BCG framework:
  * 🌟 **Stars (VIP):** Core profit drivers. High spending, high frequency, highly recent interactions.
  * 🐄 **Cash Cows (Loyal/Thân thiết):** Steady frequency and recent presence with average/moderate spend.
  * ❓ **Question Marks (Potential/Tiềm năng):** High spenders with low frequency, or active users who haven't returned recently.
  * 🐾 **Dogs (Casual/Vãng lai):** Low spenders with poor frequency who haven't interacted in a long time.

---

## 📊 Data Insights & Segmentation Summary

### Customer Distribution & Revenue Contribution
Through data synthesis, the customer ecosystem presents the following architecture:

| Customer Group | Volume Share (%) | Revenue Contribution (%) | Average Monthly Frequency | Strategic Action |
| :--- | :---: | :---: | :---: | :--- |
| **Casual (Khách vãng lai)** | **57.85%** | **49.46%** | 0.0171 | Deploy attractive conversion promos to spark a second purchase |
| **Potential (Khách tiềm năng)**| **20.34%** | **22.42%** | 0.0194 | Targeted marketing plans to convert them into Loyalists |
| **VIP (Khách hàng VIP)** | **13.43%** | **19.78%** | **0.0318** (Highest) | Exclusive loyalty programs, personalized perks, premium support |
| **Loyal (Khách thân thiết)** | **8.38%** | **8.34%** | 0.0197 | Tailored milestone rewards and gamified retention campaigns |

### Key Takeaways
1. **The Retention Challenge:** Casual customers represent the largest cohort and bring nearly half of the revenue. However, their low frequency demonstrates that turning a one-time purchaser into a recurring client is a primary operational hurdle.
2. **The Power of VIPs:** Representing only ~13% of the database, the VIP cohort drives nearly 20% of the financial value with a monthly purchase frequency almost doubling the casual base.

---

## 🗄️ Database Schema & Data Model

The analytical pipeline processes two relational entities:

### 1. `Customer_Register`
* `ID` (bigint, PK): Unique identifier for each customer.
* `Contract` (varchar): Contract identifier code.
* `LocationID` (int) / `BranchCode` (tinyint): Demographic location trackers.
* `Status` (tinyint): Current contract state.
* `Created_date` / `Stop_date` (datetime): Temporal data used to calculate Contract Age.

### 2. `Customer_Transaction`
* `ID` (bigint, PK): Transaction log ID.
* `CustomerID` (varchar, FK): Mapping indicator back to registry.
* `Purchase_date` (datetime): The stamp of purchase occurrence.
* `GMV` (bigint): Gross monetary volume per transaction line.

---

## 💻 Tech Stack & Implementation

- **Data Processing & Engineering:** SQL Server (T-SQL) utilizing Window Functions (`ROW_NUMBER`), Common Table Expressions (CTEs), and Temporary Tables for calculation performance.
- **Data Visualization & Operational BI:** Power BI Dashboarding (including Treemaps, bar distributions, and RFM mapping).

### Core SQL Pipeline Extract

```sql
-- Step 1: Aggregate Recency, Frequency, and Monetary parameters normalized by contract age
SELECT 
    CustomerID,
    DATEDIFF(day, max(CAST(purchase_date as date)), '2022-09-01') as Recency,
    ROUND(1.0*COUNT(DISTINCT Purchase_Date)/DATEDIFF(month, CAST(cr.created_date as date), '2022-09-01'),4) as Frequency,
    ROUND(1.0*SUM(GMV)/DATEDIFF(month, CAST(cr.created_date as date), '2022-09-01'),4) as Monetary,
    ROW_NUMBER() over(order by DATEDIFF(day,max(CAST(purchase_date as date)), '2022-09-01')) as rn_recency,
    ROW_NUMBER() over(order by ROUND(1.0*COUNT(DISTINCT Purchase_Date)/DATEDIFF(month, CAST(cr.created_date as date), '2022-09-01'),4)) as rn_frequency,
    ROW_NUMBER() over(order by ROUND(1.0*SUM(GMV)/DATEDIFF(month, CAST(cr.created_date as date), '2022-09-01'),4)) as rn_monetary
INTO #customer_statistics
FROM Customer_Registered cr 
JOIN Customer_Transaction ct ON cr.ID = ct.CustomerID
GROUP BY CustomerID, CAST(cr.created_date as date);

-- Step 2: Bin into Quartiles & Form Segmentations (Extract from final pipeline)
SELECT 
    CONCAT(R, F, M) as Segmentation, 
    COUNT(*) as total_customers,
    SUM(Monetary) as revenue,
    CASE
        WHEN CONCAT(R, F, M) IN ('444', '443', '434', '344', '334') THEN N'KH VIP'
        WHEN CONCAT(R, F, M) IN ('433', '424', '423', '414', '413', '343', '333', '324', '323', '314', '313') THEN N'KH thân thiết'
        WHEN CONCAT(R, F, M) IN ('442', '432', '422', '412', '342', '332', '322', '312', '242', '233', '244', '243', '234', '224', '223', '214', '213') THEN N'KH tiềm năng'
        ELSE N'KH vãng lai'
    END as Group_Customer
FROM #kq
GROUP BY CONCAT(R, F, M);
