# Master Notes: Kadane's Algorithm & The Maximum Subarray Problem

**Derivation from First Principles: From Brute Force to** $\mathcal{O}(n)$ **Time,** $\mathcal{O}(1)$ **Space**

## 1. Problem Definition

Let $A = [a_0, a_1, \dots, a_{n-1}]$ be a contiguous array of $n$ signed integers. A subarray is defined as any contiguous subsequence $A[i \dots j] = [a_i, a_{i+1}, \dots, a_j]$ where $0 \le i \le j < n$.

* **Given:** An array of integers $A \in \mathbb{Z}^n$ with $n \ge 1$.

* **Target:** Compute the maximum sum achievable over all non-empty contiguous subarrays:
  

  $$
  \text{MaxSubarraySum}(A) = \max_{0 \le i \le j < n} \sum_{k=i}^{j} a_k
  $$

* **Constraints:**

  * Subarrays must be contiguous: $A[0 \dots 1]$ is valid; choosing $A[0]$ and $A[2]$ without $A[1]$ is invalid.

  * Subarrays must be non-empty ($j \ge i$). If all elements are negative, the maximum sum corresponds to the single largest (least negative) element.

* **Representative Examples:**

  * $A = [-2, 1, -3, 4, -1, 2, 1, -5, 4]$: The optimal contiguous segment is $A[3 \dots 6] = [4, -1, 2, 1]$, yielding $\sum = 6$.

  * $A = [-8, -3, -6, -2, -5]$: The optimal non-empty segment is $A[3 \dots 3] = [-2]$, yielding $\sum = -2$.

## 2. Brute-Force Thinking

The total number of non-empty contiguous subarrays is equal to the number of pairs $(i, j)$ satisfying $0 \le i \le j < n$:

$$
\sum_{i=0}^{n-1} \sum_{j=i}^{n-1} 1 = \frac{n(n+1)}{2} = \Theta(n^2)
$$

The most direct formulation iterates through every valid pair $(i, j)$ and computes the subarray sum by explicit iteration:

```
long long max_sum = -INF;
for (int i = 0; i < n; ++i) {
    for (int j = i; j < n; ++j) {
        long long current_sum = 0;
        for (int k = i; k <= j; ++k) {
            current_sum += A[k];
        }
        max_sum = std::max(max_sum, current_sum);
    }
}

```

* **Correctness:** It exhaustively enumerates the entire search space $\mathcal{S} = \{(i, j) \mid 0 \le i \le j < n\}$ and calculates the exact sum for every interval.

* **Complexity:**

  * Outer loops enumerate $\Theta(n^2)$ intervals.

  * The inner summation loop runs in $\mathcal{O}(n)$ time.

  * Total Time Complexity: $\sum_{i=0}^{n-1} \sum_{j=i}^{n-1} (j - i + 1) = \Theta(n^3)$.

  * Auxiliary Space Complexity: $\mathcal{O}(1)$.

## 3. Identify the Bottleneck

### 3.1 Redundant Summation

The inner loop recalculates $\sum_{k=i}^j A[k]$ from scratch. Notice that:

$$
\sum_{k=i}^{j} A[k] = \left(\sum_{k=i}^{j-1} A[k]\right) + A[j]
$$


By accumulating the sum iteratively as $j$ advances from $i$ to $n-1$, the inner $\mathcal{O}(n)$ scan is eliminated, reducing runtime to $\mathcal{O}(n^2)$ time and $\mathcal{O}(1)$ space.

### 3.2 Redundant Interval Alignment

Even at $\mathcal{O}(n^2)$, the algorithm evaluates overlapping subproblems independently. Consider fixing the right boundary $j$:

$$
\max_{0 \le i \le j} \sum_{k=i}^{j} A[k] = A[j] + \max \left(0, \max_{0 \le i \le j-1} \sum_{k=i}^{j-1} A[k]\right)
$$

