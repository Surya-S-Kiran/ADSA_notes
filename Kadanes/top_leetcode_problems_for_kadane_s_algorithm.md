# Top 10 LeetCode Problems Using Kadane's Algorithm

Ranked in descending order by algorithmic relevance and structural match to the canonical boundary-anchored dynamic programming / greedy paradigm.

| Rank | Match % | Problem # | Problem Title | Kadane's Core Role & Relationship | 
 | ----- | ----- | ----- | ----- | ----- | 
| **1** | **100%** | **53** | **Maximum Subarray** | **Canonical Implementation:** Directly computes $dp[i] = \max(A[i], dp[i-1] + A[i])$. Baseline problem for the paradigm. | 
| **2** | **98%** | **918** | **Maximum Sum Circular Subarray** | **Dual Kadane (Max & Min):** The circular optimal is either non-wrapped ($\text{KadaneMax}$) or wraps around the edges ($\text{TotalSum} - \text{KadaneMin}$). | 
| **3** | **95%** | **1191** | **K-Concatenation Maximum Sum** | **Concatenated Kadane:** Executes Kadane over $k=1$ and $k=2$ repetitions; handles full-array positive accumulations for $k > 2$ modulo $10^9 + 7$. | 
| **4** | **92%** | **152** | **Maximum Product Subarray** | **Multiplicative Dual-State Kadane:** Maintains both running maximum and running minimum states at each step to handle sign inversions from negative multipliers. | 
| **5** | **90%** | **1749** | **Maximum Absolute Sum of Any Subarray** | **Simultaneous Dual Kadane:** Tracks the maximum positive subarray sum and the minimum negative subarray sum concurrently; returns $\max(\text{max\_sum}, \vert{}\text{min\_sum}\vert{})$. | 
| **6** | **88%** | **2321** | **Maximum Score Of Spliced Array** | **Difference Array Reduction:** Transforming the gain of swapping a subarray into $D[i] = B[i] - A[i]$ reduces finding the optimal swap segment directly to Kadane on $D$. | 
| **7** | **85%** | **1186** | **Maximum Subarray Sum with One Deletion** | **Multi-State Extended Kadane:** Tracks two transitions at each boundary: $dp[i][0]$ (no deletions utilized) and $dp[i][1]$ (one element omitted). | 
| **8** | **82%** | **363** | **Max Sum of Rectangle No Larger Than K** | **2D Kadane / Row-Compression:** Compresses 2D submatrices between row pairs into a 1D column-sum array; runs Kadane (with binary search / ordered set for the $\le K$ bound). | 
| **9** | **80%** | **2606** | **Find the Substring With Maximum Cost** | **Value-Mapped Canonical Kadane:** Maps string characters to custom integer weights, then applies standard linear Kadane to maximize contiguous cost. | 
| **10** | **75%** | **121** | **Best Time to Buy and Sell Stock** | **Isomorphic Difference-Array Kadane:** Converting stock prices into daily changes $\Delta_i = P[i] - P[i-1]$ makes single-transaction profit equivalent to Kadane on $\Delta$. | 

## Detailed Problem Breakdown & State Formulations

### 1. LeetCode 53: Maximum Subarray (100% Match)

* **Recurrence:**
  

  $$
  dp[i] = \max(A[i], dp[i-1] + A[i])
  $$

* **Space-Optimized:**
  

  $$
  \text{cur} = \max(A[i], \text{cur} + A[i]); \quad \text{ans} = \max(\text{ans}, \text{cur})
  $$

* **Role:** The benchmark implementation of the single-state running boundary accumulator.

### 2. LeetCode 918: Maximum Sum Circular Subarray (98% Match)

* **Core Insight:** In a circular buffer of length $n$, the optimal contiguous slice either:

  1. Does not wrap: $\text{KadaneMax}(A)$.

  2. Wraps around index $0$: The unselected elements form a contiguous **minimum** subarray in the interior.

* **Recurrence / Formulation:**
  

  $$
  \text{ans} = \begin{cases} \text{KadaneMax}(A), & \text{if } \text{TotalSum} = \text{KadaneMin}(A) \ (\text{all negative}) \\ \max(\text{KadaneMax}(A), \text{TotalSum} - \text{KadaneMin}(A)), & \text{otherwise} \end{cases}
  $$

### 3. LeetCode 1191: K-Concatenation Maximum Sum (95% Match)

* **Core Insight:** An array repeated $k$ times only needs explicit Kadane analysis on $k=1$ and $k=2$.

