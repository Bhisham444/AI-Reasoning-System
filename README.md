# Classical AI Reasoning System: Entailment over Knowledge Bases

**Course / Subject:** Artificial Intelligence 
**Name:** Bhisham Sharma  
**Roll Number:** `M26MAC003`  

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
| **RuleTaker train** | 1000 | **98.50%** | 40.60% | 100.00% | 1.17 s |
| **RuleTaker test** | 250 | **98.00%** | 37.60% | 100.00% | 0.29 s |
| **RuleTaker OOD** | 375 | **99.20%** | 65.07% | 100.00% | 0.46 s |
| **FOLIO train\*** | 1001 | **78.12%** | 61.24% | 78.42% | 0.46 s |
| **FOLIO validation\*** | 203 | **81.28%** | 64.53% | 71.43% | 0.10 s |
| **CounterBench alpha** | 1000 | **91.80%** | 50.00% | 84.60% | 0.19 s |
| **CounterBench backdoor** | 200 | **88.50%** | 52.50% | 94.00% | 0.03 s |
| **Average** | **4,029** | 90.77% | — | — | ** ** |

*\*Note: FOLIO is evaluated on the decidable First-Order Logic Horn-clause fragment.*

### 🎯 Key Validation Metrics
- **FOLIO Validation Accuracy:** `81.28%`
- **RuleTaker Test Accuracy:** `98.00%` 

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
# private data checker on model:
import csv, json, re, time, os

# private data
file_path = "private_test.csv"   


# CSV LOader
def load_dataset(path):
    path = str(path).strip().strip('"\'')
    # Google Colab path check
    if not os.path.exists(path):
        for candidate in [path, f"/content/{path}", os.path.basename(path)]:
            if os.path.exists(candidate):
                path = candidate
                break

    with open(path, "r", encoding="utf-8") as f:
        first_line = f.readline().strip()
        f.seek(0)
        full_content = f.read().strip()

    
    if first_line.startswith("{") or first_line.startswith("["):
        try:
            content = json.loads(full_content)
            if isinstance(content, dict):
                for k in ["data", "examples", "rows", "instances"]:
                    if k in content and isinstance(content[k], list):
                        return content[k]
                return [content]
            return content
        except Exception:
            rows = []
            for line in full_content.splitlines():
                line = line.strip()
                if line:
                    try:
                        rows.append(json.loads(line))
                    except Exception:
                        pass
            if rows:
                return rows

    # 2. If CSV /JSON
    with open(path, "r", encoding="utf-8") as f:
        delimiter = "\t" if "\t" in first_line and "," not in first_line else ","
        reader = csv.DictReader(f, delimiter=delimiter)
        rows = list(reader)
        if rows:
            return rows

    
    return [json.loads(l) for l in full_content.splitlines() if l.strip()]

rows = load_dataset(file_path)
print(f"Loaded {len(rows)} rows from: {file_path}")



# STEP 3: RULETAKER SCHEME LOGIC 

def parse_scheme_clause(s):
    tokens = re.findall(r'"([^"]+)"', s)
    if not tokens:
        return None
    subj = tokens[0].lower()
    is_var = subj in ["someone", "something", "x", "y", "z"]
    term = Term("?x" if is_var else subj, is_var=is_var)
    pos = (tokens[-1] == "+")
    if len(tokens) == 4 and tokens[1] == "is":
        pred = tokens[2].lower()
        return Literal(pred, [term], positive=pos)
    elif len(tokens) == 4:
        pred = tokens[1].lower()
        obj = tokens[2].lower()
        is_obj_var = obj in ["someone", "something", "x", "y", "z"]
        obj_term = Term("?y" if is_obj_var else obj, is_var=is_obj_var)
        return Literal(pred, [term, obj_term], positive=pos)
    return None

def solve_scheme_ruletaker(ctx_logic, qry_logic):
    kb = KnowledgeBase()
    for line in ctx_logic.split("\n"):
        line = line.strip()
        if not line: continue
        if "->" in line:
            body_part, head_part = line.split("->", 1)
            clauses = re.findall(r"\([^)]+\)", body_part)
            body_lits = [parse_scheme_clause(c) for c in clauses if parse_scheme_clause(c)]
            head_lit = parse_scheme_clause(head_part)
            if head_lit and body_lits:
                kb.add_rule(Rule(head=head_lit, body=body_lits))
        else:
            lit = parse_scheme_clause(line)
            if lit:
                kb.add_fact(lit)
                
    q_lit = parse_scheme_clause(qry_logic)
    if not q_lit:
        return False
        
    pos_lit = Literal(q_lit.name, q_lit.args, positive=True)
    proved, _ = agent.fc.prove(pos_lit, kb)
    if not proved:
        proved, _, _ = agent.bc.prove(pos_lit, kb)
    return (not proved) if not q_lit.positive else proved



# EVALUATION LOOP 

correct = 0
total_evaluated = 0
results = []
start_time = time.time()

for r in rows:
    row_id = r.get("id", f"row_{total_evaluated+1}")
    ctx = r.get("context") or r.get("story") or r.get("premises") or ""
    if isinstance(ctx, list):
        ctx = " ".join(str(x) for x in ctx)
    qry = r.get("statement") or r.get("query") or r.get("hypothesis") or r.get("question") or ""
    ctx_logic = r.get("context_logic") or r.get("premises_fol") or ""
    qry_logic = r.get("statement_logic") or r.get("conclusion_fol") or ""
    
    raw_label = r.get("label") or r.get("answer") or r.get("target") or r.get("gold_label")
    has_gold = raw_label is not None and str(raw_label).strip() != ""
    exp = str(raw_label).strip().lower() in ["true", "1", "yes", "t"] if has_gold else None

    # Calling my model:
    if "causes" in ctx.lower() or "would" in qry.lower():
        pred, _ = agent.solve_counterbench(ctx, qry)
    elif '("' in ctx_logic or '(""' in ctx_logic:
        pred = solve_scheme_ruletaker(ctx_logic, qry_logic)
    elif ("∀" in ctx_logic or "∃" in ctx_logic or "→" in ctx_logic) and qry_logic:
        pred, _ = agent.solve_folio({"premises_fol": ctx_logic, "conclusion_fol": qry_logic})
    else:
        pred = predict(ctx, qry)

    results.append({"id": row_id, "predicted_label": pred})
    total_evaluated += 1
    
    if has_gold and pred == exp:
        correct += 1

total_time = time.time() - start_time


# STEP 5: OUTPUT RESULT

print("=" * 60)
print("EVALUATION COMPLETED SUCCESSFULLY")
print("=" * 60)
print(f"Total Rows Evaluated : {total_evaluated}")
if total_evaluated > 0 and raw_label is not None:
    print(f"Correct Predictions  : {correct} / {total_evaluated}")
    print(f"Accuracy             : {correct / total_evaluated * 100:.2f}%")
print(f"Total Time Taken     : {total_time:.4f} seconds")
if total_evaluated > 0:
    print(f"Inference Latency    : {total_time / total_evaluated * 1000:.2f} ms / query")
print("=" * 60)

out_file = "private_predictions.csv"
with open(out_file, "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["id", "predicted_label"])
    writer.writeheader()
    writer.writerows(results)
print(f"Predictions successfully saved to: {out_file}")
