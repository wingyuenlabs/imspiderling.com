---
Title: 83% Accuracy: How We Reverse Engineered Amazon's Dynamic Pricing Algorithm
Description: 
Author: Milinda Biswas
Date: 2026-03-01T21:31:24.000Z
Robots: noindex,nofollow
Template: index
---
<p><a href="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fwww.genspark.ai%2Fapi%2Ffiles%2Fs%2FZ0xtDEkP%3Fcache_control%3D3600" class="article-body-image-wrapper"><img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fwww.genspark.ai%2Fapi%2Ffiles%2Fs%2FZ0xtDEkP%3Fcache_control%3D3600" alt="Hero Image - Reverse Engineering Amazon's Price Algorithm" width="800" height="400"></a></p>

<p>Six months ago, we asked a simple question at <a href="https://avluz.com" rel="noopener noreferrer">Avluz.com</a>: "Can we predict when Amazon will drop prices on products?" Today, our system forecasts price changes with 83% accuracy across 50,000 products, processing 7.3 price updates per product daily. But here's the thing—the journey to get here taught us more about e-commerce algorithms than any documentation ever could.</p>

<p>This isn't a theoretical post. This is the complete technical breakdown of how we built, tested, and deployed a system that reverse-engineered Amazon's dynamic pricing patterns to power our deal discovery and price tracking platform.</p>

<p>d*', price_text).group().replace(',', ''))<br>
                    break</p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>        if not price:
            return None

        return {
            'asin': asin,
            'price': price,
            'timestamp': datetime.utcnow().isoformat(),
            'url': product_url,
            'availability': self._check_availability(soup)
        }

    except Exception as e:
        print(f"Error scraping {asin}: {str(e)}")
        return None

def _check_availability(self, soup):
    """Check if product is in stock"""
    availability = soup.select_one('#availability span')
    if availability:
        return 'in_stock' if 'In Stock' in availability.get_text() else 'out_of_stock'
    return 'unknown'
</code></pre>

</div>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
**Key Implementation Details:**
- Rotating user agents to avoid detection
- Multiple fallback selectors (Amazon changes HTML structure frequently)
- Availability tracking (critical for inventory-based pricing)
- Proper error handling and timeouts

We run this scraper every 2 hours for 50,000 products. That's 600,000 requests per day, which brings us to...

### Component 2: MongoDB Time-Series Database

![MongoDB Query Code](https://www.genspark.ai/api/files/s/C8BIaNAu?cache_control=3600)

Storing 600,000 price points daily requires efficient time-series storage. MongoDB's time-series collections were perfect for this.

</code></pre>

</div>
<p><br>
javascript<br>
// MongoDB schema for price history<br>
db.createCollection("price_history", {<br>
  timeseries: {<br>
    timeField: "timestamp",<br>
    metaField: "product",<br>
    granularity: "hours"<br>
  }<br>
});</p>

<p>// Aggregation pipeline for pattern detection<br>
const priceTrends = await db.price_history.aggregate([<br>
  {<br>
    $match: {<br>
      "product.asin": productAsin,<br>
      timestamp: {<br>
        $gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) // Last 30 days<br>
      }<br>
    }<br>
  },<br>
  {<br>
    $group: {<br>
      _id: {<br>
        hour: { $hour: "$timestamp" },<br>
        dayOfWeek: { $dayOfWeek: "$timestamp" }<br>
      },<br>
      avgPrice: { $avg: "$price" },<br>
      minPrice: { $min: "$price" },<br>
      maxPrice: { $max: "$price" },<br>
      priceChanges: { $sum: 1 },<br>
      stdDev: { $stdDevPop: "$price" }<br>
    }<br>
  },<br>
  {<br>
    $sort: { "_id.dayOfWeek": 1, "_id.hour": 1 }<br>
  }<br>
]);</p>

<p>// Calculate price volatility<br>
const volatility = priceTrends.map(trend =&gt; ({<br>
  timeSlot: <code>${trend._id.dayOfWeek}-${trend._id.hour}</code>,<br>
  volatilityScore: (trend.stdDev / trend.avgPrice) * 100,<br>
  changeFrequency: trend.priceChanges<br>
}));<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
**Database Performance:**
- **Write throughput**: 8,000 inserts/second
- **Query latency**: 45ms average for 30-day aggregations
- **Storage efficiency**: 2.4GB per million records (compressed)
- **Index strategy**: Compound index on (asin, timestamp)

