# Software Requirements Specification (SRS)
## Project 3: Bike Rental Demand Analysis & Fleet Optimization System

**Document Version:** 1.0  
**Project Identifier:** SMART-MOBI-PRJ03  
**Domain:** Urban Micromobility & Smart City Transportation Analytics  
**Submission Category:** Academic / LMS Project Submission  
**Prepared by:** Senior Mobility Data Analyst  

---

## 1. Introduction

### 1.1 Purpose
This Software Requirements Specification (SRS) document details the end-to-end functional and non-functional specifications for the **Bike Rental Demand Analysis System**. The primary objective is to process historical hourly rental records, clean environmental and temporal metrics, analyze demand elasticity across seasons, diurnal hours, and weather conditions, and produce actionable operational intelligence for bike-sharing fleet allocation and rebalancing.

### 1.2 Document Conventions
- **Requirement Identifiers:** System requirements are tagged as `FR-XX` for Functional Requirements (mapped to Tasks 34–48) and `NFR-XX` for Non-Functional Requirements.
- **Priority Scale:** High (Mandatory for LMS compliance), Medium (Recommended operational enhancement).

### 1.3 Intended Audience
- **Micromobility Fleet Operations Managers:** To optimize van dispatching, battery swapping, and seasonal depot sizing.
- **Data Engineering & Analytics Teams:** To implement data ingestion, automated cleaning pipelines, and reporting dashboards.
- **Academic Evaluators & LMS Reviewers:** To verify rigorous compliance with project tasks 34 through 48.

### 1.4 Project Scope & Objectives
The system ingests the raw dataset `03_bike_rental.csv`, handles missing atmospheric sensor records, performs feature extraction (temporal partitioning), aggregates demand across environmental variables, detects demand surges (outliers) using non-parametric statistics (IQR), evaluates temperature sensitivity, and produces structured summaries and fleet management strategies.

---

## 2. Overall Description

### 2.1 Product Perspective & System Context
The system serves as the analytics and decision-support backbone for an urban bike-share operator. It ingests telemetry data from station docks and onboard IoT modules, coupled with meteorological sensor feeds.

```
[IoT Docks & User Apps]  +  [Meteorological Stations]
               \                     /
                v                   v
           [Raw Dataset: 03_bike_rental.csv]
                          |
                          v
         [ETL Pipeline: Cleaning & Imputation]
                          |
                          v
        [Cleaned Dataset: 03_bike_rental_cleaned.csv]
                          |
                          v
  [Analytical Modules: Diurnal, Seasonal, Weather, IQR]
                          |
                          v
   [Decision Engine: 5 Fleet Planning Recommendations]
```

### 2.2 User Classes and Characteristics
1. **Fleet Dispatcher / Field Operations Lead:** Requires clear peak and off-peak temporal windows to schedule station rebalancing vans.
2. **Maintenance Supervisor:** Requires low-demand maintenance windows (00:00–05:00) and seasonal overhaul targets (Winter).
3. **Data Scientist / Business Analyst:** Requires verified aggregations, correlation coefficients, and reproducible scripts.

### 2.3 Operating Environment & Technical Stack
- **Runtime Environment:** Python 3.10+ / Python 3.15
- **Core Analytics Libraries:** `pandas` (DataFrame operations, grouping, datetime extraction), `numpy` (vectorized math, percentiles, array means).
- **Visualization:** `matplotlib`, `seaborn`.
- **Operating Systems:** Windows 10/11, macOS, Linux.

---

## 3. Data Pipeline & Schema Specifications

### 3.1 Raw Data Dictionary (`03_bike_rental.csv`)
| Column Name | Data Type | Nullable | Description | Typical Range |
|:---|:---|:---|:---|:---|
| `DateTime` | String | No | Timestamp of rental record | `YYYY-MM-DD HH:MM:SS` |
| `season` | Integer | No | Season code (1:Spring, 2:Summer, 3:Fall, 4:Winter) | 1 to 4 |
| `holiday` | Integer | No | Binary flag indicating public holiday | 0 or 1 |
| `workingday` | Integer | No | Binary flag (1 if neither weekend nor holiday) | 0 or 1 |
| `weather` | Integer | No | Weather code (1:Clear, 2:Mist, 3:Light Rain/Snow, 4:Severe) | 1 to 4 |
| `temp` | Float | No | Ambient temperature in degrees Celsius | 1.0 to 40.0 °C |
| `humidity` | Float / Empty | **Yes** | Relative humidity percentage (Target of Task 35) | 15.0 to 100.0% |
| `windspeed` | Float | No | Wind speed in km/h | 0.0 to 50.0 km/h |
| `casual` | Integer | No | Count of non-registered casual rentals | $\ge 0$ |
| `registered` | Integer | No | Count of registered subscriber rentals | $\ge 0$ |
| `rental_count` | Integer | No | Total count of bike rentals (`casual + registered`) | $\ge 0$ |

