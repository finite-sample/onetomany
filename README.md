# onetomany

> An extension of the Kuhn-Munkres algorithm (Hungarian method) for 1-to-many matching in causal inference.

The Kuhn-Munkres algorithm—better known as the Hungarian algorithm—solves the classic assignment problem: finding the optimal 1-to-1 matching that minimizes total cost. It's efficient and deterministic. But in causal inference, especially when estimating the Average Treatment effect on the Treated (ATT), strict 1-to-1 matching is often too restrictive.

When control units outnumber treated units, 1-to-many matching can improve covariate balance and reduce variance. `onetomany` generalizes the Hungarian algorithm by formulating the matching problem as a **min-cost flow** network:

## 🔧 How We Implement 1-to-Many Matching

We construct a directed bipartite flow graph:

* **Nodes**:

  * One node for each treated unit
  * One node for each control unit
  * A source and a sink node

* **Edges**:

  * Source to treated units: capacity = *k*, cost = 0
  * Treated to control units: capacity = 1, cost = Euclidean distance (scaled)
  * Control units to sink: capacity = 1, cost = 0

* **Constraints**:

  * Each treated unit is matched to exactly *k* controls
  * Each control unit can be matched at most once (no replacement)

* **Solution**: We solve for the **min-cost maximum flow** using `networkx.max_flow_min_cost()`.

This approach preserves **global optimality** like the original Hungarian method, but allows each treated unit to be optimally matched to multiple control units.

## 📊 What It Does

* ♻️ Implements standard Hungarian 1-to-1 matching via `scipy.optimize.linear_sum_assignment`
* ⚐ Implements 1-to-*k* optimal matching using network flow
* 📈 Estimates ATT and computes covariate balance
* 🚪 Compares performance in simulations

## Notebook

[Notebook](hungarian-one-to-many.ipynb)

Brief Simulation Results:

```
True ATT:           2.0000
Hungarian 1-to-1:   2.0284 | Mean cov. diff: 0.2317
Hungarian 1-to-3:   2.0151 | Mean cov. diff: 0.1683
```

## Additional Results

* [Notebook](hungarian-one-to-many-alternates.ipynb)

### Additional Methods

### 1-to-1 Hungarian (Baseline)
The standard Hungarian algorithm solves the assignment problem optimally by finding minimum-cost perfect matchings in bipartite graphs. Implementation uses `scipy.optimize.linear_sum_assignment` on an (n_treated × n_controls) cost matrix where entries are Euclidean distances between covariate vectors. This serves as the baseline, guaranteeing each treated unit gets exactly one control with globally optimal total matching cost.

### Min-Cost Max-Flow
Transforms the 1-to-k problem into a network flow formulation. Constructs a directed graph with source connected to treated units (edges with capacity k), treated units connected to controls (capacity 1, weight = distance × 10⁶), and controls connected to sink (capacity 1). The min-cost max-flow solution via `networkx.max_flow_min_cost` finds the globally optimal 1-to-k assignment by maximizing flow while minimizing total cost. Edge weights are scaled to integers as required by the NetworkX implementation.

### Sequential Hungarian  
Applies the Hungarian algorithm iteratively to approximate the 1-to-k solution. In each of k rounds, runs standard Hungarian on the current (n_treated × n_remaining_controls) cost matrix, stores the optimal 1-to-1 matching, then removes matched controls from consideration. This greedy approach reuses existing Hungarian code with minimal modification and achieves O(k × n³) complexity. While not globally optimal, it provides strong local optimality guarantees since each round finds the best possible assignment given remaining controls.

### Graph Duplication
Converts the 1-to-k problem into an equivalent 1-to-1 problem through graph transformation. Creates k identical copies of each treated unit, forming an expanded (k×n_treated × n_controls) cost matrix by tiling the original cost matrix k times vertically. Applies standard Hungarian to this expanded matrix, then maps results back to original treated units. This method is theoretically elegant as it preserves the Hungarian algorithm exactly while guaranteeing global optimality - the solution is provably equivalent to the min-cost max-flow formulation.

### Greedy k-NN (Baseline)
Simple nearest neighbor matching that assigns each treated unit its k closest controls by Euclidean distance without replacement. Provides a non-optimal baseline to demonstrate the value of optimization-based approaches. While computationally efficient at O(n² log n), it lacks theoretical guarantees and typically produces inferior balance and bias compared to Hungarian-based methods.

## Results Summary

**Dataset:** 50 treated, 150 controls, 5 covariates  
**True ATT:** 2.0944

| Method | ATT Estimate | Bias | Bias % | Balance | Runtime (s) |
|--------|--------------|------|--------|---------|-------------|
| 1-to-1 Hungarian | 2.4538 | +0.3594 | 17.2% | 0.3828 | 0.000 |
| **1-to-3 Flow** | **2.2553** | **+0.1609** | **7.7%** | **0.3580** | **0.273** |
| **1-to-3 Sequential** | **2.2553** | **+0.1609** | **7.7%** | **0.3728** | **0.001** |
| **1-to-3 Duplication** | **2.2553** | **+0.1609** | **7.7%** | **0.3580** | **0.001** |
| 1-to-3 Greedy kNN | 2.2553 | +0.1609 | 7.7% | 0.4067 | 0.002 |



