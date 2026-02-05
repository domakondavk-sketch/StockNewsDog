# API Integration Guide for Market Pulse PWA

## 🔌 Connecting Real Market Data

This guide shows you how to replace mock data with real market APIs.

---

## 📰 News APIs (For Market Updates)

### 1. NewsAPI (Easiest)

**Sign up**: https://newsapi.org (Free: 100 requests/day)

**Example Implementation**:

```javascript
async function fetchMarketNews() {
    const apiKey = 'YOUR_NEWSAPI_KEY_HERE';
    const categories = ['market', 'crypto', 'commodity'];
    const allNews = [];
    
    try {
        // Fetch market news
        const marketResponse = await fetch(
            `https://newsapi.org/v2/everything?q=stock+market+OR+nifty+50+OR+dow+jones&language=en&sortBy=publishedAt&apiKey=${apiKey}`
        );
        const marketData = await marketResponse.json();
        
        marketData.articles.slice(0, 5).forEach(article => {
            allNews.push({
                category: 'market',
                title: article.title,
                content: article.description || article.content?.substring(0, 200),
                source: article.source.name,
                time: new Date(article.publishedAt).toLocaleTimeString()
            });
        });
        
        // Fetch crypto news
        const cryptoResponse = await fetch(
            `https://newsapi.org/v2/everything?q=bitcoin+OR+ethereum+OR+cryptocurrency&language=en&sortBy=publishedAt&apiKey=${apiKey}`
        );
        const cryptoData = await cryptoResponse.json();
        
        cryptoData.articles.slice(0, 3).forEach(article => {
            allNews.push({
                category: 'crypto',
                title: article.title,
                content: article.description || article.content?.substring(0, 200),
                source: article.source.name,
                time: new Date(article.publishedAt).toLocaleTimeString()
            });
        });
        
        // Fetch commodity news
        const commodityResponse = await fetch(
            `https://newsapi.org/v2/everything?q=gold+OR+crude+oil+OR+natural+gas&language=en&sortBy=publishedAt&apiKey=${apiKey}`
        );
        const commodityData = await commodityResponse.json();
        
        commodityData.articles.slice(0, 2).forEach(article => {
            allNews.push({
                category: 'commodity',
                title: article.title,
                content: article.description || article.content?.substring(0, 200),
                source: article.source.name,
                time: new Date(article.publishedAt).toLocaleTimeString()
            });
        });
        
        return allNews;
        
    } catch (error) {
        console.error('Error fetching news:', error);
        return getMockNews(); // Fallback to mock data
    }
}
```

---

### 2. Alpha Vantage (Market Data + News)

**Sign up**: https://www.alphavantage.co (Free: 25 requests/day)

**Example Implementation**:

```javascript
async function fetchMarketNews() {
    const apiKey = 'YOUR_ALPHAVANTAGE_KEY_HERE';
    const allNews = [];
    
    try {
        // Fetch news & sentiment
        const response = await fetch(
            `https://www.alphavantage.co/query?function=NEWS_SENTIMENT&topics=technology,finance&apiKey=${apiKey}`
        );
        const data = await response.json();
        
        if (data.feed) {
            data.feed.slice(0, 10).forEach(article => {
                // Determine category based on topics
                let category = 'market';
                if (article.topics?.includes('blockchain') || 
                    article.topics?.includes('cryptocurrency')) {
                    category = 'crypto';
                } else if (article.title.toLowerCase().includes('gold') || 
                           article.title.toLowerCase().includes('oil')) {
                    category = 'commodity';
                }
                
                allNews.push({
                    category: category,
                    title: article.title,
                    content: article.summary,
                    source: article.source,
                    time: new Date(article.time_published).toLocaleTimeString(),
                    sentiment: article.overall_sentiment_label // Bonus: sentiment analysis!
                });
            });
        }
        
        return allNews;
        
    } catch (error) {
        console.error('Error fetching news:', error);
        return getMockNews();
    }
}
```

---

### 3. Finnhub (Stock Market Specific)

**Sign up**: https://finnhub.io (Free: 60 requests/minute)

**Example Implementation**:

```javascript
async function fetchMarketNews() {
    const apiKey = 'YOUR_FINNHUB_KEY_HERE';
    const allNews = [];
    
    try {
        // Fetch general market news
        const response = await fetch(
            `https://finnhub.io/api/v1/news?category=general&token=${apiKey}`
        );
        const articles = await response.json();
        
        articles.slice(0, 10).forEach(article => {
            allNews.push({
                category: 'market',
                title: article.headline,
                content: article.summary,
                source: article.source,
                time: new Date(article.datetime * 1000).toLocaleTimeString()
            });
        });
        
        return allNews;
        
    } catch (error) {
        console.error('Error fetching news:', error);
        return getMockNews();
    }
}
```

---

## 💰 Crypto-Specific APIs

### CoinGecko (FREE, No API Key Needed!)

**Documentation**: https://www.coingecko.com/en/api

**Example Implementation**:

```javascript
async function fetchCryptoNews() {
    try {
        // Note: CoinGecko doesn't have a news endpoint, but we can get price updates
        const response = await fetch(
            'https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=10&page=1&sparkline=false&price_change_percentage=24h'
        );
        const coins = await response.json();
        
        const cryptoUpdates = coins.map(coin => ({
            category: 'crypto',
            title: `${coin.name} (${coin.symbol.toUpperCase()})`,
            content: `Price: $${coin.current_price.toLocaleString()} | 24h Change: ${coin.price_change_percentage_24h.toFixed(2)}%`,
            source: 'CoinGecko',
            time: new Date().toLocaleTimeString()
        }));
        
        return cryptoUpdates;
        
    } catch (error) {
        console.error('Error fetching crypto data:', error);
        return [];
    }
}
```

---

## 📊 Market Data APIs (Prices, Indices)

### Nifty 50 & Indian Markets

**Option 1: NSE India Official**
- Website: https://www.nseindia.com
- No API key needed, but requires headers
- Example:

```javascript
async function fetchNiftyData() {
    try {
        const response = await fetch(
            'https://www.nseindia.com/api/allIndices',
            {
                headers: {
                    'Accept': 'application/json',
                    'User-Agent': 'Mozilla/5.0'
                }
            }
        );
        const data = await response.json();
        
        const nifty50 = data.data.find(index => index.index === 'NIFTY 50');
        
        return {
            category: 'market',
            title: 'Nifty 50 Update',
            content: `Current: ${nifty50.last} | Change: ${nifty50.percentChange}% | ${nifty50.last > nifty50.previousClose ? '📈 Up' : '📉 Down'}`,
            source: 'NSE India',
            time: new Date().toLocaleTimeString()
        };
        
    } catch (error) {
        console.error('Error fetching Nifty data:', error);
        return null;
    }
}
```

**Option 2: Alpha Vantage (Easier)**

```javascript
async function fetchNiftyData() {
    const apiKey = 'YOUR_ALPHAVANTAGE_KEY';
    
    try {
        const response = await fetch(
            `https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=NIFTY50.BSE&apiKey=${apiKey}`
        );
        const data = await response.json();
        const quote = data['Global Quote'];
        
        return {
            category: 'market',
            title: 'Nifty 50 Update',
            content: `Price: ${quote['05. price']} | Change: ${quote['09. change']} (${quote['10. change percent']})`,
            source: 'Alpha Vantage',
            time: new Date().toLocaleTimeString()
        };
        
    } catch (error) {
        console.error('Error:', error);
        return null;
    }
}
```

---

## 🛢️ Commodity APIs (Gold, Oil, Gas)

### Alpha Vantage Commodities

```javascript
async function fetchCommodityData() {
    const apiKey = 'YOUR_ALPHAVANTAGE_KEY';
    const commodities = [];
    
    try {
        // Gold
        const goldResponse = await fetch(
            `https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=XAU&to_currency=USD&apiKey=${apiKey}`
        );
        const goldData = await goldResponse.json();
        const goldPrice = goldData['Realtime Currency Exchange Rate']['5. Exchange Rate'];
        
        commodities.push({
            category: 'commodity',
            title: 'Gold Price Update',
            content: `Current: $${parseFloat(goldPrice).toFixed(2)} per oz`,
            source: 'Alpha Vantage',
            time: new Date().toLocaleTimeString()
        });
        
        // Crude Oil (WTI)
        const oilResponse = await fetch(
            `https://www.alphavantage.co/query?function=WTI&interval=daily&apiKey=${apiKey}`
        );
        const oilData = await oilResponse.json();
        const latestOil = Object.values(oilData.data)[0];
        
        commodities.push({
            category: 'commodity',
            title: 'Crude Oil (WTI) Update',
            content: `Current: $${latestOil.value} per barrel`,
            source: 'Alpha Vantage',
            time: new Date().toLocaleTimeString()
        });
        
        return commodities;
        
    } catch (error) {
        console.error('Error fetching commodity data:', error);
        return [];
    }
}
```

---

## 🔄 Combined Implementation

Here's how to combine everything:

```javascript
async function fetchMarketNews() {
    const allNews = [];
    
    // Fetch from multiple sources
    const [newsArticles, cryptoUpdates, niftyData, commodityData] = await Promise.all([
        fetchNewsAPIData(),
        fetchCryptoNews(),
        fetchNiftyData(),
        fetchCommodityData()
    ]);
    
    // Combine all data
    allNews.push(...newsArticles);
    allNews.push(...cryptoUpdates);
    if (niftyData) allNews.push(niftyData);
    allNews.push(...commodityData);
    
    // Sort by time (most recent first)
    allNews.sort((a, b) => {
        const timeA = new Date(a.time);
        const timeB = new Date(b.time);
        return timeB - timeA;
    });
    
    return allNews;
}
```

---

## ⚡ Rate Limiting & Caching

**Important**: Free APIs have rate limits. Implement caching:

```javascript
// Cache news for 5 minutes
const CACHE_DURATION = 5 * 60 * 1000; // 5 minutes
let newsCache = null;
let lastFetchTime = 0;

