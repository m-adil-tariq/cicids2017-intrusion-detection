# Network Intrusion Detection on CICIDS2017 (Random Forest)

A Random Forest–based network intrusion detection project using the CICIDS2017 benchmark dataset, with a focus on critically evaluating results rather than reporting raw accuracy at face value.

Read Full Report here: [CICIDS-2017.pdf](./CICIDS-2017.pdf)

## Overview

This project trains a Random Forest classifier to distinguish benign from malicious network traffic, first as a **binary task** (Normal vs. Attack) and then as a **multiclass task** across seven categories (Normal, DoS, DDoS, Port Scanning, Brute Force, Web Attacks, Bots). Rather than stopping at near-perfect aggregate metrics, the project investigates *why* the results look so strong and surfaces a real, class-specific weakness the aggregate numbers hide.

## Dataset

- **Source:** CICIDS2017 (pre-processed CSV, sourced from Kaggle)
- **Size:** 2,520,751 labeled network flow records, 53 columns
- **Features:** Extracted via CICFlowMeter (packet length statistics, flow duration, inter-arrival times, flag counts, etc.)
- **Classes:** Normal Traffic, DoS, DDoS, Port Scanning, Brute Force, Web Attacks, Bots
- Notable class imbalance: Normal Traffic ≈ 83% of records; Bots and Web Attacks each < 0.1%

## Methodology

1. **Data loading & exploration** — cleaned column names, examined class distribution, checked for missing/infinite values.
2. **Data preparation** — separated features/labels, created a binary label (Normal vs. Attack), used a stratified 70/30 train/test split to preserve class ratios.
3. **Model training** — Random Forest (100 estimators) trained separately for binary and multiclass tasks.
4. **Evaluation** — precision, recall, F1-score, and confusion matrices (accuracy alone was avoided due to class imbalance).
5. **Feature importance analysis** — extracted and compared top features for both models.

## Key Findings

- **Binary classification** achieved near-perfect precision/recall/F1 (~1.00) — treated as a finding to investigate, not a result to accept outright.
- Ruled out **duplicate records** (only 161 of 2.5M+ rows) and **Destination Port leakage** (removing it had no effect) as explanations.
- Literature review confirmed near-perfect scores are a **known, documented limitation** of CICIDS2017, caused by automated attack generation producing overly consistent statistical signatures.
- **Multiclass classification** revealed a genuine weakness: **Bot traffic recall of only 0.72**, with most Bot samples misclassified as Normal Traffic — consistent with real botnet behavior designed to blend in with benign traffic.
- Class size alone doesn't explain detection difficulty — a comparably small class (Brute Force) was classified perfectly, while Bots underperformed.
- **Feature importance** showed the model relies heavily on packet-length statistics (~1/3 of total importance) in both models; Destination Port ranked low, reinforcing that it wasn't driving performance.

## Limitations & Future Work

- Uses a random train/test split, which can let flows from the same attack session appear in both sets. A **temporal, day-based split** (train on earlier days, test on a later day) would better simulate real-world generalization.
- Future work could explore **unsupervised anomaly detection**, which doesn't depend on pre-labeled attacks and may better handle novel attack types.

## References

- Sharafaldin, I., Lashkari, A. H., & Ghorbani, A. A. (2018). *Toward Generating a New Intrusion Detection Dataset and Intrusion Traffic Characterization.* ICISSP.
- Lanvin, M., Gimenez, P. F., Han, Y., Majorczyk, F., Mé, L., & Totel, É. (2023). *Errors in the CICIDS2017 Dataset and the Significant Differences in Detection Performances It Makes.* CRiSIS 2022.
