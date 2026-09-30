# Hospital Readmission Analytics — SQL Analysis

## 1. Project Overview

### Business Problem

What factors are associated with 30-day hospital readmission among patients with diabetes, and what insights can healthcare organizations derive from the data?

### Dataset

The analysis uses the UCI Diabetes 130-US Hospitals dataset.

The dataset contains hospital encounters for patients with diabetes and includes demographic, admission, clinical, medication, and hospital utilization variables.

### Dataset Size

- 101,766 hospital encounters
- 71,518 unique patients
- 50 original variables

### Analytical Objective

The SQL analysis will examine:

- The distribution of hospital readmissions
- The overall 30-day readmission rate
- Readmission patterns by demographic factors
- Prior hospital utilization
- Length of hospital stay
- Medication burden
- Number of diagnoses
- Admission characteristics
- Insulin and medication changes
- Diagnosis categories

### Important Dataset Limitation

This is a historical U.S. hospital dataset covering the period 1999–2008. Therefore, findings should not be assumed to represent current hospitals, healthcare systems, or patient populations in India.

The analysis identifies associations in the dataset and does not establish causal relationships.

---

## 2. SQL Environment

### Database

SQLite

### Database File

`hospital_readmission.db`

### Source Table

`diabetic_data`

### Data Integrity Check

The original CSV file was imported into SQLite without modifying the source data.

The imported table contains:

- 101,766 rows
- 50 columns

The row count was verified using SQL.

### SQL Validation Query

```sql
SELECT COUNT(*) AS total_rows
FROM diabetic_data;
### Validation Result

The SQL query returned 101,766 rows, confirming that the complete dataset was successfully imported into SQLite.

---

## 3. Readmission Distribution

### Business Question

What is the distribution of hospital encounters across the three readmission categories?

### Readmission Categories

- `NO` — patient was not readmitted
- `>30` — patient was readmitted after 30 days
- `<30` — patient was readmitted within 30 days

### SQL Query

```sql
SELECT
    readmitted,
    COUNT(*) AS encounters
FROM diabetic_data
GROUP BY readmitted
ORDER BY encounters DESC;
---

## 4. Overall 30-Day Readmission Rate

### Business Question

What percentage of all hospital encounters resulted in a readmission within 30 days?

### SQL Query

```sql
SELECT
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data;
---

## 5. Readmission by Age

### Business Question

Does the 30-day readmission rate vary across patient age groups?

### SQL Query

```sql
SELECT
    age,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY age
ORDER BY age;
---

## 6. Readmission by Gender

### Business Question

Does the observed 30-day readmission rate differ by gender?

### SQL Query

```sql
SELECT
    gender,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY gender
ORDER BY readmission_rate_pct DESC;
---

## 7. Readmission by Prior Inpatient Visits

### Business Question

Is the number of previous inpatient visits associated with the observed 30-day readmission rate?

### SQL Query

```sql
SELECT
    CASE
        WHEN number_inpatient = 0 THEN '0'
        WHEN number_inpatient = 1 THEN '1'
        WHEN number_inpatient = 2 THEN '2'
        WHEN number_inpatient = 3 THEN '3'
        ELSE '4+'
    END AS inpatient_visit_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY inpatient_visit_group
ORDER BY
    CASE inpatient_visit_group
        WHEN '0' THEN 1
        WHEN '1' THEN 2
        WHEN '2' THEN 3
        WHEN '3' THEN 4
        WHEN '4+' THEN 5
    END;
---

## 8. Readmission by Prior Emergency Visits

### Business Question

Is the number of prior emergency visits associated with the observed 30-day readmission rate?

### SQL Query

```sql
SELECT
    CASE
        WHEN number_emergency = 0 THEN '0'
        WHEN number_emergency = 1 THEN '1'
        WHEN number_emergency = 2 THEN '2'
        ELSE '3+'
    END AS emergency_visit_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY emergency_visit_group
ORDER BY
    CASE emergency_visit_group
        WHEN '0' THEN 1
        WHEN '1' THEN 2
        WHEN '2' THEN 3
        WHEN '3+' THEN 4
    END;
---

## 9. Readmission by Length of Stay

### Business Question

Does the length of the hospital stay appear to be associated with 30-day readmission?

### SQL Query

```sql
SELECT
    CASE
        WHEN time_in_hospital BETWEEN 1 AND 2 THEN '1-2'
        WHEN time_in_hospital BETWEEN 3 AND 4 THEN '3-4'
        WHEN time_in_hospital BETWEEN 5 AND 6 THEN '5-6'
        WHEN time_in_hospital BETWEEN 7 AND 9 THEN '7-9'
        ELSE '10+'
    END AS length_of_stay_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY length_of_stay_group
ORDER BY
    CASE length_of_stay_group
        WHEN '1-2' THEN 1
        WHEN '3-4' THEN 2
        WHEN '5-6' THEN 3
        WHEN '7-9' THEN 4
        WHEN '10+' THEN 5
    END;
---

## 10. Readmission by Number of Medications

### Business Question

Is the number of medications prescribed during the hospital encounter associated with 30-day readmission?

### SQL Query

```sql
SELECT
    CASE
        WHEN num_medications BETWEEN 1 AND 5 THEN '1-5'
        WHEN num_medications BETWEEN 6 AND 10 THEN '6-10'
        WHEN num_medications BETWEEN 11 AND 15 THEN '11-15'
        WHEN num_medications BETWEEN 16 AND 20 THEN '16-20'
        WHEN num_medications BETWEEN 21 AND 30 THEN '21-30'
        ELSE '31+'
    END AS medication_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY medication_group
ORDER BY
    CASE medication_group
        WHEN '1-5' THEN 1
        WHEN '6-10' THEN 2
        WHEN '11-15' THEN 3
        WHEN '16-20' THEN 4
        WHEN '21-30' THEN 5
        WHEN '31+' THEN 6
    END;            
---
## 11. Readmission by Number of Diagnoses

### Business Question

Is the number of diagnoses recorded during a hospital encounter associated with 30-day readmission?

### SQL Query

```sql
SELECT
    CASE
        WHEN number_diagnoses BETWEEN 1 AND 4 THEN '1-4'
        WHEN number_diagnoses BETWEEN 5 AND 6 THEN '5-6'
        WHEN number_diagnoses BETWEEN 7 AND 8 THEN '7-8'
        WHEN number_diagnoses = 9 THEN '9'
        ELSE '10+'
    END AS diagnosis_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY diagnosis_group
ORDER BY
    CASE diagnosis_group
        WHEN '1-4' THEN 1
        WHEN '5-6' THEN 2
        WHEN '7-8' THEN 3
        WHEN '9' THEN 4
        WHEN '10+' THEN 5
    END;    
---
## 12. Readmission by Number of Laboratory Procedures

### Business Question

Is the number of laboratory procedures performed during a hospital encounter associated with 30-day readmission?

### SQL Query

```sql
SELECT
    CASE
        WHEN num_lab_procedures BETWEEN 1 AND 30 THEN '1-30'
        WHEN num_lab_procedures BETWEEN 31 AND 44 THEN '31-44'
        WHEN num_lab_procedures BETWEEN 45 AND 57 THEN '45-57'
        ELSE '58+'
    END AS lab_procedure_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY lab_procedure_group
ORDER BY
    CASE lab_procedure_group
        WHEN '1-30' THEN 1
        WHEN '31-44' THEN 2
        WHEN '45-57' THEN 3
        WHEN '58+' THEN 4
    END;   
---
## 13. Readmission by Number of Procedures

### Business Question

Is the number of procedures performed during a hospital encounter associated with 30-day readmission?

### SQL Query

```sql
SELECT
    num_procedures,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY num_procedures
ORDER BY num_procedures;
---
## 14. Readmission by Insulin Treatment

### Business Question

Is the insulin treatment category during a hospital encounter associated with 30-day readmission?

### SQL Query

```sql
SELECT
    insulin,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY insulin
ORDER BY readmission_rate_pct DESC;
---
## 15. Readmission by Medication Change

### Business Question

Is the 30-day readmission rate different for patients whose diabetes medications were changed during the encounter compared with those whose medications were not changed?

### SQL Query

```sql
SELECT
    change,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY change
ORDER BY readmission_rate_pct DESC;
```

### Result

| Medication Change | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ----------------- | ---------------: | ------------------: | ---------------: |
| Ch                |           47,011 |               5,558 |           11.82% |
| No                |           54,755 |               5,799 |           10.59% |

### Key Finding

The observed 30-day readmission rate was:

* **11.82%** for encounters where diabetes medication was changed.
* **10.59%** for encounters where diabetes medication was not changed.
* The difference was **1.23 percentage points**.

### Business Interpretation

Encounters involving a diabetes medication change had a higher observed 30-day readmission rate in this dataset.

Medication changes may be a marker of differences in treatment needs, disease severity, or clinical complexity. Therefore, the observed association should not be interpreted as evidence that changing medication directly causes readmission.

This finding can be considered alongside other utilization and clinical factors in the broader readmission analysis.

### SQL Concepts Used

* `GROUP BY` — compares medication-change categories.
* `COUNT()` — counts encounters in each category.
* `CASE WHEN` — identifies 30-day readmissions.
* `SUM()` — counts 30-day readmissions.
* `ROUND()` — calculates the readmission rate to two decimal places.
* Calculated percentage — converts the readmission count into an observed rate.
---
## 16. Readmission by Admission Type

### Business Question

Does the observed 30-day readmission rate differ by the type of hospital admission?

### SQL Query

```sql
SELECT
    CASE admission_type_id
        WHEN 1 THEN 'Emergency'
        WHEN 2 THEN 'Urgent'
        WHEN 3 THEN 'Elective'
        WHEN 4 THEN 'Newborn'
        WHEN 5 THEN 'Not Available'
        WHEN 6 THEN 'NULL / Not Mapped'
        WHEN 7 THEN 'Trauma Center'
        WHEN 8 THEN 'Not Mapped'
        ELSE 'Unknown'
    END AS admission_type,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY admission_type_id
