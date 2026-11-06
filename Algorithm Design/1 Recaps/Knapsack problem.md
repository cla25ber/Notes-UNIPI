#algorithm_design #computer_science 
<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">31/03/2026 & 01/04/2026 & 08/04/2026</span></p>

[[AD index]] (1)

Let us consider $n$ items of weight $w_1,\dots,w_n$ and value $v_1,\dots,v_n$, given a knapsack that can sustain a weight of $W$, what is the combination of objects that maximizes the total value?

##### Brute-force approach

This approach has a cost of $\tilde{O}(2^n)$ where $\tilde{O}(f(n))=O(f(n)\cdot n^{O(1)})$ ignores polynomial factors. This is because it is like generating all subsets of $[n]$, and check which one has the maximal value.

##### Dynamic programming 1

We create a table with $W$ columns and $n$ rows. The $(i,j)$ cell represents the optimum if we consider only the elements from $1$ to $i$ and if we had capacity $j$, meaning that what is of interest to us is the last cell of the table $(n,W)$. The table follows this recurrence relation:$$T_{i,j}=\max(T_{i-1,j},T_{i-1,j-w_i}+v_i)$$This represents that we take the best out of not taking the element (so consider only the previous $i-1$ elements), or take the element (so we consider the previous $i-1$ elements but also with reduced capacity to $j-w_i$); the latter obviously adds value $v_i$ to the total since we decided to take the item.![[Knapsack DP1.png]]
The total cost of this algorithm is $O(n\,W)$.

##### Dynamic programming 2

We create a table with $n$ rows and $n\cdot v_\max$ columns, where $v_\max=\max\limits_{1\leq i\leq n}(v_i)$. The $(i,j)$ cell represents the minimum weight with value greater or equal than $j$ if we consider only the the elements from $1$ to $i$. This means that the cell we are looking for is not anymore the last one, but the one on the last row that is the last one before it has value $>W$. The recurrence relation is the following:$$T_{i,j}=\min(T_{i-1,j},T_{i-1,j-v_i}+w_i)$$Thus, I do not take the element (and the value to reach remains the same but I consider the first $i-1$ elements), or I do take the element and thus the value to reach is lowered by $v_i$ but the weight $w_i$ is added to the total (and now we consider just the first $i-1$ elements).![[Knapsack DP2.png]]
The total cost of the algorithm is $O(n\cdot n\cdot v_\max)$.

### Approximations

Sometimes it is better to calculate an approximation that takes less time, rather than the exact solution. Given the optimal solution $S^*$, having value $value(S^*)=\mathit{OPT}$, we say that a solution $S$ is an $r$-approximation respectively for minimization or maximization if:$$\frac{\mathit{value}(S)}{\mathit{OPT}}\leq r\qquad\qquad\qquad\qquad\qquad\qquad\frac{\mathit{OPT}}{\mathit{value}(S)}\leq r$$
##### Greedy solution

Let us order the items of the knapsack by $\frac{v_i}{w_i}$ and re-number them. We then start taking them one by one, from the first until we exceed the weight limit; we call that value $m_G$. Then, as a last step, we output as a solution the following value: $\max(m_G, v_\max)$. This last step is done avoid specific occurrences where the capacity allows to fit exactly the maximal value and adding even one element, even if more convenient in value/wight ratio, disables us from forming a good solution.

We can show that this method gives us a $2$-approximation.

**Proof:**

Since we do not know $\mathit{OPT}$, we must use an upper-bound $\mathit{UB}\geq\mathit{OPT}$ so that by proving $\frac{\mathit{UB}}{\mathit{value}(S)}\leq 2$ we also prove that $\frac{\mathit{OPT}}{\mathit{value}(S)}\leq 2$.
Let us call $j$ the first item that does not, meaning that $\bar{w}_j=\sum\limits_{i=1}^{j-1}w_i\leq W$ but $\sum\limits_{i=1}^jw_i>W$. We also call $m_G=\bar{v}_j=\sum\limits_{i=1}^{j-1}v_i$. As an upper bound we take $\bar{v}_j+v_j\geq\mathit{OPT}$. Let us then consider these 2 cases:
- $\bar{v}_j>v_j$, then $\mathit{OPT}<2\,\bar{v}_j=2\,m_G\leq2\cdot\max(m_G,v_\max)=2\cdot\mathit{value}(S)$.
- $\bar{v}_j\leq v_j$, then $\mathit{OPT}<2\,v_j\leq2\,v_\max\leq2\cdot\max(m_G,v_\max)=2\cdot\mathit{value}(S)$.
So, this greedy solution is a $2$-approximation.
$\square$

## 0-1 Knapsack

This is a variant of the knapsack problem, where we have $n$ items of weight $w_1,\dots,w_n$, and a knapsack with capacity $W$. The objective of this problem, without a value assigned to the items, is to calculate the number of possible solutions, i.e. the number of subset respecting the capacity of the knapsack. This problem is also called often times "Subset Sum".

##### Dynamic programming

We can solve this problem in a similar way to "Dynamic programming 1" for the classic knapsack problem. We create a table with $n$ rows and $W$ columns, with the following recurrence relation:$$\begin{align}
    T_{i,j} = \begin{cases}
            1 \quad\qquad\qquad\qquad\ \ \ \text{if $i=0\lor j=0$}\\
            T_{i-1,j}+T_{i-1,j-w_i} \quad\text{otherwise}
           \end{cases}
\end{align}$$We just take the number of solution that we would get by not taking $i$ and those by taking $i$ and sum them.

#### Generating a random solution

If we wanted to uniformly generate a random valid solution, we could still use the dynamic programming table defined earlier. When arriving at the cell $(i,j)$, we would have that  $T[i][j]=T[i-1][j]+T[i-1][j-w_i]$, so we decide to put $i$ in the solution according to the following:
- $\mathbb{P}(\text{take }i)=\frac{T[i-1][j]}{T[i][j]}$
- $\mathbb{P}(\text{not take }i)=\frac{T[i-1][j-w_i]}{T[i][j]}$
This makes the probability to include $i$ proportional to the number of solution generated by including it.

This method uniformly extracts a random feasible solution, i.e. the probability of extracting a specific feasible solution $S$ is $\frac{1}{T[n][W]}$.
Let $S$ be a random solution, suppose that we take element $n$, then the probability of having taken such solution must be multiplied to $\frac{T[n-1][W-w_n]}{T[n][W]}$ since, over all possible solutions, we decided to take those containing $n$. Suppose then that element $n-1$ is not taken, then the probability of not taking it is $\frac{T[n-2][W-w_n]}{T[n-1][W-w_n]}$ because by knowing we took element $n$, now the total space of probability gets restricted to the solutions containing element $n$. This causes a telescopic multiplication:$$\mathbb{P}(\text{extracting $S$})=\frac{T_{n-1}}{T_n}\cdot\frac{T_{n-2}}{T_{n-1}}\cdot\,\dots\,\cdot\frac{T_0}{T_1}=\frac{T_0}{T_n}$$However, since $T_0$ represents the base case we will always have that $T_0=1$, and since $T_n$ represents all initial solutions, we will always have that $T_n=T[n][W]$, and thus the probability of extracting a random solution is uniform.

This happens because dynamic programming avoids over-counting the same solution, it creates a partition of solution.