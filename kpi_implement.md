# KPIs and Implementation
Propose at least three KPIs that the project contributes to measuring.
Document the execution, including the code used and an interpretation of the results.

## Key Performance Indicators (KPIs)
I have defined three core metrics to evaluate the system's performance and business value:

### 1.  **Directional Accuracy (%)**
**Definition:** The percentage of days where the morning prediction ("Buy" or "Sell") matched the actual market movement at close.
**Goal:** $> 60\%$. This measures if the "Strategist" (Daily Lambda) provides a statistical edge over random guessing.

### 2.  **Sentiment Reaction Latency**
**Definition:** The time difference between a news article's publication timestamp and the SNS alert delivery.
**Goal:** $< 5$ minutes. This ensures the "Guardian" (Hourly Lambda) alerts me fast enough to react to breaking news before the market fully prices it in.

### 3.  **Operational Cost**
**Definition:** Total monthly spend on AWS resources (Lambda compute time + S3 storage + SNS messages).
**Goal:** $\$0.00$. By optimizing the code to run in milliseconds and scheduling sleep times, the project must remain entirely within the AWS Free Tier.

## Implementation & Code Execution
The execution is fully automated via **Amazon EventBridge**. I do not run the code manually; it is triggered by Cron schedules that pass a standard JSON event to the functions.

### 1. Daily Market Prep (The Strategist)
**Execution:**
Runs at 12:00 UTC (Pre-market). It fetches 100 days of history, calculates technical indicators (RSI, MACD, Bollinger Bands) using standard Python math libraries (removing Pandas for performance), and saves the decision to S3.

**Code Used (Key Logic):**
```python
# Zero-dependency implementation of Technical Indicators
def calculate_rsi(prices, period=14):
    if len(prices) < period + 1: return 50.0
    
    # Calculate gains and losses
    gains, losses = [], []
    for i in range(1, period + 1):
        change = prices[i] - prices[i-1]
        if change >= 0: gains.append(change); losses.append(0.0)
        else: gains.append(0.0); losses.append(abs(change))
            
    avg_gain = sum(gains) / len(gains)
    avg_loss = sum(losses) / len(losses)
    
    if avg_loss == 0: return 100.0
    rs = avg_gain / avg_loss
    return 100 - (100 / (1 + rs))

# Final Decision Logic
if total_score >= 2: suggestion = "Strong Buy"
elif total_score == -1: suggestion = "Sell"
# ... saves to S3/daily_trend/prediction.json
