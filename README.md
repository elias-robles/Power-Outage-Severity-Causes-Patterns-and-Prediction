# Power Outage Severity: Causes, Patterns, and Prediction

**Elias Robles**

This project explores major power outages in the United States from 2000 to 2016. I investigate what factors are associated with more severe power outages and build a model to predict outage duration.


## Introduction

Power outages can affect large numbers of people and can vary greatly in how long they last. Understanding what characteristics are associated with more severe outages can help identify patterns in when and why major outages occur.

The Power Outages dataset contains information about major power outages in the United States from 2000 to 2016. It includes information about when and where outages occurred, their causes, climate conditions, outage duration, and the number of customers affected. The dataset contains **1,534 rows**, with each row representing a major power outage.

For this project, I will focus on the question:

**What factors are associated with more severe power outages?**

I primarily measure outage severity using `OUTAGE.DURATION`, which records how long an outage lasted. I examine whether characteristics such as the cause of the outage, climate conditions, location, and year are associated with differences in outage duration.

The main columns relevant to this question are:

| Column | Description |
| --- | --- |
| `OUTAGE.DURATION` | The duration of the power outage, measured in minutes. |
| `CAUSE.CATEGORY` | The category describing the cause of the outage. |
| `CLIMATE.CATEGORY` | The climate category associated with the outage. |
| `U.S._STATE` | The U.S. state in which the outage occurred. |
| `YEAR` | The year in which the outage occurred. |
| `CUSTOMERS.AFFECTED` | The number of customers affected by the outage. |

This question is important because identifying characteristics associated with longer and more severe outages may help energy companies better understand the types of outages that could have the greatest impact.


## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The original dataset required some cleaning before beginning the analysis. When loading the Excel file, I skipped rows that did not contain observations or column names.

The outage start date and start time were originally stored in separate columns. I combined these into a single `OUTAGE.START` timestamp. I also combined the outage restoration date and restoration time into a single `OUTAGE.RESTORATION` timestamp. After creating these new timestamp columns, I removed the original date and time columns because they contained the same information.

I also kept only the columns that were relevant to the analyses in this project. This made the dataset easier to work with while preserving the information needed for the exploratory analysis, missingness analysis, hypothesis test, and prediction model.

The first five rows of the cleaned dataset are shown below:
|   YEAR | U.S._STATE   | CLIMATE.CATEGORY   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED |
|-------:|:-------------|:-------------------|:-------------------|------------------:|---------------------:|
|   2011 | Minnesota    | normal             | severe weather     |              3060 |                70000 |
|   2014 | Minnesota    | normal             | intentional attack |                 1 |                  nan |
|   2010 | Minnesota    | cold               | severe weather     |              3000 |                70000 |
|   2012 | Minnesota    | normal             | severe weather     |              2550 |                68200 |
|   2015 | Minnesota    | warm               | severe weather     |              1740 |               250000 |

### Univariate Analysis

First, I examined the distribution of outage duration.

<iframe
  src="assets/outage-duration-distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of outage duration is strongly right-skewed. Most outages have relatively shorter durations, while a small number of outages last much longer than the rest.

I also examined how frequently each cause category appears in the dataset.

<iframe
  src="assets/outages-by-cause.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Severe weather and intentional attacks are among the most common causes of major outages in the dataset, while some other cause categories occur much less frequently. This helps show which types of outages make up the largest portions of the data.

### Bivariate Analysis

Next, I compared outage duration across different cause categories.

<iframe
  src="assets/duration-by-cause.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Outage duration varies noticeably across cause categories. Severe weather and fuel supply emergencies tend to have longer outage durations, while intentional attacks and islanding generally have shorter durations.

### Interesting Aggregates

To examine how outage duration varies across both cause and climate, I created a pivot table showing the median outage duration for each combination of cause category and climate category.

| CAUSE.CATEGORY                |   cold |   normal |    warm |
|:------------------------------|-------:|---------:|--------:|
| equipment failure             |  182   |    214.5 |   447.5 |
| fuel supply emergency         | 7965   |   2880   | 18717   |
| intentional attack            |   92   |     15   |    73   |
| islanding                     |  193   |     32   |    78.5 |
| public appeal                 | 1092.5 |    495   |   300   |
| severe weather                | 2319.5 |   2511   |  2520   |
| system operability disruption |  214   |    224   |   197.5 |

The median outage duration varies substantially across cause categories. Fuel supply emergencies and severe weather tend to have much longer median durations than categories such as intentional attacks and islanding.

I used the median instead of the mean because outage duration is strongly right-skewed, so the median is less affected by unusually long outages.


