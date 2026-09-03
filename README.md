# PowerCo Customer Churn Analysis — Technical Notes

## Project Context

This analysis was completed independently as part of the **BCG X Data Science Job Simulation on Forage**.

PowerCo is a simulated energy provider experiencing above-average customer churn. The initial business hypothesis was that customer price sensitivity, particularly changes in electricity pricing, may be contributing to churn.

The objective of the analysis was to test that hypothesis, identify broader churn-related customer signals, engineer predictive features, and build a baseline classification model.

This is a simulated case study and does not represent work completed for BCG or an actual BCG client.

---

## Business Question

The analysis focused on two primary questions:

1. **Is customer price sensitivity a meaningful driver of churn?**
2. **What customer characteristics provide useful signals for predicting which customers are more likely to leave?**

The project was deliberately structured around the business hypothesis first rather than immediately building a predictive model.

---

## Data

The analysis used customer-level and monthly pricing data supplied through the Forage simulation.

### Customer Data

The customer dataset contained approximately **14,606 customers** and included information related to:

- Electricity and gas consumption
- Forecast consumption
- Customer tenure
- Contract and renewal dates
- Active products
- Gas service
- Customer margins
- Subscribed power
- Sales channel
- Customer origin
- Forecast pricing
- Churn status

The target variable was:

`churn`

- `0` = customer stayed
- `1` = customer churned

Overall churn rate:

**9.72%**

This class imbalance became important when evaluating the predictive model.

### Pricing Data

The pricing dataset contained monthly observations for each customer across:

- Off-peak pricing
- Peak pricing
- Mid-peak pricing
- Variable price components
- Fixed price components

Because each customer had multiple monthly pricing records, pricing data had to be summarized or engineered to the customer level before it could be combined with the customer dataset.

---

## Data Quality Review

Before modeling, the datasets were reviewed for issues that could affect interpretation or prediction.

Important observations included:

- Several numeric variables were strongly right-skewed.
- Consumption and margin variables contained substantial high-end outliers.
- Some categorical fields contained `"MISSING"` as a text category rather than a standard null value.
- Several price fields contained zeros that may represent structural or non-applicable values rather than literal zero pricing.
- Date variables required conversion from text into datetime format.
- `has_gas` was originally stored as `t/f` rather than numeric binary values.
- Some customer categories contained relatively small sample sizes.

Outliers were not automatically removed because unusually large customers may represent legitimate SME customers rather than data errors.

---

## Initial Price-Sensitivity Hypothesis

The initial hypothesis was:

> Customers experiencing meaningful electricity price increases should be more likely to churn.

The first test focused on the change in **off-peak variable electricity price between January and December**.

Customer-level percentage price change was calculated as:

`(December Price - January Price) / January Price`

Customers were then grouped into price-change categories:

- Decrease greater than 5%
- Decrease between 0% and 5%
- Increase between 0% and 5%
- Increase between 5% and 10%
- Increase greater than 10%

Churn rates were compared across these groups.

---

## Price-Sensitivity Results

The relationship between January-to-December off-peak price change and churn was extremely weak.

### Correlation

- Correlation with churn: approximately **-0.006**
- p-value: approximately **0.507**

The relationship was therefore not statistically significant.

The price-change buckets also did not show a consistent pattern in which larger price increases led systematically to higher churn.

For example:

- Customers experiencing increases greater than 10% showed elevated churn.
- Customers experiencing large price decreases also showed elevated churn.
- The 5–10% price-increase group contained a very small sample and should not be interpreted as a stable estimate.

### Interpretation

The analysis does **not support annual off-peak price change as a primary standalone churn driver**.

This does not prove that pricing is irrelevant.

More targeted forms of price sensitivity remain untested, including:

- Sudden monthly price shocks
- Price volatility
- Maximum price exposure
- Customer-specific price thresholds
- Price changes near renewal
- Prices relative to comparable customers

These should be treated as hypotheses for future analysis rather than demonstrated churn mechanisms.

---

## Correlation Analysis

Point-biserial correlation was used to examine relationships between continuous customer variables and the binary churn outcome.

This is equivalent to Pearson correlation when one variable is binary.

Variables investigated included:

- Consumption
- Gas consumption
- Recent consumption
- Forecast consumption
- Forecast pricing
- Margins
- Number of active products
- Customer tenure
- Maximum subscribed power
- Gas-service status

Several customer economics and tenure variables showed larger associations with churn than the original off-peak price-change feature.

However, the absolute correlations remained small.

These relationships should therefore be interpreted as:

**signals for additional segmentation and modeling, not evidence of causation.**

---

## Feature Engineering

Feature engineering was organized around four approaches:

### 1. Addition

New features were created to capture information not directly represented by individual raw variables.

Examples included:

- Average annual price change
- Maximum annual price change
- Recent consumption ratio
- Actual-versus-forecast consumption difference
- Bundled-customer indicator

### 2. Deletion

Raw variables that no longer added useful information after transformation were removed from the modeling dataset.

Examples included:

- Raw date columns after extracting useful date components
- Customer identifiers before model training

The customer `id` was retained during data processing and joins but excluded as a predictor.

### 3. Combination

Related variables were combined into more interpretable business concepts.

Examples included:

- Average forecast energy price
- Peak/off-peak price spread
- Consumption relative to subscribed power
- Margin relative to customer consumption

### 4. Mutation

Existing variables were transformed into more model-friendly representations.

Examples included:

- Log transformation of highly skewed consumption variables
- Log transformation of net margin
- Log transformation of subscribed power
- Boolean-to-binary conversion
- Categorical variables converted into dummy variables
- Dates transformed into numeric components

---

## Why Log Transformations Were Used

Exploratory analysis showed that several variables were highly right-skewed.

Examples included:

- `cons_12m`
- `forecast_cons_12m`
- `pow_max`
- `net_margin`

Large outliers can dominate some statistical relationships and modeling techniques.

Logarithmic transformations were therefore created to reduce the influence of extremely large observations while preserving the relative ordering of customers.

The original variables were not automatically assumed to be invalid.

---

## Predictive Model

A **Random Forest classifier** was used to create the baseline churn prediction model.

Random Forest was appropriate because it can:

- Handle nonlinear relationships
- Capture interactions between features
- Use many variables simultaneously
- Model complex customer behavior without requiring every relationship to be linear

The dataset was divided into:

- **80% training data**
- **20% testing data**

The split was stratified to preserve the approximately 9.72% churn rate in both samples.

The baseline model used:

- 300 decision trees
- `random_state=42`
- balanced class weighting
- all available processor cores

---

## Evaluation Metrics

Because churn represented less than 10% of the dataset, accuracy alone would provide a misleading picture of model quality.

The model was evaluated using:

### Accuracy

Measures the percentage of all predictions that were correct.

Useful as an overall measure, but heavily influenced by the large number of customers who did not churn.

### Precision

Measures how often customers predicted to churn actually churned.

High precision means PowerCo would generate relatively few unnecessary churn alerts.

### Recall

Measures how many actual churners were successfully identified.

This is particularly important for retention because missed churners represent customers PowerCo would not have an opportunity to retain.

### F1 Score

Balances precision and recall.

Useful because the target classes are highly imbalanced.

### ROC-AUC

Measures how well the model ranks churners above non-churners across different probability thresholds.

This provides information about the underlying discriminatory ability of the model rather than performance at only the default classification threshold.

---

## Baseline Random Forest Results

The final baseline model produced approximately:

| Metric | Result |
|---|---:|
| Accuracy | 91.0% |
| Precision | 88.89% |
| Recall | 8.45% |
| F1 Score | 15.43% |
| ROC-AUC | 0.7149 |

### Confusion Matrix

| | Predicted Stay | Predicted Churn |
|---|---:|---:|
| Actual Stay | 2,635 | 3 |
| Actual Churn | 260 | 24 |

The model identified only:

**24 of 284 actual churners**

while missing:

**260 churners**

---

## Model Interpretation

The model produces very few false churn alerts, resulting in high precision.

However, it is extremely conservative when assigning customers to the churn class.

This explains the combination of:

- High accuracy
- High precision
- Very low recall
- Low F1 score

The **ROC-AUC of approximately 0.715** is more encouraging.

It indicates that the model contains meaningful information for ranking customers by relative churn risk, even though the default classification threshold performs poorly at identifying churners.

Therefore, the model should be interpreted as a **baseline predictive model requiring further validation**, not as a deployment-ready churn intervention system.

---

## Business Themes Emerging From the Model

Rather than pointing to one dominant churn driver, the analysis suggests that multiple customer dimensions contribute to the risk picture.

### Customer Economics

Includes:

- Customer profitability
- Margin efficiency
- Recurring account value

### Consumption Patterns

Includes:

- Overall energy demand
- Recent changes in usage
- Deviations from forecasted consumption

### Pricing Context

Pricing variables contribute to the predictive model, but annual off-peak price change is not a dominant standalone signal.

### Customer Relationship

Tenure and product engagement may help identify customer segments with different retention behavior and warrant further segmentation.

---

## Business Interpretation

The results suggest that PowerCo should not treat churn primarily as a broad price-discounting problem.

Instead, churn appears to be a **multi-signal customer retention problem**.

The current evidence supports deeper investigation into combinations of:

- Customer economics
- Consumption behavior
- Tenure
- Product relationship
- Pricing context

Potential interventions should only be recommended after the relevant churn-risk mechanisms are validated.

---

## Potential Areas for Further Analysis

### Price Sensitivity

Test whether churn becomes more concentrated around:

- Maximum monthly price increases
- Price volatility
- High absolute prices
- Prices relative to peers
- Price changes near contract renewal

### Customer Segmentation

Test whether churn risk differs across:

- Customer tenure groups
- Product count
- Gas versus electricity-only customers
- Sales channels
- Customer origin
- Margin/value segments

### Model Validation

Evaluate:

- Churn rate by predicted-risk decile
- Alternative classification thresholds
- Cross-validation
- Hyperparameter tuning
- Segment-specific model performance
- Precision/recall trade-offs

### Additional Data

Several missing variables could materially improve the analysis:

- Actual churn or cancellation date
- Customer complaint history
- Competitor offer exposure
- Retention offer history
- Time-of-use consumption
- Customer contact history

---

## Key Limitations

### Class Imbalance

Only approximately 9.72% of customers churned.

This makes accuracy a weak standalone performance measure.

### Missing Churn Timing

The dataset contains a final churn label but not the exact month or date when each customer churned.

This prevents reliable month-by-month churn analysis and limits causal timing conclusions.

### Correlation Does Not Imply Causation

Observed relationships show association, not proof that specific variables caused churn.

### Feature Importance Does Not Imply Causation

Random Forest feature importance indicates which variables helped the model make predictions, not which variables directly caused customer behavior.

### Structural Pricing Zeros

Some zero-valued pricing fields may represent non-applicable tariff structures rather than true zero prices.

---

## Final Technical Conclusion

The initial hypothesis that broad annual price increases are a primary driver of customer churn was not supported by the available data.

Customer churn appears to contain broader predictive signals related to economics, consumption patterns, tenure, and pricing context.

The Random Forest achieved meaningful risk discrimination with an ROC-AUC of approximately **0.715**, but very low churn recall indicates that the current model is not ready for operational churn intervention.

The next stage should focus on validating risk concentration, improving churn detection, and determining whether identified customer segments support economically viable retention actions.
