# London Bike-Sharing Demand Forecasting

Forecasting hourly bike rentals in London so a bike-sharing operator can have the right number of bikes in the right place at the right time.

![Average bike rentals per hour, weekday vs weekend](https://github.com/hill526/bike-sharing-demand-forecasting/blob/main/all_images/new_cnt_by_hours.png)

*Weekdays have two sharp commuter peaks: about 3,800 rentals per hour at 8 AM and 3,200 at 5 PM. Weekends rise slowly to one afternoon peak of about 2,200 at 3 PM.*

---

## Why this project

A bike-sharing operator faces two costly problems. If there are too few bikes at a station during the morning rush, customers can't ride and may not come back. If too many bikes sit unused at night, the company is paying to maintain and move bikes nobody needs.

A reliable hourly forecast helps the operator decide **when to move bikes between stations, when to schedule maintenance, and when to run promotions.**

## Data

- **Source:** [London Bike Sharing Dataset](https://www.kaggle.com/datasets/hmavrodiev/london-bike-sharing-dataset) on Kaggle, created by Hristo Mavrodiev from Transport for London open data
- **Size:** 17,414 hourly records, January 2015 to January 2017
- **Features:** time of day, weekday/weekend, public holiday, season, actual temperature, "feels like" temperature, humidity, wind speed and weather condition
- **Target:** `cnt`, the number of new bike rentals in each hour

## Approach

1. **Data quality check:** checked for missing values, duplicates and unrealistic values (e.g. humidity above 100%).
2. **Feature preparation:** extracted hour, day, month and year from the timestamp, grouped hours into time-of-day periods, and encoded categories such as season.
3. **Exploratory analysis:** compared rentals across hours, weekdays and weekends, seasons and weather conditions to find the main demand drivers.
4. **Modelling:** compared Linear Regression, Decision Tree (base and pruned), Random Forest and Gradient Boosting, then tuned the best model.
5. **Evaluation:** measured R², RMSE and MAE on a held-out test set, and used permutation importance to see which features the final model relies on.

## Key findings

- **Time of day is the biggest driver.** On weekdays, demand jumps from about **50 rentals per hour at 3–4 AM** to about **3,800 at 8 AM**, then peaks again at about **3,200 at 5 PM**. These are commuter trips.
- **Weekends follow a completely different pattern.** There are no commuter peaks. Instead, demand rises steadily to about **2,200 rentals per hour at 3 PM**, which suggests leisure trips.
- **Weather matters, but much less than time.** Warmer, drier hours have more rentals: average demand rises from about **630 per hour below 5°C** to about **2,300 at 20–25°C**, and rainy hours have roughly **40% fewer** rentals than clear hours. But the model shows that *when* someone rides matters far more than the weather.

## Model results

![R² score comparison across models](https://github.com/hill526/bike-sharing-demand-forecasting/blob/main/all_images/comparison.png)

| Model | R² |
|---|---|
| Linear Regression | 0.22 |
| Decision Tree (base) | 0.90 |
| Decision Tree (pruned) | 0.91 |
| Random Forest (base) | 0.94 |
| **Gradient Boosting (optimised)** | **0.95** |

**The optimised Gradient Boosting model performed best.** It explains about 95% of the variation in hourly demand, and its predictions are typically off by about **[RMSE] bikes per hour**. For context, the average hour has around 1,150 rentals, and peak hours have more than 3,000.

Two results were important along the way:

- **Linear regression did poorly (R² = 0.22)** because it can only fit straight-line relationships, and demand doesn't follow a straight line over the day.
- **The base decision tree overfit.** It scored a perfect R² of 1.00 on the training data but 0.90 on the test data. Pruning helped a little, and ensemble models helped much more.

### What the model relies on

![Permutation feature importance of the optimised Gradient Boosting model](images/feature_importance.png)

To check *why* the model makes its predictions, I randomly shuffled one feature at a time and measured how much the test R² dropped. The bigger the drop, the more the model depends on that feature.

- **Hour of day** is by far the most important feature. Without it, the model falls apart.
- **Weekend vs weekday** comes second, which matches the two different daily patterns in the first chart.
- **Temperature and humidity** add a little. Wind speed, season and "feels like" temperature add almost nothing once hour and temperature are known.

So the business takeaway is clear: **plan the fleet around the daily schedule first, then adjust for the weather.**

## Explained simply

**Why did linear regression fail?** A linear model can only draw a straight line. If it sees "hour" as a number from 0 to 23, it assumes demand keeps going up (or down) as the day goes on. But real demand goes up in the morning, drops at midday, goes up again in the evening, and falls at night. A straight line can't follow that shape.

**Why did the first decision tree overfit?** It was like a student who memorises every answer from past exams. They get 100% on questions they've already seen, but struggle with new ones. The tree learned the training data perfectly, including its random noise, so it did worse on data it hadn't seen.

**Why did Random Forest and Gradient Boosting do better?** Instead of trusting one tree, they combine hundreds of them. Random Forest asks many trees and takes the average answer, like asking a group of people instead of one. Gradient Boosting builds trees one after another, with each new tree fixing the mistakes of the previous ones.

## Recommendations

1. **Rebalance bikes before 7 AM on weekdays.** Make sure stations near homes are full before the 8 AM peak, and stations near offices have free docks. Repeat before 4 PM for the evening peak.
2. **Use a different plan on weekends.** Demand builds slowly toward the afternoon, so focus on leisure areas such as parks and the riverside from late morning.
3. **Schedule maintenance between midnight and 5 AM**, when demand is lowest.
4. **Adjust for weather on top of the schedule.** Plan a larger fleet on warm, dry days and a smaller one on cold or rainy days, and use off-peak promotions in winter.

## Limitations & next steps

- The data covers **one city and two years**, so patterns may differ in other cities or in later years.
- The model predicts **total demand across London**, not demand at each station, which is what rebalancing crews need most.
- Events such as concerts, strikes or public transport disruptions aren't in the data, but they can change demand a lot.
- **Next steps:** add station-level data, test on a later time period to check real forecasting ability, and include event data.

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