async function fetchMarketNews() {
    const now = Date.now();
    
    // Return cached data if still fresh
    if (newsCache && (now - lastFetchTime) < CACHE_DURATION) {
        return newsCache;
    }
    
    // Fetch fresh data
    const freshNews = await fetchNewsFromAPIs();
    
    // Update cache
    newsCache = freshNews;
    lastFetchTime = now;
    
    return freshNews;
}
```

---

## 🔐 Securing API Keys

**NEVER expose API keys in frontend code!**

For a production app, you should:

1. **Create a backend proxy** (recommended)
   - Use services like Netlify Functions or Vercel Serverless
   - Store API keys in environment variables
   - Frontend calls your proxy, proxy calls the API

2. **Example Netlify Function**:

Create a file: `netlify/functions/fetch-news.js`

```javascript
exports.handler = async function(event, context) {
    const fetch = require('node-fetch');
    const apiKey = process.env.NEWSAPI_KEY; // Stored in Netlify
    
    const response = await fetch(
        `https://newsapi.org/v2/everything?q=stock+market&apiKey=${apiKey}`
    );
    const data = await response.json();
    
    return {
        statusCode: 200,
        body: JSON.stringify(data)
    };
}
```

Then in your frontend:

```javascript
async function fetchMarketNews() {
    const response = await fetch('/.netlify/functions/fetch-news');
    const data = await response.json();
    // Process data...
}
```

---

## 📝 Where to Add This Code

In your `market-news-pwa.html` file:

1. Find the `fetchMarketNews()` function (around line 700)
2. Replace it with your chosen API implementation
3. Add any helper functions above it
4. Save and re-upload to your hosting

---

## 🧪 Testing

After integration:

1. Open browser console (F12)
2. Click "Fetch Latest News"
3. Check for any errors
4. Verify news cards appear with real data

---

## 🆘 Need Help?

If you need help with:
- Choosing the right API
- Setting up API keys
- Debugging errors
- Setting up a backend proxy

Just ask me! I can guide you through each step.