* **Formulation:**

  * For $k = 1$: $\text{Kadane}(A)$.

  * For $k \ge 2$:

    * If $\text{TotalSum} \le 0$: The optimal segment is bounded within two concatenated copies $\implies \text{Kadane}(A + A)$.

    * If $\text{TotalSum} > 0$: Span all intermediate full arrays $\implies \text{Kadane}(A + A) + (k - 2) \cdot \text{TotalSum} \pmod{10^9 + 7}$.

### 4. LeetCode 152: Maximum Product Subarray (92% Match)

* **Core Insight:** Negative numbers invert extrema ($-\infty \leftrightarrow +\infty$). A single scalar accumulator is insufficient; both the maximum and minimum products ending at $i$ must be retained.

* **Recurrence:**
  

  $$
  dp_{\max}[i] = \max\Big(A[i], \max(dp_{\max}[i-1] \cdot A[i], dp_{\min}[i-1] \cdot A[i])\Big)
  $$

  $$
  dp_{\min}[i] = \min\Big(A[i], \min(dp_{\max}[i-1] \cdot A[i], dp_{\min}[i-1] \cdot A[i])\Big)
  $$

### 5. LeetCode 1749: Maximum Absolute Sum of Any Subarray (90% Match)

* **Core Insight:** Because the objective is $\max \left\vert{} \sum_{k=i}^j A[k] \right\vert{}$, the optimal subarray is either the most positive subarray or the most negative subarray.

* **Formulation:**
  Execute two simultaneous Kadane passes in $\mathcal{O}(1)$ space:
  

  $$
  \text{cur}_{\max} = \max(A[i], \text{cur}_{\max} + A[i]); \quad \text{ans}_{\max} = \max(\text{ans}_{\max}, \text{cur}_{\max})
  $$

  $$
  \text{cur}_{\min} = \min(A[i], \text{cur}_{\min} + A[i]); \quad \text{ans}_{\min} = \min(\text{ans}_{\min}, \text{cur}_{\min})
  $$

  $$
  \text{Result} = \max(\text{ans}_{\max}, \vert{}\text{ans}_{\min}\vert{})
  $$

### 6. LeetCode 2321: Maximum Score Of Spliced Array (88% Match)

* **Core Insight:** Swapping a subarray $A[l \dots r]$ with $B[l \dots r]$ changes the sum of $A$ by $\sum_{k=l}^r (B[k] - A[k])$.

* **Reduction:**

  1. Define difference array $D[i] = B[i] - A[i]$.

  2. The maximum boost to $A$'s total sum is simply $\text{Kadane}(D)$.

  3. Symmetrically, the maximum boost to $B$'s total sum is $\text{Kadane}(-D)$.

### 7. LeetCode 1186: Maximum Subarray Sum with One Deletion (85% Match)

* **Core Insight:** State space splits into two boundary conditions depending on whether a deletion has been consumed.

* **Recurrence:**

  * $dp[i][0]$: Maximum subarray sum ending at $i$ with **0 deletions**:
    

    $$
    dp[i][0] = \max(A[i], dp[i-1][0] + A[i])
    $$

  * $dp[i][1]$: Maximum subarray sum ending at $i$ with **1 deletion**:
    

    $$
    dp[i][1] = \max(dp[i-1][0], dp[i-1][1] + A[i])
    $$

    
    *(First term deletes* $A[i]$*; second term carries forward a deletion performed earlier).*

### 8. LeetCode 363: Max Sum of Rectangle No Larger Than K (82% Match)

* **Core Insight:** Fix the top row $r_1$ and bottom row $r_2$. Compress 2D columns into a 1D vector:
  

  $$
  C[c] = \sum_{r=r_1}^{r_2} \text{matrix}[r][c]
  $$

* **Reduction:** For unconstrained max 2D submatrix, Kadane runs on $C$ in $\mathcal{O}(W)$ time. Under the $\le K$ constraint, Kadane serves as the outer framing, combined with a prefix sum BST ($P[j] - P[i] \le K$).

### 9. LeetCode 2606: Find the Substring With Maximum Cost (80% Match)

* **Core Insight:** Direct isomorphic transformation of strings into signed integer arrays.

* **Formulation:** Map each character $s[i]$ to its explicit cost $w(s[i])$. Then execute standard Kadane:
  

  $$
  dp[i] = \max\Big(w(s[i]), dp[i-1] + w(s[i])\Big)
  $$

### 10. LeetCode 121: Best Time to Buy and Sell Stock (75% Match)

* **Core Insight:** Finding $\max_{j > i} (P[j] - P[i])$ can be rephrased via telescoping sums:
  

  $$
  P[j] - P[i] = \sum_{k=i+1}^j (P[k] - P[k-1])
  $$

* **Reduction:** Define daily delta $\Delta_k = P[k] - P[k-1]$. Finding the maximum profit of a single buy/sell operation is mathematically identical to running Kadane's algorithm on the sequence $\Delta$.