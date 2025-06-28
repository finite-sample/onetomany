# One Too Many?

The Hungarian algorithm (Kuhn–Munkres) efficiently finds optimal one-to-one matches between treated and control units by minimizing the total matching cost (typically, Euclidean distance in covariate space). It is widely used for estimating treatment effects via matching.  
However, its main limitation is that it only allows strictly one-to-one matching. In many causal inference applications—especially when estimating the Average Treatment Effect on the Treated (ATT)—the pool of controls is much larger than the number of treated units. Restricting each treated unit to a single match can waste useful data and limit statistical efficiency.

## One-to-One or One-too-Many?

The choice between one-to-one and one-to-many matching hinges on a classic trade-off between bias and variance.  
- **One-to-many matching** can be beneficial when the control pool is large relative to the treated group, when outcomes are noisy and benefit from averaging, or when match quality degrades slowly (i.e., second and third best matches are nearly as good as the first). In these settings, reducing variance by including more matches outweighs the added bias from slightly worse controls.
- **One-to-one matching** is preferred when the quality of additional matches drops sharply, when there are abundant high-quality controls, or when you require highly precise individual counterfactuals due to heterogeneous treatment effects.  
The optimal approach depends on factors like the control-to-treated ratio, covariate overlap, and outcome noise, and can be informed by examining the distribution of match qualities and conducting sensitivity analyses across different matching ratios.

## How Many Controls? (Choosing \(k\))

Selecting the number of matches (\(k\)) for each treated unit is challenging, especially without experimental benchmark data. In practice, a few strategies are commonly used:

- **Cross-validation:** Hold out a subset of controls, pretend they are "treated," match them to other controls, and measure prediction error on their outcomes as a function of \(k\). This provides a data-driven proxy for match quality.
- **Caliper-based selection:** Instead of fixing \(k\), set a maximum acceptable distance (the "caliper") based on covariate balance, domain knowledge, or rules of thumb (e.g., 0.2 SD of the propensity score). Use all controls within this threshold for matching.

## Algorithms for 1-to-\(k\) Matching

There are several approaches to implement optimal or approximate 1-to-many matching:

### 1. Min-Cost Max-Flow (Optimal 1-to-\(k\))
This generalizes the assignment problem using min-cost max-flow algorithms. Each treated unit supplies \(k\) units, each control can be used at most once, and edges are weighted by matching costs. Solving the network flow problem yields a globally optimal assignment under all constraints. This is well-established and implemented in R’s `optmatch`, Python’s `networkx`, and other optimization libraries ([Kallus 2020](https://jmlr.org/papers/volume21/19-120/19-120.pdf)).

### 2. Sequential Hungarian (Greedy, Iterative)
A more practical, though heuristic, approach is to run the Hungarian algorithm iteratively: in each of \(k\) rounds, solve the 1-to-1 problem, record matches, and remove matched controls. This is locally optimal at each step and straightforward to implement, but does not guarantee a globally optimal solution. Variants are used for quick matching in large datasets ([Abadie & Imbens 2016](https://www.jstor.org/stable/43896414)).

### 3. Graph Duplication (“Blow-up” Hungarian)
A theoretically elegant, but rarely used, method duplicates each treated unit \(k\) times, creating an expanded cost matrix. Standard Hungarian is then run on this larger matrix, and the resulting matches are mapped back to the original units to provide a globally optimal 1-to-\(k\) assignment. Although mathematically equivalent to the min-cost flow solution, this approach is limited by computational scalability when \(k\) or sample size is large. For moderate \(k\), it is a clean and optimal solution.

---

*References:*  
- Hansen, B.B. (2007). Flexible, Optimal Matching for Observational Studies. *The American Statistician.*  
- Stuart, E.A. (2010). Matching Methods for Causal Inference: A Review and a Look Forward. *Statistical Science.*  
- Kallus, N. (2020). Generalized Optimal Matching Methods for Causal Inference. *JMLR.*  
- Abadie, A. & Imbens, G. (2016). Matching on the Estimated Propensity Score. *Econometrica.*
