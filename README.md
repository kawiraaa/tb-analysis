# tuberculosis treatment outcome analysis

## executive summary

Tuberculosis (TB) treatment outcomes are influenced by more than clinical care alone. Factors such as access to healthcare facilities, employment status, patient demographics, comorbidities, and medication adherence can significantly affect whether patients successfully complete treatment.

This project analyzes integrated TB patient data to identify the key factors associated with treatment success and failure. By examining demographic, clinical, and adherence-related information, the analysis uncovers patterns that can help healthcare providers better understand the challenges faced by different patient groups.

The findings indicate that distance to healthcare facilities, employment status, age group, rural residence, comorbidities such as HIV and diabetes, and medication adherence all play an important role in TB treatment success. Patients who live farther from clinics, those in high-risk groups, and those with other health conditions are more likely to experience poor outcomes.

Based on these insights, healthcare providers can improve outcomes by introducing mobile health clinics and home-based medication delivery programs for patients living far from treatment centers, increasing follow-up support for high-risk groups, and integrating TB, HIV, and diabetes services into a single appointment. These targeted interventions can  reduce failure rates and increase overall treatment success.


![](tb_1.png)
![](tb_2.png)

---

## **Problem Statement**

Tuberculosis (TB) treatment requires patients to follow a strict medication plan for several months. When patients miss doses, discontinue treatment, or become lost to follow-up, treatment outcomes worsen and the risk of disease transmission increases.

Healthcare providers collect large amounts of patient information, including demographic, socio-economic, clinical, and treatment data. However, these data sources are often analyzed separately, making it difficult to understand how different factors work together to influence treatment outcomes.

This project addresses that challenge by integrating and analyzing patient, clinical, and treatment data to identify the key drivers of treatment success and failure. The analysis focuses on factors such as access to healthcare, patient demographics, and comorbidities.

The goal is to help healthcare providers identify high-risk patients earlier, understand the barriers to successful treatment, and implement targeted interventions that  reduce treatment dropouts and enhance overall TB treatment outcomes.


---
## Data Description

This project uses four related datasets that capture patient, treatment, medication adherence, and clinical information for tuberculosis (TB) patients.

### 1. Patients Table
Contains demographic and socio-economic information about each patient.

| Column Name | Description |
|---|---|
| `patientid` | Unique identifier for each patient |
| `age` | Patient’s age |
| `gender` | Patient’s gender |
| `locationtype` | Type of location where the patient lives, such as rural or urban |
| `employmentstatus` | Employment status of the patient |
| `distance_km` | Distance from the patient’s home to the nearest healthcare facility, measured in kilometers |

-----

### 2. Treatment Outcomes Table
Contains treatment start and end dates, as well as the final treatment status.

| Column Name | Description |
|---|---|
| `outcomeid` | Unique identifier for each treatment outcome record |
| `patientid` | Unique identifier linking the record to a patient |
| `treattmntstartdate` | Date treatment began |
| `treatmentenddate` | Date treatment ended |
| `finalstatus` | Final treatment outcome, such as recovered, failed, or lost to follow-up |

-----

### 3. Medication Log Table
Tracks medication adherence and side effects during treatment.

| Column Name | Description |
|---|---|
| `logid` | Unique identifier for each medication log record |
| `patientid` | Unique identifier linking the record to a patient |
| `datescheduled` | Date the medication was scheduled to be taken |
| `datetaken` | Date the medication was actually taken |
| `dosemissed` | Indicates whether a scheduled dose was missed |
| `sideeffectsreported` | Indicates whether the patient reported side effects |

----

### 4. Clinical Biomarkers Table
Contains clinical test results and health indicators collected during treatment.

| Column Name | Description |
|---|---|
| `biomarkerid` | Unique identifier for each clinical biomarker record |
| `patientid` | Unique identifier linking the record to a patient |
| `testdate` | Date the clinical test was performed |
| `sputumsmearresults` | Sputum smear test result |
| `bmi` | Body Mass Index of the patient |
| `comorbidities` | Other health conditions such as HIV or diabetes |

### Data Relationships

The `patientid` column is the key field that links all four tables together. This makes it possible to analyze how demographic, clinical, and treatment-related factors influence TB treatment outcomes.

All datasets were linked using a common key: `PatientID`, enabling a unified patient-level analytical view.

---
## tech stack
**MySQL** – Data cleaning, transformation, joins, aggregation, and feature engineering  
**Looker Studio** – Interactive dashboard creation and visualization

  ---

  ## Skills Demonstrated

- Data Cleaning and Preprocessing
- SQL Querying
- Business Intelligence Reporting
- Data Visualization
- Dashboard Development
- Business Insight Generation
- Data Storytelling

---

## Project Workflow
-  Data Cleaning and Preprocessing
-  SQL-Based Business Analysis
-  Dashboard Development in Looker Studio
-  Insight Generation
-  Business Recommendations
  


## Data Cleaning and Transformation

A consistent data cleaning and transformation process was applied across all TB-related datasets to improve data quality, standardize formats, and prepare the data for analysis.

### 1. Database and Table Creation

