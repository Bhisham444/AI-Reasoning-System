================================================================================
AI REASONING SYSTEM: ENTAILMENT OVER KNOWLEDGE BASES
================================================================================
Subject: Artificial Intelligence
Name: Bhisham Sharma
Roll Number: M26MAC003

--------------------------------------------------------------------------------
OVERVIEW
--------------------------------------------------------------------------------
In this assignment, I built an AI reasoning system to determine whether natural
language queries logically follow (True/False) from given facts and rules.

In accordance with the course assignment guidelines, all logical inferences
are conducted deterministically through classical symbolic reasoning algorithms
covered in the curriculum:
- Forward Chaining with Modus Ponens
- Goal-Directed Backward Chaining (DFS)
- First-Order Resolution Refutation
- Most General Unification (Robinson's MGU)

The system is implemented from first principles in pure Python using standard
data structures, translating natural language statements into First-Order Logic
Horn clauses, facts, and causal dependency rules.

--------------------------------------------------------------------------------
1. SUBMISSION STRUCTURE
--------------------------------------------------------------------------------
submission/
|-- reasoning_agent.ipynb   : Complete Jupyter Notebook containing code, benchmark
|                             results table, and step-by-step reasoning traces.
|-- report.pdf              : Academic report (3 pages) explaining representation,
|                             algorithms, full 7-dataset evaluation, and traces.
|-- submission.csv          : Predictions for all 4,029 instances across all 7 splits.
|-- README.txt              : This documentation file.
`-- data/                   : Folder containing all 7 benchmark dataset splits
                              (RuleTaker, FOLIO, CounterBench) for plug-and-play execution.

--------------------------------------------------------------------------------
2. SYSTEM PIPELINE & ALGORITHMS
--------------------------------------------------------------------------------
The system follows the pipeline specified in the assignment:
Natural Language -> Representation -> Facts + Rules -> Classical Reasoning -> Prediction

A. KNOWLEDGE REPRESENTATION:
   1. Terms: Constants ('bob', 'river', 'teacher') and Variables ('x', 'y').
   2. Literals: Predicates with positive or classical negative polarity
      (e.g., Dances_gracefully(wall), Quiet(teacher), not Plant(x)).
   3. Definite Horn Clauses (Rules): Body1 ^ Body2 ^ ... -> Head.
   4. Conjunctive Normal Form (CNF): Clauses of disjunctions for resolution refutation.
   5. Causal Dependency Rules: Directed cause-effect rules between variables.

B. REASONING ENGINES:
   1. Forward Chaining with Modus Ponens:
      Starts from known ground facts and repeatedly applies Modus Ponens. If a rule's
      conditions match known facts, its head is derived and added to the knowledge
      base until the query is proved or fixed-point closure is reached.
   2. Goal-Directed Backward Chaining (DFS):
      Starts from the target query goal and searches backward using Depth-First Search
      (DFS), matching rule heads and creating subgoals with cycle detection.
   3. First-Order Resolution Refutation:
      Adds the negated query (not Q) to the CNF knowledge base and resolves complementary
      literals to derive the empty clause (proof by contradiction).
   4. Causal Rule Forward Chaining under Interventions:
      For counterfactual queries ("Would Y occur if not X?"), condition X is set to False,
      and forward chaining traces whether target Y remains derivable.

C. NATURAL LANGUAGE PARSER:
   - Parses standard English rule formulations into Horn clauses:
     * "If X, then Y"
     * "Assuming X, we can say Y"
     * "Based on X, it is evident that Y"
     * "When X, we conclude Y"
   - Handles multi-word predicates (e.g., "dances gracefully", "fishes in lakes")
     and universal quantifiers ("All dogs are faithful").

--------------------------------------------------------------------------------
3. BENCHMARK EVALUATION RESULTS
--------------------------------------------------------------------------------
I evaluated my reasoning agent on all 7 public benchmark dataset splits spanning
4,029 total instances. Below are the empirical results:

==========================================================================================
Strict input                   Rows   Accuracy   All-False   Coverage   Time (s)
------------------------------------------------------------------------------------------
RuleTaker train                1000     98.50%      40.60%    100.00%       1.17
RuleTaker test                  250     98.00%      37.60%    100.00%       0.29
RuleTaker OOD                   375     99.20%      65.07%    100.00%       0.46
FOLIO train*                   1001     78.12%      61.24%     78.42%       0.46
FOLIO validation*               203     81.28%      64.53%     71.43%       0.10
CounterBench alpha             1000     91.80%      50.00%     84.60%       0.19
CounterBench backdoor           200     88.50%      52.50%     94.00%       0.03
==========================================================================================
TOTAL ROWS EVALUATED:          4029 instances

Key Observations:
- RuleTaker achieves 98.00% - 99.20% accuracy across multi-hop reasoning.
- CounterBench achieves 91.80% on alpha and 88.50% on backdoor through causal forward chaining.
- FOLIO achieves 78.12% on train and 81.28% on validation, beating the All-False baseline
  via first-order resolution and backward chaining.

--------------------------------------------------------------------------------
4. HOW TO RUN & EVALUATE ON PRIVATE DATASETS
--------------------------------------------------------------------------------
1. Running in Jupyter Notebook / Google Colab:
   - Open 'reasoning_agent.ipynb' in JupyterLab, Jupyter Notebook, or Google Colab.
   - All cells are already pre-executed with complete outputs, tables, and traces.
   - Running all cells reproduces the evaluation across all 7 dataset splits.

2. Evaluating on Unseen Private Test Datasets:
   The notebook defines a self-contained inference function in the final cell:
       predict(context: str, query: str) -> bool

   To evaluate an unseen test dataset (JSON/CSV), simply iterate over the items:
   --------------------------------------------------------------------------
   import json

   with open('private_test.json') as f:
       test_data = json.load(f)

   for item in test_data:
       prediction = predict(item['context'], item['query'])  # Returns True/False
       # Evaluate accuracy against item['label']
   --------------------------------------------------------------------------

   The predict() function automatically handles:
   - Multi-hop rules and facts (RuleTaker format)
   - Causal and counterfactual intervention queries ("Would Y occur if not X?")
   - General First-Order statements with universal quantifiers ("All dogs are faithful")
   - Query negations ("is not", "will not", "cannot", "never")

3. Quick Single-Query Verification:
   predict("Tommy is a dog . All dogs are faithful", "Tommy is faithful")      # -> True
   predict("Tommy is a dog . All dogs are faithful", "Tommy is not faithful")  # -> False
================================================================================
