# Market Pulse PWA - Installation & Setup Guide

## 📱 What You've Got

A Progressive Web App (PWA) that provides automated market news alerts for:
- **Nifty 50 & Indian Markets**
- **US & European Markets**
- **Cryptocurrencies**
- **Gold, Crude Oil, Natural Gas**

### Notification Schedule:
1. **Pre-Market Briefings**: 30 minutes before market opens (India, US, Europe)
2. **Intraday Updates**: Every 15 minutes during market hours
3. **Crypto & Gold**: Every 30 minutes, 24/7

---

## 🚀 Installation Instructions

### For Android Phone:

1. **Upload Files to a Web Server**
   - You need to host these files on a web server (see hosting options below)
   - All 4 files must be in the same directory:
     - `market-news-pwa.html`
     - `manifest.json`
     - `service-worker.js`
     - `icon.svg`

2. **Open in Chrome Browser**
   - Open Chrome on your Android phone
   - Navigate to: `https://your-domain.com/market-news-pwa.html`
   - You should see a banner saying "Install Market Pulse"

3. **Install the App**
   - Tap the "Install" button in the banner
   - Or tap the menu (⋮) → "Install app" → "Install"
   - The app will be added to your home screen

4. **Enable Notifications**
   - Open the installed app
   - Tap "Enable Notifications" button
   - Allow notifications when prompted
   - Done! You'll now receive automated alerts

### For Windows Laptop:

1. **Same Web Server Requirement**
   - Host the files on a web server (same as above)

2. **Open in Chrome or Edge**
   - Open Chrome or Microsoft Edge browser
   - Navigate to: `https://your-domain.com/market-news-pwa.html`

3. **Install the App**
   - Click the install icon (⊕) in the address bar
   - Or click menu (⋮) → "Install Market Pulse"
   - The app will open in its own window

4. **Enable Notifications**
   - Click "Enable Notifications" button
   - Allow notifications when prompted
   - Done!

---

## 🌐 Hosting Options (Choose ONE)

Since you're not a software engineer, here are the EASIEST hosting options:

### Option 1: GitHub Pages (FREE & RECOMMENDED)

**Steps:**
1. Create a free GitHub account at https://github.com
2. Create a new repository named "market-pulse"
3. Upload all 4 files to the repository
4. Go to Settings → Pages → Enable GitHub Pages
5. Your app URL will be: `https://YOUR-USERNAME.github.io/market-pulse/market-news-pwa.html`

**Video Tutorial**: Search YouTube for "how to host on GitHub Pages"

### Option 2: Netlify (FREE)

**Steps:**
1. Go to https://www.netlify.com
2. Sign up for free account
3. Drag and drop all 4 files into Netlify
4. Get instant URL like: `https://market-pulse-123.netlify.app`

### Option 3: Vercel (FREE)

**Steps:**
1. Go to https://vercel.com
2. Sign up for free account
3. Import your files or connect GitHub
4. Get instant URL

### Option 4: Google Drive (EASIEST but has limitations)

**NOT RECOMMENDED** - Google Drive hosting doesn't support service workers properly for PWA features.

---

## ⚙️ Customization

### To Connect Real Market Data:

The app currently shows **DEMO/MOCK data**. To get real market news:

1. **Sign up for News APIs** (choose one or more):
   - **Alpha Vantage** (Free tier): https://www.alphavantage.co
   - **NewsAPI** (Free tier): https://newsapi.org
   - **Finnhub** (Free tier): https://finnhub.io
   - **CoinGecko** (Free for crypto): https://www.coingecko.com/api

2. **Find the `fetchMarketNews()` function** in the HTML file (around line 700)

3. **Replace the mock data** with real API calls:

```javascript
async function fetchMarketNews() {
    // Example with NewsAPI
    const apiKey = 'YOUR_API_KEY_HERE';
    const response = await fetch(
        `https://newsapi.org/v2/everything?q=stock+market+OR+nifty+OR+crypto&apiKey=${apiKey}`
    );
    const data = await response.json();
    
    // Transform the data to match our format
    return data.articles.map(article => ({
        category: 'market', // or 'crypto', 'commodity'
        title: article.title,
        content: article.description,
        source: article.source.name,
        time: new Date(article.publishedAt).toLocaleTimeString()
    }));
}
```

### To Adjust Notification Times:

Find these sections in the HTML file:

```javascript
// Line ~900: Pre-market timing (currently 30 mins before)
// Change the hours and minutes values

// Line ~925: Intraday updates (currently 15 mins)
// Change 900000 to different milliseconds:
// 5 mins = 300000
// 10 mins = 600000
// 15 mins = 900000
// 30 mins = 1800000

// Line ~945: Crypto/Gold (currently 30 mins)
// Change 1800000 similarly
```

---

## 🔧 Troubleshooting

### Notifications Not Working?

1. **Check Browser Permissions**:
   - Android: Settings → Apps → Chrome → Notifications → Allowed
   - Windows: Windows Settings → Notifications → Chrome → On

2. **Check App Permissions**:
   - Open the app
   - Settings → Site Permissions → Notifications → Allow

3. **Ensure Service Worker is Active**:
   - Open browser console (F12)
   - Look for "Service Worker registered" message

### App Not Installing?

1. **HTTPS Required**: Your site MUST use HTTPS (not HTTP)
   - GitHub Pages, Netlify, Vercel provide HTTPS automatically
   
2. **All Files Present**: Make sure all 4 files are in the same directory

3. **Clear Cache**: 
   - Chrome: Settings → Privacy → Clear browsing data
   - Try again

### Icons Not Showing?

You need to convert the SVG to PNG:
1. Open `icon.svg` in a browser
2. Use an online tool like "SVG to PNG converter"
3. Create two versions:
   - `icon-192x192.png` (192x192 pixels)
   - `icon-512x512.png` (512x512 pixels)
4. Upload these PNG files to your server

---

## 📊 Features Overview

### Control Panel:
- **Enable Notifications**: One-click to start receiving alerts
- **Test Notification**: Preview how alerts will look
- **Fetch Latest News**: Manually refresh news feed
- **Clear Cache**: Reset app data

### Settings:
- ✅ **Pre-Market Alerts**: Toggle on/off
- ✅ **Intraday Updates**: Toggle on/off
- ✅ **Crypto & Gold Updates**: Toggle on/off
- ✅ **Sound Alerts**: Enable/disable 2-second notification chime

**Note**: The app plays a pleasant 2-second chime sound with each flash card notification. You can disable this in the settings panel if you prefer silent notifications.

### Market Status:
- **Live indicators** showing which markets are currently open
- Updates every minute

---

## 🎯 Next Steps

1. **Choose a hosting platform** (GitHub Pages recommended)
2. **Upload your files**
3. **Get your URL**
4. **Install on phone and laptop**
5. **Enable notifications**
6. **Customize with real APIs** (optional)

---

## 💡 Tips

- **Keep the app installed**: Don't uninstall it or notifications will stop
- **Allow background activity**: Don't force-close the browser
- **Check battery settings**: Ensure Chrome can run in background
- **Test regularly**: Use "Test Notification" to ensure it's working

---

## 🆘 Need Help?

If you need assistance with:
- Setting up hosting
- Connecting real APIs
- Customizing notification times
- Any technical issues

Just ask me, and I'll guide you through it step by step!

---

## 📝 Important Notes

1. **Battery Usage**: Frequent notifications may impact battery life
2. **Data Usage**: The app will use mobile data to fetch news
3. **Reliability**: Depends on your internet connection
4. **Market Hours**: Automatically detects when markets are open/closed

Your Market Pulse PWA is ready to deploy! 🚀