A database named `tb` was created to store all project tables. Columns were initially stored as flexible text types to preserve raw data before cleaning.

```sql
CREATE DATABASE tb;
USE tb;
```

Example of raw table creation (`biomarkers`):

```sql
CREATE TABLE biomarkers (
    BiomarkerID VARCHAR(50),
    PatientID VARCHAR(50),
    TestDate TEXT,
    SputumSmearResult TEXT,
    BMI TEXT,
    Comorbidities TEXT
);
```

---

### 2. Date Standardization

Some date columns contained multiple formats, such as:
 DD/MM/YYYY , MM/DD/YYYY, YYYY-MM-DD, month text formats  

These were converted into a standard SQL `DATE` format.

```sql
UPDATE treatment_outcome
SET TreatmentStartDate =
CASE
    WHEN TreatmentStartDate REGEXP '^(1[3-9]|2[0-9]|3[0-1])/[0-9]{2}/[0-9]{4}$'
    THEN STR_TO_DATE(TreatmentStartDate, '%d/%m/%Y')

    WHEN TreatmentStartDate REGEXP '^[0-9]{4}-[0-9]{2}-[0-9]{2}$'
    THEN STR_TO_DATE(TreatmentStartDate, '%Y-%m-%d')

    ELSE NULL
END;
```
---

### 3. Standardizing Text Values

Categorical variables had inconsistent spelling, abbreviations, and capitalization.### Why?
This ensured values like ' COMPLETE`, `Complete`,` completed.`are treated as one category.

Example:

```sql
UPDATE treatment_outcome
SET FinalStatus =
CASE 
    WHEN FinalStatus = 'Lost' THEN 'LTFU'
    WHEN FinalStatus = 'TreatmentFailed' THEN 'failed'
    WHEN FinalStatus = 'FAIL' THEN 'failed'
    WHEN FinalStatus = 'COMPLETE' THEN 'completed'
    WHEN FinalStatus = 'lost to follow-up' THEN 'LTFU'
    ELSE FinalStatus
END;
```

Lowercasing text:

```sql
UPDATE treatment_outcome
SET FinalStatus = LOWER(FinalStatus);
```

---

### 4. Handling Missing Values

Some categorical columns contained invalid entries, such as:
`'nan'`,blank values (`''`).These were replaced with valid categories.This improved data completeness and prevents missing-value issues during analysis.

Checking missing values:

```sql
SELECT EmploymentStatus, COUNT(*) AS count
FROM patients
GROUP BY EmploymentStatus;
```

Replacing invalid values:

```sql
UPDATE patients
SET EmploymentStatus =
CASE 
    WHEN EmploymentStatus = 'nan' THEN 'employed'
    WHEN EmploymentStatus = '' THEN 'employed'
    ELSE EmploymentStatus
END;
```
---

### 5. Removing Duplicate Records

Duplicate records were checked and removed. This ensured that repeated records do not bias results.

Checking duplicates:

```sql
SELECT OutcomeID, COUNT(*) AS count
FROM treatment_outcome
GROUP BY OutcomeID;
```

Creating a clean table:

```sql
CREATE TABLE treatment_outcome_clean AS
SELECT DISTINCT *
FROM treatment_outcome;
```
---

### 6. Converting Data Types

Some columns were stored as text but needed to be numeric for analysis.

Example:

```sql
ALTER TABLE patients
MODIFY Age INT;
```

---

### 7. Medication Log Aggregation

The medication log contained daily records, so it was compressed into patient-level summary statistics.

```sql
CREATE TABLE combined_medication AS 
SELECT 
    PatientID,
    COUNT(LogID) AS Total_Scheduled_Doses,
    SUM(CASE WHEN DoseMissed = 1 THEN 1 ELSE 0 END) AS Total_Missed_Doses,

    MIN(DateScheduled2) AS Treatment_Start_Date,
    MAX(DateScheduled2) AS Last_Tracked_Date

FROM medication_log_clean
GROUP BY PatientID;
```
---

### 8. Joining All Tables

All cleaned datasets were combined into one analytical table using `PatientID`.
This created one unified dataset for complete patient analysis.


```sql
CREATE TABLE joined AS
SELECT 
    p.PatientID,
    p.Age,
    p.Gender,
    p.LocationType,
    p.EmploymentStatus,
    p.Distance_km,

    b.BMI AS Baseline_BMI,
    b.Comorbidities,
    b.SputumSmearResult AS Initial_Smear_Result,

    m.Total_Scheduled_Doses,
    m.Total_Missed_Doses,
    m.Adherence_Rate,

    o.FinalStatus AS Final_Treatment_Outcome

FROM patients_clean p

LEFT JOIN biomarkers_clean b 
    ON p.PatientID = b.PatientID

LEFT JOIN combined_medication m 
    ON p.PatientID = m.PatientID

LEFT JOIN treatment_outcome_clean o 
    ON p.PatientID = o.PatientID;
```
---

### 9. Feature Engineering

Continuous variables were grouped into categories for easier analysis.

Example: Distance grouping

