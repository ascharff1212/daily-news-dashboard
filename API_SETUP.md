# API Setup Guide

How to wire live data into Alan's Daily News Dashboard. Demo mode works out of the box — this guide is for upgrading to Hybrid or Live mode.

## TL;DR

1. Sign up for the APIs you want (free tiers exist for all of them).
2. Paste your keys into `CONFIG.apiKeys` near the top of `dashboard.html`.
3. Switch the mode pill (top banner) from **Demo** to **Hybrid** (recommended) or **Live**.
4. Implement the live fetch in the relevant `get*()` functions (current code returns demo data; example code below).

## Recommended APIs and what each is for

| API | Free tier? | Use for | Sign up |
|---|---|---|---|
| **NewsAPI** | Yes, 100/day | General U.S. headlines, business headlines, Colorado regional news | [newsapi.org](https://newsapi.org) |
| **GNews** | Yes, 100/day | Alternative general news source; better for non-English / regional | [gnews.io](https://gnews.io) |
| **Marketaux** | Yes, 100/day | Business news with sentiment tagging built in | [marketaux.com](https://www.marketaux.com) |
| **Finnhub** | Yes, generous | Real-time quotes, company news, earnings calendar, recommendations | [finnhub.io](https://finnhub.io) |
| **Financial Modeling Prep (FMP)** | Yes, 250/day | Economic calendar, earnings calendar, fundamentals, analyst estimates | [financialmodelingprep.com](https://financialmodelingprep.com) |
| **Alpha Vantage** | Yes, 25/day | Market data, technical indicators, fundamentals | [alphavantage.co](https://www.alphavantage.co) |
| **Polygon.io** | Yes, 5/min | Real-time and historical market data, ticker news | [polygon.io](https://polygon.io) |
| **Trading Economics** | Yes, limited | Global economic calendar, indicators, central bank events | [tradingeconomics.com](https://tradingeconomics.com) |
| **Benzinga** | Paid | Deal news, earnings, analyst rating changes | [benzinga.com](https://www.benzinga.com) |

**Minimum recommended stack** for a fully-live dashboard:
- NewsAPI (headlines, Colorado, national)
- Finnhub (market quotes, company news, earnings)
- Financial Modeling Prep (economic calendar)
- Marketaux (business news with sentiment)

## Where to put the keys

Open `dashboard.html` and find this block near the top of the `<script>` section:

```js
const CONFIG = {
  mode: 'demo',
  // ...
  apiKeys: {
    newsapi:         '',
    gnews:           '',
    finnhub:         '',
    fmp:             '',
    alphavantage:    '',
    polygon:         '',
    marketaux:       '',
    tradingeconomics:'',
    benzinga:        '',
  },
  // ...
};
```

Paste your keys between the quotes. Save and reload.

## Wiring a section to live data — example

Each section has a `get*()` function that currently returns demo data:

```js
async function getHeadlines() { return DEMO.headlines; }
```

Replace it with a live fetch:

```js
async function getHeadlines() {
  const key = CONFIG.apiKeys.newsapi;
  if (!key || STATE.mode === 'demo') return DEMO.headlines;

  try {
    const url = CONFIG.endpoints.headlinesUS(key);
    const r = await fetch(url);
    const data = await r.json();
    return data.articles.map((a, i) => ({
      id: 'h' + i,
      title: a.title,
      src: a.source.name,
      cat: 'Markets',
      published: timeAgo(a.publishedAt),
      summary: a.description || '',
      why: '',                          // generate separately or leave blank
      sentiment: 'neutral',             // run through a sentiment model or skip
      urgency: 5,                       // default mid-tier; tune by source/recency
      marketSensitive: /market|fed|inflation|earnings/i.test(a.title),
      sector: null,
      breaking: false,
      url: a.url,
    }));
  } catch (e) {
    console.warn('NewsAPI fetch failed; falling back to demo', e);
    return STATE.mode === 'hybrid' ? DEMO.headlines : [];
  }
}
```

The same pattern applies to every `get*()` function. The dashboard already routes:
- Hybrid mode → live with demo fallback
- Live mode → live only, empty on failure
- Demo mode → always demo

## CORS and the "frontend key" problem

Two issues come up when you call third-party APIs directly from `dashboard.html`:

**1. CORS.** Some APIs (notably NewsAPI on free plans) block browser requests. Workarounds:
- Use APIs that explicitly support browser CORS (Finnhub, Marketaux, Polygon do).
- Open the file via a local web server (`python -m http.server`) rather than `file://`, which helps with some CORS edge cases.
- Use a proxy (see next section).

**2. Key exposure.** Any key in `dashboard.html` is visible to anyone who views the page source. Acceptable for:
- Local-only use on your own machine
- Private deployment behind authentication
- APIs where the key has tight per-domain or per-IP scoping

**Not acceptable for** a publicly hosted page on the open internet — you'll burn through quota or worse.

## Production setup: backend proxy

For a publicly hosted dashboard with live data, proxy the API calls. Minimal Node/Express example:

```js
// server.js
const express = require('express');
const app = express();
require('dotenv').config();

app.get('/api/headlines', async (req, res) => {
  const r = await fetch(`https://newsapi.org/v2/top-headlines?country=us&apiKey=${process.env.NEWSAPI_KEY}`);
  res.json(await r.json());
});

app.use(express.static('.'));
app.listen(3000);
```

In `dashboard.html`, change the endpoint:

```js
endpoints: {
  headlinesUS: () => '/api/headlines',
  // ...
}
```

Now keys live in `.env` (server-side) and never reach the browser.

### Serverless alternatives

Even simpler for low-volume:

- **Vercel Functions** — Drop `api/headlines.js` in the project; Vercel auto-routes it.
- **Cloudflare Workers** — Free 100K req/day; great latency.
- **Netlify Functions** — Similar to Vercel.

Each lets you store keys as environment variables in their dashboard.

## Auto-refresh and rate limits

The sidebar auto-refresh control (Off / 5m / 10m / 30m) will hit your APIs every cycle. For free tiers:

| API | Free tier | Safe interval |
|---|---|---|
| NewsAPI | 100/day | 30m+ (under typical section count) |
| Finnhub | 60/min | 5m (with the watchlist) is fine |
| FMP | 250/day | 30m+ |
| Alpha Vantage | 25/day | Manual refresh only |
| Polygon | 5/min | 5m+ |

Recommend **10m or 30m** for daily personal use. Override the intervals by editing the `data-ref` values on the auto-refresh buttons in the HTML if you want different presets.

## Sentiment generation (optional)

The demo data has sentiment pre-labeled. For live data, you have three options:

1. **Use a sentiment-labeled API** — Marketaux returns sentiment scores natively. Easiest path.
2. **Keyword heuristic** — Simple rules ("beat" → positive, "miss" → negative, "warn" → negative, etc.). Works surprisingly well for headlines.
3. **LLM sentiment** — Send the headline + summary to a cheap LLM (Haiku, GPT-4o-mini) with a prompt like: *"Classify the market sentiment of this headline as one of: positive, negative, neutral, mixed, market-sensitive. Return only the label."* Costs fractions of a cent per call.

## "Why it matters" generation (optional)

Same three options:
1. **Pre-written** — fastest but doesn't scale to live news.
2. **Template-based** — generate from category + sector + ticker + sentiment.
3. **LLM** — best quality. Prompt: *"In one sentence, explain why this headline matters for a public-equity investor. Be specific. Cite affected sectors or tickers when relevant."*

The dashboard's `buildCard()` function already shows the `why` field — just populate it in your `get*()` fetch implementations.

## GitHub Pages deployment

If you want this hosted on GitHub Pages:

```bash
cd "Alan's Daily News Dashboard"
git init
git add dashboard.html README.md API_SETUP.md
git commit -m "Initial dashboard"
git remote add origin https://github.com/YOUR-USERNAME/daily-news-dashboard.git
git push -u origin main
```

Then in the repo's Settings → Pages, set source to `main` / `(root)`. Your dashboard will be live at `https://YOUR-USERNAME.github.io/daily-news-dashboard/dashboard.html`.

**For live APIs on GitHub Pages**, you'll need a backend proxy (GitHub Pages is static-only). Cloudflare Workers is the lightest option.

## Need a section the dashboard doesn't have?

Add a new section by:
1. Creating an HTML container in the content column (`<section class="block" id="my-section">…</section>`).
2. Adding a nav link (`<a href="#my-section">My Section</a>`).
3. Writing a `getMyData()` function.
4. Writing a `renderMySection()` function.
5. Calling it from `renderAll()`.

The existing sections are good templates to copy from.
