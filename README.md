# E-Commerce Behavior Mining: PrefixSpan & Association Rules

> Uncovering hidden customer journeys and sequential purchase patterns in large-scale e-commerce event data using **Sequential Pattern Mining (PrefixSpan)** and **Association Rule Learning**.

---

## Project Overview

In modern e-commerce, understanding how customers behave and not just what they buy is a critical competitive advantage. This project applies **Sequential Pattern Mining (SPM)** to raw e-commerce event logs to extract ordered behavioral patterns such as:

- `view → cart → purchase`
- `cart → view → (abandon)`
- Repeated browsing loops before conversion

By mining ~6 million user interaction events, I discover statistically significant behavioral sequences and compute confidence-based conversion rates for each brand journey. The results are visualized through an interactive **Sankey diagram** to map the full customer flow.

---

## Dataset

| Property | Value |
|---|---|
| Source | eCommerce behavior data from multi category store (Oct-2019) [Kaggle](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store/data) |
| Files | `sequence_event1.json`, `sequence_event2.json` |
| Total Events | **5,974,703** sequences |
| Minimum Support | 0.1% of total events (~5,974 occurrences) |
| Event Types | `view`, `cart`, `purchase` |
| Brands Covered | Samsung, Apple, Xiaomi, Unknown |

Each sequence represents a user session encoded as an ordered list of `event_brand` strings (e.g., `['view_samsung', 'cart_samsung', 'purchase_samsung']`).

---

## Methodology

### 1. Sequential Pattern Mining — PrefixSpan