```sql
UPDATE joined
SET Distance_Group =
CASE
    WHEN Distance_km BETWEEN 0 AND 5 THEN '0–5 km'
    WHEN Distance_km BETWEEN 6 AND 10 THEN '6–10 km'
    WHEN Distance_km BETWEEN 11 AND 15 THEN '11–15 km'
    ELSE '16+ km'
END;
```
---
            

## sql querrying

After cleaning and integrating all datasets into the `joined` table, several analytical questions were explored to understand how socio-economic, clinical, and behavioral factors influence TB treatment outcomes.

The main objective of this analysis is to identify patterns associated with:
- Treatment success  
- Treatment failure 

---

### 1. Employment Status vs Treatment Outcome
Insight: Employment status is strongly associated with treatment outcomes. Patients with stable employment have higher treatment completion rates, while informal or unemployed groups show higher failure rates.

```sql
SELECT EmploymentStatus, Final_Treatment_Outcome,
ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM joined), 2) AS percentage
FROM joined
GROUP BY EmploymentStatus, Final_Treatment_Outcome;
```

### results
![](employmentstatus.png)

---

### 2. Location Type (Urban vs Rural)
Insight: Urban patients achieve better treatment outcomes than rural patients, suggesting that access to healthcare services and infrastructure plays a key role in treatment success.

```sql
SELECT LocationType, Final_Treatment_Outcome,
ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM joined), 2) AS percentage
FROM joined
GROUP BY LocationType, Final_Treatment_Outcome;
```

### results
![](locationvssucess.png)

---

### 3. Distance to Clinic vs Treatment Outcome
Insight: Distance to healthcare facilities is a major factor influencing treatment success. Patients living within 0–5 km show significantly higher success rates, while those beyond 10 km experience increased failure risk.
```sql
SELECT Distance_Group, Final_Treatment_Outcome,
COUNT(*) AS count,
ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM joined), 2) AS percentage
FROM joined
GROUP BY Distance_Group, Final_Treatment_Outcome;
```

### results
![](distancevssucess.png)

---

### 4. Age Distribution Analysis
Insight: Young adults (18–30 years) show a higher proportion of treatment failure compared to other age groups, indicating this group may require additional adherence support.

```sql
SELECT Age_Group, Final_Treatment_Outcome,
COUNT(*) AS count,
ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM joined), 2) AS percentage
FROM joined
GROUP BY Age_Group, Final_Treatment_Outcome;
```

### results
![](agevssucess.png)

---

### 5. BMI vs Treatment Outcome
Insight: Baseline BMI does not show a strong variation across treatment outcomes, suggesting it is not a major predictor of TB treatment success in this dataset.


```sql
SELECT ROUND(AVG(Baseline_BMI), 2) AS avg_bmi,
Final_Treatment_Outcome
FROM joined
GROUP BY Final_Treatment_Outcome;
```

### results
![](bmivssucess.png)

---

### 6. Sputum Smear Results vs Outcome
insights: Even patients with positive initial smear results show high recovery rates, indicating that treatment adherence plays a critical role in overcoming initial disease severity.

```sql
SELECT Initial_Smear_Result, Final_Treatment_Outcome,
COUNT(*),
ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM joined), 2) AS percentage
FROM joined
WHERE Initial_Smear_Result = 'positive'
GROUP BY Initial_Smear_Result, Final_Treatment_Outcome;
```

### results
![](spurtumvssucess.png)

---

### 7. Comorbidities vs Treatment Failure
Insight: Patients with comorbidities such as HIV and diabetes show higher failure rates, indicating increased clinical complexity and risk

```sql
SELECT Comorbidities, Final_Treatment_Outcome,
COUNT(*) AS count
FROM joined
WHERE Final_Treatment_Outcome = 'Failure'
GROUP BY Comorbidities, Final_Treatment_Outcome;
```

### results
![](/cormoditiesvssucess.png)

---

### 8. High-Risk Socioeconomic Combination (Failure Cases)
Insight: Failure cases are often associated with a combination of socio-economic and accessibility challenges, highlighting the importance of multi-factor risk assessment.

```sql
SELECT LocationType, EmploymentStatus, Distance_Group,
Final_Treatment_Outcome, COUNT(*) AS count
FROM joined
WHERE Final_Treatment_Outcome = 'Failure'
GROUP BY LocationType, EmploymentStatus, Distance_Group, Final_Treatment_Outcome
ORDER BY count DESC;
```

### results
![](patient.png)

---

## dashboard 
An interactive dashboard was developed using Looker Studio to visualize key patterns in tuberculosis treatment outcomes. It allows healthcare stakeholders to explore relationships between variables and identify high-risk patient groups more efficiently than static analysis alone.
[view dashboard](https://datastudio.google.com/reporting/1ca5e04d-71f4-41c9-9784-5a918050825d)

## recommendations
- introduce mobile health clinics to deliver medicine directly to patients who live more than 6 km away from the nearest clinic.
- Implement home-based medication delivery programs for accessible medicine
- increase follow-up frequency in the high-risk group
- Combine TB, HIV, and diabetes care into a single appointment so patients do not have to make multiple clinic trips.



