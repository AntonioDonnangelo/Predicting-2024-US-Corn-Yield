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
  
- Feature selection:
  To build a machine learning model to predict Yield, we want features that have a strong correlation with Yield but they shouldn't be too redundant with each other (to avoid multicollinearity).

![avg_yield](./images/mtrx_corr_yield_weather.png)

- Examinating the relationships between weather patterns and yield outcomes, several things shown up: <br>
    - A sharp decrease in yield around 2012;<br>
    - The decrease in yield coincides with a drastic drop in total rainfall; <br>
    - Predictably, in addition to the decrease in rainfall, soil moisture also decreased. <br>
    - During the same period, it can be observed that the average maximum temperatures were the highest (drought). <br>
    - Average maximum temperatures around 25 degrees, combined with good rainfall/soil moisture, allowed for the best yields. <br>

![avg_yield](./images/feature_comp_yield.png)

### 5. Model Development & Evaluation

As initial experiment i use SARIMAX/SARIMAX with all data from 1910. these methods assumes stationarity and regular patterns over time,  <br>
which may not fully capture yield variations caused by highly variable weather patterns or regional effects. and then I have incorporated the features variables, <br>
but with only 24 years of overlapping yield + weather data, obviously its predictive power is limited. 

- ML models:
    To ensure consistency and valid feature-target relationships, I have aggregated weather data to yearly resolution, aligning it with the annual yield data at county level. <br>
    With this method it was possible:
    - Increase the effective sample size by modeling many counties across multiple years (instead of just one national average per year).
    - Exploit spatial variation because weather effects vary by region, and modeling at the county level let me capture these localized patterns.

    | Model                               | Why                                                                                                       |
    | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
    | **Linear Regression**               | Baseline model to understand linear relationships and feature importance. Simple, interpretable.                          |
    | **ElasticNet Regression**           | Regularized regression to prevent overfitting and handle multicollinearity between weather features.                      |
    | **Support Vector Regression (SVR)** | Good for non-linear relationships; works well with smaller datasets and can model complex feature interactions.           |
    | **Random Forest Regression**        | Non-parametric, captures non-linearities and feature interactions. Robust to overfitting and provides feature importance. |


### 6. Predict 2024 Yield

2024 weather was not fully available, especially for the latter part of the growing season (e.g. July–September), which are crucial for final yield determination.
My approach was to use the historical average for each feature across previous years as a proxy for the missing 2024 values. This offered was fast, stable, and unbiased estimate.

One potential extension is to incorporate short-term weather forecasts or use time-series models to predict future weather features.

Results for the different models:

![mean_pred](./images/mean_pred.png)

All models provided reasonable yield predictions, but their performance varied significantly. 
The Support Vector Regression (SVR) model performed best, with the lowest error and a strong R² (0.78), offering accurate and stable predictions. 
Linear Regression showed moderate explanatory power but high variability, reflecting its sensitivity to the complex nature of agricultural data. 
Random Forest captured some non-linear patterns but had inconsistent results with high variance.

### 7. Reporting


**Key Findings:**
- Weather plays a significant role in annual yield fluctuations, especially drought years.
- Strong positive correlation betweeen long-term water availability (`precip_sum_3yr`)

Yield is highest when:
- Tmax is ~25°C
- Rainfall and soil moisture are above average.
- Spatial modeling at county level enabled richer pattern recognition than national-only data.
- The SVR model captures non-linear interactions and is resilient to limited data volume

**Limitations:**
- Weather gaps: Proxying with historical averages for 2024 introduces uncertainty.
- Data scope: Only 24 years of overlapping yield-weather data limits the training sample.
- No management factors: Models do not account for irrigation, hybrid varieties, fertilizer usage, or pest events.
- Aggregation tradeoff: Yearly aggregation removes intra-seasonal dynamics.

**Future Improvements:**
- Cross-validation: Applying K-Fold cross-validation allows for a better estimation of the model's actual performance. 
- Integrate season weather: Use real data or forecasts for July–September weather features.
- Add agronomic variables: Incorporate crop type, irrigation levels, planting dates, etc.
- Use deep learning models: LSTM or transformer-based models could model sequential weather-yield dynamics.

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
