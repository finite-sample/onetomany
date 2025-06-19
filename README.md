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