ORDER BY readmission_rate_pct DESC;
```

### Result

| Admission Type    | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ----------------- | ---------------: | ------------------: | ---------------: |
| Emergency         |           53,990 |               6,221 |           11.52% |
| Urgent            |           18,480 |               2,066 |           11.18% |
| NULL / Not Mapped |            5,291 |                 586 |           11.08% |
| Elective          |           18,869 |               1,961 |           10.39% |
| Not Available     |            4,785 |                 495 |           10.34% |
| Newborn           |               10 |                   1 |           10.00% |
| Not Mapped        |              320 |                  27 |            8.44% |
| Trauma Center     |               21 |                   0 |            0.00% |

### Key Finding

Among the larger admission groups, Emergency admissions had the highest observed 30-day readmission rate at **11.52%**, followed by Urgent admissions at **11.18%**.

Elective admissions had an observed rate of **10.39%**.

The difference between Emergency and Elective admissions was approximately **1.13 percentage points**.

The Newborn and Trauma Center categories contain only 10 and 21 encounters respectively, so their observed rates should not be interpreted as reliable comparisons.

### Business Interpretation

Admission type shows some differences in observed 30-day readmission rates in this dataset.

Emergency admissions had a somewhat higher observed readmission rate than Elective admissions. This may reflect differences in patient characteristics, clinical urgency, or underlying complexity associated with how patients enter the hospital.

However, admission type alone does not establish why readmission occurs. The finding should therefore be considered together with prior hospital utilization, diagnoses, medication burden, length of stay, and other patient and encounter characteristics.

### SQL Concepts Used

* `CASE` — converts numeric admission type IDs into meaningful categories.
* `GROUP BY` — calculates results separately for each admission type.
* `COUNT()` — counts encounters.
* `SUM(CASE WHEN ...)` — counts 30-day readmissions.
* `ROUND()` — calculates the observed readmission rate.
* `ORDER BY` — sorts admission types by observed readmission rate.
---
## 17. Readmission by Race

### Business Question

Does the observed 30-day readmission rate differ across the race categories represented in the dataset?

### SQL Query

```sql id="x3k7m2"
SELECT
    CASE
        WHEN race = '?' THEN 'Missing / Unknown'
        ELSE race
    END AS race_category,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY race_category
ORDER BY readmission_rate_pct DESC;
```

### Result

| Race Category     | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ----------------- | ---------------: | ------------------: | ---------------: |
| Caucasian         |           76,099 |               8,592 |           11.29% |
| AfricanAmerican   |           19,210 |               2,155 |           11.22% |
| Hispanic          |            2,037 |                 212 |           10.41% |
| Asian             |              641 |                  65 |           10.14% |
| Other             |            1,506 |                 145 |            9.63% |
| Missing / Unknown |            2,273 |                 188 |            8.27% |

### Key Finding

The observed 30-day readmission rates ranged from **9.63% to 11.29%** across the named race categories.

Caucasian encounters had an observed rate of **11.29%**, while AfricanAmerican encounters had a very similar rate of **11.22%**.

The difference between these two largest groups was only **0.07 percentage points**.

The Missing / Unknown category had an observed rate of **8.27%**, but this represents missing race information rather than a meaningful race category.

### Business Interpretation

Race categories show relatively modest differences in observed 30-day readmission rates in this dataset.

The two largest groups, Caucasian and AfricanAmerican encounters, had very similar observed rates. The smaller Asian, Hispanic, and Other groups should be interpreted with consideration of their substantially smaller sample sizes.

These descriptive differences should not be interpreted as evidence that race itself causes hospital readmission. Observed differences can be influenced by other patient, clinical, socioeconomic, healthcare-utilization, and data-quality factors.

### SQL Concepts Used

* `CASE` — converts the missing `?` value into a readable category.
* `GROUP BY` — calculates results separately for each race category.
* `COUNT()` — counts encounters in each category.
* `SUM(CASE WHEN ...)` — counts 30-day readmissions.
* `ROUND()` — calculates the observed readmission rate.
* `ORDER BY` — sorts categories by observed readmission rate.
---
## 18. Readmission by Primary Diagnosis Category

### Business Question

Do observed 30-day readmission rates differ across the major primary diagnosis categories recorded for diabetic hospital encounters?

### SQL Query

```sql id="r6m2q8"
SELECT
    CASE
        WHEN diag_1 BETWEEN 390 AND 459 THEN 'Circulatory'
        WHEN diag_1 BETWEEN 460 AND 519 THEN 'Respiratory'
        WHEN diag_1 BETWEEN 520 AND 579 THEN 'Digestive'
        WHEN diag_1 BETWEEN 580 AND 629 THEN 'Genitourinary'
        WHEN diag_1 BETWEEN 680 AND 709 THEN 'Skin / Subcutaneous'
        WHEN diag_1 BETWEEN 710 AND 739 THEN 'Musculoskeletal'
        WHEN diag_1 BETWEEN 780 AND 799 THEN 'Symptoms / Signs'
        WHEN diag_1 BETWEEN 250 AND 259 THEN 'Diabetes'
        WHEN diag_1 BETWEEN 240 AND 249 THEN 'Endocrine / Metabolic'
        WHEN diag_1 BETWEEN 140 AND 239 THEN 'Neoplasms'
        WHEN diag_1 BETWEEN 800 AND 999 THEN 'Injury / Poisoning'
        ELSE 'Other / Unclassified'
    END AS diagnosis_category,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY diagnosis_category
ORDER BY total_encounters DESC;
```

### Result

| Diagnosis Category    | Total Encounters | 30-Day Readmissions | Readmission Rate |
| --------------------- | ---------------: | ------------------: | ---------------: |
| Circulatory           |           30,336 |               3,474 |           11.45% |
| Other / Unclassified  |           12,238 |               1,489 |           12.17% |
| Respiratory           |           10,407 |               1,112 |           10.69% |
| Digestive             |            9,208 |                 966 |           10.49% |
| Diabetes              |            8,869 |               1,150 |           12.97% |
| Symptoms / Signs      |            7,636 |                 687 |            9.00% |
| Injury / Poisoning    |            6,974 |                 854 |           12.25% |
| Genitourinary         |            5,078 |                 549 |           10.81% |
| Musculoskeletal       |            4,957 |                 471 |            9.50% |
| Neoplasms             |            3,433 |                 346 |           10.08% |
| Skin / Subcutaneous   |            2,530 |                 250 |            9.88% |
| Endocrine / Metabolic |              100 |                   9 |            9.00% |

### Key Finding

The observed 30-day readmission rate varied across primary diagnosis categories.

The highest observed rate among the broad categories was:

* **Diabetes — 12.97%**
* **Injury / Poisoning — 12.25%**
* **Other / Unclassified — 12.17%**
* **Circulatory — 11.45%**

The lowest observed rates were:

* **Symptoms / Signs — 9.00%**
* **Endocrine / Metabolic — 9.00%**
* **Musculoskeletal — 9.50%**

Circulatory conditions represented the largest group, with **30,336 encounters** and an observed readmission rate of **11.45%**.

### Business Interpretation

Primary diagnosis category shows differences in observed 30-day readmission rates in this dataset.

Encounters classified under Diabetes had an observed readmission rate of **12.97%**, compared with **11.16% overall** in the dataset. Circulatory conditions also represented a substantial volume of encounters and had an observed rate of **11.45%**.

These findings suggest that the clinical context of an encounter may be relevant when examining readmission patterns. However, diagnosis category alone does not establish the cause of readmission because patients can have multiple comorbidities and differences in prior utilization, treatment intensity, length of stay, and other characteristics.

The **Other / Unclassified** category should also be interpreted cautiously because it combines diagnosis codes that were not captured by the broad categories defined in this query.

### SQL Concepts Used

* `CASE` — converts numeric diagnosis codes into broader clinical categories.
* `BETWEEN` — identifies ranges of ICD-9 diagnosis codes.
* `GROUP BY` — calculates results separately for each diagnosis category.
* `COUNT()` — counts encounters.
* `SUM(CASE WHEN ...)` — counts 30-day readmissions.
* `ROUND()` — calculates the observed readmission rate.
* `ORDER BY` — sorts categories by encounter volume.
---
## 19. Detailed Primary Diagnosis Analysis

### Business Question

Which frequently occurring primary diagnosis codes have higher or lower observed 30-day readmission rates?

### SQL Query

```sql id="m8q4t1"
SELECT
    diag_1 AS primary_diagnosis_code,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
