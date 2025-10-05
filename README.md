# United Airlines Hackathon — Flight Difficulty & Ops Insights

> **Goal:** Predict which flights will be hardest to operate on time, explain *why*, and turn that into an operations-ready playbook.
> **Data:** Flight-level, PNR/customer-level, bag-level, and station-level CSVs provided by the challenge.

---

## 🔎 What this project delivers

* **Clean + de-dup** the 5 source datasets (robust keys, time parsing, PNR de-dup).
* Complete, slide-ready **EDA** for delays, tight turns, bag complexity, SSR impact, and loads.
* A transparent **Flight Difficulty Score (0–100)** per flight, **ranked daily** and **classified** (Difficult / Medium / Easy).
* **UA-styled charts** and a **1-page operational playbook** for quick adoption by stations.

---


## ⚙️ Setup

**Conda**

```bash
conda env create -f environment.yml
conda activate ua-hack
jupyter lab
```

**Pip**

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

**Run**

1. Place the 5 CSVs in `data/raw/`.
2. Open `notebooks/05_flight_difficulty_score.ipynb` → **Run All**.
3. Final CSVs are written to `outputs/`, charts to `reports/figures/`.

---

## 🧱 Key modeling choices & assumptions

* **Universal flight key**
  `flight_key = COMPANY | FLIGHT | YYYY-MM-DD | ORIGIN | DEST`

* **Time parsing**
  Robust ISO-8601 + local date handling (`pandas.to_datetime(..., utc=True, errors="coerce")`).

* **PNR de-dup (prevents double counting)**
  Collapse to one row per **(flight_key, record_locator)** using `max(total_pax)` and `max(lap_child_count)`;
  **Seated pax (infants removed)**: `seated_pax = max(total_pax - lap_child_count, 0)`.

* **Best seat denominator**
  `best_seats = max(total_seats, configured_seats, available_seats, seats)`.

* **Load factor**
  `LF = seated_pax / best_seats` (we keep >100% if present to reflect reality like oversells).

* **Bag Complexity Index (BCI)**
  `transfer_bags / checked_bags` (from counts, or derived from flags/categories when needed).

* **SSR intensity**
  Count distinct PNR-level SSRs and **assign to first segment only**;
  `ssr_per_100 = 100 * ssr_count / seated_pax`.

* **Turn buffer**
  `turn_buffer_min = scheduled_ground_time_minutes − minimum_turn_minutes`.

* **Bank congestion**
  Within a station-day, count departures in ±30 minutes; **peak flag** = top quartile for that station-day.

---

## 📊 EDA questions covered

1. **Average departure delay & % late** — overall, by **Carrier × Fleet**, and by **Segment (Mainline/Express)**.
2. **Tight turns** — below minimum / within +5; station/fleet exposures; buffer distribution.
3. **Bag complexity** — overall BCI, by station, and late-bag patterns.
4. **Loads vs difficulty** — scatter/heatmaps, decile summaries.
5. **SSR vs delay (controlling for load)** — stratified comparisons & smooth partial-effect visuals.

All plots are exported in UA-styled colors for slide-readiness.

---

## 🧮 Flight Difficulty Score (transparent & daily-relative)

We compute **daily percentile risks (0–1)** for each flight:

* `risk_turn`  — smaller turn buffer = higher risk
* `risk_load`  — higher load factor = higher risk
* `risk_bci`   — higher bag complexity = higher risk
* `risk_ssr`   — more SSRs per 100 pax = higher risk
* `risk_bank`  — heavier local bank congestion = higher risk
* `risk_peak`  — in top-quartile “push bank” = higher risk

**Score (weights auto-rescaled if any feature is missing):**

```
FDS = 100 × [ 0.30*risk_turn
            + 0.20*risk_load
            + 0.15*risk_bci
            + 0.15*risk_ssr
            + 0.12*risk_bank
            + 0.08*risk_peak ]
```

**Rank + Class (per dep_date):**

* **Ranking:** sort by FDS (highest = most difficult).
* **Classification:** tertiles → **Difficult / Medium / Easy**.

**Exports**

* `UA_Flight_Difficulty_Score_FINAL.csv` — full columns for analysts.
* `UA_FDS_Minimal_ORDERED_HighScoreIsDifficult.csv` — panel-friendly:
  `Carrier Code, Carrier Name, Carrier Segment, Fleet Type, Difficulty Class, Difficulty Score` *(plus optional factor values).*

---

## 🧭 Operational efficiency one-pager

A station-friendly playbook built from the score:

* **Daily Top-10 by station** (D-1 21:00 & D-0 04:30 refresh).
* **Tight turns:** pre-assign bin-runner, aisle-helper, SSR lead; stage carts/ULDs; keep push sequence.
* **High BCI:** T-30/T-15 bag runners; proactive knock-and-go for late bags.
* **High SSR:** stage lift + aisle-chair; jetbridge escort; pre-seat forward rows.
* **Peak banks:** avoid remote stands for Top-10; roll tugs early; preserve Top-10 order.

The PDF render lives in `reports/UA_Operational_Playbook.pdf`.

---

## 🔒 Limitations & next steps

* Finer **time-zone alignment** and **stand/gate distances** would improve bank/turn modeling.
* Direct **bag counts** (vs derived) would tighten BCI estimates.
* Add features such as **inbound delay**, **de-icing**, **MEL/deferrals**, **crew swaps**, **staffing**, **remote stand usage**.

---

## 🧪 Quick judge walkthrough (3 steps)

1. Open `notebooks/05_flight_difficulty_score.ipynb` → **Run All**.
2. Open `outputs/UA_FDS_Minimal_ORDERED_HighScoreIsDifficult.csv` to see top “Difficult” flights.
3. Grab charts from `reports/figures/` for the deck.

---

## 📦 Requirements

`requirements.txt`

```text
pandas
numpy
matplotlib
seaborn
statsmodels
scikit-learn
jupyterlab
```

*(or use `environment.yml` if you prefer conda)*

---

## 📜 License & credits

* **Code:** MIT License
* **Data:** Remains the property of the provider; not redistributed.
* Built by **[Your Team Name]** for the **United Airlines Hackathon**.