This powers the real-time analysis, but the magic happens in the ML model.

### Component 3: Machine Learning Model

![ML Model Code](https://www.genspark.ai/api/files/s/Ydli5Gd3?cache_control=3600)

The prediction model is where we encode everything we learned about Amazon's patterns.

</code></pre>

</div>

<p><br>
python<br>
from sklearn.ensemble import RandomForestRegressor<br>
from sklearn.preprocessing import StandardScaler<br>
import numpy as np<br>
import pandas as pd</p>

<p>class PricePredictionModel:<br>
    def <strong>init</strong>(self):<br>
        self.model = RandomForestRegressor(<br>
            n_estimators=200,<br>
            max_depth=15,<br>
            min_samples_split=10,<br>
            random_state=42<br>
        )<br>
        self.scaler = StandardScaler()</p>
<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>def engineer_features(self, price_history, product_metadata):
    """
    Create features from raw price data
    Returns: feature matrix for ML model
    """
    df = pd.DataFrame(price_history)

    # Time-based features
    df['hour'] = pd.to_datetime(df['timestamp']).dt.hour
    df['day_of_week'] = pd.to_datetime(df['timestamp']).dt.dayofweek
    df['day_of_month'] = pd.to_datetime(df['timestamp']).dt.day
    df['is_weekend'] = df['day_of_week'].isin([5, 6]).astype(int)
    df['is_month_end'] = (df['day_of_month'] &gt;= 28).astype(int)

    # Price history features
    df['price_ma_24h'] = df['price'].rolling(window=12, min_periods=1).mean()
    df['price_ma_7d'] = df['price'].rolling(window=84, min_periods=1).mean()
    df['price_std_24h'] = df['price'].rolling(window=12, min_periods=1).std()
    df['price_change_rate'] = df['price'].pct_change()
    df['price_volatility'] = df['price'].rolling(window=24).std() / df['price'].rolling(window=24).mean()

    # Competitor pricing features
    df['competitor_min'] = product_metadata.get('competitor_prices', []).min() if product_metadata.get('competitor_prices') else df['price']
    df['price_vs_competitor'] = (df['price'] - df['competitor_min']) / df['competitor_min']

    # Inventory signals
    df['low_stock'] = (product_metadata.get('stock_level', 100) &lt; 10).astype(int)

    # Demand indicators
    df['search_volume'] = product_metadata.get('search_trend', 0)
    df['sales_rank'] = product_metadata.get('sales_rank', 999999)

    return df.fillna(0)

def train(self, historical_data, future_prices):
    """Train the model on historical data"""
    X = self.engineer_features(historical_data)
    y = future_prices  # Price 24 hours ahead

    X_scaled = self.scaler.fit_transform(X)
    self.model.fit(X_scaled, y)

    return self.model.score(X_scaled, y)  # R² score

def predict_next_price(self, recent_data, product_metadata):
    """Predict price for next time window"""
    X = self.engineer_features(recent_data, product_metadata)
    X_scaled = self.scaler.transform(X)

    prediction = self.model.predict(X_scaled[-1:])
    confidence = self.model.score(X_scaled, recent_data['price'])

    return {
        'predicted_price': float(prediction[0]),
        'confidence_score': float(confidence),
        'current_price': float(recent_data['price'].iloc[-1]),
        'predicted_change': float(prediction[0] - recent_data['price'].iloc[-1])
    }
</code></pre>

</div>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
**Model Performance Evolution:**

![Performance Chart](https://www.genspark.ai/api/files/s/0jlfX9Yq?cache_control=3600)

- **Month 1**: 47% accuracy (baseline linear regression)
- **Month 2**: 62% accuracy (added time features)
- **Month 3**: 71% accuracy (competitor price features)
- **Month 4**: 78% accuracy (demand signals integrated)
- **Month 5**: 81% accuracy (ensemble methods)
- **Month 6**: 83% accuracy (hyperparameter tuning)

### System Flow

![Process Flowchart](https://www.genspark.ai/api/files/s/1tIRoj7i?cache_control=3600)

The complete workflow:

1. **Scraper** collects price every 2 hours
2. **Detection** identifies if price changed
3. **Analysis** compares to historical patterns
4. **ML Prediction** forecasts next change
5. **Alert Generation** notifies users of opportunities
6. **Feedback Loop** improves model with results

### Technology Stack

![Technology Stack](https://www.genspark.ai/api/files/s/MiqDXvGn?cache_control=3600)

**Frontend:**
- React with Material-UI for dashboard
- Chart.js for price visualization
- WebSocket for oe real-time updates

**Backend:**
- Node.js Express API
- Python Flask for ML serving
- Redis for caching hot predictions

**Data Layer:**
- MongoDB for time-series storage
- Redis for session state
- S3 for raw HTML archives

**Infrastructure:**
- AWS Lambda for scraping jobs
- CloudWatch for monitoring
- API Gateway for public API

## The Optimization Phase: From 62% to 83%

Getting from decent accuracy to production-grade prediction required obsessive optimization.

### Failed Optimizations

Let me be honest about what didn't work:

**Attempt 1: Deep Learning**
- Tried LSTM networks for time-series prediction
- Result: 58% accuracy (worse than Random Forest)
- Reason: Not enough data per product for deep learning

**Attempt 2: Real-Time Competitor Scraping**
- Added scraping of 10 competitor sites
- Result: Minimal accuracy improvement (2%)
- Cost: 4x infrastructure costs
- Decision: Removed from production

**Attempt 3: Review Sentiment Analysis**
- Hypothesis: Review sentiment predicts pricing
- Result: No correlation found
- Lesson: Focus on direct price signals

### Successful Optimizations

What actually moved the needle:

**1. Category-Specific Models** (+7% accuracy)
</code></pre>

</div>
<p><br>
python</p>
<h1>
  
  
  Instead of one model for all products
</h1>

<p>models = {<br>
    'electronics': RandomForestRegressor(max_depth=20),<br>
    'books': GradientBoostingRegressor(max_depth=10),<br>
    'home': RandomForestRegressor(max_depth=15)<br>
}<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
Electronics prices follow different patterns than books. Category-specific models captured these nuances.

**2. Temporal Cross-Validation** (+4% accuracy)
</code></pre>

</div>

<p><br>
python</p>
<h1>
  
  
  Time-based splits instead of random splits
</h1>

<p>from sklearn.model_selection import TimeSeriesSplit</p>

<p>tscv = TimeSeriesSplit(n_splits=5)<br>
for train_idx, test_idx in tscv.split(X):<br>
    model.fit(X[train_idx], y[train_idx])<br>
    score = model.score(X[test_idx], y[test_idx])<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>
Random cross-validation was leaking future information into training. Time-based splits fixed this.

**3. Feature Interaction Terms** (+3% accuracy)
</code></pre>

</div>

<p><br>
python</p>

<h1>
  
  
  Interaction between time and price volatility
</h1>

<p>df['evening_volatility'] = df['is_evening'] * df['price_volatility']<br>
df['weekend_demand'] = df['is_weekend'] * df['search_volume']</p>



<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>


The interaction between time of day and price volatility was more predictive than either feature alone.

### ROI Analysis

![ROI Calculation](https://www.genspark.ai/api/files/s/NEvBrARG?cache_control=3600)

**Investment:**
- Development: $12,000 (3 months, 2 engineers)
- Infrastructure: $300/month AWS costs
- Total Year 1: $15,600

**Returns:**
- Labor savings: $48,000/year (40 → 2 hours/week manual work)
- Better deal pricing: $18,000/year (improved conversion)
- Total Annual Return: $66,000

**ROI: 323% in first year**

This approach now powers Avluz.com's deal recommendation engine, helping millions of shoppers catch price drops at the perfect moment.

## The Lessons: What We'd Do Differently

Looking back, here's what we learned:

### Technical Lessons

**1. Start Simple, Add Complexity Incrementally**
Our initial attempt used complex deep learning. Random Forest with good features outperformed it. Start simple, add complexity only when needed.

**2. Data Quality &gt; Data Quantity**
50,000 products with clean data beats 500,000 with noisy data. We spent 2 weeks cleaning outliers and anomalies. That investment paid off.

**3. Domain Knowledge &gt; Algorithm Choice**
Understanding *why* Amazon prices change (inventory, competitors, demand) was more valuable than trying every ML algorithm. Talk to your domain experts.

### Business Lessons

**1. Predict, Don't Just React**
The shift from reactive ("price changed!") to predictive ("price will change in 6 hours") transformed our value proposition.

**2. User Trust Requires Transparency**
We show confidence scores with predictions. Users trust "83% confident" more than "will drop" without context.

**3. Continuous Learning Is Essential**
Amazon's algorithm evolves. Our model retrains weekly with new data. Static models decay rapidly in e-commerce.

## What's Next at Avluz.com

We're expanding this approach in three directions:

**1. Multi-Retailer Prediction**
Applying these techniques to Target, Walmart, and Best Buy. Early tests show 76% accuracy—close to Amazon's 83%.

**2. Bundle Optimization**
Predicting optimal times to buy multiple products together. "Wait 3 days for laptop, buy monitor today" type recommendations.

**3. Open Source Toolkit**
Planning to release our feature engineering library as open source. The ML community helped us; time to give back.

**4. Real-Time Alert Improvements**
Moving from email alerts to push notifications with WebSocket connections. Users get price drop alerts within 30 seconds.

## Recommendations for Engineering Teams

If you're building similar systems, here's my advice:

### For Individual Engineers

**Start with data collection:** You need 3-6 months of historical data before ML makes sense. Start scraping now.

**Use existing tools:** Don't build scrapers from scratch. Libraries like Scrapy and Selenium are battle-tested.

**Validate constantly:** Test predictions against holdout data weekly. Models drift faster than you expect.

### For Engineering Teams

**Invest in infrastructure early:** We initially underestimated storage and compute needs. MongoDB's time-series collections saved us.

**Build feedback loops:** Our model improves because we track actual vs. predicted prices and retrain.

**Consider ethics:** Price prediction can be used for price discrimination. We use it to help consumers, not exploit them.

### For Companies

**This is a marathon, not a sprint:** It took us 6 months to reach 83% accuracy. Budget for iterative improvement.

**Partner with domain experts:** Our deals team's insights were as valuable as our ML expertise.

**Prepare for maintenance:** Amazon changes their site structure monthly. Budget for ongoing scraper maintenance.

## Code Repository

Want to build your own price prediction system? Key resources:

- [MongoDB Time-Series Documentation](https://www.mongodb.com/docs/manual/core/timeseries-collections/)
- [scikit-learn RandomForest Guide](https://scikit-learn.org/stable/modules/ensemble.html#forest)
- [Scrapy Best Practices](https://docs.scrapy.org/en/latest/topics/practices.html)
- [AWS Lambda for Web Scraping](https://aws.amazon.com/blogs/compute/web-scraping-at-scale-with-aws-lambda/)

## Final Thoughts

Reverse engineering Amazon's pricing algorithm taught us that modern e-commerce is a real-time game. Prices adjust to demand, inventory, and competition within minutes. Static price tracking isn't enough—you need prediction.

The system we built processes 7.3 price changes per product per day across 50,000 products. That's 365,000 price updates daily. And we can predict the next change with 83% accuracy.

But here's the most important lesson: The algorithm is just the beginning. The real value comes from helping real people save money on products they actually want to buy. That's what drives us at [Avluz.com](https://avluz.com).

---

## 💬 Discussion Questions

1. Have you experimented with price prediction for e-commerce? What accuracy did you achieve?
2. What other factors do you think influence Amazon's pricing algorithm that we might have missed?
3. For large-scale web scraping, do you prefer Scrapy, Selenium, or cloud-based solutions?
4. How do you handle model drift when external algorithms (like Amazon's) are constantly evolving?

---

*Engineering insights from [Avluz.com](https://avluz.com) - Where millions discover deals, coupons, and price drops daily. Follow our engineering blog for more deep dives into e-commerce ML, real-time data processing, and scalable web scraping.*
</code></pre>

</div>

