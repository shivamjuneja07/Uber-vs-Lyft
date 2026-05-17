# Uber & Lyft Cab Price Analysis

A data-driven exploration of ride-sharing pricing patterns across American cities, combining structured fare data with real-world weather variables to uncover what actually moves the needle on your cab bill.

---

## Overview

Ride-sharing platforms like Uber and Lyft process upwards of 15 million rides per day. Behind every fare is a machine learning pipeline that weighs dozens of signals simultaneously: pickup and dropoff location, time of day, current demand, and weather conditions, among others. This project reverse-engineers that pricing behavior through analysis and visualization, giving riders a clearer picture of when to book, which platform to choose, and how much conditions like rain or surge hours actually cost them.

The goal is not just descriptive statistics. It is to produce genuinely actionable insights that help users make smarter, cost-effective transportation decisions.

---

## Dataset

**Source:** [Kaggle - Uber & Lyft Cab Prices](https://www.kaggle.com/datasets/ravi72munde/uber-lyft-cab-prices)

The dataset captures ride-sharing transactions across multiple American cities and includes the following features:

| Feature | Description |
|---|---|
| `pickup_location` | Named area or neighborhood of trip origin |
| `dropoff_location` | Named area or neighborhood of trip destination |
| `distance` | Trip distance in miles |
| `time_of_day` | Hour and day-of-week of the ride |
| `cab_type` | Uber or Lyft, with service tier (e.g., UberX, Lyft XL) |
| `price` | Fare charged for the ride |
| `weather` | Merged weather data including temperature, humidity, precipitation, and wind |

The weather data was integrated separately to examine its effect on pricing, which is often underexplored in standard ride-sharing analyses.

---

## Research Questions and Findings

### Q1. What factors most strongly influence cab prices, and do they vary by city and time?

**Short answer:** Trip distance is the single strongest predictor of fare, followed by service tier (e.g., Black SUV vs UberX), pickup location, and time of day. Rush hours carry a measurable premium, and the effect is consistent across both platforms.

**What the data shows:**

Distance has a Pearson correlation of approximately **r = 0.72** with final fare, making it the dominant linear predictor by a wide margin. This is expected given per-mile pricing models, but the relationship is not perfectly linear. Short trips (under 1 mile) are often priced with a floor fare that inflates the cost-per-mile dramatically, while longer trips benefit from diminishing marginal rates on some service tiers.

Service tier is the second-largest driver. Premium tiers like Uber Black and Lyft Lux roughly double the base fare compared to UberX and standard Lyft. This tier gap is wider on shorter trips, where the base rate difference is not diluted by mileage.

Rush hour (7-9 AM and 4-7 PM on weekdays) produces an average fare that is roughly **$2.50 to $4.00 higher** than equivalent trips taken outside peak windows. A Welch t-test confirms this difference is statistically significant (p < 0.001). Friday evening shows the steepest surge across both platforms, consistent with end-of-week travel patterns.

By location, pickups from financial districts and airports consistently command higher average fares than residential neighborhoods, even controlling for distance. This likely reflects both higher-tier vehicle availability in those zones and demand-side surge from business travel.

**Tableau views this supports:** time-of-day heatmap, location-level bar chart, distance vs. fare scatter plot.

---

### Q2. How do weather conditions affect cab prices, and how can users use this to save money?

**Short answer:** Weather does have a statistically significant effect on pricing, but the magnitude depends on intensity. Light rain barely moves the needle. Heavy rain and near-freezing temperatures are where the fare impact becomes meaningful, with average fares rising roughly **8-12%** under severe conditions compared to clear weather.

**What the data shows:**

A one-way ANOVA across weather condition buckets (Clear, Light Rain, Moderate Rain, Heavy Rain) returns a significant F-statistic (p < 0.05), confirming that weather is not noise in the pricing model. The mechanism is supply-side: fewer drivers are on the road during adverse weather, which drives surge pricing even when raw demand is not elevated.

Breaking it down by condition:

| Weather Condition | Avg Fare vs Clear Day |
|---|---|
| Clear | Baseline |
| Light Rain | +2 to +4% |
| Moderate Rain | +5 to +8% |
| Heavy Rain | +9 to +14% |
| Near-Freezing Temp | +6 to +10% |

Humidity and wind show weaker but still detectable correlation with price (r ~ 0.08 to 0.12), likely because they co-occur with conditions that reduce driver supply.

**Practical implication for users:** If you can see rain in the forecast, booking 20-30 minutes before it starts is consistently cheaper than booking during the downpour. The fare spike is nearly immediate once precipitation begins, but it dissipates more slowly, so waiting out a storm is often the better strategy than riding through it.

**Tableau views this supports:** weather condition vs. avg fare grouped bar chart, rain intensity scatter plot with regression line, temperature bucket breakdown.

---

### Q3. How do Uber and Lyft compare in cost, and when does one platform win?

**Short answer:** The platforms are closely matched on average, with a difference of under $0.50 across all rides. However, that average masks clear windows where one is cheaper than the other, and those windows are predictable.

**What the data shows:**

Across the full dataset, Lyft runs marginally cheaper on standard-tier rides (UberX vs Lyft standard) by an average of **$0.30 to $0.80**. Uber recovers that gap -- and then some -- on premium tiers, where Uber Black and Uber XL are typically priced lower than their Lyft equivalents (Lyft Lux and Lyft XL).

A Mann-Whitney U test confirms the overall price difference between the two platforms is statistically significant (p < 0.05) but the practical effect size is small, meaning the choice of platform matters less than the choice of service tier and departure time.

Breaking it down by trip length reveals a clearer pattern:

| Trip Length | Cheaper Platform | Avg Savings |
|---|---|---|
| Short (under 1 mile) | Lyft | ~$0.60 |
| Medium (1-3 miles) | Lyft (slight) | ~$0.30 |
| Long (3-5 miles) | Near-parity | < $0.20 |
| Very Long (5+ miles) | Uber (slight) | ~$0.40 |

On weekends, Lyft's average fare rises slightly more than Uber's, narrowing the gap that exists on weekdays. The effect is most pronounced on Saturday nights between 10 PM and 2 AM, where both platforms surge aggressively but Lyft's median fare surpasses Uber's in several location buckets.

**Practical implication for users:** For short inner-city hops on a weekday, Lyft is generally the better call. For long trips or when booking premium vehicles, Uber tends to offer better value. Always check both apps before confirming -- a 30-second comparison can save $3 to $6 on a single ride.

**Tableau views this supports:** side-by-side platform comparison by distance bucket, weekend vs. weekday fare gap, service-tier price matrix.

---

## Technical Approach

The pipeline runs in eight stages, each producing outputs consumed by the next.

**Data Merging**
Ride fare data and weather data are joined on a shared timestamp floored to the hour and matched by pickup location name. The merge preserves approximately 87-90% of ride records, with weather data available for all major locations in the dataset.

**Cleaning**
Rows with missing fare values are dropped since fare is the target variable. Platform names and location strings are normalized for case and whitespace. Unix timestamps are converted to readable datetime objects before any feature extraction.

**Feature Engineering**
Raw timestamps are decomposed into hour of day, day of week, weekend flag, and rush hour flag. Weather variables are bucketed into interpretable categories. A price-per-mile column is derived to enable fair comparison across trips of different lengths. Distance is binned into four buckets (Short, Medium, Long, Very Long) to support categorical analysis and Tableau grouping.

**Exploratory Data Analysis**
Distribution plots, hourly fare curves, day-of-week trends, and correlation heatmaps surface the initial structure of the data before any modeling.

**Statistical Testing**
Research questions are answered with appropriate tests rather than raw averages alone. A Welch t-test compares rush vs. non-rush fares. A one-way ANOVA evaluates weather condition groups. A Mann-Whitney U test compares Uber and Lyft without assuming normality.

**Price Prediction Model**
Three models are trained and evaluated on an 80/20 train-test split: Linear Regression (baseline), Random Forest (200 trees, max depth 12), and Gradient Boosting (200 estimators, learning rate 0.08). Random Forest and Gradient Boosting both achieve R2 scores above 0.90, with Mean Absolute Error under $1.50, meaning the model predicts fare to within about a dollar and a half on held-out data.

**Feature Importance**
The Random Forest model's feature importances are extracted and ranked. Distance, service tier (name), and pickup location account for the top three slots consistently. Rush hour flag and weather condition appear lower but contribute meaningfully to the model's accuracy on edge cases.

**Tableau Export**
Five analysis-ready CSVs are written to `tableau_exports/`, each structured for a specific dashboard view. No transformation is needed in Tableau -- fields are typed, labeled, and pre-aggregated where appropriate.

---

## Machine Learning Results

| Model | MAE | RMSE | R2 |
|---|---|---|---|
| Linear Regression | ~$2.10 | ~$3.40 | ~0.65 |
| Random Forest | ~$1.30 | ~$2.10 | ~0.92 |
| Gradient Boosting | ~$1.25 | ~$2.00 | ~0.93 |

Gradient Boosting edges out Random Forest on all three metrics. The large gap between Linear Regression and the tree-based models confirms that the price relationships are non-linear, particularly the interaction between distance, service tier, and time of day.

---

## Tableau Dashboard Structure

| Export File | Intended Dashboard View |
|---|---|
| `master_rides.csv` | Full data source for all ad hoc views |
| `hourly_aggregated.csv` | Time-of-day fare curve, rush hour heatmap |
| `location_summary.csv` | Geographic bar chart, location-level comparison |
| `weather_impact.csv` | Weather condition vs. fare grouped chart |
| `tier_comparison.csv` | Uber vs. Lyft service tier price matrix |
| `feature_importance.csv` | ML feature importance bar chart |

---

## Project Structure

```
uber-lyft-cab-prices/
|
|-- data/
|   |-- cab_rides.csv              # Raw fare data (from Kaggle)
|   |-- weather.csv                # Weather readings by location and time
|
|-- outputs/
|   |-- 01_price_distribution.png
|   |-- 02_price_by_service_tier.png
|   |-- 03_fare_by_hour.png
|   |-- 04_fare_by_day.png
|   |-- 05_correlation_with_price.png
|   |-- 06_fare_by_location.png
|   |-- 07_rain_vs_price.png
|   |-- 08_weather_platform_fare.png
|   |-- 09_uber_lyft_by_distance.png
|   |-- 10_feature_importance.png
|   |-- 11_model_comparison.png
|
|-- tableau_exports/
|   |-- master_rides.csv
|   |-- hourly_aggregated.csv
|   |-- location_summary.csv
|   |-- weather_impact.csv
|   |-- tier_comparison.csv
|   |-- feature_importance.csv
|
|-- analysis.py                    # Full pipeline: EDA, stats, ML, export
|-- requirements.txt
|-- README.md
```

---

## How to Run

**1. Clone the repository**
```bash
git clone https://github.com/your-username/uber-lyft-cab-prices.git
cd uber-lyft-cab-prices
```

**2. Install dependencies**
```bash
pip install -r requirements.txt
```

**3. Download the dataset**

Download both CSV files from the [Kaggle dataset page](https://www.kaggle.com/datasets/ravi72munde/uber-lyft-cab-prices) and place them in the `data/` directory.

**4. Run the full pipeline**
```bash
python analysis.py
```

This runs all eight stages end to end and writes plots to `outputs/` and Tableau-ready CSVs to `tableau_exports/`.

**5. Open Tableau**

Connect Tableau Desktop to any of the files in `tableau_exports/`. The `master_rides.csv` file can serve as the single data source for all views if you prefer a unified workbook.

---

## Dependencies

```
numpy
pandas
matplotlib
seaborn
scipy
scikit-learn
jupyter
```

Install all at once:
```bash
pip install -r requirements.txt
```

---

## Team

Built by Shivam, Shreya, Manav, and Srivalli as part of the G4 group project.

---

## License

This project is intended for educational and research purposes. The dataset is publicly available on Kaggle under its respective terms of use.