WHERE diag_1 IS NOT NULL
GROUP BY diag_1
HAVING COUNT(*) >= 500
ORDER BY readmission_rate_pct DESC;
```

### Result

| Primary Diagnosis Code | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ---------------------- | ---------------: | ------------------: | ---------------: |
| 250.7                  |              871 |                 165 |           18.94% |
| 250.6                  |            1,183 |                 219 |           18.51% |
| 434                    |            2,028 |                 329 |           16.22% |
| 440                    |              840 |                 133 |           15.83% |
| 820                    |            1,082 |                 171 |           15.80% |
| 403                    |              513 |                  79 |           15.40% |
| 507                    |              610 |                  90 |           14.75% |
| 428                    |            6,862 |                 968 |           14.11% |
| 250.11                 |              625 |                  88 |           14.08% |
| 8                      |              515 |                  71 |           13.79% |
| 276                    |            1,889 |                 257 |           13.61% |
| V57                    |            1,207 |                 163 |           13.50% |
| 577                    |            1,057 |                 142 |           13.43% |
| 996                    |            1,967 |                 264 |           13.42% |
| 584                    |            1,520 |                 202 |           13.29% |
| 250.13                 |              851 |                 113 |           13.28% |
| 491                    |            2,275 |                 287 |           12.62% |
| 789                    |              561 |                  65 |           11.59% |
| 296                    |              896 |                 103 |           11.50% |
| 998                    |              784 |                  87 |           11.10% |
| 38                     |            1,688 |                 187 |           11.08% |
| 453                    |              546 |                  60 |           10.99% |
| 599                    |            1,595 |                 171 |           10.72% |
| 578                    |              663 |                  71 |           10.71% |
| 250.8                  |            1,680 |                 179 |           10.65% |
| 560                    |              876 |                  92 |           10.50% |
| 410                    |            3,614 |                 373 |           10.32% |
| 715                    |            2,151 |                 215 |           10.00% |
| 433                    |              789 |                  78 |            9.89% |
| 518                    |            1,115 |                 110 |            9.87% |
| 780                    |            2,019 |                 191 |            9.46% |
| 493                    |            1,056 |                  98 |            9.28% |
| 530                    |              531 |                  49 |            9.23% |
| 427                    |            2,766 |                 252 |            9.11% |
| 414                    |            6,581 |                 595 |            9.04% |
| 562                    |              989 |                  89 |            9.00% |
| 682                    |            2,042 |                 183 |            8.96% |
| 486                    |            3,508 |                 314 |            8.95% |
| 574                    |              965 |                  84 |            8.70% |
| 250.02                 |              675 |                  53 |            7.85% |
| 435                    |            1,016 |                  78 |            7.68% |
| 786                    |            4,016 |                 291 |            7.25% |
| 722                    |              771 |                  50 |            6.49% |

### Key Finding

Among primary diagnosis codes with at least 500 encounters, the observed 30-day readmission rates varied substantially.

The highest observed rates in this analysis were:

* `250.7` — **18.94%**
* `250.6` — **18.51%**
* `434` — **16.22%**
* `440` — **15.83%**
* `820` — **15.80%**

Several high-volume diagnosis codes also showed notable differences. For example:

* `428` had **6,862 encounters** and an observed readmission rate of **14.11%**.
* `414` had **6,581 encounters** and an observed rate of **9.04%**.
* `786` had **4,016 encounters** and an observed rate of **7.25%**.
* `722` had **771 encounters** and an observed rate of **6.49%**.

The observed rates therefore differed considerably across specific primary diagnosis codes.

### Business Interpretation

Primary diagnosis appears to be an important descriptive dimension when examining readmission patterns in this dataset.

Some diagnosis codes were associated with substantially higher observed 30-day readmission rates than the overall dataset rate of **11.16%**. Other frequently occurring diagnosis codes had rates below the overall rate.

However, these are **unadjusted descriptive comparisons**. A diagnosis code may be associated with other factors such as age, prior inpatient utilization, number of diagnoses, medication burden, length of stay, and other clinical characteristics.

Therefore, these results should not be interpreted as evidence that a particular diagnosis independently causes or prevents readmission.

The minimum threshold of 500 encounters was used to reduce the influence of extremely small diagnosis groups, but it does not eliminate statistical uncertainty or confounding.

### SQL Concepts Used

* `GROUP BY` — calculates results separately for each primary diagnosis code.
* `COUNT()` — counts encounters for each diagnosis.
* `SUM(CASE WHEN ...)` — counts 30-day readmissions.
* `HAVING COUNT(*) >= 500` — filters diagnosis groups based on encounter volume.
* `ROUND()` — calculates the observed readmission rate.
* `ORDER BY` — sorts diagnosis codes by observed readmission rate.
* `IS NOT NULL` — excludes missing primary diagnosis values.
---
## 20. Readmission by Prior Outpatient Visits

### Business Question

Is the observed 30-day readmission rate different according to the number of outpatient visits before the current hospital encounter?

### SQL Query

```sql id="p4x8n2"
SELECT
    CASE
        WHEN number_outpatient = 0 THEN '0'
        WHEN number_outpatient = 1 THEN '1'
        WHEN number_outpatient = 2 THEN '2'
        WHEN number_outpatient = 3 THEN '3'
        ELSE '4+'
    END AS outpatient_visit_group,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY outpatient_visit_group
ORDER BY
    CASE outpatient_visit_group
        WHEN '0' THEN 1
        WHEN '1' THEN 2
        WHEN '2' THEN 3
        WHEN '3' THEN 4
        WHEN '4+' THEN 5
    END;
```

### Result

| Prior Outpatient Visits | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ----------------------- | ---------------: | ------------------: | ---------------: |
| 0                       |           85,027 |               9,076 |           10.67% |
| 1                       |            8,547 |               1,190 |           13.92% |
| 2                       |            3,594 |                 494 |           13.75% |
| 3                       |            2,042 |                 251 |           12.29% |
| 4+                      |            2,556 |                 346 |           13.54% |

### Key Finding

Encounters with no prior outpatient visits had an observed 30-day readmission rate of **10.67%**.

The observed rate was higher among encounters with at least one prior outpatient visit:

* 1 visit — **13.92%**
* 2 visits — **13.75%**
* 3 visits — **12.29%**
* 4+ visits — **13.54%**

The highest observed rate was **13.92%** among encounters with one prior outpatient visit.

The relationship was **not monotonic**, because the readmission rate did not consistently increase as the number of outpatient visits increased.

### Business Interpretation

Prior outpatient utilization shows an observed association with 30-day readmission in this dataset.

Encounters with at least one prior outpatient visit generally had higher observed readmission rates than encounters with no prior outpatient visits. However, increasing outpatient utilization beyond one visit did not produce a consistent step-by-step increase.

This pattern differs from the stronger monotonic relationships observed for prior inpatient and emergency utilization.

Prior outpatient visits may reflect differences in healthcare utilization patterns, chronic disease management, or underlying patient characteristics. The analysis is descriptive and does not establish that outpatient utilization itself causes readmission.

### SQL Concepts Used

* `CASE` — creates practical outpatient-utilization groups.
* `GROUP BY` — calculates results separately for each utilization group.
* `COUNT()` — counts encounters.
* `SUM(CASE WHEN ...)` — counts 30-day readmissions.
* `ROUND()` — calculates the observed readmission rate.
* `ORDER BY CASE` — preserves the logical order of outpatient visit groups.
---
## 21. Readmission by Admission Source

### Business Question

Does the observed 30-day readmission rate differ according to the source from which the patient was admitted to the hospital?

### SQL Query

```sql id="q7m3v9"
SELECT
    CASE admission_source_id
        WHEN 1 THEN 'Physician Referral'
        WHEN 2 THEN 'Clinic Referral'
        WHEN 3 THEN 'HMO Referral'
        WHEN 4 THEN 'Transfer from Hospital'
        WHEN 5 THEN 'Transfer from SNF'
        WHEN 6 THEN 'Transfer from Other Healthcare Facility'
        WHEN 7 THEN 'Emergency Room'
        WHEN 8 THEN 'Court / Legal System'
        WHEN 9 THEN 'Not Available'
        WHEN 10 THEN 'Transfer from Critical Access Hospital'
        WHEN 11 THEN 'Normal Delivery'
        WHEN 12 THEN 'Premature Delivery'
        WHEN 13 THEN 'Sick Baby'
        WHEN 14 THEN 'Extramural Birth'
        WHEN 15 THEN 'Not Available'
        WHEN 17 THEN 'Transfer from Other Facility'
        WHEN 20 THEN 'Not Mapped'
        WHEN 21 THEN 'Unknown'
        WHEN 22 THEN 'Transfer from Other Facility'
        WHEN 25 THEN 'Transfer from Hospital'
        ELSE 'Other / Unknown'
    END AS admission_source,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY admission_source_id
