# Alan's Daily News Dashboard

A professional, single-file HTML dashboard delivering a daily news and business intelligence briefing for investors, financial advisors, and market-focused professionals. Premium financial-terminal aesthetic, modern news-app usability.

## What's in the box

| File | Purpose |
|---|---|
| `dashboard.html` | The dashboard. Open in any modern browser. All HTML, CSS, and JavaScript in one file. |
| `API_SETUP.md` | How to connect live news and market data APIs (NewsAPI, Finnhub, Financial Modeling Prep, etc.) |
| `README.md` | This file. |

## Quick start

1. Double-click `dashboard.html` to open it in your default browser.
2. The dashboard loads in **Demo Mode** with realistic sample data so you can see every feature working immediately.
3. To enable live data, follow `API_SETUP.md`.

## Three operating modes

The mode toggle (top right of the banner) controls how data is sourced:

- **Demo** — Sample data only. Fastest path to a working dashboard. No keys needed.
- **Hybrid** *(recommended for daily use)* — Uses live APIs wherever keys are configured; falls back to demo data for missing sources.
- **Live** — APIs only. Missing keys leave sections empty.

## Dashboard sections

1. **Daily Morning Briefing** — Top 5 headlines, top 3 risks, top 3 opportunities, key econ report, key earnings, sector leadership, one-paragraph outlook, "what I'd watch today"
2. **Market Snapshot** — S&P/Nasdaq/Russell futures, UST yields, 2s10s, DXY, EUR/USD, USD/JPY, WTI, Gold, BTC, VIX
3. **Sector Heatmap** — All 11 S&P sectors color-coded by news-flow sentiment; click any tile to filter
4. **General Headlines** — Top market-relevant stories with summary, why-it-matters, sentiment, urgency, source
5. **Colorado News** — Denver metro, Boulder County, Longmont, Niwot; politics, economy, real estate, weather/wildfire, local business, transportation
6. **National News** — U.S. politics, federal policy, regulation, Fed, tax/budget, consumer confidence
7. **Business: Economy & Macro** — Inflation, jobs, GDP, retail sales, credit, FX, commodities
8. **Upcoming Economic Reports** — Date, time, consensus, prior, importance, why-it-matters, affected sectors, watchlist linkage
9. **S&P 500 Earnings** — Reporting today/tomorrow/this week, estimate revisions, sector-flagged, watchlist-flagged
10. **Deals, M&A, IPOs & Corporate Actions** — Deal flow with values, premiums, watchlist linkage, market reaction
11. **S&P 500 Sector News** — All 11 sectors with sentiment, drivers, earnings trend, rate/inflation/valuation sensitivity
12. **Watchlist** — Pre-populated with your full ticker list, auto-categorized into 8 groups. Watchlist Alerts panel ranks news by direct impact using the 30/20/15/15/10/10 model. LocalStorage persistence.
13. **Possible Market Opportunity** — Research-oriented ideas (never buy/sell recommendations) with bull/bear case, key risks, time horizon, confidence level, and a specific next research step. Shows a quiet message when nothing rises to the threshold.

## Features

- Auto-refresh (off / 5m / 10m / 30m)
- Manual refresh button
- Last-updated timestamp
- Source attribution on every item
- Global search across all sections
- Filters: sentiment, sector, minimum urgency, watchlist-only, favorites-only
- Dark/light mode toggle (persists across sessions)
- Mobile-responsive layout (single column under 1100px)
- Save favorites (★)
- Export to PDF (print to PDF — works in every browser)
- Copy briefing to clipboard
- Collapse/expand any section, or all at once
- Breaking-news and market-moving badges
- Scrolling deal-news ticker strip at top
- Watchlist add/remove with auto-categorization
- Click any sector tile to filter the feed

## Watchlist

The watchlist starts pre-populated with your full set:

