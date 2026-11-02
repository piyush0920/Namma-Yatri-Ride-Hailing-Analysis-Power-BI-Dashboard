# 🚖 Namma Yatri — Data Visualization & Analysis

> **Executive PG Diploma in Data Science & AI** | IIIT Bangalore  
> **Tool:** Microsoft Power BI | **Module:** Advanced Data Visualisation

## Deliverables and viewing the work

This repository includes the source workbook and exported presentation/report files:

- [Dynamic dashboard assignment PDF](./DV%20Assignment%20Dynamic%20Dashboard.pdf)
- [Technical report PDF](./DV_Namma_Yatri_Piyush_Sharma_Report.pdf)
- [Management presentation](./DV_Namma_Yatri_Piyush_Sharma_Management.pptx)
- [Technical presentation](./DV_Namma_Yatri_Piyush_Sharma_Technical.pptx)
- `nammayatri.xlsx` — analysis data workbook

**Power BI file availability:** An editable `.pbix` file is not currently included. The repository provides the workbook and exported documentation/presentations, rather than a directly openable Power BI Desktop project.

---

## 📌 Project Overview

This project involves end-to-end data analysis and visualization of the **Namma Yatri** ride-hailing dataset using Microsoft Power BI. The analysis covers data preparation, exploratory data analysis (EDA), and strategic recommendations for operational and marketing improvements.

Namma Yatri is an open-source ride-hailing platform operating in Bengaluru, India. The dataset captures the full trip lifecycle — from search attempts through to completed rides — across 37 assembly zones in the city.

---

## 📁 Dataset

The dataset consists of **5 tables** imported into Power BI:

| Table | Rows | Description |
|-------|------|-------------|
| `Trips` | 983 | Core fact table — fare, distance, location IDs, payment method, duration |
| `Trip_Details` | 2,161 | Full search-to-ride funnel with binary flags at each stage |
| `Duration` | 24 | Lookup table — hour buckets (0-1 through 23-24) |
| `Assembly` | 37 | Lookup table — zone names across Bengaluru |
| `Payment` | 4 | Lookup table — payment methods (cash, UPI, debit card, credit card) |

> ⚠️ `Trip_Details` has more rows than `Trips` because it captures every search attempt, including those that never converted into a completed ride. This is expected, not a data error.

> **Relationship validation:** Confirm key uniqueness and relationship cardinality in Power BI before reusing the model. Table row counts alone do not prove a one-to-one relationship, and search attempts should not be assumed to correspond to completed trips.

---

## 🔗 Data Model

The following relationships were established in Power BI:

```
Trips.tripid        ──(1:1)──>   Trip_Details.tripid
Trips.duration      ──(M:1)──>   Duration.id
Trips.faremethod    ──(M:1)──>   Payment.id
Trips.loc_from      ──(M:1)──>   Assembly_Pickup.ID
Trips.loc_to        ──(M:1)──>   Assembly_Drop.ID
```

> 📝 **Note:** The `Assembly` table was duplicated into `Assembly_Pickup` and `Assembly_Drop` using the **role-playing dimension** pattern. This allows both pickup and drop zone names to be used simultaneously in visuals without requiring `USERELATIONSHIP()` DAX.

---

## 🧹 Data Preparation

### Inconsistencies Checked
- ✅ No duplicate trip IDs in any table
- ✅ No null or missing values across all sheets
- ✅ Fares range ₹44–₹1,500 — no zero or negative values
- ✅ Distances all positive (2–27 km)
- ✅ Trip_Details funnel logically consistent — no row skips a stage
- ✅ All ID columns correctly typed as whole numbers in Power Query
- ✅ Duration text column sorted by `Duration.id` to ensure correct hour-sequence ordering in visuals

### Calculated Fields Created