HAVING COUNT(*) >= 500
ORDER BY readmission_rate_pct DESC;
```

### Result

| Admission Source                        | Total Encounters | 30-Day Readmissions | Readmission Rate |
| --------------------------------------- | ---------------: | ------------------: | ---------------: |
| Transfer from SNF                       |              855 |                 101 |           11.81% |
| Emergency Room                          |           57,494 |               6,720 |           11.69% |
| Physician Referral                      |           29,565 |               3,130 |           10.59% |
| Transfer from Other Facility            |            6,781 |                 706 |           10.41% |
| Clinic Referral                         |            1,104 |                 111 |           10.05% |
| Transfer from Hospital                  |            3,187 |                 309 |            9.70% |
| Transfer from Other Healthcare Facility |            2,264 |                 212 |            9.36% |

### Key Finding

Observed 30-day readmission rates differed across the major admission-source groups.

The highest observed rates were:

* Transfer from SNF — **11.81%**
* Emergency Room — **11.69%**
* Physician Referral — **10.59%**

The lowest observed rate among the displayed groups was **9.36%** for transfers from other healthcare facilities.

Emergency Room admissions represented the largest group, with **57,494 encounters** and an observed readmission rate of **11.69%**.

### Business Interpretation

Admission source shows differences in observed 30-day readmission rates in this dataset.

Emergency Room admissions had an observed rate of 11.69%, while physician referrals had a rate of 10.59%. Transfers from SNF had a slightly higher observed rate of 11.81%, although this group contained only 855 encounters.

Admission source may reflect differences in patient condition, healthcare utilization, referral pathways, or care setting before hospitalization. These descriptive results should therefore be considered alongside other patient and encounter characteristics rather than interpreted as evidence that a particular admission source causes readmission.

The analysis uses a minimum threshold of 500 encounters to reduce the influence of very small admission-source groups.

### SQL Concepts Used

* `CASE` — converts admission-source IDs into readable categories.
* `GROUP BY` — calculates results separately for each admission source.
* `COUNT()` — counts encounters.
* `SUM(CASE WHEN ...)` — counts 30-day readmissions.
* `HAVING COUNT(*) >= 500` — retains admission sources with sufficient encounter volume.
* `ROUND()` — calculates the observed readmission rate.
* `ORDER BY` — sorts admission sources by observed readmission rate.
---
## 22. Readmission by Discharge Disposition

### Business Question

Does the observed 30-day readmission rate differ according to the patient's discharge disposition?

### SQL Query

```sql
SELECT
    CASE discharge_disposition_id
        WHEN 1 THEN 'Home'
        WHEN 2 THEN 'Short-Term Hospital'
        WHEN 3 THEN 'SNF'
        WHEN 4 THEN 'Intermediate Care'
        WHEN 5 THEN 'Another Institution'
        WHEN 6 THEN 'Home with Home Health'
        WHEN 7 THEN 'Left AMA'
        WHEN 8 THEN 'Home with IV Provider'
        WHEN 9 THEN 'Admitted as Inpatient'
        WHEN 10 THEN 'Hospice'
        WHEN 11 THEN 'Home'
        WHEN 12 THEN 'Other Facility'
        WHEN 13 THEN 'Hospice'
        WHEN 14 THEN 'Other Facility'
        WHEN 15 THEN 'Home'
        WHEN 16 THEN 'Hospice'
        WHEN 17 THEN 'Home'
        WHEN 18 THEN 'Home'
        WHEN 19 THEN 'Expired'
        WHEN 20 THEN 'Expired'
        WHEN 21 THEN 'Expired'
        WHEN 22 THEN 'Expired'
        WHEN 23 THEN 'Expired'
        WHEN 24 THEN 'Expired'
        WHEN 25 THEN 'Home'
        WHEN 26 THEN 'Expired'
        WHEN 27 THEN 'Expired'
        WHEN 28 THEN 'Unknown'
        WHEN 29 THEN 'Transferred to another institution'
        WHEN 30 THEN 'Expired'
        ELSE 'Other / Unknown'
    END AS discharge_disposition,
    COUNT(*) AS total_encounters,
    SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END) AS readmitted_30d,
    ROUND(
        100.0 * SUM(CASE WHEN readmitted = '<30' THEN 1 ELSE 0 END)
        / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY discharge_disposition_id
HAVING COUNT(*) >= 500
ORDER BY readmission_rate_pct DESC;
```

### Result

| Discharge Disposition | Total Encounters | 30-Day Readmissions | Readmission Rate |
| --------------------- | ---------------: | ------------------: | ---------------: |
| Expired               |            1,993 |                 552 |           27.70% |
| Another Institution   |            1,184 |                 247 |           20.86% |
| Short-Term Hospital   |            2,128 |                 342 |           16.07% |
| SNF                   |           13,954 |               2,046 |           14.66% |
| Left AMA              |              623 |                  90 |           14.45% |
| Intermediate Care     |              815 |                 104 |           12.76% |
| Home with Home Health |           12,902 |               1,638 |           12.70% |
| Home                  |            3,691 |                 459 |           12.44% |
| Home                  |              989 |                  92 |            9.30% |
| Home                  |           60,234 |               5,602 |            9.30% |
| Home                  |            1,642 |                   0 |            0.00% |

### Key Finding

Observed 30-day readmission rates varied considerably across discharge disposition codes.

Among the reported categories, the `Expired` disposition had an observed rate of 27.70%, followed by `Another Institution` at 20.86% and `Short-Term Hospital` at 16.07%. SNF had a 14.66% observed rate across 13,954 encounters, while Home with Home Health had a 12.70% rate across 12,902 encounters.

The largest group in the output was the Home disposition with 60,234 encounters and an observed 30-day readmission rate of 9.30%.

### Important Data Interpretation Note

Several different `discharge_disposition_id` values are mapped to the descriptive label `Home`. Therefore, the repeated `Home` rows should not automatically be combined without retaining the original disposition ID.

This demonstrates the importance of preserving the original coded variable when creating business-friendly labels.

### Business Interpretation

Discharge disposition appears to be associated with meaningful differences in observed 30-day readmission rates in this dataset.

Patients discharged to skilled nursing facilities, home with home health, other institutions, or short-term hospitals show different observed readmission patterns from patients discharged to home.

These differences may reflect differences in patient complexity, severity of illness, post-discharge care requirements, or other characteristics. The analysis is descriptive and does not establish that discharge disposition itself causes readmission.

The `Expired` category requires particular caution because it represents a fundamentally different disposition from ordinary post-hospital discharge and should not be interpreted as a conventional discharge-risk category.

### SQL Concepts Used

* `CASE` for translating coded discharge disposition values into descriptive labels
* `COUNT()` for encounter volume
* `SUM(CASE WHEN...)` for counting 30-day readmissions
* `ROUND()` for calculating percentages
* `GROUP BY` for category-level analysis
* `HAVING` for excluding categories with fewer than 500 encounters
* `ORDER BY` for sorting observed readmission rates
---
## 23. Readmission by Prior Inpatient Visits and Number of Diagnoses

### Business Question

How does the observed 30-day readmission rate vary when prior inpatient utilization and the number of diagnoses recorded during the encounter are considered together?

### SQL Query

```sql
SELECT
    CASE
        WHEN number_inpatient = 0 THEN '0'
        WHEN number_inpatient = 1 THEN '1'
        WHEN number_inpatient = 2 THEN '2'
        ELSE '3+'
    END AS inpatient_visit_group,
    CASE
        WHEN number_diagnoses BETWEEN 1 AND 4 THEN '1-4'
        WHEN number_diagnoses BETWEEN 5 AND 6 THEN '5-6'
        WHEN number_diagnoses BETWEEN 7 AND 8 THEN '7-8'
        ELSE '9+'
    END AS diagnosis_group,
    COUNT(*) AS total_encounters,
    SUM(
        CASE
            WHEN readmitted = '<30' THEN 1
            ELSE 0
        END
    ) AS readmitted_30d,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY
    inpatient_visit_group,
    diagnosis_group
ORDER BY
    CASE inpatient_visit_group
        WHEN '0' THEN 1
        WHEN '1' THEN 2
        WHEN '2' THEN 3
        WHEN '3+' THEN 4
    END,
    CASE diagnosis_group
        WHEN '1-4' THEN 1
        WHEN '5-6' THEN 2
        WHEN '7-8' THEN 3
        WHEN '9+' THEN 4
    END;
```

### Result

| Prior Inpatient Visits | Diagnosis Group | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ---------------------- | --------------- | ---------------: | ------------------: | ---------------: |
| 0                      | 1–4             |            7,608 |                 441 |            5.80% |
| 0                      | 5–6             |           15,607 |               1,145 |            7.34% |
| 0                      | 7–8             |           14,230 |               1,222 |            8.59% |
| 0                      | 9+              |           30,185 |               2,898 |            9.60% |
| 1                      | 1–4             |            1,277 |                 127 |            9.95% |
| 1                      | 5–6             |            3,597 |                 440 |           12.23% |
| 1                      | 7–8             |            4,024 |                 537 |           13.34% |
| 1                      | 9+              |           10,623 |               1,419 |           13.36% |
| 2                      | 1–4             |              347 |                  66 |           19.02% |
| 2                      | 5–6             |            1,265 |                 223 |           17.63% |
| 2                      | 7–8             |            1,490 |                 272 |           18.26% |
| 2                      | 9+              |            4,464 |                 758 |           16.98% |
| 3+                     | 1–4             |              382 |                 107 |           28.01% |
| 3+                     | 5–6             |            1,085 |                 293 |           27.00% |
| 3+                     | 7–8             |            1,265 |                 342 |           27.04% |
| 3+                     | 9+              |            4,317 |               1,067 |           24.72% |

### Key Finding

The observed 30-day readmission rate generally increases substantially as prior inpatient utilization increases.

Among encounters with no prior inpatient visits, observed readmission rates ranged from 5.80% to 9.60% across diagnosis groups.

Among encounters with 3 or more prior inpatient visits, observed readmission rates ranged from 24.72% to 28.01%.

Within the 9+ diagnosis group, the observed readmission rate increased from:

* 9.60% with 0 prior inpatient visits
* 13.36% with 1 prior visit
* 16.98% with 2 prior visits
* 24.72% with 3+ prior visits

This represents a 15.12 percentage-point difference between 0 and 3+ prior inpatient visits within the same diagnosis group.

### Business Interpretation

The combination of prior inpatient utilization and diagnosis burden provides a more detailed view of readmission patterns than either variable considered alone.

Prior inpatient utilization shows a particularly strong descriptive gradient in this dataset. Patients with repeated prior inpatient encounters had substantially higher observed 30-day readmission rates across diagnosis groups.

Diagnosis burden also shows an increasing pattern among patients with 0 or 1 prior inpatient visit. However, the relationship becomes less monotonic among patients with 2 or more prior inpatient visits.

This suggests that prior inpatient utilization may act as an important marker of underlying healthcare utilization and patient complexity.

The results remain observational. Prior inpatient utilization or number of diagnoses should not be interpreted as independently causing readmission. Other patient, clinical, and healthcare-utilization characteristics may contribute to the observed relationships.

### Important Sample Size Consideration

Some combinations contain relatively few encounters. For example:

* 2 prior inpatient visits + 1–4 diagnoses: 347 encounters
* 3+ prior inpatient visits + 1–4 diagnoses: 382 encounters

Therefore, these subgroup rates should be interpreted with more caution than results from larger groups.

### SQL Concepts Used

* `CASE` for creating analytical groups
* Multiple `CASE` expressions within the same query
* `COUNT()` for encounter volume
* `SUM(CASE WHEN...)` for outcome counts
* `ROUND()` for percentage calculation
* `GROUP BY` using multiple dimensions
* Multiple `ORDER BY CASE` expressions for logical category ordering
* Two-dimensional segmentation for business analysis
---
## 24. Identifying High-Volume Groups with Above-Average Readmission

### Business Question

Which prior-inpatient-utilization groups have both a meaningful number of encounters and an observed 30-day readmission rate above the overall dataset average?

### Benchmark

The overall observed 30-day readmission rate in the dataset is 11.16%.

For this analysis, a group must satisfy both conditions:

* At least 1,000 encounters
* Observed 30-day readmission rate above 11.16%

### SQL Query

```sql id="n4x7q2"
SELECT
    CASE
        WHEN number_inpatient = 0 THEN '0'
        WHEN number_inpatient = 1 THEN '1'
        WHEN number_inpatient = 2 THEN '2'
        WHEN number_inpatient = 3 THEN '3'
        ELSE '4+'
    END AS inpatient_visit_group,
    COUNT(*) AS total_encounters,
    SUM(
        CASE
            WHEN readmitted = '<30' THEN 1
            ELSE 0
        END
    ) AS readmitted_30d,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS readmission_rate_pct
FROM diabetic_data
GROUP BY inpatient_visit_group
HAVING COUNT(*) >= 1000
   AND (
        100.0 * SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) / COUNT(*)
   ) > 11.16