- **Individual Stocks** (52): AAPL, AMZN, ANET, APH, AVGO, BAC, BE, BK, BLK, BRK.B, C, CI, COST, CTVA, EMR, ETR, GE, GEV, GM, GOOGL, GS, IBM, ISRG, JCI, JPM, KMI, KR, LIN, LLY, LRCX, MAR, MCK, META, MRK, MS, MSFT, NFLX, NVDA, NVO, PLTR, PPL, RTX, RXST, SHEL, SPG, SYK, TJX, TMUS, VLO, WELL, WMB, WMT
- **Bond / Defined Maturity ETFs** (6): BSCQ, BSCR, BSCS, BSCT, BSCU, BSCV
- **Preferred / Income ETFs** (2): FPE, PGF
- **ETFs (Broad / Style / Factor)** (9): CEFZ, IDEV, IVE, IWD, IWF, IWL, MGC, RWK, VONG
- **Sector / Industry ETFs** (1): KRE
- **Mutual Funds** (45): APDKX, AQMIX, ARTKX, BDMIX, CSCZX, DREVX, EPGAX, EQPGX, FGTAX, FTHSX, FTXSX, GARIX, GENIX, GINDX, GONIX, GSFTX, GSIMX, GTSGX, HFMDX, HILIX, HULIX, JUESX, MADVX, MBXIX, MEIIX, MIOFX, NASDX, NEAGX, OAKMX, PEIYX, PIPAX, PIUHX, PMJPX, PRCOX, PRPFX, QAACX, QCENX, QLEIX, QLENX, QMNNX, SCIEX, SEMNX, SVFAX, TRIGX, VSMIX

**To add a ticker:** type in the "Add ticker" box and press Enter. Auto-classification rules:
- 5 letters ending in X → Mutual Funds
- BSCx pattern → Bond / Defined Maturity ETFs
- FPE/PGF → Preferred / Income ETFs
- Known broad/style ETFs → ETFs (Broad / Style / Factor)
- Sector ETFs → Sector / Industry ETFs
- Other → Individual Stocks (or Needs Manual Classification)

**To remove:** hover any ticker tile and click the × that appears.

**To reset:** click "Reset to default" in the Watchlist section header.

All watchlist changes persist in browser `localStorage`. Clearing browser data resets the watchlist to default.

## Ranking logic

**Business news** uses the 30/20/15/15/10/10 weighted model:
- 30% market impact
- 20% economic importance
- 15% earnings relevance
- 15% sector relevance
- 10% recency
- 10% source credibility

**Watchlist alerts** use a separate 30/20/15/15/10/10 model:
- 30% direct impact on a watchlist holding
- 20% market-moving potential
- 15% earnings or revenue impact
- 15% sector or asset-class relevance
- 10% recency
- 10% source credibility

**Urgency** is scored 1-10:
- 9-10: Major market-moving event
- 7-8: Important business / economic / policy development
- 5-6: Relevant but not urgent
- 3-4: Interesting but lower impact
- 1-2: Background noise

Sources are weighted by editorial credibility, with Bloomberg/WSJ/FT/Reuters/Fed releases at the top tier.

Edit the weights in `CONFIG.rankBusiness` and `CONFIG.rankWatchlist` at the top of `dashboard.html` to tune the rankings.

## Deployment options

### Option 1: Local file (simplest)

Just open `dashboard.html` directly. No server needed for demo mode. For live APIs from a local file, you may hit CORS — see API_SETUP for workarounds.

### Option 2: Static hosting (recommended)

Drop `dashboard.html` on any static host:

- **GitHub Pages** — push to a public repo, enable Pages, point to root.
- **Netlify** — drag and drop the folder into the Netlify dashboard.
- **Vercel** — `vercel deploy` from the folder.
- **Cloudflare Pages** — connect to a Git repo and deploy.

### Option 3: Private deployment with backend proxy (most secure)

For live mode with API keys you don't want exposed:
1. Add a small Node/Express, Cloudflare Worker, or Vercel/Netlify Function that proxies API calls.
2. Store keys server-side as environment variables.
3. Update `CONFIG.endpoints` in `dashboard.html` to point at your proxy.

See `API_SETUP.md` for full details.

## Customization

Everything is in `dashboard.html`. Key edit points:

- **Demo data:** Edit the `DEMO` object near the top of the `<script>` block.
- **Watchlist defaults:** Edit `DEFAULT_WATCHLIST`.
- **API keys:** Edit `CONFIG.apiKeys`.
- **Ranking weights:** Edit `CONFIG.rankBusiness` and `CONFIG.rankWatchlist`.
- **Theme colors:** Edit the `:root` and `html[data-theme="light"]` CSS variables at the top of the `<style>` block.
- **Categories / filters:** Edit the sidebar HTML in the `<aside class="sidebar">` block.

## Browser support

Tested on modern Chrome, Edge, Firefox, and Safari. Uses standard ES2020 features (no transpilation needed).

## Disclaimer

This dashboard is for **research and informational purposes only**. It is not personalized investment advice, a recommendation to buy or sell any security, or a solicitation of any kind. Sentiment, urgency scores, and "Why it matters" notes are model-generated heuristics. Always verify with the original source and consult a qualified financial advisor before acting on any information.
