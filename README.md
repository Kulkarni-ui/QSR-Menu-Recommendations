# QSR-Menu-Recommendations
This project addresses the WWT Unstop Round 2 challenge of predicting a missing item in a customer’s partial cart and recommending the top 3 likely items, evaluated using **Recall@3**. 
## Executive Summary

This project addresses the WWT Unstop Round 2 challenge of predicting a missing item in a customer’s partial cart and recommending the top 3 likely items, evaluated using **Recall@3**.Using **historical transaction data**, we applied a **co-occurrence-based recommendation algorithm** with a popularity fallback to ensure robust suggestions even for sparse cases.The solution achieved a **Recall@3 score of 0.327**, meaning ~33% of true missing items appeared in the top 3 recommendations.The approach is **fully reproducible**, well-structured, and ready for scaling with personalization, seasonal adjustments, and hybrid recommendation models in future iterations.This documentation provides a complete breakdown of **dataset details, methodology, codebase structure, evaluation, and future work**.

## 1. Overview
This project optimizes **menu recommendations** and **customer segmentation** for a Quick Service Restaurant (QSR) chain.  
It uses transaction data to identify co-occurrence patterns between items and generate top-3 recommendations for partial customer carts.

---

## 2. Problem Statement
**Given:**
- Historical transaction logs (orders with items purchased)
- Customer demographic fields (not used in baseline)
- Partial cart with one missing item

**We aim to:**
1. Predict the missing item in the customer’s cart.
2. Generate **top 3** recommendations per order.
3. Evaluate performance using **Recall@3**.

**Competition Alignment:**  
- Input: Partial cart with 1 missing item  
- Output: Top 3 recommended items (**Recall@3** evaluation)

---

## 3. Dataset Description
| Column Name           | Description                              |
|-----------------------|------------------------------------------|
| CUSTOMER_ID           | Unique ID for customer                   |
| ORDER_ID              | Order identifier                         |
| item1, item2, item3   | Items in the order (partial in test set)  |
| CUSTOMER_TYPE         | Guest or Registered (not used in baseline) |
| ORDER_OCCASION_NAME   | Occasion type (not used in baseline)      |

---

## 4. Methodology
1. **Data Preprocessing** – Normalize item names, handle missing values.
2. **EDA** – Identify top-selling items and frequent item pairs.
3. **Recommendation Generation** –  
   - Build item co-occurrence counts from historical data.  
   - Recommend items most frequently bought together with items in the cart.  
   - Use popular items as fallback if co-occurrence is insufficient.
4. **Evaluation** – Calculate **Recall@3**.

---

## 5. Codebase Structure

```plaintext
qsr-recommendation/
├── data/
│   ├── raw/               # Raw data (or sample)
│   ├── processed/         # Cleaned datasets
├── notebooks/             # Jupyter notebooks for exploration
├── src/
│   ├── data/              # Data loading & preprocessing scripts
│   ├── models/            # Prediction, evaluation scripts
│   ├── utils/             # Helper functions
├── outputs/               # Final output sheet & figures
├── requirements.txt       # Dependencies
└── README.md              # Project documentation
```

---

## 6. Installation & Usage

### Requirements
- Python 3.9+
- Install dependencies:
```bash
pip install -r requirements.txt
```

### End-to-End Run
```bash
# 1. Preprocess data
python src/data/preprocess.py --config config.yml
```
```bash
# 2. Predict recommendations
python src/models/predict.py     --model-path experiments/best_model.pkl     --test data/raw/test_data_question.csv     --out outputs/Output_Sheet.xlsx
```
```bash
# 3. Evaluate Recall@3 (optional local check)
python src/models/evaluate.py     --pred outputs/Output_Sheet.xlsx     --gold data/raw/test_gold.csv     --metric recall@3
```

---

## 7. Core Recommendation Logic
```python
from collections import Counter
from itertools import combinations
import pandas as pd

def normalize(text):
    return str(text).strip().lower()

def build_pair_counts(df):
    pair_counts = Counter()
    for _, row in df.iterrows():
        items = [normalize(row['item1']), normalize(row['item2']), normalize(row['item3'])]
        for pair in combinations(set(items), 2):
            pair_counts[tuple(sorted(pair))] += 1
    return pair_counts

def recommend_items(cart, pair_counts, popular_items, top_n=3):
    cart = [normalize(i) for i in cart]
    recs = Counter()
    for item in cart:
        for pair, count in pair_counts.items():
            if item in pair:
                other = pair[0] if pair[1] == item else pair[1]
                if other not in cart:
                    recs[other] += count
    results = [item for item, _ in recs.most_common(top_n)]
    for fallback in popular_items:
        if fallback not in cart and fallback not in results:
            results.append(fallback)
        if len(results) >= top_n:
            break
    return results[:top_n]
```

---

## 8. Output Format
**`Output_Sheet.xlsx`** contains:
- CUSTOMER_ID  
- ORDER_ID  
- item1, item2, item3 (partial cart in test set)  
- RECOMMENDATION 1 / 2 / 3

---

## 9. Results Summary
- **Recall@3 Achieved:** `0.327` (~33% of true missing items are in top-3)  
- This is a **baseline** co-occurrence + popularity fallback method.  
- No personalization or seasonal adjustment applied yet.

---

## 10. Future Work
- Integrate personalization using `CUSTOMER_TYPE` and `ORDER_OCCASION_NAME`.
- Explore hybrid recommendation approaches.
- Add seasonal trend adjustments.
- Optimize scoring weights for Recall@3 improvement.

---

## Conclusion

This project successfully addresses the WWT Unstop Round 2 challenge by delivering a functional, interpretable, and reproducible recommendation system tailored for a Quick Service Restaurant (QSR) chain. Leveraging historical transaction data and a co-occurrence-based recommendation strategy with a popularity fallback, the solution achieved a **Recall@3 score of 0.327**, demonstrating a solid baseline performance.

The approach not only meets the competition’s requirements but also establishes a scalable framework that can be extended with personalization, contextual awareness, and hybrid recommendation techniques. With minimal computational overhead, the system is capable of generating relevant top-3 item suggestions even for sparse or unseen customer scenarios.

This work showcases the ability to convert raw transactional data into actionable business insights, ultimately contributing to enhanced customer experience, higher order value, and data-driven decision-making in the QSR domain. It sets a strong foundation for further improvements and real-world deployment.

## Authors
- [Atharv Kulkarni](https://github.com/Kulkarni-ui/)
- [Hrithik Rayapati](https://github.com/MRG-Hazmatz)