The optimal subarray ending at index $j$ depends exclusively on the optimal subarray ending at index $j-1$. Evaluating every left boundary $i$ independently ignores this Markovian dependency structure across adjacent endpoints.

## 4. Search for a DP State

### 4.1 State Failure: Global Best Up to $i$

Let $F[i] = \max_{0 \le p \le q \le i} \sum_{k=p}^q A[k]$.

* Problem: If $F[i-1]$ represents the best subarray anywhere inside the prefix $A[0 \dots i-1]$, we cannot determine whether adding $A[i]$ forms a contiguous segment.

* $F[i-1]$ may reside entirely within $A[0 \dots i-3]$. To attach $A[i]$, elements $A[i-2]$ and $A[i-1]$ must also be included, which could degrade the sum. $F[i-1]$ lacks the boundary information needed to maintain contiguity.

### 4.2 State Formulation: Conditioning on the Boundary

To enforce contiguity, we constrain the subproblem to subarrays anchored at the current index:

$$
dp[i] = \text{maximum sum of a non-empty contiguous subarray ending strictly at index } i
$$

* **Sufficiency:** Any contiguous subarray ending at $i$ is either:

  1. Exactly the single element $A[i]$.

  2. A contiguous subarray ending at $i-1$, concatenated with $A[i]$.

* **Global Answer Reconstruction:** Every valid non-empty subarray in the array must terminate at some index $i \in [0, n-1]$. Therefore:
  

  $$
  \text{MaxSubarraySum}(A) = \max_{0 \le i < n} dp[i]
  $$

## 5. Derive the Recurrence

Let $S(i)$ denote the set of all non-empty subarrays ending at index $i$:

$$
S(i) = \{ A[k \dots i] \mid 0 \le k \le i \}
$$

We partition $S(i)$ into two mutually exclusive, exhaustive subsets:

1. **Size-1 Subarray:** $\{ A[i \dots i] \}$. The sum is identically $A[i]$.

2. **Size** $\ge 2$ **Subarrays:** $\{ A[k \dots i] \mid 0 \le k < i \}$.

For any subarray $A[k \dots i]$ with $k < i$:

$$
\sum_{m=k}^{i} A[m] = \left(\sum_{m=k}^{i-1} A[m]\right) + A[i]
$$

Taking the maximum over all $0 \le k < i$:

$$
\max_{0 \le k < i} \sum_{m=k}^{i} A[m] = \left( \max_{0 \le k \le i-1} \sum_{m=k}^{i-1} A[m] \right) + A[i] = dp[i-1] + A[i]
$$

Combining both partitions:

$$
dp[i] = \max(A[i], dp[i-1] + A[i])
$$

Factoring out $A[i]$:

$$
dp[i] = A[i] + \max(0, dp[i-1])
$$

* **Base Case:** For $i = 0$, the only non-empty subarray ending at index $0$ is $A[0 \dots 0]$:
  

  $$
  dp[0] = A[0]
  $$

## 6. Work Through a Complete Example

Let $A = [-2, 1, -3, 4, -1, 2, 1, -5, 4]$:

| $i$ | $A[i]$ | $dp[i-1]$ | Choices: $\max(A[i], dp[i-1] + A[i])$ | $dp[i]$ | $\max_{0 \le k \le i} dp[k]$ | Decision | 
 | ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| $0$ | $-2$ | N/A | Base case: $A[0]$ | $-2$ | $-2$ | Start at $0$ | 
