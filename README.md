# SCEG: Structured Event Graphs for Next-Day Stock Prediction

SCEG is a research framework for studying whether **structured financial events, market regimes, and inter-company relationships** provide additional signal for next-day stock movement and cross-sectional stock ranking.

The framework combines LLM-based event extraction, text embeddings, temporal modeling, market-regime detection, and graph-based information propagation. Rather than focusing only on maximizing predictive accuracy, the project uses **controlled, leakage-aware experiments** to test each proposed mechanism against appropriate baselines and placebos.

---

## Research Questions

SCEG investigates four main questions:

1. **Do structured financial events provide additional predictive signal compared with raw text representations?**
2. **Can a learned inter-company dependency graph improve prediction compared with no graph, sector graphs, or matched-random graphs?**
3. **Do detected market regimes provide useful information beyond shuffled-regime controls?**
4. **Is apparent directional prediction skill due to stock-specific selection, or simply predicting the overall market direction?**

---

## SCEG Framework

```text
Financial News / Tweets + Daily Prices
                │
                ▼
      Qwen2.5-1.5B Event Extraction
                │
                ▼
      Structured Event Representation
                │
                ▼
       BGE Embeddings + Price Features
                │
                ▼
          5-Day GRU Encoder
                │
          + Filtered HMM Regime
                │
                ▼
     Optional Graph Message Passing
                │
                ▼
       ┌────────┴────────┐
       ▼                 ▼
 Direction Prediction   Stock Ranking
   Up / Down            Score / Rank
```
The graph stage uses signed lead-lag dependencies and controlled graph alternatives. The graph component is retained only when validation shows an improvement.


DYNOTEARS is treated as a directed statistical dependency prior, not as proof of causal relationships.

---

## Key Components
### 1. Structured Event Extraction

Financial news and tweets are processed using Qwen2.5-1.5B-Instruct.


Each item is converted into a structured representation containing:
- Subject
- Event type
- Polarity
- Magnitude
- Temporal horizon


The structured representation is evaluated against a raw-text control to determine whether explicit event structure provides additional predictive information.

### 2. Text Representation

BGE embeddings are generated for textual and structured-event representations and combined with market-derived features.

### 3. Temporal Modeling

A 5-day GRU processes recent information for each company and captures short-term temporal dependencies.<br>
The model also receives the filtered market-regime representation produced by the HMM.

### 4. Market Regime Detection

A Hidden Markov Model (HMM) identifies latent market states.<br>
The final implementation uses filtered regime probabilities, meaning the regime estimate for day t only uses information available up to day t.<br>
Real regimes are compared against shuffled-regime controls.

### 5. Inter-Company Dependency Graph

SCEG evaluates multiple graph constructions:
- DYNOTEARS signed lead-lag dependency graph
- Sector graph
- Correlation graph
- Matched-random graph


The matched-random graph preserves the number and weighting of relationships while randomizing the company connections.

### 6. Graph Message Passing

The final model uses gated residual graph message passing.


A company can receive information from linked companies at:

- the same day
- one day earlier
- two days earlier

  
The graph contribution is initialized at zero and retained only when validation performance improves.

---

## Prediction Tasks
### Direction Prediction

The model predicts whether a company's stock will move up or down.


Metrics:

- Accuracy
- Matthews Correlation Coefficient (MCC)
- AUC
### Cross-Sectional Ranking

The model produces a stock score that is used to rank companies relative to each other.


Metrics:

- IC
- RankIC
### Magnitude Prediction

Magnitude prediction is not part of the final SCEG-R model and is treated only as a post-hoc analysis.

---

## Datasets
### StockNet / ACL18

The ACL18 benchmark contains stock prices and Twitter data.


SCEG applies a strict temporal cutoff of 16:00 ET. Tweets or news appearing after the cutoff are assigned to the following trading day.

### FNSPID

FNSPID provides financial news and historical stock-price information.


The final evaluation uses chronological walk-forward testing across multiple test years.

---

## Experimental Protocol

The project is designed as a controlled mechanism study.

| Mechanism | Comparison |
|---|---|
| Structured events | Structured events vs. raw text |
| Market regimes | Real HMM regimes vs. shuffled regimes |
| Graph propagation | Learned graph vs. sector/correlation/matched-random graphs |
| Temporal modeling | GRU vs. simpler and alternative temporal baselines |


The experiments use:

- 5 random seeds
- Chronological train/validation/test splits
- Training-only fitting of preprocessing components
- Strict 16:00 ET news timing
- Multiple baseline models
- Block bootstrap
- Newey–West adjustment
- Holm multiple-testing correction
- TOST equivalence testing

