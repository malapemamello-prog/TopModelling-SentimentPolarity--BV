# Topic Modelling, Sentiment Polarity and Bitcoin Volatility


---

## Overview

This project answers two questions about Bitcoin-related tweets (Kaggle "Bitcoin Tweets Dataset 2025-2026"):

1. **What themes** are being discussed, and how does their prevalence change over time?
2. **Does daily estimated sentiment** improve next-day Bitcoin volatility forecasts once market information and tweet volume are controlled for?

**Key results:** a nine-topic NMF model gave the most distinct themes, led by trading and market discussion. Sentiment did **not** give a consistent forecasting gain: it improved random forest MAE by 5.6% but slightly worsened both regression models. CryptoBERT agreed with a human reviewer on only 43% of 100 tweets.

---

## Data

| Stage | Records | Note |
|---|---|---|
| Successfully parsed source | 4,689,348 | Malformed records were skipped |
| Excluded (bad date/text) | 109 | |
| Retained modelling base | 4,689,239 | Shared by both questions |
| Topic training sample | 26,000 | Unique English tweets, 2,000 per month over 13 months |
| Full-corpus topic assignments | 4,641,399 | 47,840 left unassigned |
| CryptoBERT scored sample | 40,083 | Up to 200 tweets per observed day |
| Manual review sample | 100 | Human labels for validation |
| Forecasting origins | 200 | 160 train / 40 test (days, not tweets) |

- **Prices:** daily BTC-USD closes from Yahoo Finance (`yfinance`).
- **Coverage:** tweets exist for only 201 of 362 days (6 Mar 2025 to 2 Mar 2026); gaps are collection gaps, not quiet days.
- **Timestamp caveat:** some tweet text refers to 2021-2022 despite 2025-2026 timestamps, so tweet-market alignment cannot be fully trusted.

---

## Methods

| Step | Approach |
|---|---|
| Topic cleaning | Removed URLs, mentions, retweet markers, punctuation, numbers, stop words, and the words "bitcoin"/"btc" |
| Topic models | LDA (count vectors) vs NMF (TF-IDF); up to 5,000 uni/bigram features; seed 42 |
| Topic count | Nine, chosen from 5 to 13 using reconstruction error, coherence, diversity and stability |
| Sentiment | `ElKulako/cryptobert` on original text (negation, emoji and punctuation kept); polarity = P(positive) - P(negative) |
| Daily features | Mean polarity, negative share, positive share, log(1 + full tweet count) |
| Target | Next-day absolute log return (%) of BTC-USD |
| Split | Chronological 80/20 (no random split, no test-driven tuning) |
| Models | Linear regression and random forest (200 trees, depth 4) on three matched input groups: market; market + volume; market + volume + sentiment |
| Benchmarks | Training mean, training median, persistence |

---

## Results

### Topic model comparison (balanced training sample)

| Model | Coherence | Diversity | Topic balance | Largest topic (%) |
|---|---|---|---|---|
| LDA | -2.275 | 0.767 | 0.953 | 19.9 |
| **NMF (selected)** | **-1.249** | **0.956** | 0.732 | 42.1 |

### Nine NMF topics (share of balanced sample)

| Topic | Share |
|---|---|
| Trading and market discussion | 42.13% |
| NFTs, metaverse and meme coins | 22.33% |
| General crypto news and blockchain | 15.12% |
| Airdrops and promotional campaigns | 10.42% |
| Whale and volume alerts | 2.42% |
| Lightning network and Bitcoin faucets | 2.27% |
| Cardano price alerts | 2.23% |
| Bitcoin mining and block rewards | 1.75% |
| Coinhunt cryptocurrency gaming | 1.33% |

These are training-sample shares, not full-corpus shares. In the full corpus, trading is strongest around June 2025, airdrops peak around July to August, and Cardano alerts spike in September.


## Limitations

| Limitation | Implication |
|---|---|
| Timestamp integrity | Text referencing earlier years conflicts with 2025-2026 dates, limiting event interpretation and tweet-market alignment |
| Incomplete coverage | 161 of 362 days have no tweets; partial boundary months complicate monthly comparisons |
| Sampling | Topic fitting uses equal monthly English samples and sentiment uses a daily cap; neither is a census |
| Sentiment measurement | Only 43% agreement in a single-reviewer, 100-tweet check |
| Target and evaluation | Absolute return is one volatility proxy; one chronological split with 40 test origins and no uncertainty intervals |
| Interpretability | Topic scores are not probabilities; coefficients and importances do not show causal effects |

**Recommended next steps:** verify timestamps, expand manual review and improve sentiment measurement, use expanding-window evaluation, test sensitivity to sampling, language and an alternative volatility target (such as rolling standard deviation), and examine errors around large price moves.

---

## Contributors


Mamello Malape and
Reatile Seekoei 



---

## References

- Blei, D. M., Ng, A. Y. & Jordan, M. I. (2003). Latent Dirichlet allocation. *JMLR*, 3, 993-1022.
- ElKulako. CryptoBERT model card. https://huggingface.co/ElKulako/cryptobert
- pokeash. Bitcoin Tweets Dataset 2025-2026. Kaggle.
- Provost, F. & Fawcett, T. (2013). *Data Science for Business*. O'Reilly Media.
- scikit-learn developers. Topic extraction with NMF and LDA.
- Yahoo Finance. Bitcoin USD (BTC-USD) historical data.