### 3.2 Data Cleaning & Imputation Strategy (Task 35)
- **Problem Statement:** Atmospheric sensor dropout results in missing values in the `humidity` feature (~4.85% of records).
- **Cleaning Decision:** **Median Imputation** is chosen over dropping rows or mean imputation.
  - *Non-removal Justification:* Deleting records destroys chronological continuity, breaking hourly lags and skewing cumulative fleet demand.
  - *Median Justification:* Humidity distributions are constrained to $[0, 100]$ and prone to precipitation-induced skewness. The median ($67.5\%$) preserves central tendency without being distorted by extreme storm events.

### 3.3 Engineered Features (Task 36)
- `date`: Extracted calendar date (`YYYY-MM-DD`).
- `month`: Extracted integer month (`1` to `12`) via `.dt.month`.
- `hour`: Extracted integer hour (`0` to `23`) via `.dt.hour`.
- `day_type`: Categorical label derived from day of week (`'Weekend'` vs `'Weekday'`).
- `is_weekend`: Binary flag (1 for Saturday/Sunday, 0 for Monday–Friday).

---

## 4. Functional Requirements (Mapped to Tasks 34–48)

| Req ID | Task Ref | Description & Acceptance Criteria |
|:---|:---|:---|
| **FR-34** | Task 34 | **Dataset Ingestion:** System must load `03_bike_rental.csv`, inspect row/column dimensions (8760 rows, 11 columns), display schema, and summarize column datatypes. |
| **FR-35** | Task 35 | **Missing Data Handling:** System must identify missing values in `humidity`, document the missing percentage, and impute using median value ($67.5\%$). |
| **FR-36** | Task 36 | **Datetime Decomposition:** System must parse `DateTime` into datetime objects and derive `date`, `month` (`.dt.month`), and `hour` (`.dt.hour`), exporting `03_bike_rental_cleaned.csv`. |
| **FR-37** | Task 37 | **Fleet Volume Aggregation:** System must compute cumulative rentals ($1,086,770$ rides) and average hourly demand ($124.06$ rides/hr). |
| **FR-38** | Task 38 | **Surge Record Extraction:** System must isolate and display the top 10 highest-demand hourly timestamps with associated environmental attributes. |
| **FR-39** | Task 39 | **Seasonal Grouping:** System must execute `groupby('season')['rental_count'].mean()` to quantify demand variation across all four seasons. |
| **FR-40** | Task 40 | **Day Type Comparison:** System must contrast weekend vs. weekday demand patterns, including casual vs. registered rider segmentation. |
| **FR-41** | Task 41 | **Meteorological Sensitivity:** System must compute average demand across weather conditions 1 (Clear) through 4 (Severe). |
| **FR-42** | Task 42 | **Diurnal Peak Window Identification:** System must group demand by hour ($0$–$23$) and distinguish peak rush hours from off-peak maintenance windows. |
| **FR-43** | Task 43 | **NumPy Metric Computation:** System must convert series to NumPy arrays and calculate average temperature ($17.00^\circ\text{C}$) and average rental count ($124.06$). |
| **FR-44** | Task 44 | **IQR Outlier Identification:** System must calculate $Q_1$, $Q_3$, $\text{IQR}$, and $\text{Upper Bound} = Q_3 + 1.5 \times \text{IQR}$, isolating surge records above $446.50$ rides/hr. |
| **FR-45** | Task 45 | **Bivariate Matrix Summary:** System must generate a cross-tabulated summary grouping both `season` and `weather` with count, mean, median, std, min, and max. |
| **FR-46** | Task 46 | **Demand Sorting:** System must sort records and calendar days by rental volume, outputting the top 5 peak days and top 5 lowest-demand days. |
| **FR-47** | Task 47 | **Correlation Analysis:** System must evaluate the Pearson correlation coefficient between temperature and rental count using `Series.corr()` ($r = +0.5560$). |
| **FR-48** | Task 48 | **Decision Support:** System must formulate five concrete, data-backed operational fleet planning recommendations. |