ORDER BY readmission_rate_pct DESC;
```

### Result

| Prior Inpatient Visits | Total Encounters | 30-Day Readmissions | Readmission Rate |
| ---------------------- | ---------------: | ------------------: | ---------------: |
| 4+                     |            3,638 |               1,117 |           30.70% |
| 3                      |            3,411 |                 692 |           20.29% |
| 2                      |            7,566 |               1,319 |           17.43% |
| 1                      |           19,521 |               2,523 |           12.92% |

### Key Finding

Four prior inpatient-utilization groups met both business criteria.

The observed readmission rate increased substantially as prior inpatient utilization increased:

* 1 prior visit: 12.92%
* 2 prior visits: 17.43%
* 3 prior visits: 20.29%
* 4+ prior visits: 30.70%

All four groups also contain at least 1,000 encounters, making them sufficiently large for meaningful descriptive analysis compared with very small subgroups.

The 4+ group had an observed readmission rate approximately 2.4 times the 1-visit group's rate.

### Business Interpretation

Prior inpatient utilization identifies a set of patient groups with observed 30-day readmission rates above the overall dataset benchmark.

From an analytics perspective, these groups could be considered **priority segments for further investigation**. A healthcare organization could examine these groups more closely to understand associated clinical characteristics, utilization patterns, discharge pathways, and other factors.

However, this analysis does not establish that prior inpatient utilization itself causes readmission. It should be interpreted as a descriptive segmentation of historical encounters.

The threshold of 1,000 encounters and the 11.16% benchmark are analytical choices used to demonstrate a business-filtering approach; different operational objectives could justify different thresholds.

### SQL Concepts Used

* `CASE` for creating utilization groups
* `COUNT()` for group volume
* `SUM(CASE WHEN...)` for outcome counts
* `ROUND()` for percentage calculation
* `GROUP BY` for aggregation
* `HAVING` for filtering aggregated results
* Multiple conditions inside `HAVING`
* `ORDER BY` for ranking the resulting groups
* Business-rule-based segmentation
---
## 25. Excess Readmissions Compared with the Overall Benchmark

### Business Question

How many observed 30-day readmissions occur above or below the number expected if each inpatient-utilization group had the overall dataset readmission rate?

### Benchmark

The overall observed 30-day readmission rate is 11.16%.

For each prior-inpatient-utilization group:

**Expected readmissions** = Total encounters × 11.16%

**Excess readmissions vs benchmark** = Observed readmissions − Expected readmissions

This is a descriptive benchmark calculation. It does not represent the number of preventable readmissions.

### SQL Query

```sql id="p73m8k"
SELECT
    CASE
        WHEN number_inpatient = 0 THEN '0'
        WHEN number_inpatient = 1 THEN '1'
        WHEN number_inpatient = 2 THEN '2'
        WHEN number_inpatient = 3 THEN '3'
        ELSE '4+'
    END AS inpatient_visit_group,
    COUNT(*) AS total_encounters,
    SUM(
        CASE
            WHEN readmitted = '<30' THEN 1
            ELSE 0
        END
    ) AS observed_readmissions,
    ROUND(
        COUNT(*) * 0.1116,
        1
    ) AS expected_readmissions_at_overall_rate,
    ROUND(
        SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) - COUNT(*) * 0.1116,
        1
    ) AS excess_readmissions_vs_benchmark
FROM diabetic_data
GROUP BY inpatient_visit_group
ORDER BY
    excess_readmissions_vs_benchmark DESC;
```

### Result

| Prior Inpatient Visits | Total Encounters | Observed Readmissions | Expected Readmissions at 11.16% | Excess Readmissions vs Benchmark |
| ---------------------- | ---------------: | --------------------: | ------------------------------: | -------------------------------: |
| 4+                     |            3,638 |                 1,117 |                           406.0 |                            711.0 |
| 2                      |            7,566 |                 1,319 |                           844.4 |                            474.6 |
| 1                      |           19,521 |                 2,523 |                         2,178.5 |                            344.5 |
| 3                      |            3,411 |                   692 |                           380.7 |                            311.3 |
| 0                      |           67,630 |                 5,706 |                         7,547.5 |                         -1,841.5 |

### Key Finding

The `4+` prior-inpatient-visit group had the highest observed readmission rate at 30.70% and approximately 711 more observed readmissions than would be expected using the overall 11.16% benchmark.

The `2` prior-visit group had approximately 475 excess observed readmissions, while the `1` and `3` visit groups had approximately 345 and 311 excess observed readmissions respectively.

The `0` prior-visit group had an observed rate below the overall benchmark, resulting in approximately 1,842 fewer observed readmissions than would be expected at the overall rate.

### Important Business Insight

This analysis demonstrates an important distinction between **rate-based prioritization** and **volume-based prioritization**.

A group with the highest readmission rate does not necessarily contribute the largest number of observations above a benchmark. Both the size of the population and the difference from the benchmark influence the resulting excess count.

For example, the `4+` group has a substantially elevated rate, while the `2` and `1` groups contain more encounters and therefore also contribute a considerable number of excess observed readmissions relative to the benchmark.

This type of analysis can help healthcare analysts identify groups for deeper investigation.

### Important Interpretation Limitation

"Excess readmissions vs benchmark" is a mathematical comparison against the overall dataset rate. It should **not** be interpreted as:

* preventable readmissions
* avoidable readmissions
* readmissions caused by prior inpatient visits
* the number of readmissions that an intervention would eliminate

The benchmark is descriptive and does not adjust for patient characteristics, clinical severity, or other confounding factors.

### Business Interpretation

Prior inpatient utilization shows a strong observed relationship with 30-day readmission in this historical dataset.

The benchmark analysis adds another perspective by incorporating both group size and observed readmission rate. This illustrates how healthcare analytics can move from simple descriptive statistics toward metrics that are more directly useful for operational investigation.

Further analysis would be required before using these findings to design clinical interventions or estimate preventable readmissions.

### SQL Concepts Used

* `CASE` for creating analytical groups
* `COUNT()` for encounter volume
* `SUM(CASE WHEN...)` for observed outcomes
* Arithmetic calculations for expected values
* `ROUND()` for derived metrics
* `GROUP BY` for segmentation
* `ORDER BY` using a calculated business metric
* Benchmark-based analysis
* Derived business metrics
---
## 26. Patient-Level 30-Day Readmission Analysis

### Business Question

How many unique patients experienced at least one 30-day readmission, and how does the patient-level readmission rate differ from the encounter-level rate?

### Why Patient-Level Analysis Matters

The dataset contains multiple hospital encounters for some patients. Therefore, an encounter-level readmission rate and a patient-level readmission rate answer different questions.

* **Encounter-level analysis:** What proportion of hospital encounters were followed by a 30-day readmission?
* **Patient-level analysis:** What proportion of unique patients experienced at least one 30-day readmission?

Both perspectives can be useful in healthcare analytics.

### SQL Query

```sql id="q68qmz"
SELECT
    COUNT(*) AS total_patients,
    SUM(
        CASE
            WHEN readmission_30d_count > 0 THEN 1
            ELSE 0
        END
    ) AS patients_with_30d_readmission,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN readmission_30d_count > 0 THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS patient_readmission_rate_pct
