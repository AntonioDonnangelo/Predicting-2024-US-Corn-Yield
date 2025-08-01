# Predicting 2024 U.S. Corn Yield  

## Objective

Build a model to **predict the 2024 U.S. national corn yield (bu/acre)** using historical USDA yield data and weather data. The final output includes:

- A **point prediction** and **uncertainty estimate** for 2024.
- An evaluation of model performance on past years.
- Clear documentation of assumptions, logic, process, and key findings.

---

## Project Outline

### 1. Data Acquisition

- Download **historical corn yield data** at county, state, and national level from USDA quick stats trough the [API](https://quickstats.nass.usda.gov/api) provided.
- Load **daily county-level weather data** already in possession with the project, *wx_hist_df.parquet*.

### 2. Data Sanity Check

- Merging data of yield in one .csv since the request allowed not more than 50.0000 records togheter
- Validation of the yield data and weather data.
- Handling missing values, check for unit consistency, and verifying dimensions.
- Different granyularity: The yield data from USDA quick stats is from 1910 at year level and the weather data is at daily level from 2000. <br>

### 3. Exploratory Data Analysis

- Exploring yield trends over time and space: it is clear that Yields rose sharply from 1920 to 2020, <br>
especially after 1960 due to hybrid crops, with fluctuations likely from weather or global events.

![corn_yield](./images/corn_yield_over_years.png)

- First 5 states with the highest average yield where:
    
    | State Name  | Value      |
    |-------------|------------|
    | IDAHO       | 129.907670 |
    | CALIFORNIA  | 119.936595 |
    | UTAH        | 111.171765 |
    | NEW YORK    | 109.179003 |
    | ARIZONA     | 107.542978 |

![avg_yield](./images/avg_yield_states.png)

 
### 4. Feature Engineering

- Derive relevant features from the raw weather data: <br>
It’s better to have less-designed, biologically meaningful features than a lot of noisy or redundant features. For this reason the feature feature engineering included: <br>
    - **Biological and stress features**: <br>
    - `soil_moisture_delta`: Captures the difference between surface and sub-surface soil moisture.
    - `term_stress`: If high, stress plants by disrupting respiration and photosynthesis.
    - `dry_heat_index`: Combines heat and dryness into a single stress indicator. 
    - **Rolling Features and multi-year rolling averages**: to capture temporal patterns <br>
    - `tmax_mean_lag1`: to capture carry-over effects of temperature.
    - `precip_mean_lag1`: to capture carry-over effects of precipitations.
    - `tmax_mean_3yr`: to capture longer-term trends for temperature.
    - `precip_mean_3yr`: to capture longer-term trends of precipitations.

![avg_yield](./images/mtrx_corr_yield_weather.png)

- Examinating the relationships between weather patterns and yield outcomes, several things shown up: <br>
    - A sharp decrease in yield around 2012;<br>
    - The decrease in yield coincides with a drastic drop in total rainfall; <br>
    - Predictably, in addition to the decrease in rainfall, soil moisture also decreased. <br>
    - During the same period, it can be observed that the average maximum temperatures were the highest (drought). <br>
    - Average maximum temperatures around 25 degrees, combined with good rainfall/soil moisture, allowed for the best yields. <br>

![avg_yield](./images/feature_comp_yield.png)

### 5. Model Development & Evaluation

- Train one or more models to predict national yield.
- Evaluate performance using recent years as holdout.
- Include performance metrics and interpretability (if applicable).

### 6. Predict 2024 Yield

- Create input variables.
- Make a national yield prediction with uncertainty estimates.
- Describe assumptions behind missing/future data handling.

### 7. Reporting

- Summarize your modeling process, prediction, and findings.
- Include a brief discussion of limitations and potential improvements.

---

## Deliverables

- One clean, self-contained Jupyter notebook (`corn_yield_2024.ipynb`)
- Your notebook should:
  - Run top-to-bottom without error
  - Include markdown comments explaining key steps
  - Present your final 2024 prediction and model performance clearly

Optionally include:
- `requirements.txt` for dependencies
- `README.md` with brief setup or notes
