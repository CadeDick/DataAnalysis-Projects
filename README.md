# Tech Layoffs & AI Impact Analysis
### A Business Analyst Portfolio Project

**Dataset:** 12,000 records across 7 tech industries | **Tools:** SQL · AI · Data Visualization  
**Role target:** Business Analyst | **Topics:** Workforce trends, AI adoption, financial impact

---

## Project Overview

This project investigates three questions that dominated tech industry headlines between 2022 and 2024:

1. Are certain industries laying off workers at significantly higher rates — and is AI adoption driving it?
2. Do companies that cut headcount actually see their stock prices improve?
3. What does the data actually say vs. what the media narrative claims?

Using a dataset of 12,000 company-level records spanning industry, layoff counts, AI metrics, financial performance, and market conditions, I applied SQL aggregation and data analysis to surface findings that challenge common assumptions.

---

## Dataset

| Field | Description |
|---|---|
| `industry` | Tech sub-sector (AI, Cloud, FinTech, Gaming, Cybersecurity, E-Commerce, Social Media) |
| `layoff_percentag` | Share of workforce laid off per event |
| `ai_adoption_leve` | Company AI adoption score (0–10 scale) |
| `ai_automation_imp` | Estimated automation impact score |
| `ai_replacement_r` | AI job replacement risk score |
| `stock_growth_per` | Stock price change associated with the period |
| `revenue_growth_p` | Revenue change for the same period |
| `market_condition` | Bull Market / Stable / Recession |
| `reason_for_layof` | Stated reason (AI Automation, Overhiring, Restructuring, Cost Cutting, Market Slowdown) |
| `hiring_trend` | Direction of hiring activity |
| `remote_jobs_perc` | Share of open roles that are remote |

---

## Finding 1 — Industry layoff rates are nearly identical, and AI adoption doesn't explain them

### What I expected
Industries investing heavily in AI (like the "AI Sector" or Cloud) would show higher layoff rates as automation replaced roles.

### What the data shows

| Industry | Avg Layoff % | Avg AI Adoption Score |
|---|---|---|
| Social Media | 13.16% | 5.48 |
| E-Commerce | 12.97% | 5.52 |
| Cybersecurity | 12.89% | 5.48 |
| Cloud | 12.80% | 5.61 |
| AI Sector | 12.76% | 5.59 |
| FinTech | 12.48% | 5.52 |
| Gaming | 12.39% | 5.57 |

The spread between the highest and lowest layoff industry is **less than 0.8 percentage points**. Every industry clusters around the same AI adoption score (~5.5/10). There is no meaningful correlation between how AI-forward a company is and how many people it laid off.

### The "AI narrative" finding

Layoff reasons were distributed almost perfectly evenly across five categories:

| Reason | Count | Share |
|---|---|---|
| AI Automation | 2,435 | 20.3% |
| Overhiring Correction | 2,433 | 20.3% |
| Restructuring | 2,401 | 20.0% |
| Cost Cutting | 2,397 | 20.0% |
| Market Slowdown | 2,334 | 19.4% |

If AI automation were a genuine structural driver of layoffs, its share would be significantly higher than the other reasons. The even distribution suggests companies may be citing AI as a convenient narrative rather than a root cause.

### SQL used

```sql
-- Industry layoff rates ranked
SELECT
  industry,
  ROUND(AVG(layoff_percentag), 2)   AS avg_layoff_pct,
  ROUND(AVG(ai_adoption_leve), 2)   AS avg_ai_adoption,
  COUNT(*)                           AS records
FROM tech_layoffs_hiring_trends_elite_v2
GROUP BY industry
ORDER BY avg_layoff_pct DESC;

-- Layoff reason distribution
SELECT
  reason_for_layof,
  COUNT(*)                                              AS total,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1)   AS pct_of_total
FROM tech_layoffs_hiring_trends_elite_v2
GROUP BY reason_for_layoffs
ORDER BY total DESC;
```

**Key SQL concepts:** `GROUP BY`, `AVG()`, `COUNT()`, `ORDER BY DESC`, window function `OVER()`

---

## Finding 2 — Cutting headcount does not reliably boost stock prices

### What I expected
Companies that announced large layoffs would show higher stock growth, consistent with the market's typical short-term positive reaction to "cost discipline" announcements.

### What the data shows

| Layoff severity | Avg stock growth | % with positive stock |
|---|---|---|
| No layoffs (0%) | +1.8% | 40% |
| Small (1–5%) | +23.0% | 67% |
| Medium (6–15%) | +22.4% | 66% |
| Large (16–25%) | +22.4% | 66% |
| Severe (26%+) | +22.3% | 67% |