FROM (
    SELECT
        patient_nbr,
        SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) AS readmission_30d_count
    FROM diabetic_data
    GROUP BY patient_nbr
) AS patient_summary;
```

### Result

| Metric                                        |  Value |
| --------------------------------------------- | -----: |
| Unique patients                               | 71,518 |
| Patients with at least one 30-day readmission |  8,834 |
| Patient-level readmission rate                | 12.35% |

### Comparison with Encounter-Level Analysis

The earlier encounter-level analysis found:

* 101,766 total encounters
* 11,357 30-day readmissions
* 11.16% encounter-level readmission rate

The patient-level analysis found:

* 71,518 unique patients
* 8,834 patients with at least one 30-day readmission
* 12.35% patient-level readmission rate

These percentages should not be interpreted as conflicting results because their denominators are different.

### Key Finding

Approximately **12.35% of unique patients** experienced at least one 30-day readmission in the dataset.

This differs from the **11.16% encounter-level rate** because some patients contributed multiple hospital encounters and may therefore appear more than once in encounter-level calculations.

### Business Interpretation

Patient-level analysis provides a complementary perspective for healthcare organizations.

An encounter-level metric is useful for understanding the frequency of readmissions relative to hospital encounters, while a patient-level metric provides insight into how widespread the readmission experience is across the patient population.

For patient outreach or care-management analysis, the patient-level perspective can be particularly relevant because interventions are generally directed toward individual patients rather than individual rows in a hospital encounter table.

However, this calculation only identifies whether a patient had **at least one** 30-day readmission. It does not measure the number of distinct patients who were repeatedly readmitted, nor does it establish why a patient was readmitted.

### Important Limitation

The patient-level rate is descriptive and specific to this dataset.

It should not be interpreted as a current population estimate or as evidence that the observed patient characteristics caused readmission.

The dataset represents historical U.S. hospital encounters from 1999–2008 and may not reflect current healthcare utilization patterns.

### SQL Concepts Used

* Subqueries
* `GROUP BY` at the patient level
* `COUNT()`
* `SUM()`
* `CASE WHEN`
* Calculated percentages
* Nested aggregation
* Distinguishing encounter-level and patient-level analysis
---
## 27. Repeat 30-Day Readmission Burden

### Business Question

Among patients who experienced at least one 30-day readmission, how many experienced repeated 30-day readmissions?

### SQL Query

```sql id="r42nqp"
SELECT
    CASE
        WHEN readmission_30d_count = 1 THEN '1'
        WHEN readmission_30d_count = 2 THEN '2'
        WHEN readmission_30d_count = 3 THEN '3'
        ELSE '4+'
    END AS readmission_group,
    COUNT(*) AS patients
FROM (
    SELECT
        patient_nbr,
        SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) AS readmission_30d_count
    FROM diabetic_data
    GROUP BY patient_nbr
) AS patient_summary
WHERE readmission_30d_count > 0
GROUP BY readmission_group
ORDER BY
    CASE readmission_group
        WHEN '1' THEN 1
        WHEN '2' THEN 2
        WHEN '3' THEN 3
        WHEN '4+' THEN 4
    END;
```

### Result

| Number of 30-Day Readmissions |  Patients | Share of Patients with ≥1 Readmission |
| ----------------------------- | --------: | ------------------------------------: |
| 1                             |     7,295 |                                82.59% |
| 2                             |     1,051 |                                11.89% |
| 3                             |       290 |                                 3.28% |
| 4+                            |       198 |                                 2.24% |
| **Total**                     | **8,834** |                           **100.00%** |

### Key Finding

Among the 8,834 patients who experienced at least one 30-day readmission:

* 7,295 patients had exactly one 30-day readmission.
* 1,051 had two.
* 290 had three.
* 198 had four or more.

Overall, **1,539 patients, or approximately 17.41% of patients with at least one 30-day readmission, experienced two or more 30-day readmissions.**

### Business Interpretation

Most patients with at least one 30-day readmission experienced a single such encounter. However, a smaller subset experienced repeated 30-day readmissions.

This demonstrates why patient-level analysis can provide information that is not visible from a simple overall readmission rate.

For healthcare analytics, identifying the distribution of repeated readmissions can help analysts understand whether the overall readmission burden is concentrated among patients with repeated events.

Further analysis would be required to understand the clinical characteristics, utilization patterns, or other factors associated with repeated readmissions.

### Important Interpretation Limitation

Repeated `<30` encounters in this dataset should not automatically be interpreted as independent clinical events or as evidence of preventable readmissions.

The analysis is descriptive and does not establish why repeated readmissions occurred.

The dataset is also historical and represents U.S. hospital encounters from 1999–2008.

### SQL Concepts Used

* Subqueries
* Patient-level aggregation
* `SUM(CASE WHEN...)`
* `GROUP BY`
* `WHERE`
* `CASE` for creating analytical groups
* Nested aggregation
* Patient-level cohort analysis
---
## 28. Patient-Level Utilization Profile

### Business Question

How does healthcare utilization differ across patients with different numbers of recorded hospital encounters?

### Analytical Approach

Instead of analyzing each hospital encounter independently, this analysis first creates a patient-level summary.

Each patient is summarized using:

* number of recorded encounters
* number of 30-day readmissions
* inpatient utilization
* emergency utilization
* outpatient utilization
* average length of hospital stay
* average number of diagnoses

Patients are then grouped according to the number of encounters recorded in the dataset.

### SQL Query

```sql id="m51xrd"
SELECT
    CASE
        WHEN encounter_count = 1 THEN '1'
        WHEN encounter_count BETWEEN 2 AND 3 THEN '2-3'
        WHEN encounter_count BETWEEN 4 AND 5 THEN '4-5'
        ELSE '6+'
    END AS encounter_group,
    COUNT(*) AS patients,
    ROUND(AVG(encounter_count), 2) AS avg_encounters,
    ROUND(AVG(readmission_30d_count), 2) AS avg_30d_readmissions,
    ROUND(AVG(total_inpatient_visits), 2) AS avg_inpatient_visits,
    ROUND(AVG(total_emergency_visits), 2) AS avg_emergency_visits,
    ROUND(AVG(total_outpatient_visits), 2) AS avg_outpatient_visits,
    ROUND(AVG(avg_length_of_stay), 2) AS avg_length_of_stay,
    ROUND(AVG(avg_diagnoses), 2) AS avg_diagnoses
FROM (
    SELECT
        patient_nbr,
        COUNT(*) AS encounter_count,
        SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) AS readmission_30d_count,
        SUM(number_inpatient) AS total_inpatient_visits,
        SUM(number_emergency) AS total_emergency_visits,
        SUM(number_outpatient) AS total_outpatient_visits,
        AVG(time_in_hospital) AS avg_length_of_stay,
        AVG(number_diagnoses) AS avg_diagnoses
    FROM diabetic_data
    GROUP BY patient_nbr
) AS patient_profile
GROUP BY encounter_group
ORDER BY
    CASE encounter_group
        WHEN '1' THEN 1
        WHEN '2-3' THEN 2
        WHEN '4-5' THEN 3
        WHEN '6+' THEN 4
    END;
