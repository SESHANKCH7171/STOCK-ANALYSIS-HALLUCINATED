# STOCK-ANALYSIS-HALLUCINATED

> **A cautionary demo showing exactly where and how LLM-powered "analysis" goes wrong — producing confident, plausible-sounding, but entirely fabricated stock insights.**

---

## What This Project Claims To Do

This notebook uses **CrewAI multi-agent framework** to analyze three Indian defense stocks:
- `HAL.NS` — Hindustan Aeronautics Limited
- `BEL.NS` — Bharat Electronics Limited
- `BDL.NS` — Bharat Dynamics Limited

It defines specialized AI agents (Stock Data Analyst, Technical Analyst, Fundamental Analyst, Sentiment Analyst, Risk Analyst, Report Writer) that supposedly gather real-time data, compute technical indicators, and generate an investment recommendation report.

---

## Where It Goes Wrong (The Hallucinations)

### 1. No Real Stock Data Is Ever Fetched

**What it looks like:** The agents have titles like "Stock Data Analyst" and mention RSI, MACD, moving averages, volume, P/E ratios.

**What actually happens:** There is **zero call** to any stock data API (`yfinance`, `NSEpy`, Alpha Vantage, etc.). No dataframe is created. No OHLCV data is downloaded. The LLM simply *invents* all numbers from memory.

```python
# There is NO code like this anywhere in the notebook:
import yfinance as yf
data = yf.download('HAL.NS', period='6mo')
```

> The agents hallucinate specific figures like "RSI at 62.4", "200-day MA crossover confirmed", "volume surge of 34%" — none of which are computed from real data.

---

### 2. Technical Indicators Are Fabricated, Not Calculated

**What it looks like:** The report confidently states specific RSI values, MACD signals, Bollinger Band positions, and moving average crossovers.

**What actually happens:** Libraries like `pandas-ta`, `TA-Lib`, or `scipy` are never imported or used. There is no price series to compute on. The LLM generates numbers that *sound* like technical analysis outputs but are pure fiction.

**Why this is dangerous:** A reader without domain knowledge would have no way to tell these numbers are made up. They look exactly like real analysis.

---

### 3. The Serper Search Tool Is Misused

**What it looks like:** A `SerperDevTool` is configured, implying the agents browse real news and filings.

**What actually happens:** Even when Serper returns real headlines/snippets, the agents **ignore the actual content** and instead generate their own fabricated geopolitical narratives. The search results serve as a veneer of legitimacy, not as actual grounding.

> Example: Serper might return a news headline about a HAL contract. The agent then invents detailed order-book figures, delivery timelines, and margin impacts that were never in the article.

---

### 4. Fundamental Analysis Numbers Are Invented

**What it looks like:** The Fundamental Analyst agent produces P/E ratios, revenue growth percentages, debt-to-equity ratios, and EPS figures.

**What actually happens:** No financial statements are parsed. No APIs like Screener.in, Tickertape, or NSE filings are called. The LLM recalls approximate values from its training data — which may be months or years out of date — and presents them as current analysis.

---

### 5. No Validation, No Backtesting, No Sanity Checks

There is:
- No assertion that checks if fetched prices are within a realistic range
- No comparison of LLM-stated values against any ground truth
- No backtesting of the recommended strategy
- No confidence intervals or uncertainty quantification
- No disclaimer in the output that the analysis is LLM-generated

The final report reads as if produced by a human analyst who ran real computations.

---

### 6. Circular Reasoning Between Agents

The CrewAI task chain passes outputs from one agent as context to the next. Since the first agent's output is already hallucinated, every subsequent agent builds on fabricated data — amplifying the errors rather than correcting them.

```
Stock Data Agent (invents data)
        ↓
Technical Analyst (invents indicators based on invented data)
        ↓
Fundamental Analyst (invents ratios based on invented data)
        ↓
Report Writer (writes confident report based on all hallucinations)
```

Each handoff adds a layer of false confidence.

---

## Why The Output *Looks* Convincing

| Feature | Looks Real | Actually Is |
|---|---|---|
| Specific RSI/MACD values | ✅ Precise numbers given | ❌ LLM guesses |
| News-based sentiment | ✅ Serper tool used | ❌ Summaries fabricated |
| Defense sector narrative | ✅ Geopolitically detailed | ❌ No source cited |
| Buy/Hold/Sell recommendation | ✅ Clear conclusion | ❌ Based on nothing real |
| Agent specialization | ✅ Named roles | ❌ All same underlying LLM |

---

## What A Correct Implementation Would Look Like

To make this project actually work, you would need:

1. **Real data fetching:**
   ```python
   import yfinance as yf
   df = yf.download('HAL.NS', period='6mo', interval='1d')
   ```

2. **Computed technical indicators:**
   ```python
   import pandas_ta as ta
   df['RSI'] = ta.rsi(df['Close'], length=14)
   df['MACD'] = ta.macd(df['Close'])['MACD_12_26_9']
   ```

3. **Grounded agent context** — pass the actual computed dataframe or its summary as part of the agent's task input, not just a stock ticker string.

4. **Output validation** — compare LLM-stated figures against computed figures and flag discrepancies.

5. **Explicit uncertainty** — the final report should state which values are LLM-inferred vs. computed from real data.

---

## Key Takeaway

> **Giving an LLM a stock ticker and asking it to "analyze" it does not constitute analysis. Without grounding agents in real, freshly-fetched, computed data, the entire pipeline is an elaborate hallucination machine that produces dangerous misinformation at scale.**

This repo exists to make that failure mode visible.

---

## Tech Stack (As Used)

- `crewai` — Multi-agent orchestration
- `crewai_tools` — SerperDevTool for web search
- Google Gemini (via `GEMINI_API_KEY`) — LLM backbone
- Serper API (via `SERPER_API_KEY`) — Web search (misused)

## License

MIT — Use freely for educational purposes.