Stock growth is **essentially flat** across all layoff severity buckets. Companies cutting more than a quarter of their workforce see the same stock performance as companies making modest cuts.

### The market condition smoking gun

The most revealing cut is by market condition:

| Market condition | Avg layoff % | Avg stock growth |
|---|---|---|
| Bull Market | 5.1% | +22.4% |
| Stable | 9.6% | +22.5% |
| Recession | 24.1% | +22.5% |

During a recession, companies laid off nearly **5× more workers** than during a bull market — yet stock growth was statistically identical across all three conditions. This strongly suggests that **macroeconomic conditions drive stock prices**, not headcount decisions.

### Business implication

The conventional wisdom that "layoffs signal discipline and boost shareholder value" is not supported by this dataset. Companies in all market conditions averaged ~22.5% stock growth regardless of how aggressively they cut staff. This has implications for how layoff announcements should be interpreted by analysts and investors.

### SQL used

```sql
-- Bucket layoff severity and compare stock growth
SELECT
  CASE
    WHEN layoff_percentag = 0        THEN 'No layoffs (0%)'
    WHEN layoff_percentag <= 5       THEN 'Small (1-5%)'
    WHEN layoff_percentag <= 15      THEN 'Medium (6-15%)'
    WHEN layoff_percentag <= 25      THEN 'Large (16-25%)'
    ELSE                                   'Severe (26%+)'
  END                                 AS layoff_bucket,
  ROUND(AVG(stock_growth_per), 2) AS avg_stock_growth,
  ROUND(AVG(layoff_percentag), 2)    AS avg_layoff_pct,
  COUNT(*)                            AS records
FROM tech_layoffs_hiring_trends_elite_v2
GROUP BY layoff_bucket
ORDER BY avg_layoff_pct ASC;

-- Market condition breakdown: the key comparison
SELECT
  market_condition,
  ROUND(AVG(layoff_percentag), 2)      AS avg_layoff_pct,
  ROUND(AVG(stock_growth_per), 2)   AS avg_stock_growth,
  ROUND(AVG(revenue_growth_per), 2) AS avg_rev_growth,
  COUNT(*)                              AS records
FROM tech_layoffs_hiring_trends_elite_v2
GROUP BY market_condition
ORDER BY avg_layoff_pct ASC;
```

**Key SQL concepts:** `CASE WHEN / THEN / ELSE / END`, multiple `AVG()` in one query, `GROUP BY` on a text column

---

## Key Takeaways

| # | Finding | Business implication |
|---|---|---|
| 1 | Industry layoff rates vary by less than 1pp | No single sector is uniquely exposed; workforce risk is distributed evenly across tech |
| 2 | AI adoption score has no correlation with layoff rate | AI investment and headcount reduction are independent decisions |
| 3 | AI automation cited in only 20% of layoffs — same as 4 other reasons | Companies may be using AI as post-hoc justification rather than a real driver |
| 4 | Severe layoffs produce the same stock growth as minor ones | Cutting headcount is not a reliable lever for improving shareholder value |
| 5 | Stock growth is identical across bull, stable, and recession markets | Macro conditions dominate stock performance; layoff decisions are secondary |

---

## SQL Skills Demonstrated

| Concept | Where used |
|---|---|
| `GROUP BY` + `AVG()` | All analyses — core aggregation pattern |
| `ORDER BY DESC / ASC` | Ranking industries, sorting results |
| `COUNT(*)` | Record counts alongside every aggregation |
| `ROUND()` nested in `AVG()` | Clean numeric output |
| `CASE WHEN` | Creating layoff severity buckets from raw numbers |
| Window function `OVER()` | Calculating % share of total within a result set |
| Multiple aggregates in one query | Comparing layoff %, stock growth, revenue growth side by side |
| `WHERE` for pre-filter | Excluding zero-layoff outliers before grouping |

---

## Methodology Notes

- Records with `layoff_percentag = 0` represent a very small sample (n=20) and were treated as an edge case rather than a meaningful group in the stock analysis.
- All averages are unweighted — each company-event record counts equally regardless of company size.
- Correlation analysis was visual (bucket comparison) rather than statistical (Pearson r). A follow-on analysis could compute formal correlation coefficients using `CORR()` in PostgreSQL or equivalent.
- Dataset covers multiple years; temporal trends (month/year analysis) are reserved for a follow-on analysis of hiring trends.

---

## About This Project

Built as part of a self-directed business analyst portfolio. Analysis performed in SQL. Dataset: `tech_layoffs_hiring_trends_elite_v2.csv` (12,000 rows, 23 columns).

*Interested in collaborating or have feedback? Open an issue or connect on LinkedIn.*
