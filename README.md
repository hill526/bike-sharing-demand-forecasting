# London Bike-Sharing Demand Forecasting

Forecasting hourly bike rentals in London, so a bike-sharing operator can have the right number of bikes in the right place at the right time.

![Average rentals by hour, weekday vs weekend](images/rentals_by_hour.png)

*On weekdays, demand has two sharp commuter peaks (8 AM and 5–6 PM). On weekends, it rises slowly to one afternoon peak.*

---

## Why this project

A bike-sharing operator faces two costly problems. If there are too few bikes at a station during the morning rush, customers can't ride and may not come back. If there are too many bikes sitting unused at night, the company is paying to maintain and move bikes nobody needs.

A reliable hourly forecast helps the operator decide **when to move bikes between stations, when to schedule maintenance, and when to run promotions.**

## Data

- **Source:** [London Bike Sharing Dataset](https://www.kaggle.com/datasets/hmavrodiev/london-bike-sharing-dataset) on Kaggle, created by Hristo Mavrodiev from Transport for London open data
- **Size:** 17,414 hourly records, January 2015 to January 2017
- **Features:** time of day, weekday/weekend, public holiday, season, actual temperature, "feels like" temperature, humidity, wind speed, and weather condition
- **Target:** `cnt`, the number of new bike rentals in each hour

## Approach

1. **Data quality check:** checked for missing values, duplicates and unrealistic values (e.g. humidity above 100%).
2. **Feature preparation:** extracted hour, day, month and year from the timestamp, converted season codes into names, and one-hot encoded categories such as weather condition.
3. **Exploratory analysis:** compared rentals across hours, days, seasons and weather conditions to find the main demand drivers.
4. **Modelling:** compared four regression models: Linear Regression, Decision Tree, Random Forest and Gradient Boosting.
5. **Evaluation:** measured performance with R², RMSE and MAE on a held-out test set. Tree pruning settings were tuned with cross-validation on the training data only, so the test set stayed unseen until the final check.

## Key findings

- **Time of day matters most.** Weekday rentals peak at around **3,800 bikes per hour at 8 AM** and **3,200 at 5 PM**, compared with about **50 per hour at 3 AM**.
- **Weekends follow a different pattern.** Instead of commuter peaks, demand rises steadily to around **2,100–2,200 bikes per hour between 2 PM and 4 PM**, which suggests leisure trips.
- **Warmer weather means more riders.** Average hourly rentals rise from about **630 below 5°C** to about **2,300 at 20–25°C**.
- **Rain cuts demand.** Rainy hours average roughly **40% fewer** rentals than clear hours.
- **Summer is the busiest season**, and winter is the quietest (around 1,460 vs 820 rentals per hour on average).

![Average rentals by temperature](images/rentals_vs_temperature.png)

## Model results

| Model | R² | RMSE | MAE |
|---|---|---|---|
| Linear Regression | 0.22 | 967 | 703 |
| Decision Tree (pruned) | 0.91 | 325 | 210 |
| Random Forest | 0.94 | 261 | 138 |
| **Gradient Boosting** | **0.95** | **255** | 143 |

**Gradient Boosting performed best.** It explains about 95% of the variation in hourly demand, and its predictions are typically off by around **255 bikes per hour**. For context, the average hour has around 1,150 rentals, and peak hours have more than 3,000.

Two results were important along the way:

- **Linear regression did poorly (R² = 0.22)** because it can only fit straight-line relationships, and demand doesn't follow a straight line over the day.
- **The unpruned decision tree overfit.** It scored a perfect R² of 1.00 on the training data but only 0.90 on the test data. Pruning the tree, and then moving to ensemble models, fixed this.

![Feature importance of the best model](images/feature_importance.png)

## Explained simply

**Why did linear regression fail?** A linear model can only draw a straight line. If it sees "hour" as a number from 0 to 23, it assumes demand keeps going up (or down) as the day goes on. But real demand goes up in the morning, drops at midday, goes up again in the evening, and falls at night. A straight line can't follow that shape.

**Why did the first decision tree overfit?** It was like a student who memorises every answer from past exams. They get 100% on questions they've already seen, but struggle with new ones. The tree learned the training data perfectly, including its random noise, so it did worse on data it hadn't seen.

**Why did Random Forest and Gradient Boosting do better?** Instead of trusting one tree, they combine hundreds of them. Random Forest asks many trees and takes the average answer, like asking a group of people instead of one. Gradient Boosting builds trees one after another, with each new tree fixing the mistakes of the previous ones.

## Recommendations

1. **Rebalance bikes before 7 AM on weekdays.** Make sure stations near homes are full before the 8 AM peak, and stations near offices have free docks.
2. **Schedule maintenance in the quiet hours** (late night and early morning), when demand is lowest and fewer bikes are needed.
3. **Combine the forecast with weather forecasts.** Plan a larger fleet on warm, dry days and a smaller one on cold or rainy days.
4. **Run promotions in off-peak periods**, such as winter or weekday midday, to use bikes that would otherwise sit idle.

## Limitations & next steps

- The data covers **one city and two years**, so patterns may differ in other cities or in later years.
- The model predicts **total demand across London**, not demand at each station, which is what rebalancing crews need most.
- Events such as concerts, strikes or public transport disruptions aren't in the data, but they can change demand a lot.
- **Next steps:** add station-level data, test time-series models, and include event data.

## How I used AI tools

I used AI tools to help debug Python code and to review the wording of this README. I checked all code, results and conclusions myself.

## How to run

```bash
git clone https://github.com/hill526/bike-sharing-demand-forecasting.git
cd bike-sharing-demand-forecasting
pip install -r requirements.txt
jupyter notebook notebooks/bike_demand_forecasting.ipynb
```

## Repository structure

```
bike-sharing-demand-forecasting/
├── data/            # london_merged.csv (from Kaggle)
├── images/          # charts used in this README
├── notebooks/       # analysis and modelling notebook
├── requirements.txt
└── README.md
```

## Acknowledgements

Thanks to Hristo Mavrodiev for publishing the dataset on Kaggle, and to Transport for London for the original open data.

---

**Minh Hieu (Hill) Huynh** · Bachelor of Business Analytics, Deakin University · [LinkedIn](https://www.linkedin.com/in/hmh06206/)
