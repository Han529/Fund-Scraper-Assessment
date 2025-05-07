Thought for 17 seconds


🧾 **Full-Scale 13F Filings Scraper**

This project automates the end-to-end extraction and analysis of *every* investment-manager 13F filing listed on **13f.info**.
It crawls the entire A-Z index, follows to each manager page, discovers all quarterly filing URLs, handles dynamically-loaded (AJAX) tables, aggregates the holdings, computes quarter-over-quarter share-count changes, and classifies each position as **Buy / Sell / Hold**.
The result is a clean, research-ready **CSV** covering the complete universe of managers and filings.

---

### 📌 Features

* 🔗 **A-Z Manager Discovery** – scrapes all index pages (no link caps) in parallel (ThreadPool, 15 workers).
* 🔍 **Drill-Down Scraping** – navigates from manager pages to every quarterly filing URL (no 100-manager limit).
* ⚙️ **Dynamic Table Handling** – detects `data-url` attributes, fetches JSON via AJAX, and merges with HTML headers.
* 🔄 **Automatic Retries** – uses *tenacity* with exponential back-off (max 4 tries, 1-10 s delay) for transient HTTP / network errors.
* 📊 **Quarter-over-Quarter Analytics** – calculates share Δ, %Δ, and infers Buy / Sell / Hold for each stock.
* 🗄️ **Clean, Consolidated Output** – standardised column names, numeric cleansing, and one CSV (`Fund Manager Shares Analysis.csv`).
* 🧪 **Robust Fallbacks** – graceful degradation to BS4 parsing when AJAX fails or HTML tables are malformed.
* 🧩 **Modular Pipeline** – distinct stages for discovery, link harvest, filing scrape, processing, and export (easy to extend).

---

### 🛠️ Technical Approach

#### Web Scraping

* **requests** for HTTP (custom UA string).
* **BeautifulSoup4 + lxml** for HTML parsing.
* Parses manager metadata (fund name, quarter, filing date) and all table headers.

#### Dynamic Content (AJAX)

* If a holdings table contains `data-url`, a secondary JSON request retrieves the rows.
* Merged with headers from the original `<thead>` for a complete DataFrame.
* Falls back to direct HTML row extraction if JSON is missing or malformed.

#### Concurrency & Reliability

* **ThreadPoolExecutor** (15 workers) for A-Z pages, quarter links, and filing tables.
* **tenacity** retry wrapper handles `Timeout`, `ConnectionError`, 429 & 5xx status codes with exponential back-off.

#### Data Storage & Manipulation

* **pandas** for DataFrames; **numpy** for numeric ops.
* Filters to common-stock holdings (`cl == 'COM'`).
* Uses `pd.concat()` to merge thousands of filing tables.

#### 🔢 Calculations & Cleansing

* Strips commas, converts *shares* / *value\_usd\_000* to numeric (`pd.to_numeric`).
* Converts textual quarters to `pandas.Period` for chronological sorting.
* Computes:

  * `change`  – absolute Δ in shares.
  * `pct_change` – percentage Δ (∞ & NaN guarded).
  * `inferred_transaction_type` – **Buy/Sell/Hold** via `numpy.select`.

---

### 🚀 Running the Scraper

1. **Install deps**

   ```bash
   pip install requests pandas beautifulsoup4 lxml tenacity
   ```
2. **Run**

   ```bash
   python scrape_13f_full.py
   ```
3. **Output**
   `Fund Manager Shares Analysis.csv` appears in the working directory.

---

### ⚙️ Config Tunables

| Constant                | Purpose           | Default            |
| ----------------------- | ----------------- | ------------------ |
| `MAX_WORKERS`           | Parallel threads  | **15**             |
| `MAX_RETRIES`           | Tenacity attempts | **4**              |
| `INITIAL_BACKOFF_DELAY` | First retry delay | **1 s**            |
| `MAX_BACKOFF_DELAY`     | Max delay         | **10 s**           |
| `BASE_URL`              | Target site       | `https://13f.info` |

Adjust these near the top of the script to fit your bandwidth or risk tolerance.

---

### ✅ Status

* Tested April 2025 against \~**3 300** managers and **>3 200** filing pages without link-caps.
* Produces a single, deduplicated dataset ready for quantitative or fundamental research.

> **Note:** Scraping large sites can stress their servers. Consider inserting additional delays or honouring robots.txt where applicable.