## Assessment of Missingness

### MNAR Analysis

I believe that `CUSTOMERS.AFFECTED` could be **MNAR**. The likelihood that this value is missing may depend on the actual number of customers affected. For example, outages that affect a very large number of customers may make it more difficult to determine or record the exact number affected. In this case, the missingness would depend on the unobserved value of `CUSTOMERS.AFFECTED` itself.

Additional information about how the number of customers affected was collected and reported could help explain the missingness. If factors such as the reporting method, utility company, or type of outage explained why the value was missing, then the missingness could instead be considered MAR.

### Missingness Dependency

I analyzed whether the missingness of `CUSTOMERS.AFFECTED` depends on other columns in the dataset using permutation tests.

First, I tested whether the missingness of `CUSTOMERS.AFFECTED` depends on `YEAR`.

**Null Hypothesis:** The missingness of `CUSTOMERS.AFFECTED` does not depend on `YEAR`. Any observed difference in mean year between rows where `CUSTOMERS.AFFECTED` is missing and not missing is due to random chance.

**Alternative Hypothesis:** The missingness of `CUSTOMERS.AFFECTED` does depend on `YEAR`.

I used the absolute difference in mean `YEAR` between the missing and non-missing groups as the test statistic. The observed difference was approximately **1.85 years**.

<iframe
  src="assets/missingness-year-permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram shows the permutation distribution of the absolute difference in mean `YEAR` under the null hypothesis, while the red line shows the observed difference. None of the 1000 permutations produced a difference at least as large as the observed statistic, giving a simulated p-value of **0.0**. Since this is below the significance level of 0.05, I reject the null hypothesis and conclude that there is strong evidence that the missingness of `CUSTOMERS.AFFECTED` depends on `YEAR`.

I then tested whether the missingness of `CUSTOMERS.AFFECTED` depends on `TOTAL.CUSTOMERS`.

**Null Hypothesis:** The missingness of `CUSTOMERS.AFFECTED` does not depend on `TOTAL.CUSTOMERS`.

**Alternative Hypothesis:** The missingness of `CUSTOMERS.AFFECTED` does depend on `TOTAL.CUSTOMERS`.

The simulated p-value for this test was approximately **0.83**. Since this is greater than 0.05, I fail to reject the null hypothesis. There is not enough evidence to conclude that the missingness of `CUSTOMERS.AFFECTED` depends on `TOTAL.CUSTOMERS`.


## Hypothesis Testing

To investigate whether outage cause is associated with outage severity, I compared the duration of outages caused by severe weather with the duration of outages caused by intentional attacks.

**Null Hypothesis:** Severe weather outages and intentional attack outages have the same distribution of outage duration. Any observed difference in their mean durations is due to random chance.

**Alternative Hypothesis:** Severe weather outages have a greater mean outage duration than intentional attack outages.

**Test Statistic:** Mean outage duration for severe weather outages minus mean outage duration for intentional attack outages.

**Significance Level:** 0.05

I chose the difference in mean outage duration as the test statistic because the question compares how long outages last between two cause categories. A larger positive value indicates that severe weather outages have a greater average duration than intentional attack outages. I used a significance level of 0.05 as the cutoff for determining whether the observed difference would be unlikely under the null hypothesis.

The observed difference in mean outage duration was approximately **3454 minutes**, with severe weather outages having the larger mean duration.

<iframe
  src="assets/hypothesis-test-permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram shows the permutation distribution of the difference in mean outage duration under the null hypothesis, while the red line shows the observed difference. The observed statistic lies far to the right of the simulated differences, which is consistent with the very small simulated p-value.

I performed a permutation test with 1000 repetitions. None of the simulated differences were at least as large as the observed difference, giving a simulated p-value of **0.0**.

Since the p-value is below the significance level of 0.05, I reject the null hypothesis. The results provide strong evidence that severe weather outages tend to have longer durations than outages caused by intentional attacks.


## Framing a Prediction Problem

For the prediction portion of this project, I will predict the duration of a major power outage once its cause category has been identified, but before the outage has been restored.

This is a **regression** problem because the response variable, `OUTAGE.DURATION`, is quantitative.

The response variable is `OUTAGE.DURATION`, which measures the length of an outage in minutes. I chose this variable because outage duration is one way to measure the severity of a power outage and was also a major focus of my earlier analysis.

I will evaluate the model using **Root Mean Squared Error (RMSE)**. RMSE measures the size of the model's prediction errors and gives more weight to larger errors. I chose RMSE instead of a metric such as R² because RMSE is measured in the same units as the response variable, so the model's error can be directly interpreted in minutes.

