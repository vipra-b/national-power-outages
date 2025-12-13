# Analyzing Power Outage Duration in the United States

**Name**: Vipra Bindal
**Website Link**: https://vipra-b.github.io/national-power-outages/

---

## Introduction

### Dataset Overview

 Power outages have significant impacts every year, whether that be economically through shutdown of production and workplaces, or socially through individual homes' loss of electricity. Since a power outage is such a significant interruption, it would make sense for states to want to minimize their impact and length. Understanding which factors contribute to the length of a power outage can help states focus on minimizing the risk of those particular factors and improve infrastructure in areas that need it to hopefully minimize the length of power outages. The leads us to our research question?

### Research Question

**What characteristics affect the length of a power outage?**

This question is central to our analysis as we explore various factors including geographic location, climate conditions, cause categories, and economic indicators to understand their relationship with outage duration.

### Dataset Information

There were more columns than this in the original dataset, but I highlight the ones we use for analysis below
- **Number of rows**: 1,476 (after initial cleaning)
- **Relevant columns**:

| Column Name | Description |
|------------|-------------|
| `OUTAGE.DURATION` | Duration of the power outage in minutes |
| `YEAR`, `MONTH` | Temporal information about when outages occurred |
| `U.S._STATE` | State where the outage occurred |
| `NERC.REGION` | North American Electric Reliability Corporation region |
| `CLIMATE.REGION`, `CLIMATE.CATEGORY` | Climate classification information |
| `ANOMALY.LEVEL` | Climate anomaly level |
| `CAUSE.CATEGORY`, `CAUSE.CATEGORY.DETAIL` | Cause of the outage |
| `CUSTOMERS.AFFECTED` | Number of customers impacted |
| `RES.PRICE`, `COM.PRICE`, `IND.PRICE` | Electricity prices by sector |
| `TOTAL.CUSTOMERS`, `POPULATION` | Demographic information |
| `POPPCT_URBAN`, `POPDEN_URBAN` | Urban population metrics |
| `PC.REALGSP.STATE`, `UTIL.CONTRI` | Economic indicators |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The data cleaning process involved several steps to prepare the dataset for analysis:

1. **Removed header rows and metadata**: The Excel file contained header rows and variable definitions that were removed to isolate the actual data.

2. **Numeric conversion**: All numeric columns were converted to appropriate numeric types using `pd.to_numeric()` with error handling to convert invalid values to NaN.

3. **Date/time handling**: 
   - Combined `OUTAGE.START.DATE` and `OUTAGE.START.TIME` into a single `OUTAGE.START` timestamp column
   - Combined `OUTAGE.RESTORATION.DATE` and `OUTAGE.RESTORATION.TIME` into a single `OUTAGE.RESTORATION` timestamp column
   - Created a `DATE` column from `YEAR` and `MONTH` for temporal analysis

4. **Missing value standardization**: Replaced various missing value indicators (empty strings, 'N/A', 'n/a', 'NA', 'null', 'NULL', 'None') with proper NaN values.

5. **Column dropping**:
   - Created `PRICE_DIFF` = `RES.PRICE` - `IND.PRICE` to capture price differentials
   - Created `GDP_CONTR` = `UTIL.CONTRI` × `PC.REALGSP.STATE` to capture economic contribution
   - Created `SEASON` from `MONTH` to categorize outages by season
   - Calculated `OUTAGE.DURATION_CALC` from start and restoration timestamps when available

6. **Data filtering**: Removed rows where `OUTAGE.DURATION` was missing, as this is our primary variable of interest.

