# Pipeline Architecture
Describe the designed data pipeline, the architecture, the costs, and how these support the business goals.

<img width="1037" height="551" alt="non_transparent_final_pipeline drawio" src="https://github.com/user-attachments/assets/591ba405-549a-4be4-a102-9d8ee74af241" />

# API
I am using the `AlphaVantage API` as from the free tier we are able to collect historical data daily and also news and sentiment data, which is just the two datasources I need.

# Initial Upload
It is not visible in the pipeline, but I used another Lambda function for the initial upload of historical data as a reference point, the code is very similar to the daily data. That is also in the bucket. 

# Lambda
## Daily Stock Data
The Lambda function for this section is fetching the data from the API and is calculating a prediction on that, by appending a new column to the dataframe called and checking its value. The prediction/suggestion can be "Strong Sell", "Sell", "Hold", "Buy" and "Strong Buy", reflecting the stock's predicted movement for the day. The newest data is always in `t-1`.

## Hourly NLP Sentiment Data
This function fetches the **sentiment** data from the same API every hour when the NYSE is open to search for news that have an impact on the public sentiment and can influence our prediction for the day. The function checks news from the last hour.

# EventBridges
## Daily
I have added a trigger to the lambda function to invoke it every morning 1 hour before the NYSE opens, so we can have a prediction ready for the day.

## Hourly
In the same way, I have added a trigger to this lambda function too to invoke it every hour the NYSE is open.

# S3 Simple Storage Bucket and JSON prediction check
The bucket has 3 folders, one for the daily stock data and prediction, one for the "live" sentiments and third technical one for the initial upload.
The predictions are extracted from the folders as JSON and it alerts the SNS if there is a sentiment change in the market.

# SNS
A trigger that instant alerts when critical anomalies occur (e.g., Model predicts "Buy" but Live News indicates "Market Crash"), enabling "rapid" response to breaking trends.

# Cost
The architecture is 100% serverless. By optimizing the Python code to remove heavy dependencies (like Pandas) and scheduling Lambdas to sleep when the market is closed, the system operates entirely within the AWS Free Tier.