At the time of prediction, I will only use information that could reasonably be known once the cause category of the outage has been identified. In my models, this includes information such as the outage's cause category, state, climate category, and year. I will not use information such as `OUTAGE.RESTORATION`, because the restoration time would not yet be known when predicting how long the outage will last.


## Baseline Model

For my baseline model, I used a **Linear Regression** model to predict `OUTAGE.DURATION`.

The model used two features:

- `CAUSE.CATEGORY`: a **nominal categorical** feature describing the cause of the outage.
- `U.S._STATE`: a **nominal categorical** feature describing the state where the outage occurred.

Because both features are nominal categorical variables, I applied **one-hot encoding** before fitting the linear regression model.

I evaluated the model using RMSE on both the training data and the test data so that I could measure not only how well the model fit the data it was trained on, but also how well it generalized to unseen outages.

The baseline model had:

- **Training RMSE:** approximately 4727 minutes
- **Test RMSE:** approximately 7240 minutes

The test RMSE is noticeably larger than the training RMSE, which suggests that the model does not generalize especially well to unseen data. The test RMSE is also fairly large, so I would not consider this baseline model to be particularly accurate.

However, it provides a simple starting point that can be improved by adding and transforming additional features.


## Final Model

For my final model, I kept the two baseline features, `CAUSE.CATEGORY` and `U.S._STATE`, and added `CLIMATE.CATEGORY` and `YEAR`.

I added `CLIMATE.CATEGORY` because climate conditions are part of the environment in which an outage occurs and may be related to the severity of the outage. Different climate conditions may be associated with different types of disruptions, so this feature could provide additional information about outage duration beyond the outage's cause and location.

I also added `YEAR` because characteristics of the power grid, reporting practices, and outage patterns may change over time. Instead of assuming that the relationship between year and outage duration is strictly linear, I applied polynomial features to `YEAR` so that the model could capture a possible non-linear relationship.

The final model used:

- `CAUSE.CATEGORY`: a **nominal categorical** feature
- `U.S._STATE`: a **nominal categorical** feature
- `CLIMATE.CATEGORY`: a **nominal categorical** feature
- `YEAR`: a **quantitative** feature

I one-hot encoded the categorical features and applied polynomial features to `YEAR`.

I continued to use a **Linear Regression** model. I kept the same modeling algorithm as the baseline model so that I could focus on whether the additional features and feature engineering improved the model.

To choose the polynomial degree for `YEAR`, I used **GridSearchCV with 5-fold cross-validation** on the training data. I tested polynomial degrees of 1, 2, 3, and 4. The best-performing hyperparameter was a polynomial degree of **2**.

The final model had:

- **Training RMSE:** approximately 4718 minutes
- **Test RMSE:** approximately 7174 minutes

The baseline model had a test RMSE of approximately 7240 minutes, so the final model reduced the test RMSE by about **66 minutes**. This means the final model performed slightly better on unseen outages than the baseline model.

Although the improvement was relatively small, the lower test RMSE shows that the additional features and transformations provided some useful information for predicting outage duration. The test RMSE is still considerably larger than the training RMSE, so the model still has limitations in how well it generalizes to unseen data.


## Fairness Analysis

For my fairness analysis, I compared the performance of my final model across two outage cause groups:

- **Group X:** outages caused by severe weather
- **Group Y:** outages caused by intentional attacks

Because my prediction task is a regression problem, I used **RMSE** as the evaluation metric. A larger RMSE means that the model's predictions are farther from the actual outage durations.

**Null Hypothesis:** The model performs equally well for severe weather outages and intentional attack outages. Any observed difference in RMSE between the two groups is due to random chance.

**Alternative Hypothesis:** The model performs worse for severe weather outages than for intentional attack outages, meaning that the RMSE for severe weather outages is greater.

**Test Statistic:** RMSE for severe weather outages minus RMSE for intentional attack outages.

**Significance Level:** 0.05

I used this test statistic because a positive difference indicates that the model has greater prediction error for severe weather outages than for intentional attack outages.

The observed difference in RMSE was approximately **4027 minutes**, with severe weather outages having the larger RMSE.

I performed a permutation test with 1000 repetitions. None of the simulated differences were at least as large as the observed difference, giving a simulated p-value of **0.0**.

Since the simulated p-value is below the significance level of 0.05, I reject the null hypothesis. The results provide strong evidence that the model performs worse for severe weather outages than for intentional attack outages.

This suggests that the model's prediction error is not equally distributed across these two outage cause categories.