| $1$ | $1$ | $-2$ | $\max(1, -2 + 1) = \max(1, -1)$ | $1$ | $1$ | Restart at $1$ | 
| $2$ | $-3$ | $1$ | $\max(-3, 1 + (-3)) = \max(-3, -2)$ | $-2$ | $1$ | Extend from $1$ | 
| $3$ | $4$ | $-2$ | $\max(4, -2 + 4) = \max(4, 2)$ | $4$ | $4$ | Restart at $3$ | 
| $4$ | $-1$ | $4$ | $\max(-1, 4 + (-1)) = \max(-1, 3)$ | $3$ | $4$ | Extend from $3$ | 
| $5$ | $2$ | $3$ | $\max(2, 3 + 2) = \max(2, 5)$ | $5$ | $5$ | Extend from $3$ | 
| $6$ | $1$ | $5$ | $\max(1, 5 + 1) = \max(1, 6)$ | $6$ | $6$ | Extend from $3$ | 
| $7$ | $-5$ | $6$ | $\max(-5, 6 + (-5)) = \max(-5, 1)$ | $1$ | $6$ | Extend from $3$ | 
| $8$ | $4$ | $1$ | $\max(4, 1 + 4) = \max(4, 5)$ | $5$ | $6$ | Extend from $3$ | 

Final result: $\max_i dp[i] = 6$, corresponding to $A[3 \dots 6] = [4, -1, 2, 1]$.

## 7. From DP to Optimization

### 7.1 Dependency Analysis

Examining the recurrence relation:

$$
dp[i] = \max(A[i], dp[i-1] + A[i])
$$

The computation of $dp[i]$ depends **strictly** on the immediately preceding state $dp[i-1]$. It does not reference $dp[i-2], dp[i-3], \dots, dp[0]$.

```
Full DP Table:
[ dp[0] ] ---> [ dp[1] ] ---> [ dp[2] ] ---> ... ---> [ dp[n-1] ]
   |              |              |                       |
   v              v              v                       v
                  Global Max Tracking: max(dp[k])

```

### 7.2 Memory Compression

Because past states $dp[0 \dots i-2]$ are never queried again, maintaining an array of size $n$ is redundant. We can compress storage into a single register:

$$
\text{current\_max} \leftarrow \max(A[i], \text{current\_max} + A[i])
$$

$$
\text{global\_max} \leftarrow \max(\text{global\_max}, \text{current\_max})
$$

* Space decreases from $\mathcal{O}(n)$ to $\mathcal{O}(1)$.

* Time remains $\mathcal{O}(n)$, performing a single linear pass.

## 8. Greedy Interpretation

### 8.1 The Local Choice

At each index $i$, the algorithm decides whether to **extend** the running subarray or **reset** it:

$$
\text{Extend if } dp[i-1] > 0, \quad \text{Reset if } dp[i-1] \le 0
$$

### 8.2 Why the Greedy Choice is Globally Optimal

Suppose $dp[i-1] \le 0$. Can prefixing $A[0 \dots i-1]$ (or any suffix of it) ever benefit any future subarray ending at $j \ge i$?

$$
\sum_{k=p}^{j} A[k] = \left(\sum_{k=p}^{i-1} A[k]\right) + \sum_{k=i}^{j} A[k]
$$


Because $\max_{0 \le p \le i-1} \sum_{k=p}^{i-1} A[k] = dp[i-1] \le 0$, for any valid starting point $p \le i-1$:

$$
\sum_{k=p}^{i-1} A[k] \le 0 \implies \sum_{k=p}^{j} A[k] \le \sum_{k=i}^{j} A[k]
$$

Concatenating an accumulated non-positive sum cannot increase the sum of any subsequent segment. Discarding the prefix when $dp[i-1] \le 0$ does not eliminate the optimal subarray.

Kadane's algorithm is simultaneously a Dynamic Programming formulation (decomposing via optimal substructure of boundary-anchored states) and a Greedy strategy (pruning non-positive history at each boundary).

## 9. Correctness Proof

We prove the correctness of the recurrence via mathematical induction.

* **Theorem:** For every $m \in \{0, 1, \dots, n-1\}$, $dp[m] = \max_{0 \le k \le m} \sum_{j=k}^m A[j]$.

* **Base Case (**$m = 0$**):**
  The set of non-empty subarrays ending at index $0$ is solely $\{A[0 \dots 0]\}$.
  

  $$
  dp[0] = A[0] = \max_{0 \le k \le 0} \sum_{j=k}^0 A[j]
  $$

  
  The base case holds.