---

## 5. Non-Functional Requirements

### 5.1 Performance & Scalability (NFR-01)
- The end-to-end data processing script must execute in under 10 seconds on standard desktop hardware for an annual dataset (8,760+ records).

### 5.2 Reliability & Fault Tolerance (NFR-02)
- Script must handle flexible column naming conventions (`count` vs `rental_count`, `datetime` vs `DateTime`) without throwing uncaught exceptions.
- Zero data loss: All original records must be preserved post-cleaning.

### 5.3 Maintainability & Code Quality (NFR-03)
- Modular script design adhering to PEP 8 standards with comprehensive docstrings and comments.
- Dual deliverable format: Python source file (`.py`) for automated batch jobs and Jupyter Notebook (`.ipynb`) for interactive visual analysis.

---

## 6. Analytical Deliverables & Structured Summaries

### 6.1 Hourly Demand Summary (Task 42)
- **Morning Commute Peak:** 08:00 ($207.94$ rides/hr) and 09:00 ($259.53$ rides/hr).
- **Evening Commute Peak:** 17:00 ($332.75$ rides/hr) and 18:00 ($305.90$ rides/hr).
- **Nocturnal Trough (Off-Peak):** 00:00 to 05:00 ($12.40$ to $13.07$ rides/hr).

### 6.2 Seasonal Comparison Summary (Task 39)
- **Summer (Season 2):** $175.22$ rides/hr (Peak season, $386,892$ total rides).
- **Fall (Season 3):** $153.64$ rides/hr (High sustained demand, $335,542$ total rides).
- **Spring (Season 1):** $111.77$ rides/hr (Moderate recovery, $246,797$ total rides).
- **Winter (Season 4):** $54.42$ rides/hr (Low demand trough, $117,539$ total rides).

### 6.3 Weather Comparison Summary (Task 41)
- **Clear / Few Clouds (1):** $143.15$ rides/hr ($817,938$ rides, 75.3% of annual demand).
- **Mist / Cloudy (2):** $108.35$ rides/hr ($219,089$ rides, 20.2% of annual demand).
- **Light Rain / Snow (3):** $51.49$ rides/hr ($48,400$ rides, 64% drop vs. Clear).
- **Heavy Rain / Severe (4):** $15.99$ rides/hr ($1,343$ rides, 88.8% collapse vs. Clear).

### 6.4 Season x Weather Interaction Matrix (Task 45)
- Clear Summer conditions achieve the highest average demand ($202.29$ rides/hr, max $1,038$).
- Severe Winter weather exhibits the lowest demand ($7.71$ rides/hr, max $28$).

---

## 7. Operational Fleet Planning Recommendations (Task 48)

1. **Peak Commuter Rebalancing Buffer:** Pre-allocate 75% of station capacity in residential origins prior to 07:00 and deploy active van shuttles between 15:30 and 16:30 toward downtown subway interchanges.
2. **Seasonal Fleet Rotation:** Operate 100% active fleet during May–October. During the winter trough (December–February), mothball 35–40% of the fleet for scheduled workshop overhauls and battery cell balancing.
3. **Weather-Integrated Operations & Dynamic Pricing:** Connect dispatch telemetry to radar APIs. Suspend manual van transit during storm warnings to ensure driver safety; trigger automatic digital vouchers to stimulate rides as soon as rain bands pass.
4. **Dedicated Maintenance Window (00:30–05:30):** Perform battery swaps, dock terminal updates, and mobile mechanical servicing during the overnight off-peak trough ($<13$ rides/hr) to guarantee 99%+ station uptime during daytime hours.
5. **Weekend Tourist & Recreational Repositioning:** Redirect weekend fleet deployment away from business districts toward parks, riverfront trails, and cultural corridors, capitalizing on the 3.5x surge in casual riders.

---

## 8. Verification and LMS Submission Sign-Off
- [x] Python Script: `bike_rental_analysis.py` (Runnable & documented)
- [x] Jupyter Notebook: `bike_rental_analysis.ipynb` (Formatted with Markdown & code cells)
- [x] Cleaned Dataset: `03_bike_rental_cleaned.csv` (Exported & verified)
- [x] Raw Dataset: `03_bike_rental.csv` (Generated & populated)
- [x] SRS Specification: `SRS_Bike_Rental_Demand_Analysis.md` (Formal specification)