```

### Result

| Encounter Group | Patients | Avg Encounters | Avg 30-Day Readmissions | Avg Inpatient Visits | Avg Emergency Visits | Avg Outpatient Visits | Avg Length of Stay | Avg Diagnoses |
| --------------- | -------: | -------------: | ----------------------: | -------------------: | -------------------: | --------------------: | -----------------: | ------------: |
| 1               |   54,745 |           1.00 |                    0.04 |                 0.17 |                 0.09 |                  0.27 |               4.22 |          7.18 |
| 2-3             |   13,762 |           2.24 |                    0.37 |                 1.60 |                 0.42 |                  0.92 |               4.55 |          7.59 |
| 4-5             |    2,138 |           4.34 |                    0.97 |                 6.10 |                 1.44 |                  2.33 |               4.73 |          7.87 |
| 6+              |      873 |           7.90 |                    2.38 |                23.09 |                 7.16 |                  6.12 |               4.62 |          7.89 |

### Key Findings

A clear utilization gradient is visible across the encounter groups.

Patients with only one recorded encounter had:

* 0.17 average inpatient visits
* 0.09 average emergency visits
* 0.27 average outpatient visits
* 4.22 average days in hospital
* 7.18 average diagnoses

In comparison, patients in the `6+` encounter group had:

* 23.09 average inpatient visits
* 7.16 average emergency visits
* 6.12 average outpatient visits
* 4.62 average days in hospital
* 7.89 average diagnoses

The `6+` group also had an average of **2.38 30-day readmissions per patient**, compared with **0.04** among patients with only one recorded encounter.

### Business Interpretation

The results show that patients with more recorded encounters also tend to have substantially higher healthcare utilization across inpatient, emergency, and outpatient measures.

This suggests that **overall healthcare utilization is an important dimension for understanding readmission burden** in this dataset.

The findings also demonstrate the value of patient-level feature engineering. Instead of examining isolated encounter variables, a healthcare analyst can create a consolidated patient profile that can subsequently be used for:

* segmentation
* dashboard reporting
* statistical analysis
* predictive modeling
* care-management research
* identification of high-utilization populations

### Important Interpretation Limitation

The `encounter_group` represents the number of encounters recorded for each patient within this dataset. It should not automatically be interpreted as the patient's complete lifetime healthcare utilization.

Similarly, inpatient, emergency, and outpatient values are utilization measures recorded in the source data and should not necessarily be interpreted as unique visits generated exclusively within this dataset.

The observed relationships are descriptive and do not establish that higher utilization causes readmission.

### SQL Concepts Used

* Nested subqueries
* Patient-level aggregation
* `COUNT()`
* `SUM()`
* `AVG()`
* `CASE`
* `GROUP BY`
* Derived analytical features
* Patient segmentation
* Multi-metric aggregation
---
## 29. Overall Hospital Readmission KPI Summary

### Business Question

What are the core hospital readmission KPIs for the complete dataset, and how do encounter-level and patient-level readmission measures compare?

### SQL Query

```sql id="t83vpl"
SELECT
    COUNT(*) AS total_encounters,
    COUNT(DISTINCT patient_nbr) AS unique_patients,
    SUM(
        CASE
            WHEN readmitted = '<30' THEN 1
            ELSE 0
        END
    ) AS readmitted_30d_encounters,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS encounter_readmission_rate_pct,
    COUNT(
        DISTINCT CASE
            WHEN readmitted = '<30' THEN patient_nbr
        END
    ) AS patients_with_30d_readmission,
    ROUND(
        100.0 * COUNT(
            DISTINCT CASE
                WHEN readmitted = '<30' THEN patient_nbr
            END
        ) / COUNT(DISTINCT patient_nbr),
        2
    ) AS patient_readmission_rate_pct
FROM diabetic_data;
```

### Result

| KPI                                           |   Value |
| --------------------------------------------- | ------: |
| Total encounters                              | 101,766 |
| Unique patients                               |  71,518 |
| 30-day readmission encounters                 |  11,357 |
| Encounter-level readmission rate              |  11.16% |
| Patients with at least one 30-day readmission |   8,834 |
| Patient-level readmission rate                |  12.35% |

### Key Findings

The dataset contains **101,766 hospital encounters involving 71,518 unique patients**.

There were **11,357 encounters classified as 30-day readmissions**, producing an encounter-level readmission rate of **11.16%**.

At the patient level, **8,834 unique patients experienced at least one 30-day readmission**, corresponding to a patient-level rate of **12.35%**.

### Encounter-Level vs Patient-Level Readmission

The two rates measure different things:

**Encounter-level rate — 11.16%**

Measures the proportion of hospital encounters associated with a `<30` readmission outcome.

**Patient-level rate — 12.35%**

Measures the proportion of unique patients who experienced at least one `<30` readmission.

The patient-level rate is therefore not a replacement for the encounter-level rate. Both metrics should be retained because they provide different perspectives on readmission burden.

### Business Interpretation

These KPIs provide the high-level baseline for the entire analysis.

The dataset represents a substantial number of hospital encounters and patients, while the readmission analysis shows that 30-day readmission is a meaningful outcome within the dataset.

The KPI summary can serve as the foundation for an executive healthcare analytics dashboard, with additional analysis breaking the overall rate down by:

* prior inpatient utilization
* emergency utilization
* age
* length of stay
* medication burden
* diagnosis burden
* admission characteristics
* primary diagnosis category
* patient-level utilization

### Portfolio Relevance

This type of KPI summary demonstrates the ability to convert a large clinical dataset into a small number of business-facing metrics.

The same KPIs can later be represented as Power BI cards and used as the starting point for an interactive hospital readmission dashboard.

### Important Interpretation Limitation

These metrics describe the historical dataset and should not be interpreted as current hospital performance benchmarks.

The dataset represents U.S. hospital encounters from 1999–2008 and may not represent current healthcare systems or patient populations.

### SQL Concepts Used

* `COUNT()`
* `COUNT(DISTINCT ...)`
* `SUM()`
* `CASE WHEN`
* Calculated percentages
* `ROUND()`
* Conditional distinct counting
* Encounter-level aggregation
* Patient-level aggregation
---
## 30. Summary of Major Observed Readmission Patterns

### Business Question

Which utilization and clinical-complexity factors show the largest observed differences in 30-day readmission rates between lower- and higher-level groups?

### Analytical Approach

Previous SQL analyses identified several variables with observable differences in 30-day readmission rates.

This summary focuses on three factors with particularly clear descriptive patterns:

1. Prior inpatient utilization
2. Prior emergency utilization
3. Number of diagnoses

The comparison is expressed as a **percentage-point difference** between the selected lower- and higher-utilization groups.

### SQL Query

```sql id="u94kcs"
SELECT
    'Prior Inpatient Visits' AS factor,
    '0 vs 4+' AS comparison,
    8.44 AS lower_group_rate_pct,
    30.70 AS higher_group_rate_pct,
    ROUND(30.70 - 8.44, 2) AS rate_difference_pp

UNION ALL

SELECT
    'Prior Emergency Visits' AS factor,
    '0 vs 3+' AS comparison,
    10.47 AS lower_group_rate_pct,
    24.94 AS higher_group_rate_pct,
    ROUND(24.94 - 10.47, 2) AS rate_difference_pp

UNION ALL

SELECT
    'Number of Diagnoses' AS factor,
    '1-4 vs 9' AS comparison,
    7.71 AS lower_group_rate_pct,
    12.38 AS higher_group_rate_pct,
    ROUND(12.38 - 7.71, 2) AS rate_difference_pp;
```

### Result

| Factor                 | Comparison | Lower Group Rate | Higher Group Rate | Difference |
| ---------------------- | ---------- | ---------------: | ----------------: | ---------: |
| Prior Inpatient Visits | 0 vs 4+    |            8.44% |            30.70% |   22.26 pp |
| Prior Emergency Visits | 0 vs 3+    |           10.47% |            24.94% |   14.47 pp |
| Number of Diagnoses    | 1-4 vs 9   |            7.71% |            12.38% |    4.67 pp |

### Key Findings

The largest observed difference among the selected factors was associated with prior inpatient utilization.

Patients in the `0` prior-inpatient-visit group had an observed 30-day readmission rate of **8.44%**, compared with **30.70%** among patients with `4+` prior inpatient visits, a difference of **22.26 percentage points**.

Prior emergency utilization also showed a substantial difference:

* 0 prior emergency visits: **10.47%**
* 3+ prior emergency visits: **24.94%**
* Difference: **14.47 percentage points**

Number of diagnoses showed a smaller but still observable difference:

* 1–4 diagnoses: **7.71%**
* 9 diagnoses: **12.38%**
* Difference: **4.67 percentage points**

### Business Interpretation

The analysis indicates that historical healthcare utilization and clinical complexity measures are associated with different observed levels of 30-day readmission in this dataset.

Among the selected comparisons, prior inpatient utilization shows the largest descriptive difference.

This supports the use of utilization history as an important analytical dimension when investigating hospital readmissions.

However, these comparisons are **unadjusted**. The groups may differ in age, disease severity, diagnosis, treatment, and other characteristics.

Therefore, the results should be interpreted as **observed associations rather than causal effects**.

### Portfolio Insight

This summary demonstrates an important analytics workflow:

**Raw clinical data → segmentation → KPI calculation → comparison → business interpretation**

The results can be translated into Power BI visualizations such as:

* readmission rate by prior inpatient visits
* readmission rate by prior emergency visits
* readmission rate by diagnosis burden
* KPI cards for overall readmission metrics

### Important Interpretation Limitation

The percentage-point differences do not represent causal effects or expected reductions from an intervention.

They are descriptive comparisons within the historical dataset.

The dataset represents U.S. hospital encounters from 1999–2008 and should not be treated as a current benchmark for hospitals in India or other healthcare systems.

### SQL Concepts Used

* `UNION ALL`
* Literal analytical values
* Calculated columns
* `ROUND()`
* Percentage-point calculations
* Presentation-oriented summary queries
* Translating exploratory analysis into business-facing metrics
---
## 31. Dashboard-Ready Readmission Summary

### Business Question

How does the observed 30-day readmission rate vary across prior inpatient-utilization groups, and how does each group compare with the overall dataset benchmark?

### Analytical Objective

This analysis converts the previously explored inpatient-utilization relationship into a compact reporting dataset suitable for downstream visualization.

The resulting output contains:

* encounter volume
* 30-day readmission count
* observed readmission rate
* overall benchmark
* difference from the benchmark

This represents the transition from exploratory SQL analysis toward dashboard-ready reporting.

### SQL Query

```sql id="x27nfd"
SELECT
    CASE
        WHEN number_inpatient = 0 THEN '0'
        WHEN number_inpatient = 1 THEN '1'
        WHEN number_inpatient = 2 THEN '2'
        WHEN number_inpatient = 3 THEN '3'
        ELSE '4+'
    END AS inpatient_visit_group,
    COUNT(*) AS total_encounters,
    SUM(
        CASE
            WHEN readmitted = '<30' THEN 1
            ELSE 0
        END
    ) AS readmitted_30d,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN readmitted = '<30' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS readmission_rate_pct,
    11.16 AS overall_benchmark_pct,
    ROUND(
        (
            100.0 * SUM(
                CASE
                    WHEN readmitted = '<30' THEN 1
                    ELSE 0
                END
            ) / COUNT(*)
        ) - 11.16,
        2
    ) AS difference_vs_benchmark_pp