* **Inductive Hypothesis:** Assume the statement holds for $m = i - 1$:
  

  $$
  dp[i-1] = \max_{0 \le k \le i-1} \sum_{j=k}^{i-1} A[j]
  $$

* **Inductive Step (**$m = i$**):**
  Any subarray ending at index $i$ starts at some $k \in \{0, \dots, i\}$.
  

  $$
  \max_{0 \le k \le i} \sum_{j=k}^i A[j] = \max \left( \sum_{j=i}^i A[j], \max_{0 \le k \le i-1} \sum_{j=k}^i A[j] \right)
  $$

  
  For $k \le i - 1$:
  

  $$
  \sum_{j=k}^i A[j] = \left(\sum_{j=k}^{i-1} A[j]\right) + A[i]
  $$

  
  Substituting this identity:
  

  $$
  \max_{0 \le k \le i-1} \sum_{j=k}^i A[j] = \left(\max_{0 \le k \le i-1} \sum_{j=k}^{i-1} A[j]\right) + A[i] = dp[i-1] + A[i]
  $$

  
  Therefore:
  

  $$
  \max_{0 \le k \le i} \sum_{j=k}^i A[j] = \max(A[i], dp[i-1] + A[i]) = dp[i]
  $$

* **Conclusion:** By induction, $dp[i]$ computes the exact maximum contiguous subarray sum ending at $i$. Because the global optimum must end at some index $i \in [0, n-1]$, $\max_{0 \le i < n} dp[i]$ yields the globally optimal maximum subarray sum. $\blacksquare$

## 10. Complexity Analysis

* **Time Complexity:** $\mathcal{O}(n)$. The algorithm performs a single pass over $A$ with $n$ elements. Each step performs $\mathcal{O}(1)$ basic arithmetic comparisons and additions.

* **Space Complexity:** $\mathcal{O}(1)$. The algorithm requires two scalar registers: one for the current running boundary sum and one for the global maximum.

| Approach | Time | Auxiliary Space | Main Idea | 
 | ----- | ----- | ----- | ----- | 
| Naive Brute Force | $\Theta(n^3)$ | $\mathcal{O}(1)$ | Enumerate all $(i, j)$ intervals; sum each with an inner loop | 
| Prefix Sum / Accumulator | $\Theta(n^2)$ | $\mathcal{O}(1)$ | Reuse running sum for interval updates | 
| Prefix Sum Min-Reduction | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | Compute $P[j] - \min_{i < j} P[i]$ on the fly | 
| Dynamic Programming | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | State $dp[i]$ anchored at index $i$; full table stored | 
| Kadane's Algorithm | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | State dependency compression of DP to one scalar variable | 

## 11. Edge Cases

* **All Negative Values:** e.g., $A = [-7, -2, -4]$.

  * The transition $\max(A[i], dp[i-1] + A[i])$ chooses $A[i]$ when $dp[i-1] < 0$.

  * Initializing the global answer to $0$ fails here; it must be initialized to $A[0]$ or $-\infty$.

  * Output: $\max(-7, -2, -4) = -2$.

* **Size** $n = 1$**:**

  * Loop terminates immediately; returns $A[0]$ correctly.

* **All Positive Values:**

  * $dp[i-1] > 0$ holds at every step. Subarrays never restart; returns $\sum_{k=0}^{n-1} A[k]$.

* **Zero Values Present:** e.g., $A = [0, -1, 0, 2]$.

  * If $dp[i-1] = 0$, then $\max(A[i], 0 + A[i]) = A[i]$. Restarting or extending preserves correctness.

## 12. Common Incorrect Approaches

### 12.1 Resetting Running Sum to 0 on Negative Elements

* **Flawed Idea:** "Whenever we encounter a negative number, reset the running sum to zero."

* **Counterexample:** $A = [4, -1, 5]$.

