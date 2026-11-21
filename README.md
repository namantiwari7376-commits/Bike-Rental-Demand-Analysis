# Bike Rental Project

This repository contains the data, analysis, and documentation for the bike rental demand project.

## Files

- **bike_analysis.ipynb** – Jupyter notebook with exploratory data analysis, visualizations, and model fitting for bike rental demand.
- **bike_rental_analysis.pdf** – PDF version of the analysis report generated from the notebook.
- **bike_rental_cleaned.csv** – Cleaned dataset used for analysis (contains the primary features and target variable after preprocessing).
- **03_bike_rental_cleaned.csv** – Alternate cleaned dataset (e.g., a subset or a different preprocessing pipeline).
- **03_bike_rental.csv** – Raw bike rental data before cleaning.
- **Bike_Rental_Demand_Analysis_SRS_Individual_Naman.pdf** – Software Requirements Specification (SRS) document outlining the project goals, functional and non‑functional requirements, and design considerations.

## How to Use

1. **Explore the data**
   ```bash
   # Open the raw CSV
   pandas.read_csv('03_bike_rental.csv')
   ```

2. **Run the analysis notebook**
   - Launch Jupyter Notebook or JupyterLab:
   ```bash
   jupyter notebook bike_analysis.ipynb
   ```
   - Execute the cells to reproduce the visualizations and model results.

3. **Reference the reports**
   - Open `bike_rental_analysis.pdf` for a nicely formatted version of the notebook.
   - Review `Bike_Rental_Demand_Analysis_SRS_Individual_Naman.pdf` for the project specifications.

## Project Overview

The goal of this project is to understand and predict bike rental demand based on historical usage data, weather, and temporal factors. The analysis includes:
- Data cleaning and feature engineering.
- Descriptive statistics and visual exploration.
- Time‑series and regression modeling.
- Evaluation of model performance.

Feel free to modify the notebooks, add additional analyses, or extend the dataset.

---
*Generated on 2026-11-21*