7. **Column dropping**: Dropped columns not relevant to the analysis or had much missingness to analyze properly:
   - `OBS`: Observation number (not informative)
   - `POSTAL.CODE`: Postal code (redundant with state information)
   - `HURRICANE.NAMES`: Hurricane names (not needed for analysis)
   - `DEMAND.LOSS.MW`: Demand loss in megawatts (not used in final model)
   - `TOTAL.PRICE`: Total price (redundant with individual sector prices)
   - `RES.SALES`, `COM.SALES`, `IND.SALES`, `TOTAL.SALES`: Sales data by sector (not used)
   - `RES.PERCEN`, `COM.PERCEN`, `IND.PERCEN`: Percentage sales by sector (not used)
   - `RES.CUSTOMERS`, `COM.CUSTOMERS`, `IND.CUSTOMERS`: Customer counts by sector (redundant with `TOTAL.CUSTOMERS`)
   - `RES.CUST.PCT`, `COM.CUST.PCT`, `IND.CUST.PCT`: Customer percentages by sector (not used)
   - `PC.REALGSP.USA`, `PC.REALGSP.REL`, `PC.REALGSP.CHANGE`: USA-level and relative GSP metrics (not used)
   - `UTIL.REALGSP`, `TOTAL.REALGSP`: Utility and total real GSP (redundant with `PC.REALGSP.STATE`)
   - `OUTAGE.START.DATE`, `OUTAGE.START.TIME`: Individual date/time components (combined into `OUTAGE.START`)
   - `OUTAGE.RESTORATION.DATE`, `OUTAGE.RESTORATION.TIME`: Individual date/time components (combined into `OUTAGE.RESTORATION`)
   - `DATE`: Redundant date column (we have `YEAR` and `MONTH` separately)

After completing all cleaning steps, here is the head of the cleaned DataFrame (showing key columns because about 30 columns):

|   YEAR |   MONTH | U.S._STATE   | NERC.REGION   | CLIMATE.CATEGORY   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED |   PRICE_DIFF |   GDP_CONTR |
|-------:|--------:|:-------------|:--------------|:-------------------|:-------------------|------------------:|---------------------:|-------------:|------------:|
|   2011 |       7 | Minnesota    | MRO           | normal             | severe weather     |              3060 |                70000 |         4.79 |     89790.3 |
|   2014 |       5 | Minnesota    | MRO           | normal             | intentional attack |                 1 |                  nan |         5.63 |     95763.3 |
|   2010 |      10 | Minnesota    | MRO           | cold               | severe weather     |              3000 |                70000 |         4.8  |     86076   |
|   2012 |       6 | Minnesota    | MRO           | normal             | severe weather     |              2550 |                68200 |         5.08 |     99691.9 |
|   2015 |       7 | Minnesota    | MRO           | warm               | severe weather     |              1740 |               250000 |         5.33 |     90829.2 |

### Univariate Analysis

This is a histogram of the distribution of # of power outages, which shows that California, Michigan, and Texas have had the highest # of power outages from 2000-2016.

<iframe
  src="assets/state_outages_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Number of Outages by Climate Category

The distribution of # of power outages shows that the most power outages happened under "normal" conditions.

<iframe
  src="assets/climate_category_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

Analysis of the relationship between climate category and outage duration reveals interesting patterns. Warm climate regions tend to have slightly different outage characteristics compared to cold and normal climate regions.

<iframe
  src="assets/climate_duration_box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


The relationship between NERC region and average outage duration shows a distribution skeweed right and significant variation across different regions, with ECAR, FRCC, and RFC having the most frequent outages.

<iframe
  src="assets/nerc_duration_box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

*

### Interesting Aggregates

A pivot table showing average outage duration by NERC Region and Climate Category reveals interesting interactions:

| NERC Region | Cold | Normal | Warm |
|------------|------|--------|------|
| ECAR | 92.49 | 83.58 | 120.58 |
| FRCC | 8.24 | 101.82 | 74.58 |
| WECC | 20.42 | 14.08 | 49.16 |
| ... | ... | ... | ... |

This table builds upon the visualizations we did above and is significant because it shows that the interaction between geographic region and climate conditions affects outage duration differently across regions. For example, ECAR experiences longer outages in warm conditions, while FRCC has longer outages in normal conditions.

---

## Assessment of Missingness

### NMAR Analysis

I believe the `CAUSE.CATEGORY.DETAIL` column is **NMAR (Not Missing At Random)**. because the missingness likely depends on the reporting standards of a particular company, which determains whether more details are added to the cause. However, that information isn't present in our dataset.