FROM diabetic_data
GROUP BY inpatient_visit_group
ORDER BY
    CASE inpatient_visit_group
        WHEN '0' THEN 1
        WHEN '1' THEN 2
        WHEN '2' THEN 3
        WHEN '3' THEN 4
        WHEN '4+' THEN 5
    END;
```

### Result

| Prior Inpatient Visits | Total Encounters | 30-Day Readmissions | Readmission Rate | Overall Benchmark | Difference vs Benchmark |
| ---------------------- | ---------------: | ------------------: | ---------------: | ----------------: | ----------------------: |
| 0                      |           67,630 |               5,706 |            8.44% |            11.16% |                -2.72 pp |
| 1                      |           19,521 |               2,523 |           12.92% |            11.16% |                +1.76 pp |
| 2                      |            7,566 |               1,319 |           17.43% |            11.16% |                +6.27 pp |
| 3                      |            3,411 |                 692 |           20.29% |            11.16% |                +9.13 pp |
| 4+                     |            3,638 |               1,117 |           30.70% |            11.16% |               +19.54 pp |

### Key Finding

A clear increasing pattern is observed between prior inpatient utilization and 30-day readmission rate.

The observed rate increases from:

**8.44% → 12.92% → 17.43% → 20.29% → 30.70%**

as prior inpatient visits increase from `0` to `4+`.

Compared with the overall 11.16% benchmark:

* The `0` group is **2.72 percentage points below** the benchmark.
* The `1` group is **1.76 percentage points above** the benchmark.
* The `2` group is **6.27 percentage points above** the benchmark.
* The `3` group is **9.13 percentage points above** the benchmark.
* The `4+` group is **19.54 percentage points above** the benchmark.

### Business Interpretation

Prior inpatient utilization is one of the clearest descriptive patterns identified during the SQL analysis.

Patients with greater prior inpatient utilization have progressively higher observed 30-day readmission rates in this dataset.

This makes prior inpatient utilization a useful dimension for dashboard segmentation and further analytical investigation.

A healthcare organization could use this type of analysis to monitor how readmission rates vary across utilization segments and identify populations requiring further investigation.

However, the analysis does not establish that prior inpatient visits cause readmission. Higher prior utilization may reflect underlying disease severity, chronic conditions, or other patient characteristics.

### Dashboard Application

This output is suitable for a Power BI visualization such as:

**30-Day Readmission Rate by Prior Inpatient Visits**

Recommended fields:

* **Axis:** `inpatient_visit_group`
* **Value:** `readmission_rate_pct`
* **Reference line:** `overall_benchmark_pct`
* **Tooltip:** `total_encounters`, `readmitted_30d`, `difference_vs_benchmark_pp`

This allows the dashboard user to see both the readmission rate and its position relative to the overall benchmark.

### Portfolio Relevance

This section demonstrates the transition from:

**Raw clinical data → SQL analysis → derived reporting dataset → BI visualization**

The query is no longer simply answering an exploratory question. It produces a structured analytical output designed for communication to a business or healthcare stakeholder.

### Important Interpretation Limitation

The benchmark comparison is descriptive.

A positive difference from the overall benchmark does not represent preventable or excess readmissions in a clinical or policy sense.

The dataset is historical U.S. hospital data from 1999–2008 and should not be treated as a current hospital benchmark.

### SQL Concepts Used

* `CASE`
* `COUNT()`
* `SUM(CASE WHEN...)`
* `ROUND()`
* Derived metrics
* Percentage-point calculations
* `GROUP BY`
* Ordered analytical categories
* Dashboard-oriented reporting datasets
---
## 32. Final SQL Findings Summary

### Business Purpose

The SQL analysis phase consolidated the major readmission patterns identified in the diabetic hospital dataset into a concise set of validated findings.

These findings will serve as the analytical foundation for the Power BI dashboard and final portfolio report.

### Final SQL Findings

| Finding                                        | Metric | Population                                                   |
| ---------------------------------------------- | -----: | ------------------------------------------------------------ |
| Overall Encounter Readmission Rate             | 11.16% | 101,766 encounters                                           |
| Patient-Level Readmission Rate                 | 12.35% | 71,518 unique patients                                       |
| Highest Prior Inpatient Group Readmission Rate | 30.70% | 3,638 encounters with 4+ prior inpatient visits              |
| Highest Prior Emergency Group Readmission Rate | 24.94% | Encounters with 3+ prior emergency visits                    |
| High Diagnosis-Burden Group Readmission Rate   | 14.78% | 115 encounters with 10+ diagnoses                            |
| Patients with Repeated 30-Day Readmissions     | 17.41% | 1,539 of 8,834 patients with at least one 30-day readmission |

### Key Analytical Findings

#### 1. Overall readmission burden

The overall encounter-level 30-day readmission rate was **11.16%** across 101,766 encounters.

At the patient level, **8,834 of 71,518 patients** experienced at least one 30-day readmission, corresponding to **12.35%**.

These two metrics use different denominators and should therefore be presented separately.

#### 2. Prior inpatient utilization shows the strongest descriptive pattern

Observed readmission rates increased substantially as the number of prior inpatient visits increased:

* 0 prior inpatient visits: **8.44%**
* 1 prior inpatient visit: **12.92%**
* 2 prior inpatient visits: **17.43%**
* 3 prior inpatient visits: **20.29%**
* 4+ prior inpatient visits: **30.70%**

This represents a **22.26 percentage-point difference** between the 0-visit and 4+ groups.

This is one of the strongest descriptive patterns identified in the SQL analysis.

#### 3. Prior emergency utilization is also associated with higher observed readmission

The observed rate increased from approximately **10.47%** among encounters with no prior emergency visits to **24.94%** among encounters with 3+ prior emergency visits.

This represents a **14.47 percentage-point difference**.

#### 4. Repeated readmissions affect a smaller subset of patients

Among the 8,834 patients who experienced at least one 30-day readmission:

* 7,295 had one 30-day readmission
* 1,051 had two
* 290 had three
* 198 had four or more

Therefore, **1,539 patients (17.41%)** experienced two or more 30-day readmissions.

This indicates that repeated readmission is concentrated in a smaller subset of patients.

#### 5. Diagnosis burden shows a positive descriptive pattern, but small groups require caution

Readmission rates generally increased across diagnosis-count groups, with the selected 10+ diagnosis group showing **14.78%**.

However, this particular group contains only **115 encounters**. Therefore, it should not be treated as a major standalone business finding without additional statistical or clinical validation.

### Findings Selected for Power BI

The Power BI dashboard should prioritize findings with stronger volume and clearer business relevance:

1. Overall 30-day readmission rate
2. Patient-level readmission rate
3. Readmission by prior inpatient visits
4. Readmission by prior emergency visits
5. Readmission by age
6. Readmission by insulin treatment
7. Readmission by length of stay
8. Readmission by number of diagnoses
9. Repeat-readmission burden
10. Key demographic and admission characteristics

The dashboard should avoid presenting every exploratory SQL analysis simultaneously. The objective is to communicate the most meaningful patterns clearly.

### Important Analytical Caveats

* These are **observational associations**, not evidence of causation.
* The prior-utilization groups should not automatically be interpreted as causes of readmission.
* The dataset represents historical U.S. hospital encounters from approximately 1999–2008.
* Results should not be assumed to represent current Indian hospitals or the current diabetes population.
* Small groups should be interpreted cautiously.
* The model developed during the Python phase demonstrated only modest discrimination and should not be presented as a clinically deployable prediction system.
* “Excess readmissions” calculated against the overall benchmark represent deviations from the observed benchmark; they should **not** be described as preventable readmissions.

### SQL Portfolio Skills Demonstrated

The SQL phase demonstrates:

* Database creation and CSV import
* SQLite
* Table inspection using `PRAGMA`
* Aggregation with `COUNT()` and `SUM()`
* Conditional aggregation
* `GROUP BY`
* `ORDER BY`
* `CASE` expressions
* `WHERE` filtering
* Percentage calculations
* Benchmark comparisons
* Patient-level aggregation
* Repeat-event analysis
* Multi-dimensional grouping
* Dashboard-ready SQL outputs
* Translating analytical findings into business insights

### SQL Phase Conclusion

The SQL analysis transformed the raw diabetic hospital dataset into a set of validated, business-oriented readmission metrics.

The strongest recurring descriptive pattern was the relationship between **previous inpatient utilization and observed 30-day readmission**, while prior emergency utilization also showed a substantial association.

The analysis is now ready to transition into Power BI, where these findings will be converted into an interactive healthcare analytics dashboard.
---