[PrefixSpan](https://github.com/chuanconggao/PrefixSpan-py) is a projection-based sequential pattern mining algorithm that efficiently discovers frequent ordered subsequences without candidate generation, making it ideal for large-scale datasets.

```
Input  : User event sequences  →  [view_samsung, cart_samsung, purchase_samsung]
Process: PrefixSpan (min_support = 0.1%)
Output : Frequent patterns with occurrence counts
```

### 2. Association Rule Mining (Confidence-Based)

For each frequent sequential pattern of length > 1, I derive **association rules** by treating the subsequence `[A, B]` as:

```
Antecedent: A  →  Consequent: B
Confidence: freq(A → B) / freq(A)
```

This confidence score is interpreted as the **behavioral conversion rate**, which represent the probability that a user who performed action A will subsequently perform action B.

---

## Key Results

### Top Frequent Sequential Patterns

The most dominant single-event patterns reveal brand viewing dominance:

| Rank | Pattern | Frequency |
|---|---|---|
| 1 | `view_Unknown` | 1,512,378 |
| 2 | `view_samsung` | 1,285,599 |
| 3 | `view_Unknown → view_Unknown` | 1,051,460 |
| 4 | `view_apple` | 1,002,335 |
| 5 | `view_samsung → view_samsung` | 971,106 |
| 6 | `view_apple → view_apple` | 791,250 |
| 7 | `view_xiaomi` | 737,611 |

> **Observation:** Samsung and Apple dominate the top browsing patterns. The high frequency of repeated `view → view` sequences indicates strong consideration loops, which means users repeatedly browse a brand before deciding.

---

### Cart and Purchase Patterns

Filtering for patterns that involve `cart` or `purchase` reveals the most commercially valuable user journeys:

| Rank | Pattern | Frequency |
|---|---|---|
| 1 | `cart_samsung` | 185,786 |
| 2 | `view_samsung → cart_samsung` | 184,926 |
| 3 | `purchase_samsung` | 149,358 |
| 4 | `view_samsung → purchase_samsung` | 148,934 |
| 5 | `cart_apple` | 136,037 |
| 6 | `view_apple → cart_apple` | 135,572 |
| 7 | `cart_samsung → view_samsung` | 127,766 |
| 8 | `view_samsung → cart_samsung → view_samsung` | 127,086 |
| 9 | `purchase_apple` | 121,635 |
| 10 | `view_apple → purchase_apple` | 121,342 |
| 11 | `cart_samsung → purchase_samsung` | 104,910 |
| 12 | `view_samsung → cart_samsung → purchase_samsung` | 104,409 |

---

### Conversion Rate Analysis

Using association rule confidence, we compute the behavioral conversion rate for each sequential step:

| Pattern (Antecedent → Consequent) | Events | Conversion Rate |
|---|---|---|
| `view_samsung` → `cart_samsung` | 184,926 | **14.38%** |
| `view_samsung` → `purchase_samsung` | 148,934 | **11.58%** |
| `view_apple` → `cart_apple` | 135,572 | **13.53%** |
| `view_apple` → `purchase_apple` | 121,342 | **12.11%** |
| `cart_samsung` → `purchase_samsung` | 104,910 | **56.47%** |
| `view_samsung → cart_samsung` → `purchase_samsung` | 104,409 | **56.46%** |
| `cart_samsung` → `view_samsung` | 127,766 | **68.77%** |
| `view_samsung → cart_samsung` → `view_samsung` | 127,086 | **68.72%** |
| `cart_apple` → `view_apple` | 92,655 | **68.11%** |
| `view_apple → cart_apple` → `view_apple` | 92,293 | **68.08%** |
| `view_samsung → view_samsung` → `cart_samsung` | 94,500 | **9.73%** |

> **Key Metric:** Once a user adds to cart, there is a **~56% probability** they will complete the purchase, which is a strong indicator of high purchase intent at the cart stage.

---

## Business Insights & Golden Path

### The Golden Path

The most high-value, end-to-end customer journey discovered from the data is:

```
view_samsung  →  cart_samsung  →  purchase_samsung
```

This three-step path occurred 104,409 times and represents the clearest signal of a high-intent buyer journey. Any intervention that accelerates this path directly drives revenue.

---

### Insight 1: The Cart is the Conversion Tipping Point

The conversion rate jumps dramatically once a user adds an item to cart:

- **View → Cart**: ~14% (Samsung), ~13.5% (Apple)
- **Cart → Purchase**: **~56%** (Samsung)

This means the cart stage is the most powerful momentum indicator. Once a user carts an item, they are more than 4× more likely to purchase compared to someone who only viewed. Strategies to push users from `view` to `cart` will have the highest ROI.

**Recommended Actions:**
- Use retargeting campaigns specifically for users in the `view` stage who haven't carted yet
- Deploy product page nudges such as "Only 3 left in stock" or limited-time discounts to reduce view-to-cart friction

---

### Insight 2: Cart Abandonment is a Browse-Back Loop

The pattern `cart_samsung → view_samsung` has a confidence of **68.77%**, meaning that nearly 7 out of 10 users who add to cart will return to browse again rather than purchasing immediately. This "browse-back" behavior is a classic signal of price sensitivity or comparison shopping.

**Recommended Actions:**
- Implement cart abandonment email/notification sequences within 1–2 hours of the cart event
- Show side-by-side brand comparisons on the cart page to reduce external comparison-seeking
- Offer installment payment options on the cart page to overcome price hesitation

---

### Insight 3: Samsung Dominates But Apple Converts More Efficiently

While Samsung leads in raw frequency across all event types, the view-to-purchase gap is narrower for Apple:

| Brand | View Freq | Purchase Freq | View-to-Purchase Rate |
|---|---|---|---|
| Samsung | 1,285,599 | 149,358 | ~11.6% |
| Apple | 1,002,335 | 121,635 | ~12.1% |

Apple users show slightly higher purchase intent per view, suggesting a more decisive buyer persona with higher brand loyalty.

**Recommended Actions:**
- For Samsung: focus on reducing decision friction by providing more reviews, filter tools, comparison features
- For Apple: focus on upselling and cross-selling (Apple buyers are more committed; a well-placed accessory recommendation can significantly increase basket size)

---

### Insight 4: Repeat-View Behavior Signals Warm Audiences

Patterns like `view_samsung → view_samsung → cart_samsung` (9.73% conversion) confirm that users who browse a brand repeatedly are warming up toward a purchase, even if their conversion rate per session looks low.

**Recommended Actions:**
- Build a "frequent viewer" cohort for remarketing
- Trigger personalized promotions after the 2nd or 3rd repeated view event
- Use homepage and category page personalization to keep returning viewers engaged

---

### Insight 5: Xiaomi is Underperforming in the Funnel

Xiaomi is the 3rd most-viewed brand (`view_xiaomi`: 737,611) but does not appear in the top business-critical patterns for cart or purchase, suggesting a significant funnel drop-off between awareness and action.

**Recommended Actions:**
- Audit Xiaomi product pages for UX friction points
- Investigate whether Xiaomi products are competitively priced relative to Samsung and Apple
- Consider targeted promotions or bundle deals for Xiaomi to push users past the view stage

---

## Customer Journey Visualization

An interactive **Sankey diagram** was built using Plotly to visualize the flow of user events across the top business-critical patterns. The diagram maps weighted transitions between `view`, `cart`, and `purchase` events across brands, providing an intuitive view of where users enter and exit the conversion funnel.

> The visualization is generated in the notebook via `plotly.graph_objects.Sankey` and is viewable interactively in Google Colab or any Jupyter environment.

---

## Tech Stack

| Tool | Purpose |
|---|---|
| `Python 3.12` | Core language |
| `PrefixSpan 0.5.2` | Sequential pattern mining |
| `pandas` | Data manipulation |
| `plotly` | Interactive Sankey diagram visualization |
| `collections.defaultdict` | Transition aggregation |
| Google Colab | Notebook execution environment |
| Google Drive | Data storage |