* **Failure:** Resetting on $-1$ yields segments $[4]$ and $[5]$, missing $[4, -1, 5] = 8$. A negative number is acceptable as long as the cumulative prefix remains strictly positive.

* **Correction:** Reset only when the *cumulative sum* drops $\le 0$, not on the occurrence of an individual negative element.

### 12.2 Initializing Answer to Zero (Empty Subarray Fallacy)

* **Flawed Idea:** `long long max_so_far = 0;`

* **Counterexample:** $A = [-5, -3, -9]$.

* **Failure:** The code outputs `0`, but the problem requires a non-empty subarray, so the true answer is `-3`.

* **Correction:** Initialize `max_so_far = A[0]` (or $-\infty$).

### 12.3 Standard Sliding Window (Two Pointers)

* **Flawed Idea:** "Expand right pointer to increase sum; shrink left pointer when sum decreases."

* **Failure:** Sum does not vary monotonically with window size when negative numbers exist. Sliding window monotonic invariants break over arbitrary signed integer inputs.

## 13. Implementation

### C++ (Competitive Programming Style)

```
#include <vector>
#include <algorithm>
#include <cstdint>

long long maxSubarraySum(const std::vector<int>& nums) {
    long long current_max = nums[0];
    long long global_max = nums[0];

    for (size_t i = 1; i < nums.size(); ++i) {
        current_max = std::max(static_cast<long long>(nums[i]), current_max + nums[i]);
        global_max = std::max(global_max, current_max);
    }

    return global_max;
}

```

### Java

```
public class Solution {
    public static long maxSubarraySum(int[] nums) {
        long currentMax = nums[0];
        long globalMax = nums[0];

        for (int i = 1; i < nums.length; i++) {
            currentMax = Math.max((long) nums[i], currentMax + nums[i]);
            globalMax = Math.max(globalMax, currentMax);
        }

        return globalMax;
    }
}

```

## 14. Dry Run of the Final Optimized Algorithm

Input: $A = [3, -4, 2, -1, 6, -8]$

* Initialization:

  * `current_max` = $A[0] = 3$

  * `global_max` = $A[0] = 3$

| $i$ | $A[i]$ | Evaluation: `std::max((long long)A[i], current_max + A[i])` | `current_max` | `global_max` | Explanation | 
 | ----- | ----- | ----- | ----- | ----- | ----- | 
| $1$ | $-4$ | $\max(-4, 3 + (-4)) = \max(-4, -1)$ | $-1$ | $3$ | Extends previous subarray; sum drops to $-1$. | 
| $2$ | $2$ | $\max(2, -1 + 2) = \max(2, 1)$ | $2$ | $3$ | Restarts at index $2$ because $-1$ is a net deficit. | 
| $3$ | $-1$ | $\max(-1, 2 + (-1)) = \max(-1, 1)$ | $1$ | $3$ | Extends; cumulative sum remains positive ($1$). | 
| $4$ | $6$ | $\max(6, 1 + 6) = \max(6, 7)$ | $7$ | $7$ | Extends; exceeds previous `global_max`, update to $7$. | 
| $5$ | $-8$ | $\max(-8, 7 + (-8)) = \max(-8, -1)$ | $-1$ | $7$ | Extends; sum falls to $-1$. | 

Final Output: `global_max` = $7$, corresponding to subarray $A[2 \dots 4] = [2, -1, 6]$.

## 15. Why the Algorithm Works

Kadane's algorithm works because the search space of $\Theta(n^2)$ subarrays can be partitioned by their ending positions into $n$ disjoint equivalence classes:

$$
\mathcal{S} = \bigsqcup_{i=0}^{n-1} S(i), \quad S(i) = \{A[k \dots i] \mid 0 \le k \le i\}
$$

For each endpoint $i$, finding the optimal subarray does not require evaluating all $k \in [0, i]$. Due to associativity:

$$
\sum_{m=k}^i A[m] = \sum_{m=k}^{i-1} A[m] + A[i]
$$


The optimal choice for $S(i)$ reuses the optimal choice from $S(i-1)$:

$$
dp[i] = A[i] + \max(0, dp[i-1])
$$

The algorithm discards all suboptimal history within $S(i-1)$ in constant time, retaining only a single scalar summary ($dp[i-1]$). Since $dp[i]$ depends only on $dp[i-1]$, previous states $dp[0 \dots i-2]$ can be discarded immediately, achieving $\mathcal{O}(n)$ time and $\mathcal{O}(1)$ space.

## 16. Pattern Recognition

Identify this algorithmic pattern when a problem features:

* **"Contiguous subarray" + "Maximize / Minimize an additive metric":** A signal that two-pointer or DP interval caching applies.

* **"Extend or Reset" Decision Space:** Whenever prepending past accumulated state helps only if that state exceeds an identity threshold ($> 0$ for addition, $> 1$ for positive multiplication).

* **Prefix Sum Difference Formulation:** Subarray sums satisfy $\sum_{k=i}^j A[k] = P[j] - P[i-1]$. Maximizing this difference is equivalent to:
  

  $$
  \max_{0 \le j < n} \left( P[j] - \min_{0 \le i \le j} P[i-1] \right)
  $$

  
  Tracking running prefix minima on the fly directly mirrors Kadane’s single-variable state accumulation.

## 17. Generalization

### 17.1 Reconstructing Subarray Indices

To find the actual slice $[L, R]$ producing the maximum sum, track the starting pointer:

```
long long max_sum = A[0], cur_sum = A[0];
int best_l = 0, best_r = 0, cur_l = 0;

for (int r = 1; r < n; ++r) {
    if (cur_sum + A[r] < A[r]) {
        cur_sum = A[r];
        cur_l = r;
    } else {
        cur_sum += A[r];
    }
    if (cur_sum > max_sum) {
        max_sum = cur_sum;
        best_l = cur_l;
        best_r = r;
    }
}

```

### 17.2 Maximum Circular Subarray Sum

For an array that wraps around, the optimal subarray is either:

1. Entirely non-wrapped: Solved via standard Kadane: $\text{Kadane}(A)$.

2. Wrapped across array ends: The elements *not* taken form a contiguous minimum subarray in the middle.
   

   $$
   \text{MaxCircular} = \max(\text{KadaneMax}(A), \text{TotalSum} - \text{KadaneMin}(A))
   $$

   
   *(Special case: If all elements are negative,* $\text{TotalSum} - \text{KadaneMin}(A) = 0$*, representing an empty set. Return* $\text{KadaneMax}(A)$ *instead.)*

### 17.3 Maximum Product Subarray

Because multiplying two negative numbers yields a positive number, tracking only the maximum product ending at $i$ is insufficient. Maintain both the maximum and minimum products:

$$
dp_{\max}[i] = \max(A[i], \max(dp_{\max}[i-1] \cdot A[i], dp_{\min}[i-1] \cdot A[i]))
$$

$$
dp_{\min}[i] = \min(A[i], \min(dp_{\max}[i-1] \cdot A[i], dp_{\min}[i-1] \cdot A[i]))
$$

## 18. Deep Insight

The core insight behind Kadane's algorithm is:

> **Conditioning on a boundary turns an unmanageable global search over** $\Theta(n^2)$ **intervals into** $n$ **locally coupled subproblems.**

Directly asking for "the best subarray within $A[0 \dots i]$" fails because it destroys contiguity. By instead asking for "the best subarray **ending precisely at index** $i$," contiguity is strictly enforced. Under this formulation, each subproblem links to the next through a single operation:

$$
dp[i] = A[i] + \max(0, dp[i-1])
$$

This transforms an unstructured combinatorial search into a linear transition, allowing the algorithm to find the global optimum in $\mathcal{O}(n)$ time and $\mathcal{O}(1)$ space.