### Missingness Dependency

We will test for dependency on CAUSE.CATEGORY

**Hypotheses:**
- **H₀:** Missingness of `CAUSE.CATEGORY.DETAIL` does NOT depend on `CAUSE.CATEGORY`
- **H₁:** Missingness of `CAUSE.CATEGORY.DETAIL` DOES depend on `CAUSE.CATEGORY`

We will use Total Variation Distance of missingness rates across categories. I got a TVD of 0.8859. We get a p-value of 0, so We reject the null hypothesis. Missingness DOES depend on `CAUSE.CATEGORY`.

<iframe
  src="assets/missingness_cause_category_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Test 2: Dependency on OUTAGE.DURATION

**Hypotheses:**
- **H₀:** Missingness of `CAUSE.CATEGORY.DETAIL` does NOT depend on `OUTAGE.DURATION`
- **H₁:** Missingness of `CAUSE.CATEGORY.DETAIL` DOES depend on `OUTAGE.DURATION`


<iframe
  src="assets/missingness_duration_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Missingness Dependency Summary

The missingness in `CAUSE.CATEGORY.DETAIL` is **MAR (Missing At Random)** with respect to `CAUSE.CATEGORY` (p-value = 0.0000) but does not depend on `OUTAGE.DURATION` (p-value = 0.0770). This indicates that the probability of missingness depends on the observed `CAUSE.CATEGORY` value.

---

## Hypothesis Testing

### Research Question

Do power outages last longer in winter compared to summer?

### Hypotheses

- **H₀:** μ_summer = μ_winter (The mean outage duration is the same for summer and winter outages)
- **H₁:** μ_summer ≠ μ_winter (The mean outage duration differs between summer and winter outages)

- **Test:** Two-sample t-test
- **Test Statistic:** Difference in sample means
- **Sample sizes:**
  - Summer (June-August): n = 514
  - Winter (December-February): n = 373

### Results

The two-sample t-test yielded a test statistic of -2.1438 and a p-value of 0.0323, leading us to reject the null hypothesis. There is statistically significant evidence that outage duration differs between summer and winter, with winter outages tending to be longer than summer outages.

*Note: This conclusion is based on statistical evidence from our sample. We cannot prove with absolute certainty that winter outages are always longer, but we have evidence suggesting a difference in the populations.*

<iframe
  src="assets/summer_winter_histogram.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Framing a Prediction Problem

### Prediction Problem

We aim to predict the duration of power outages (in hours) based on characteristics known at the time of the outage. This is a regression problem since we are predicting `OUTAGE.DURATION`, a continuous numerical value. 

Predicting duration can help utilities allocate resources and set expectations for restoration times.



We use Mean Absolute Error (MAE) as our evaluation metric because it is interpretable in the original units and gives equal weight to all errors regardless of direction. We chose MAE over RMSE because it is less sensitive to outliers, as there some significantly long outages (the longest outage is about 108563 minutes, or 75 days).

### Features and Temporal Considerations

All features used in our models are known at the time of prediction, including temporal features (`YEAR`, `MONTH`), geographic features (`U.S._STATE`, `NERC.REGION`, `CLIMATE.REGION`), climate features (`CLIMATE.CATEGORY`, `ANOMALY.LEVEL`), cause category (`CAUSE.CATEGORY`), economic features (`RES.PRICE`, `COM.PRICE`, `IND.PRICE`, `PC.REALGSP.STATE`, `UTIL.CONTRI`, `PRICE_DIFF`, `GDP_CONTR`), and demographic features (`TOTAL.CUSTOMERS`, `POPULATION`, `POPPCT_URBAN`, `POPDEN_URBAN`). We do not use features that would only be known after the outage is resolved, such as `OUTAGE.RESTORATION`.

---

## Baseline Model

Our baseline model uses Ridge Regression, a linear regression technique that adds L2 regularization to by penalizing large coefficients. The model uses two nominal categorical features: `CAUSE.CATEGORY` and `NERC.REGION`. Both features were one-hot encoded using `OneHotEncoder` with `drop='first'` to avoid multicollinearity, and all preprocessing and modeling steps were implemented in a single sklearn Pipeline with Ridge regression (alpha=1.0).

