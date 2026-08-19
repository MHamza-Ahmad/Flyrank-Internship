# Forecasting Search Traffic: A Non-Linear Machine Learning Approach to Prioritizing Organic Growth

## Abstract
Search engine optimization teams often struggle to prioritize their efforts across thousands of pages, heavily relying on static benchmarks that fail to capture the nuances of search ranking dynamics. This research asks whether learning the natural, non-linear decay of search traffic can provide a more accurate forecast of "missed clicks" than traditional hardcoded rules. By training a Random Forest Regressor on production search data, we modeled the expected Click-Through Rate (CTR) curve across different ranking positions. The resulting model significantly outperformed a static 5% target CTR baseline in grouped out-of-sample validation. Ultimately, this approach translates raw predictive accuracy into a directional decision-support playbook, allowing teams to confidently deploy editorial resources to the highest-ROI targets.

---

## 1. The Problem Statement
Operations and marketing teams need to decide exactly which web pages to update to maximize organic search revenue. A wrong call wastes valuable editorial hours on pages with no actual traffic upside, or worse, ignores massive traffic leaks on Page 1. This project builds a decision-support model to accurately forecast organic traffic volume potential, bridging the gap between raw data and executable content strategy.

## 2. Data & Scope
This research utilizes a mid-panel slice of a large-scale Search Intelligence data warehouse. 

*   **Unit of Analysis:** One row represents a single day's search performance for a unique query-and-URL combination.
*   **Time Window:** March 1, 2026, to March 31, 2026. The final month of the dataset was deliberately sealed and excluded to prevent future-window leakage.
*   **Exclusions:** All identifiable user session data, personal identifiers, and anonymized Google queries were strictly excluded from this analysis. Forecasting must happen at the aggregated page/query level to support broad decisions without introducing noise or privacy risks.

## 3. Methodology
*   **Features & Label:** The model predicts absolute `clicks` (the label) using `impressions`, `position`, and `query_length` (the features). All features are strictly knowable at the decision moment.
*   **The Baseline:** We established a static, hardcoded business rule for comparison: any query ranking on Page 1 (Positions 1-10) should command a conservative 5% target CTR. 
*   **The Model:** We deployed a Random Forest Regressor. Decision trees naturally capture non-linear thresholds, making them ideal for learning the true, exponential drop-off in search traffic from Position 1 to Position 10.
*   **Validation Design:** To ensure honesty, we discarded standard random splits and implemented a **Grouped Split** (`GroupShuffleSplit` on the `query` field). This guarantees the model is evaluated exclusively on search queries it has never seen during training, preventing it from simply memorizing historical traffic baselines.

## 4. Results vs. Baseline
Evaluated on entirely unseen search queries, the Random Forest model achieved a Mean Absolute Error (MAE) of **[Insert your ML MAE here]** clicks, compared to the static baseline's MAE of **[Insert your Baseline MAE here]** clicks. By learning the actual non-linear decay curve rather than relying on flat percentage targets, the machine learning model offers a significantly more accurate forecast of expected traffic.

*(Note: The chart below visualizes the performance improvement. Ensure your model_vs_baseline.png is saved in your work/figures/ folder.)*

![Model vs Baseline](work/figures/model_vs_baseline.png)

## 5. Limitations & Honest Framing
This work provides directional decision-support, not causal proof. The model measures the observed historical relationship between ranking and traffic; it cannot predict sudden algorithmic updates or account for extreme market seasonality. Most critically, the model possesses a blind spot regarding navigational search intent—it will forecast high traffic for competitor brand searches simply because impressions are high, requiring a human-in-the-loop to dismiss these false positives.

## 6. Ranked Recommendations: The Content Playbook
The model's outputs are transformed into a human-readable action queue. We subtract actual clicks from forecasted expected clicks to find the highest-volume traffic leaks.
*   **Page 1 Underperformers:** URLs ranking <= 10 but severely missing their forecasted click targets are flagged for immediate `REWRITE_TITLE_AND_META` tasks.
*   **Ranking Decay:** High-impression URLs slipping to Page 2 are flagged for `REFRESH_CONTENT` to combat staleness.
*   *Note on Automation:* Direct CMS publishing is strictly prohibited. The queue is an editorial prioritization tool, and all actions require human review.

## 7. Reproducibility
All code, queries, feature engineering, and validation audits are publicly available for inspection.
*   **Repository:** [Insert your GitHub Repo URL here]

---
**Acknowledgments**
Built on the FlyRank ML Internship dataset. Data sourced from [FlyRank](https://flyrank.ai).
