# GoodThought NGO

This comprehensive dataset includes the following three tables:

## Dataset Overview

| Dataset    | Description                                                                                                           |
|------------|-----------------------------------------------------------------------------------------------------------------------|
| **Assignments** | Details about each project, including its name, duration (start and end dates), budget, geographical region, and the impact score. |
| **Donations**   | Records of financial contributions, linked to specific donors and assignments, highlighting how financial support is allocated and utilized. |
| **Donors**      | Information on individuals and organizations that fund GoodThought’s projects, including donor types.              |

![ERD Diagram](images/GoodThought_ERD.png)

## Code

```sql
-- Calculate total donation amounts by assignment and donor type,
-- rounding amounts to 2 decimal places
WITH donation_details AS (
    SELECT
        d.assignment_id,
        ROUND(SUM(d.amount), 2) AS rounded_total_donation_amount,
        dn.donor_type
    FROM
        donations d
    JOIN donors dn ON d.donor_id = dn.donor_id
    GROUP BY
        d.assignment_id, dn.donor_type
)

-- Select assignment details along with total donations and donor type,
-- showing the top 5 assignments by donation amount
SELECT
    a.assignment_name,
    a.region,
    dd.rounded_total_donation_amount,
    dd.donor_type
FROM
    assignments AS a
JOIN
    donation_details dd ON a.assignment_id = dd.assignment_id
ORDER BY
    dd.rounded_total_donation_amount DESC
LIMIT 5;
```

## Top 5 Assignments by Donation Amount and Donor Type

| Index | Assignment Name | Region | Total Donation Amount (USD) | Donor Type   |
|:-----:|:----------------|:-------|----------------------------:|:-------------|
| 0     | Assignment_3033 | East   |                     3,840.66 | Individual   |
| 1     | Assignment_300  | West   |                     3,133.98 | Organization |
| 2     | Assignment_4114 | North  |                     2,778.57 | Organization |
| 3     | Assignment_1765 | West   |                     2,626.98 | Organization |
| 4     | Assignment_268  | East   |                     2,488.69 | Individual   |

```sql

-- Calculate total number of donations per assignment
WITH donation_info AS (
    SELECT 
        assignment_id, 
        COUNT(donation_id) AS num_total_donations
    FROM donations
    GROUP BY assignment_id
),

-- Rank assignments by impact score within each region, including only assignments with at least one donation
assignment_rank AS (
    SELECT 
        a.assignment_name, 
        a.region, 
        a.impact_score, 
        di.num_total_donations,
        ROW_NUMBER() OVER (PARTITION BY a.region ORDER BY a.impact_score DESC) AS rank_in_region
    FROM assignments AS a
    JOIN donation_info AS di ON a.assignment_id = di.assignment_id
    WHERE di.num_total_donations >= 1
)

-- Select top-ranked assignment by impact score in each region
SELECT 
    assignment_name, 
    region, 
    impact_score, 
    num_total_donations
FROM assignment_rank
WHERE rank_in_region = 1
ORDER BY region;
```

## Top Impact Assignments by Region (With Donations)

| Index | Assignment Name | Region | Impact Score | Total Donations |
|:-----:|:----------------|:-------|-------------:|----------------:|
| 0     | Assignment_316  | East   |          10  |               2 |
| 1     | Assignment_2253 | North  |         9.99 |               1 |
| 2     | Assignment_3547 | South  |          10  |               1 |
| 3     | Assignment_2794 | West   |         9.99 |               2 |


## Summary

Leveraged SQL CTEs, joins, aggregation, and window functions to analyze donation data.
Identified the top assignments by donation amount and donor type.
Ranked assignments by impact score regionally while considering only those with donations.
Provided clear and actionable insights to support GoodThought NGO's mission strategy.