| Field | Type | Logic |
|-------|------|-------|
| `Fare_per_KM` | Calculated Column | `fare / distance` |
| `Count_of_Trips` | Measure | `COUNTROWS(Trips)` |
| `Sum_of_trips_revenue` | Measure | `SUM(Trips[fare])` |
| `Customer_Cancellation_Rate` | Measure | % rows where `customer_not_cancelled = 0` |
| `Driver_Cancellation_Rate` | Measure | % rows where `driver_not_cancelled = 0` |
| `Successful_Ride_Rate` | Measure | % rows where `end_ride = 1` |
| `Quote_To_Completion_Rate` | Measure | `end_ride = 1` ÷ `searches_for_quotes = 1` |
| `Time_Period` | Calculated Column | Groups duration into Night / Morning / Afternoon / Evening |

---

## 📊 Exploratory Data Analysis

### 2.1 Variable Classification

**Categorical:**
- Assembly zone names (Pickup & Drop) — nominal
- Payment method (cash, UPI, debit card, credit card) — nominal
- Duration bucket (0-1 through 23-24) — ordinal, sorted by `Duration.id`
- ID fields: tripid, custid, driverid, loc_from, loc_to, faremethod — nominal numeric
- Trip_Details funnel flags — binary (0/1)

**Numerical:**
- `fare` — continuous (₹44–₹1,500)
- `distance` — continuous (2–27 km)
- `fare_per_km` — derived continuous

---

### 2.2 Demand Over Time

**Visual:** Line chart — Count of Trips by Duration bucket

| Peak Hours | Trips |
|------------|-------|
| 0-1 | 53 |
| 13-14 | 52 |
| 6-7 / 11-12 / 17-18 | 48 each |

| Lowest Hours | Trips |
|--------------|-------|
| 20-21 / 23-24 | 32 each |
| 8-9 / 4-5 | 33 each |

> Demand oscillates in a 4-6 hour wave pattern throughout the day rather than a simple morning/evening rush.

---

### 2.3 Revenue by Time Period

**Visual:** Donut chart — Sum of fare by Time Period

| Time Period | Revenue | Share |
|-------------|---------|-------|
| Night (21–05) | ₹2,47,537 | **32.95%** |
| Morning (05–12) | ₹2,20,297 | 29.32% |
| Afternoon (12–17) | ₹1,66,965 | 22.22% |
| Evening (17–21) | ₹1,16,544 | 15.51% |

> Night dominates revenue despite evening having a demand spike — late-night rides are longer and higher-value.

---

### 2.4 Trip Hour vs Revenue

**Visual:** Bar chart — Sum of fare by Duration bucket

- Highest revenue hour: **0-1** (₹45,019 — 5.99% of total revenue)
- Second: **6-7** (₹39,624) and **22-23** (₹38,880)
- Lowest: **8-9** (₹21,174)

---

### 2.5 Payment Method Popularity

**Visual:** Donut chart — Count of Trips by Payment Method

| Method | Trips | Share |
|--------|-------|-------|
| UPI | 262 | 26.65% |
| Cash | 243 | 24.72% |
| Debit Card | 243 | 24.72% |
| Credit Card | 235 | 23.91% |

> Near-equal distribution — no single payment mode dominates.

---

### 2.6 High-Performing Zones

**Visual:** Line and Clustered Column Chart — Trip count (bars) and Revenue (line) by pickup zone

| Zone | Trips | Revenue |
|------|-------|---------|
| Ramanagaram | 39 | ₹26,225 |
| Yeshwantpur | 36 | ₹29,035 |
| Bangalore South | 33 | ₹30,295 |
| Dasarahalli | 33 | ₹23,624 |
| Gandhi Nagar | 32 | ₹23,065 |

> Bangalore South ties Dasarahalli on trip count but generates 28% more revenue — indicating longer or higher-fare routes.

---

### 2.7 Ride Time Periods Across Zones

**Visual:** Heatmap Matrix — Zones × Duration buckets, colored by trip count

Notable hotspots:
- **Basavanagudi 6-7** — 7 trips (highest single cell)
- **Jayanagar 1-2** and **Chamrajpet 11-12** — 5 trips each
- **0-1 column** — active across almost every zone (platform-wide midnight demand)

---

