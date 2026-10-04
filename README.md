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

A 5-day GRU processes recent information for each company and captures short-term temporal dependencies.


The model also receives the filtered market-regime representation produced by the HMM.

### 4. Market Regime Detection

A Hidden Markov Model (HMM) identifies latent market states.


The final implementation uses filtered regime probabilities, meaning the regime estimate for day t only uses information available up to day t.


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

