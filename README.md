# Classical AI Reasoning System: Entailment over Knowledge Bases

**Course / Subject:** Artificial Intelligence (`CSL7610`)  
**Name:** Bhisham Sharma  
**Roll Number:** `M26MAC003`  
**Methodology:** 100% Classical Symbolic AI (Zero Generative AI / LLMs for Inference)

---

## 📌 Overview

This repository implements an end-to-end symbolic reasoning agent that determines whether a natural language query is logically entailed by an unstructured knowledge base of facts and rules:

$$\text{Natural Language Knowledge Base} \longrightarrow \text{Internal Representation} \longrightarrow \text{Classical Symbolic Reasoning} \longrightarrow \{\text{True}, \text{False}\}$$

In accordance with course guidelines, **no Generative AI or black-box neural networks** are used for the final inference. All reasoning is executed deterministically using classical algorithms implemented from first principles.

---

## 🧠 Reasoning Algorithms Implemented

1. **Forward Chaining with Modus Ponens:**
   - Starts from ground facts and derives consequences iteratively until fixed-point closure is reached.
2. **Goal-Directed Backward Chaining (DFS):**
   - Resolves subgoals from query to premises with cycle / loop detection to prevent infinite recursion on cyclic rules.
3. **Robinson's Most General Unification (MGU):**
   - Full occurs-check with variable standardization apart (`x_1`, `x_2`, ...) to eliminate variable capture.
4. **First-Order Resolution Refutation:**
   - Negates the query, converts to Conjunctive Normal Form (CNF), and derives the empty clause ($\Box$) by contradiction.
5. **Causal Forward Chaining under Interventions:**
   - Handles counterfactual queries (*"Would Y occur if not X instead of X?"*) via intervention operations on causal dependency DAGs.

---

## 📊 Empirical Evaluation Results (All 7 Benchmark Splits)

The reasoning agent was evaluated across **4,029 instances** spanning all required public benchmark datasets (**RuleTaker**, **FOLIO**, and **CounterBench**):

| Strict input | Rows | Accuracy | All-False Baseline | Coverage | Time (s) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **RuleTaker train** | 1000 | **98.50%** | 40.60% | 100.00% | 0.49 s |
| **RuleTaker test** | 250 | **98.00%** | 37.60% | 100.00% | 0.11 s |
| **RuleTaker OOD** | 375 | **99.20%** | 65.07% | 100.00% | 0.18 s |
| **FOLIO train\*** | 1001 | **78.12%** | 61.24% | 78.42% | 0.18 s |
| **FOLIO validation\*** | 203 | **81.28%** | 64.53% | 71.43% | 0.03 s |
| **CounterBench alpha** | 1000 | **91.80%** | 50.00% | 84.60% | 0.07 s |
| **CounterBench backdoor** | 200 | **88.50%** | 52.50% | 94.00% | 0.01 s |
| **TOTAL** | **4,029** | — | — | — | **~1.07 s** |

*\*Note: FOLIO is evaluated on the decidable First-Order Logic Horn-clause fragment.*

### 🎯 Key Validation Metrics (for Submission Form)
- **FOLIO Validation Accuracy:** `81.28%` (Inference Time: `0.03s` / `30 ms` across 203 rows, `~0.15 ms/query`)
- **RuleTaker Test Accuracy:** `98.00%` (Inference Time: `0.11s` / `110 ms` across 250 rows, `~0.44 ms/query`)

---

## 📂 Repository Structure

```text
├── README.md               # Project documentation, methodology, and results
├── README.txt              # Plaintext submission README
├── reasoning_agent.ipynb   # Complete self-contained notebook with pre-run outputs & traces
├── report.pdf              # Academic report explaining design, proofs, and metrics
├── submission.csv          # Pre-computed predictions for all 4,029 benchmark rows
└── data/                   # Benchmark datasets (RuleTaker, FOLIO, CounterBench)
    ├── ruletaker_train_1000.json
    ├── ruletaker_test_250.json
    ├── ruletaker_ood_375.json
    ├── folio_v2_train.jsonl
    ├── folio_v2_validation.jsonl
    ├── data_balanced_alpha_V1.json
    └── data_balanced_backdoor_V2.json
```

---

## 🚀 How to Run

### 1. In Jupyter Notebook / Google Colab
Open [`reasoning_agent.ipynb`](reasoning_agent.ipynb). All cells are pre-executed with outputs, accuracy tables, and full proof derivation trees. You can click **Run All** to reproduce all results from scratch.

### 2. Evaluating on Unseen Private Test Datasets
The notebook exposes a plug-and-play evaluation interface:

```python
from reasoning_agent import predict   # or run the final notebook cell

# Example single query:
context = "Tommy is a dog . All dogs are faithful"
query = "Tommy is faithful"

result = predict(context, query)
print(result)  # Output: True
```

For batch evaluation of private test sets:
```python
import json

with open("private_test.json", "r") as f:
    test_cases = json.load(f)

predictions = [predict(item["context"], item["query"]) for item in test_cases]
```
