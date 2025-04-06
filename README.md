# Blocktime Prediction


## Project Objective
The objective of this project is to predict the actual block time (gate-to-gate time) for American Airlines (AA) flights using historical flight data from the U.S. Department of Transportation (DOT) for the year 2024. In addition to prediction, exploratory data analysis (EDA) was performed to uncover patterns and understand which factors impact block time.

## Dataset
-	Source: U.S. Department of Transportation (DOT) [DOT](https://www.transtats.bts.gov/DL_SelectFields.aspx?gnoyr_VQ=FGK&QO_fu146_anzr=b0-gvzr)
-	Scope: All American Airlines (AA) flights for the year 2024.
-	Size: Final cleaned dataset contains ~1.8 million rows and 54 features initially.

## Data Preprocessing & Cleaning
Multiple monthly Excel files (January to December) were appended to the original Master file using Python.

### Cleaning Steps
-	Removed columns with more than 50% missing values
-	Renamed ActualElapsedTime to ActualBlock, and CRSElapsedTime to ScheduleBlock
-	Removed cancelled and diverted flights
### Feature Engineering
-	BlockDiff = ActualBlock - ScheduleBlock
-	TaxiTotal = TaxiOut + TaxiIn
-	MarketPair = Origin-Dest
-	BlockCategory created based on block time differences:
    - Early: BlockDiff < -5
    -	On Time: -5 ≤ BlockDiff ≤ 5
    -	Late: BlockDiff > 5

## Exploratory Data Analysis (EDA)
### Taxi Time vs. Block Time
-  General Trend: As the ActualBlock time increases, TaxiTotal (sum of TaxiIn and TaxiOut) generally increases up to a point.
-  Interesting Observation: After around 300 minutes of block time, the taxi time starts to stabilize or slightly decline, rather than continuing to grow.
- Explanation:
   - Flights with longer block times (300+ minutes) are usually long-haul or cross-country. These flights often operate from larger hub airports with better infrastructure and dedicated gates, possibly reducing taxi delays.
    - On the other hand, short- to mid-haul flights might face more congestion or frequent takeoff/landing queues, leading to more variability in taxi time.
[Taxi Time vs. Block Time Result ](https://drive.google.com/file/d/1FHCcMRB0cTVeXvtiVVeth2kBoApnod5k/view?usp=sharing) 

### Flight Outcomes by Day of Week
- Insight: The distribution of Early, On-Time, and Late flights is fairly consistent across all weekdays.
-	Friday (Day 5) shows a slight increase in late arrivals.
-	Saturday (Day 6) has the least number of total flights, which may correlate with fewer operational challenges.
- Explanation: Weekdays like Friday might require better block time planning to reduce late arrivals due to heavier traffic.
[Block Category by Day of Week Result ](https://drive.google.com/file/d/1OUYErKozlyNr4T2yXKEIPri2KlGVUfWJ/view?usp=sharing) 

### Flight Outcomes by Month
- There is a seasonal trend:
    -	October shows a notable increase in early arrivals.
    -	July and August (summer months) have a higher number of late arrivals, which might reflect seasonal demand and congestion.
- Explanation: Seasonality plays a role in operational performance, and block time buffers may need to be adjusted accordingly.
[Block Category by Month Result ](https://drive.google.com/file/d/1PkBrk-QydlrN8rMx8DeyzAJ9X3RKXxaq/view?usp=sharing) 

### Routes with Frequent Early Arrivals
- Insight: Routes like LGA-DFW, ORD-DFW, and DFW-MIA have consistently earlier-than-scheduled arrivals (more than 10 minutes early on average).
- Explanation: These routes might have overestimated schedule blocks, indicating potential optimization opportunities to save time and operational costs.
[Routes with Frequent Early Arrivals Result ](https://drive.google.com/file/d/14XDIjTVsgwnorsSFu-N-3nYwVB9z5ylC/view?usp=sharing) 

### Routes with Frequent Delays
- Insight: Routes like MIA-DFW, SAT-DFW, and AUS-DFW show average delays of 18–21 minutes in block time.
- Explanation: These market pairs might be impacted by consistent congestion or operational inefficiencies, and could benefit from re-evaluation of scheduled blocks.
[Routes with Frequent Delays Result ](https://drive.google.com/file/d/1JotptBl_O--GAUxh5HktJ_LReupoS9k8/view?usp=sharing) 

## Variable Selection and Reduction
In order to improve the model’s performance and reduce multicollinearity, I performed a thorough selection process by dropping variables that were redundant or highly correlated with the target variable, Block. This helps in ensuring that the model is not overwhelmed with irrelevant or duplicate information.

### The following reasons guided my decision to drop certain variables:
- Redundant Variables: Many columns, such as "OriginCityName" and "DestCityName", were repetitive and contained overlapping information that would not contribute new insights to the analysis.
- High Correlation with the Target: Variables that were highly correlated with the target variable, Block, were removed to prevent overfitting and ensure the model remained generalizable.
- Irrelevant or Identifier Variables: Columns like "Tail_Number" and "Flight_Number_Operating_Airline" are mostly unique identifiers that don't contribute to the predictive power of the model.
- By reducing the dataset from over 119 variables to just 27, I ensured that the remaining features are meaningful, non-redundant, and improve the overall efficiency and interpretability of the model.

## Model Building
Model Used: Random Forest

### Preprocessing:
Removed redundant, highly correlated, or low-variance features:
-	Dropped variables like ScheduleBlock, TaxiIn, TaxiOut, and AirTime to prevent data leakage, as they are highly correlated with the target (ActualBlock).
-	Removed low-predictive-power features such as DOT_ID_Marketing_Airline since the dataset was filtered to include only American Airlines (AA), making these columns contain a single constant value.

### Hyperparameter Choices:

- n_estimators: 25
Started with 100 trees but reduced to speed up training with minimal accuracy loss.

- max_depth: 10
Initially set to 15 but lowered to prevent overfitting and reduce runtime.

- n_jobs:
Set to 1 to manage memory usage and improve execution speed, as the model was taking too long to run.



### Evaluation:
- RMSE: ≈ 18.03 minutes
- R^2 Score: ≈ 0.9226
- Adjusted R^2 ≈ 0.9226
- Training R^2 ≈ 0.9231  
Good model performance due to a clean and predictable dataset.

## Feature Importance
Random Forest revealed that Distance is by far the most important predictor of actual block time, followed by airline identifier and arrival delay metrics. This aligns with expectations, as longer routes naturally take more time.
[Feature Importance Result](https://drive.google.com/file/d/1n9812YBJE-3duLaAQE_bivLLtcaXZVL-/view?usp=sharing) 

## Cross Validation
I also tried to perform K-Fold Cross Validation (3 folds) with Random Forest on a subsample of 100,000 rows due to memory limits in Google Colab and to mitigate overfitting. The results remained quite similar, likely because my dataset was clean and well-structured, with a diverse set of variables that contributed to high accuracy

## Conclusion
-	This project successfully built a model to predict actual block time using real DOT data for AA flights.
-	EDA showed interesting patterns in scheduling efficiency across routes, days, and months.
-	The model can be extended to support schedule optimization for airlines.