On the test set, the model achieved an MAE of 46.01 hours, RMSE of 139.04 hours, and R² of 0.114. Cross-validation MAE was 38.55 hours (std: 2.65). The model explains only about 11-19% of the variance in outage duration, indicating that most variation is not captured by these two features alone.

The baseline model is not particularly good due to large prediction errors (average error of nearly 2 days) and overfitting concerns, as evidenced by the gap between training (RMSE: 74.95 hours) and test performance (RMSE: 139.04 hours). However, it provides a starting point for comparison and demonstrates that even simple models can capture some relationship between cause category, region, and outage duration.

---

## Final Model

Our final model uses Random Forest Regressor, an ensemble method that builds multiple decision trees and averages their predictions. We expanded the feature set to include temporal features (`YEAR`, `MONTH`), geographic features (`U.S._STATE`, `NERC.REGION`, `CLIMATE.REGION`, `CLIMATE.CATEGORY`), climate features (`ANOMALY.LEVEL`), economic features (`RES.PRICE`, `COM.PRICE`, `IND.PRICE`, `PC.REALGSP.STATE`, `UTIL.CONTRI`, `PRICE_DIFF`, `GDP_CONTR`), and demographic features (`TOTAL.CUSTOMERS`, `POPULATION`, `POPPCT_URBAN`, `POPDEN_URBAN`).

We applied `QuantileTransformer` to economic and price variables (`RES.PRICE`, `COM.PRICE`, `IND.PRICE`, `ANOMALY.LEVEL`, `PC.REALGSP.STATE`, `UTIL.CONTRI`) to normalize their skewed distributions. Other numeric features were passed through unchanged, and categorical features (`U.S._STATE`, `CAUSE.CATEGORY`, `NERC.REGION`, `CLIMATE.REGION`, `CLIMATE.CATEGORY`) were one-hot encoded in the ColumnTransformer.

We tuned hyperparameters using GridSearchCV with 5-fold cross-validation, searching over `n_estimators` [100, 200], `max_depth` [10, 15, 20], `min_samples_split` [10, 20], `min_samples_leaf` [5, 10], and `max_features` ['sqrt', 0.5]. These parameters control model complexity and help prevent overfitting.

On the test set, the final model achieved an MAE of 41.74 hours, RMSE of 142.18 hours, and R² of 0.0735. Cross-validation MAE was 35.31 hours (std: 2.17). The model shows improvement over the baseline, with test MAE decreasing from 46.01 to 41.74 hours (9% improvement) and training R² increasing from 0.188 to 0.5485.


---

## Fairness Analysis

### Groups and Evaluation Metric

**Group X:** Urban areas (defined as areas with `POPPCT_URBAN` ≥ 50%)  
**Group Y:** Rural areas (defined as areas with `POPPCT_URBAN` < 50%)

**Evaluation Metric:** RMSE (Root Mean Squared Error)

**Why this metric:** Since we built a regression model, we use RMSE rather than classification metrics. RMSE measures the average magnitude of prediction errors, which is appropriate for understanding fairness in regression contexts.

### Hypotheses

- **H₀:** The model is fair. RMSE for rural and urban areas are roughly the same, and any differences are due to random chance.
- **H₁:** The model is unfair. RMSE for rural areas is higher than RMSE for urban areas (i.e., the model performs worse for rural areas).

### Test Statistic and Significance Level

- **Test Statistic:** Difference in RMSE (RMSE_rural - RMSE_urban)

### Results

The observed test statistic was -74.02 hours, with RMSE of 144.08 hours for urban areas and 70.06 hours for rural areas. The p-value of 0.2260 leads us to fail to reject the null hypothesis, indicating no statistically significant evidence that the model performs differently for rural versus urban areas. While the model appears to perform better for rural areas, this difference is not statistically significant.

<iframe
  src="assets/fairness_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