Scalers, PCA transformations, HMMs, and graph structures are fitted using training data only.

---

## Baseline Models

The framework is compared against:

- Logistic Regression
- MLP
- GRU
- Transformer
- LightGBM
- Momentum / reversal-style rules

---

## Results

The main finding is not that SCEG produces a large predictive advantage.


Instead, the controlled experiments found that the proposed mechanisms did not provide reliable stock-specific predictive signal under the strict evaluation protocol.

### Direction Prediction

On ACL18, SCEG-R achieved approximately:

- 55.8% Accuracy
- 0.125 MCC

However, stock-specific accuracy was approximately 50.9%, indicating that much of the apparent directional skill came from predicting the overall market direction rather than reliably selecting individual stocks.


On FNSPID:

- 50.2% Accuracy
- 0.001 MCC

A logistic-regression baseline performed similarly.

### Cross-Sectional Ranking
| Dataset | RankIC |
|---------|-------:|
| ACL18   | 0.029  |
| FNSPID  | 0.005  |

The confidence intervals included zero, so the observed ranking signal could not be reliably distinguished from zero.

---

## Main Research Findings

The controlled experiments found:

- Structured events performed approximately like raw text.
- Real market regimes did not outperform shuffled regimes.
- Learned company graphs did not outperform appropriate graph controls.
- Ranking performance remained close to zero.
- The apparent ACL18 directional advantage was largely market-wide timing rather than stock-specific selection.

These findings suggest that adding structured events, learned inter-company dependencies, and market regimes does not automatically provide useful stock-specific predictive information under strict next-day timing.

---

## Diagnostics

The project investigates why the proposed mechanisms fail to produce reliable stock-specific signal.


Key diagnostics include:

- Limited information content in extracted events
- Instability of learned graph relationships
- Information dilution during graph propagation
- Strong market-wide component in model predictions
- Limited regime variation in some evaluation periods
- Limited statistical power of the ACL18 test period

---

## Leakage and Reproducibility Controls

The project applies several safeguards against temporal leakage:

- News after 16:00 ET is assigned to the following trading day.
- Scalers and PCA are fitted using training data only.
- HMM regimes are filtered rather than smoothed.
- Graph structures are estimated using training data.
- Temporal windows do not cross dataset split boundaries.
- Test predictions are generated only after the evaluation protocol is frozen.
- Results are reported across five random seeds rather than selecting the best run.
- Statistical corrections are applied for multiple comparisons.

Known limitations and unresolved checks are explicitly documented.

---

## Limitations
- ACL18 contains only 64 test days.
- FNSPID has survivorship-bias concerns because the selected firms survive through the end of the sample.
- A large portion of FNSPID news lacks precise timestamps.
- Qwen and BGE were trained after portions of the historical evaluation period, creating a potential knowledge-cutoff limitation.
- One automated price-feature leakage check remains unresolved and is disclosed as a limitation.
- Published models such as MAN-SF were not yet reimplemented under exactly the same strict timing protocol.

---

## Future Work
- Resolve the remaining leakage check.
- Complete the FNSPID ranking study.
- Re-run published stock-prediction models under the same strict timing protocol.
- Conduct a pre-registered evaluation on an Indian-market dataset.
- Use an event extractor with a knowledge cutoff before the evaluation period.
- Investigate economically grounded relationships such as supplier-customer networks.

---

## Research Contribution

The main contribution of SCEG is not a claim of superior stock-prediction accuracy.


Instead, the project provides a controlled framework for testing whether:

- structured financial events,
- market regimes, and
- inter-company graph propagation

add measurable stock-specific predictive information.


The experiments provide a replicated and bounded null result across two datasets, together with diagnostics explaining where the proposed mechanisms fail.

---

## Technologies
- Python
- PyTorch
- Qwen2.5-1.5B-Instruct
- BGE Embeddings
- GRU
- Hidden Markov Models
DYNOTEARS
- NumPy
- Pandas
- Scikit-learn
- SciPy
- Jupyter
- Google Colab

---

## Repository Structure
```text
SCEG-Stock-Prediction/
│
├── README.md
├── SCEG_ACL18_Final.ipynb
└── SCEG_FNSPID_Final.ipynb
```
The notebooks contain the implementation, experiments, evaluation procedures, generated outputs, and research analysis.
---

## Author

Himanshi Mittal<br>
B.Tech Computer Science & Engineering<br>
Indira Gandhi Delhi Technical University for Women (IGDTUW)

---

## Disclaimer

This repository is intended for research and educational purposes. The reported results should not be interpreted as evidence of a reliable trading strategy or financial advice.

---