### 2.8 Top 5 Zones by Trip Volume

**Visual:** Treemap — Count of Trips by pickup zone (Top 5 filter returns 6 due to ties)

| Zone | Trips |
|------|-------|
| Ramanagaram | 39 |
| Yeshwantpur | 36 |
| Bangalore South | 33 |
| Dasarahalli | 33 |
| Gandhi Nagar | 32 |
| Rajarajeshwarinagar | 32 |

---

### 2.9 Cancellation & Completion Analysis

#### 2.9.1 Cancellation Rates

**Visual:** Two donut charts + Card visual

| Metric | Value |
|--------|-------|
| Customer Cancellation Rate | **48.17%** |
| Driver Cancellation Rate | **47.25%** |
| Successful Ride Rate | **45.49%** |

#### 2.9.2 Quote-to-Completion

**Visual:** Card visual (67.56%) with written funnel analysis

| Stage | Count | % of Total |
|-------|-------|------------|
| Total Searches | 2,161 | 100% |
| Searched for Quotes | 1,455 | 67.33% |
| Completed Rides | 983 | 45.49% |
| Quote → Completion | — | **67.56%** |

---

### 2.10 Parameter & Filter

**Visual:** Payment Method slicer (field parameter) filtering zone trip count bar chart

Allows interactive comparison of which zones prefer which payment method — useful for targeted campaign planning.

---

## 📋 Recommendations

### Operational Efficiency

| Priority | Recommendation | Evidence |
|----------|---------------|----------|
| 🔴 High | Cancellation penalty system for both drivers and customers | 48% cancellation rate |
| 🔴 High | Predictive driver positioning in peak zones/hours | Demand heatmap (2.7) |
| 🟡 Medium | Dynamic pricing in Bangalore South and Hebbal | Revenue vs trip count divergence |
| 🟡 Medium | Streamline quote-to-booking flow — auto-rematch on cancellation | 67.56% quote completion rate |

### Marketing Strategy

| Strategy | Target |
|----------|--------|
| UPI cashback campaign | Cash + debit card users (49% of trips) |
| Loyalty program (discount after 10 rides) | Ramanagaram, Yeshwantpur, Bangalore South |
| First-ride discount campaign | Nelamangala, Doddaballapur, Rajaji Nagar |
| Guaranteed minimum hourly driver earnings | Off-peak slots — 8-9 AM and 20-21 PM |

---

## 📈 Key Metrics Summary

| Metric | Value |
|--------|-------|
| Total Completed Trips | 983 |
| Total Revenue | ₹7,51,343 |
| Trip Completion Rate | 45.49% |
| Customer Cancellation Rate | 48.17% |
| Driver Cancellation Rate | 47.25% |
| Quote-to-Completion Rate | 67.56% |
| Top Pickup Zone (Volume) | Ramanagaram — 39 trips |
| Top Pickup Zone (Revenue) | Bangalore South — ₹30,295 |
| Peak Demand Hour | 0-1 (53 trips) |
| Total Zones Analysed | 37 |

---

## 🛠️ Technical Notes

- **Role-playing dimension:** Assembly table duplicated into `Assembly_Pickup` and `Assembly_Drop` to enable two simultaneous active relationships in the data model
- **Duration sort:** Duration text column (`0-1`, `1-2` etc.) sorted by `Duration.id` integer to prevent alphabetical missorting in visuals
- **Funnel time breakdown limitation:** `Trip_Details` has no duration column — quote-to-completion rate cannot be broken down by hour without producing a misleading 100% flat line
- **Parameter implementation:** Power BI's Top N filter UI does not natively accept a parameter measure — a Payment Method field slicer was used as the interactive parameter instead

---

## 👤 Author

**Piyush Sharma**  
Senior Executive — P&C Insurance Operations, Xceedance  
Executive PG Diploma in Data Science & AI, IIIT Bangalore (2024–2026)  
🔗 [LinkedIn](https://linkedin.com/in/piyush-sharma-6591171ba) | 💻 [GitHub](https://github.com/piyush0920)
